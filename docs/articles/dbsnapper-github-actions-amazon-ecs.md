---
title: Use DBSnapper with GitHub Actions and Amazon ECS to Snapshot Amazon RDS Databases
description: In this article we describe how to setup a GitHub Action to snapshot an Amazon RDS database using DBSnapper and Amazon ECS Fargate.
---

## Overview

The motivation behind this article is to provide an automated way to snapshot database workloads running on private infrastructure. In this article, we'll use GitHub Actions to trigger a snapshot and sanitization of an Amazon RDS database using DBSnapper and Amazon ECS Fargate.

### Services and components used

- DBSnapper Agent to take the snapshot, sanitize it, and store it.
- DBSnapper Cloud providing target configuration and snapshot storage.
- GitHub Actions (GHA) as CI/CD provider to automate the environment setup and trigger the snapshot.
- GitHub Actions self-hosted runners to run the DBSnapper Agent in our private infrastructure.
- Amazon ECS Fargate to run an ephemeral GHA self-hosted runner

### Other requirements

- We use GitHub's OIDC Provider to provide AWS credentials needed for the actions. We used the instructions given in the [aws-actions/configure-aws-credentials OIDC Section](https://github.com/aws-actions/configure-aws-credentials?tab=readme-ov-file#oidc) to setup the federation between GitHub and AWS, using the [CloudFormation Template](https://github.com/aws-actions/configure-aws-credentials?tab=readme-ov-file#configuring-iam-to-trust-github) to speed things up.
- We use the [Github official `actions-runner` container](https://github.com/actions/runner/pkgs/container/actions-runner) for our self-hosted runner.

### Github Secrets:

We will need the following GitHub Secrets configured for the GitHub Action:

- `FG_PAT`' - [Fine-Grained GitHub Personal Access Token (PAT)](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-fine-grained-personal-access-token) with `repo:actions:(read/write)` and `org:self-hosted-runners:(read/write)` scopes.
- `DBSNAPPER_AUTHTOKEN` - Your DBSnapper Cloud Auth Token used to authenticate to the DBSnapper API.
- `DBSNAPPER_SECRET_KEY` - Your DBSnapper config secret key used to encrypt certain configuration values.

### Overview of the steps

1. Setup the Github Actions workflow basics.
2. Get a Registration token to register a self-hosted runner.
3. Launch an Amazon ECS Fargate Task to run the self-hosted runner.
4. Execute the DBSnapper Agent snapshot and sanitization commands.
5. Cleanup the Amazon ECS Fargate Tasks.

## Step 1 - Setup the GitHub Actions Workflow

```yaml title="Step 1 - Setup the GitHub Actions Workflow" linenums="1"
name: "DBSnapper Build Snapshot"
on: [push]

# Permissions are needed for github OIDC provider - AWS federation
# https://github.com/aws-actions/configure-aws-credentials?tab=readme-ov-file#oidc
permissions:
  id-token: write
  contents: read
  actions: write

# Some necessary environment variables
env:
  ORGANIZATION: dbsnapper # Your Github Organization
  ORGANIZATION_URL: https://github.com/dbsnapper # Your Github Organization URL
  AWS_REGION: us-east-1
  AWS_IAM_ROLE: arn:aws:iam::<acct_id>:role/<name of role> # From OIDC Federation
  ECS_CLUSTER: dbsnapper
  ECS_SERVICE: github_runner
  ECS_TASK_DEFINITION: .aws/ecs_task_definition_github_runner.json # Initial task definition file in the repo for the ECS Task
```

Here we setup the basic elements of the GitHub Actions workflow. We define the name of the workflow, the event that triggers it, and some environment variables that we will use throughout the workflow.

### IAM Policy for the AWS_IAM_ROLE

The `permissions` section is needed for the GitHub OIDC provider to provide AWS credentials to the actions and the `AWS_IAM_ROLE` is the ARN of the role that the OIDC provider will assume to provide the AWS credentials. This role should have the necessary permissions to interact with the services we are using, specifically ECS in our case:

```json title="Example IAM Policy for the AWS_IAM_ROLE"
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "RegisterTaskDefinition",
      "Effect": "Allow",
      "Action": ["ecs:RegisterTaskDefinition"],
      "Resource": "*"
    },
    {
      "Sid": "PassRolesInTaskDefinition",
      "Effect": "Allow",
      "Action": ["iam:PassRole"],
      "Resource": "*"
    },
    {
      "Sid": "DeployService",
      "Effect": "Allow",
      "Action": ["ecs:UpdateService", "ecs:DescribeServices"],
      "Resource": "*"
    }
  ]
}
```

Above is an example of the IAM policy that the `AWS_IAM_ROLE` should have to interact with ECS. You should set specific Resource ARNs for the `ecs:UpdateService` and `ecs:DescribeServices` actions to limit the access to the ECS cluster and service you are using. I've wildcarded them here for simplicity.

## Step 2 - Get Registration Token to Register a Runner

```yaml title="Step 2 - Get Registration Token" linenums="1"
jobs:
  register-runner:
    runs-on: ubuntu-latest
    outputs:
      registration_token: ${{ steps.create-token.outputs.registration_token }}
    steps:
      - name: Create Registration Token
        id: create-token
        # https://docs.github.com/en/rest/actions/self-hosted-runners?apiVersion=2022-11-28#about-self-hosted-runners-in-github-actions
        # use jq -r to output the raw version of the .token field
        run: |
          token=$(curl -L \
          -X POST \
          -H "Accept: application/vnd.github+json" \
          -H "Authorization: Bearer ${{ secrets.FG_PAT }}" \
          -H "X-GitHub-Api-Version: 2022-11-28" \
          "https://api.github.com/orgs/${{ env.ORGANIZATION }}/actions/runners/registration-token" | jq -r '.token')

          echo "registration_token=$token" >> $GITHUB_OUTPUT
```

### Inputs

- `secrets.FG_PAT `- The Fine-Grained GitHub Personal Access Token (PAT) used in the `Authentication` header for the API call
- `env.ORGANIZATION` - The GitHub Organization that the runner will be registered with, used in the API URL.

### Outputs

- `create_token.registration_token` - The registration token that will be used to register the self-hosted runner

### Description

Step two gives us a Registration Token that we can pass to the self-hosted runner configuration script to register the runner with the GitHub Actions service. We use the [GitHub REST API](https://docs.github.com/en/rest/actions/self-hosted-runners?apiVersion=2022-11-28#create-a-registration-token-for-an-organization) to get the registration token and output it to a file that we can access in the next step.

We use the `jq` utility to parse the JSON response from the API and extract the `token` field from the API response. We use the `-r` flag to output the raw version of the `token` field, otherwise it would be quoted and cause issues later in the workflow.

in line 19 we save the output of the `token` to the [$GITHUB_OUTPUT variable](https://docs.github.com/en/actions/using-workflows/workflow-commands-for-github-actions#setting-an-output-parameter) which is the new way to set outputs in GitHub Actions. This is assigned to the `registration_token` output of the job in line 5 for use later in the workflow.

## Step 3 - Launch Amazon ECS Task

```yaml title="Step 3 - Launch Amazon ECS Task" linenums="1"
runner-ecs:
  runs-on: ubuntu-latest
  needs: register-runner
  steps:
    - name: Checkout for access to task definition
      uses: actions/checkout@v4

    # Uses OIDC connector for auth
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-region: ${{ env.AWS_REGION }}
        role-to-assume: ${{ env.AWS_IAM_ROLE }}

    - name: Update Task Definition JSON
      id: update-task-def
      uses: restackio/update-json-file-action@main
      with:
        file: ${{ env.ECS_TASK_DEFINITION }}
        fields: |
          {
            "containerDefinitions[0].command": [
            "sh",
            "-c",
            "./config.sh --unattended --ephemeral --labels x64,linux,ecs --name ECS_Ephemeral_Runner --url ${{ env.ORGANIZATION_URL }} --token ${{ needs.register-runner.outputs.registration_token }} && ./run.sh"
            ]
          }

    - name: Deploy Amazon ECS task definition
      uses: aws-actions/amazon-ecs-deploy-task-definition@v1
      with:
        task-definition: ${{ env.ECS_TASK_DEFINITION }}
        service: ${{ env.ECS_SERVICE }}
        cluster: ${{ env.ECS_CLUSTER }}
        wait-for-service-stability: false

    - name: ECS Service Set desired tasks to 1
      run: |
        aws ecs update-service --cluster ${{ env.ECS_CLUSTER }} --service ${{ env.ECS_SERVICE }} --desired-count 1
```

### Inputs

- `env.AWS_REGION` - The AWS region where the ECS cluster is located
- `env.AWS_IAM_ROLE` - The ARN of the IAM role that the OIDC provider will assume to provide the AWS credentials.
- `env.ECS_CLUSTER` - The name of the ECS cluster where the task will be run.
- `env.ECS_SERVICE` - The name of the ECS service that will run the task.
- `env.ECS_TASK_DEFINITION` - The path and filename of the task definition file in the repository.
- `.aws/ecs_task_definition_github_runner.json` - The initial task definition file in the repository for the ECS Task.

### Actions

- [`actions/checkout@v4`](https://github.com/actions/checkout/tree/releases/v4.0.0) - This action checks out the repository so that we can access the ECS task definition file.
- [`aws-actions/configure-aws-credentials@v4`](https://github.com/aws-actions/configure-aws-credentials/tree/v4) - This action configures the AWS credentials for the actions to use. It uses the OIDC connector to authenticate with AWS.
- [`restackio/update-json-file-action@main`](https://github.com/restackio/update-json-file-action/tree/main/) - This action updates the ECS task definition JSON file with the container command that provides the `--url` of the GitHub organization and the `--token` registration token to register the self-hosted runner. The main branch includes an updated version of node used in the action. We use this instead of [`amazon-ecs-render-task-definition`](https://github.com/aws-actions/amazon-ecs-render-task-definition/tree/v1) because it improperly parses the provided command string, causing issues with the runner registration [per this pull request](https://github.com/aws-actions/amazon-ecs-render-task-definition/pull/290).

### Description

#### Checkout and Configure AWS Credentials

In this step, we're defining the `runner-ecs` job (that depends on successful execution of the `register-runner` job) that will launch an Amazon ECS Fargate Task to run the self-hosted runner. The first two steps are straightforward. We use the `actions/checkout@v4` action on line 6 to checkout the repository so that we can access the ECS task definition file. We then use the `aws-actions/configure-aws-credentials@v4` action on line 10 to configure the AWS credentials for the actions to use. This action uses the OIDC connector to authenticate with AWS.

#### Update ECS Task Definition JSON

The next step starting on line 15, we update the ECS task definition file in our repository to reflect the command we'll run to configure the self-hosted runner. In this command we use the `config.sh` script to configure the runner with the `--url` and `--token` arguments, which are needed to register the runner in the GitHub Actions service. We can find our GHA runners in the organization actions settings URL: `https://github.com/organizations/<your_organization>/settings/actions/runners`

<!-- prettier-ignore-start -->
!!! note "GHA Runner Docs"

    It wasn't immediately obvious how to run the GHA runner container, so there was a bit of trial and error locally to get the commands right. Documentation can be found in the repository, and the [Automate Configuring Self-Hosted Runners](https://github.com/actions/runner/blob/main/docs/automate.md) is especially useful. Two  scripts in the runner container are used to configure and run the runner:

    - `config.sh` - This script configures the runner with the provided `--url` and `--token` arguments. The `--ephemeral` flag is used to run the runner in ephemeral mode, meaning it will be removed when the task is stopped, and the `--unattended` flag is used to run the runner in unattended mode, meaning it won't prompt for user input.
    - `run.sh` - This script starts the runner, registering it with GitHub Actions, starting the runner service and waiting for jobs to run. This script is run after the runner is configured with the `config.sh` script.

    Both scripts will display help and available command line options by passing `--help` as an argument. The `config.sh` script is the only one that requires arguments, and the `run.sh` script will run the runner with the configuration provided by the `config.sh` script.
<!-- prettier-ignore-end -->

The `--ephemeral` flag is used to run the runner in ephemeral mode, meaning the runner will be removed from the organization after it has completed a single job. This ensures we don't have any lingering runners in the organization.

#### Deploy Amazon ECS Task Definition

The next step on line 29 uses the `aws-actions/amazon-ecs-deploy-task-definition@v1` action to deploy the updated task definition to the ECS cluster. This action takes the path to the task definition file, the cluster, and service name as inputs. We set the `wait-for-service-stability` input to `false` to avoid waiting for the service to stabilize before continuing with the workflow, since [Step 4 - Execute the DBSnapper Agent Commands](#step-4---execute-the-dbsnapper-agent-commands) won't proceed until the runner is registered, running, and able to execute the job.

#### ECS Service: Set Desired Tasks to 1

Finally, on line 39, we use the `aws ecs update-service` command (from the aws-cli) to set the desired task count for the service to 1. This will start a task in the service to run the self-hosted runner. The updated task definition will not start without this command. We use a similar command in [Step 5 - Cleanup the Amazon ECS Task](#step-5---cleanup-the-amazon-ecs-task) to set the desired task count to 0 to stop the runner after the job is complete.

### Task Definition File

To launch an ECS task via github actions and the `aws-actions/amazon-ecs-deploy-task-definition@v1` action, we need to provide a task definition file. This file is a JSON file that defines the properties of the task that we want to run. We can create this file using the ECS task definition wizard in the AWS console and then copy the task definition JSON to a file in our repository. In this example, we're using the `ECS_TASK_DEFINITION` environment variable to specify the path to the task definition file.

```json title="ecs_task_definition_github_runner.json" linenums="1"
{
  "containerDefinitions": [
    {
      "name": "github_runner",
      "image": "ghcr.io/actions/actions-runner:2.317.0",
      "cpu": 0,
      "portMappings": [],
      "essential": true,
      "command": [],
      "mountPoints": [],
      "volumesFrom": [],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/github_runner",
          "awslogs-create-group": "true",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      },
      "systemControls": []
    }
  ],
  "family": "github_runner",
  "taskRoleArn": "arn:aws:iam::<account_id>:role/<task_execution_role>",
  "executionRoleArn": "arn:aws:iam::<account_id>:role/<task_execution_role>",
  "networkMode": "awsvpc",
  "volumes": [],
  "placementConstraints": [],
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "2048",
  "runtimePlatform": {
    "cpuArchitecture": "X86_64",
    "operatingSystemFamily": "LINUX"
  },
  "tags": []
}
```

In this task definition, we set the default GHA runner image on line 5. If needed, we can update this image in the workflow using the `restackio/update-json-file-action@main` action. We also set the `cpu` and `memory` values for this task on lines 31 and 32 to the minimum values needed to run the runner and any additional services.

## Step 4 - Execute the DBSnapper Agent Commands

```yaml title="Step 4 - Execute the DBSnapper Agent Commands" linenums="1"
dbsnapper:
  runs-on: self-hosted
  needs: runner-ecs
  env:
    DBSNAPPER_SECRET_KEY: ${{ secrets.DBSNAPPER_SECRET_KEY }}
    DBSNAPPER_AUTHTOKEN: ${{ secrets.DBSNAPPER_AUTHTOKEN }}
  steps:
    - name: Download and install DBSnapper release
      run: |
        sudo apt-get update && sudo apt-get install -y curl && \
        curl -L -o dbsnapper.deb https://github.com/dbsnapper/dbsnapper/releases/download/v2.7.0/dbsnapper_linux_x86_64.deb && \
        sudo dpkg -i dbsnapper.deb
    - name: Run DBSnapper build command
      run: dbsnapper build dvdrental-prod
```

### Inputs

- `secrets.DBSNAPPER_SECRET_KEY` - Your DBSnapper config secret key used to encrypt certain configuration values.
- `secrets.DBSNAPPER_AUTHTOKEN` - Your DBSnapper Cloud Auth Token used to authenticate to the DBSnapper API.
- [`DBSnapper Debian Package`](https://github.com/dbsnapper/dbsnapper/releases/latest) - The latest release of the DBSnapper Agent. Download the `.deb` package that matches your architecture. We use the `dbsnapper_linux_x86_64.deb` package in this example.

### Discussion

The steps prior to this one were necessary to setup a self-hosted runner in our private infrastructure with access to the workloads in our network. In this step, we're finally able to run the DBSnapper Agent commands to snapshot and sanitize the database.

#### Runs-on: Self-Hosted

On line 2, we specify the `runs-on` property as `self-hosted` to run the job on the self-hosted runner we registered in the previous step. This property can take additional labels to be more specific about the runner that should run the job, in cases where you have many different types of runners registered in your organization. In line 3 we specify that this job depends on the `runner-ecs` job to ensure the ecs jobs are run before running the dbsnapper job.

#### Environment Variables

We set our environment variables starting on line 4. In this example, we set the minimum required `DBSNAPPER_SECRET_KEY` and `DBSNAPPER_AUTHTOKEN` environment variables necessary to run DBSnapper without a configuration file.

<!-- prettier-ignore-start -->
!!! note "DBSnapper Configuration via Environment Variables"
    DBSnapper can be configured exclusively through environment variables if you don't want to rely on a configuration file. All the configuration options can be represented as environment variables through a specific naming convention involving prefixing the environment variable with `DBSNAPPER` and replacing periods with two underscores `__`. Some examples from the [DBSnapper Configuration Documentation](/docs/configuration.md) include:

    - `docker.images.postgres` -> `DBSNAPPER_DOCKER__IMAGES__POSTGRES: postgres:latest` # Sets the docker image to use for the postgres containers
    - `defaults.shared_target_dst_db_url` -> `DBSNAPPER_DEFAULTS__SHARED_TARGET_DST_DB_URL: <connstring>` # Sets the default destination database URL for shared targets
    - `override.san_query` -> `DBSNAPPER_OVERRIDE__SAN_QUERY: <base-64-encoded-value>` # Sets a query to use for sanitization overriding any existing queries.
<!-- prettier-ignore-end -->

#### Install and Run DBSnapper

Starting on line 8, we run commands to update our apt repository, install `curl`, and use `curl` to download the latest release of the DBSnapper Agent. We then use `dpkg` to install the `.deb` package.

<!-- prettier-ignore-start -->
!!! question "Why Not use the DBSnapper Docker Image?"
    At this point, it would have been convenient to use the [DBSnapper Docker image](https://ghcr.io/dbsnapper/dbsnapper) to run the DBSnapper commands. Unfortunately the GitHub Actions runner does not have Docker installed by default, so we would need to install Docker in the runner before we could use the Docker image. To avoid the additional complexity we decided to download and install the `.deb` package instead.
<!-- prettier-ignore-end -->

On line 13, we run the `dbsnapper build dvdrental-prod` command which will use the `DBSNAPPER_AUTHTOKEN` to authenticate to the DBSnapper Cloud, create a snapshot of the database specified in the `dvdrental-prod` target, and store it in the cloud storage specified in the target configuration. Once this is complete and no additional steps are needed, the task will be cleaned up in the next step.

## Step 5 - Cleanup the Amazon ECS Task

```yaml title="Step 5 - Cleanup the Amazon ECS Task" linenums="1"
deprovision:
  runs-on: ubuntu-latest
  if: ${{ always() }}
  needs: dbsnapper
  steps:
    # Uses OIDC connector for auth
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-region: ${{ env.AWS_REGION }}
        role-to-assume: ${{ env.AWS_IAM_ROLE }}

    - name: ECS Add desired tasks to 0
      run: |
        aws ecs update-service --cluster ${{ env.ECS_CLUSTER }} --service ${{ env.ECS_SERVICE }} --desired-count 0
```

### Inputs

- `env.AWS_REGION` - The AWS region where the ECS cluster is located
- `env.AWS_IAM_ROLE` - The ARN of the IAM role that the OIDC provider will assume to provide the AWS credentials.
- `env.ECS_CLUSTER` - The name of the ECS cluster where the task will be run.
- `env.ECS_SERVICE` - The name of the ECS service that will run the task.

### Actions

- [`aws-actions/configure-aws-credentials@v4`](https://github.com/aws-actions/configure-aws-credentials/tree/v4) - This action configures the AWS credentials for the actions to use. It uses the OIDC connector to authenticate with AWS.

### Description

In this final step, we need to use the `aws-actions/configure-aws-credentials@v4` action to configure the AWS credentials so we can use the AWS CLI to run the `aws ecs update-service` command. We use this command to set the desired task count for the service to 0, which will stop all ECS runner tasks, ensuring we don't get billed for unnecessary resources. Because we configured the runners with the `--ephemeral` flag, they will be automatically removed from the GitHub organization when they are terminated.

Note: By using the `if: ${{ always() }}` condition on line 3, we ensure this step runs even if the previous steps fail. This is important to ensure that the ECS service is cleaned up even other steps fail, which under normal circumstances would not be the case.

## Conclusion

In this article, we've shown how to use GitHub Actions to trigger a snapshot and sanitization of an Amazon RDS database using DBSnapper and Amazon ECS Fargate. We've covered the setup of the GitHub Actions workflow, the registration of a self-hosted runner, the launch of an Amazon ECS Fargate task to run the runner, the execution of the DBSnapper Agent commands, and the cleanup of the Amazon ECS task. This workflow can be used to automate the snapshot and sanitization of database workloads running on private infrastructure, providing an automated and reliable manage database backups.

## Full Workflow

Here is the full Github Actions workflow that we've described in this article. Be sure to replace any placeholders with your own values.

```yaml title="DBSnapper Build Snapshot - Full Workflow" linenums="1"
name: "DBSnapper Build Snapshot"
on: [push]

# Permissions are needed for github OIDC provider - AWS federation
# https://github.com/aws-actions/configure-aws-credentials?tab=readme-ov-file#oidc
permissions:
  id-token: write
  contents: read
  actions: write

# Some necessary environment variables
env:
  ORGANIZATION: dbsnapper # Your Github Organization
  ORGANIZATION_URL: https://github.com/dbsnapper # Your Github Organization URL
  AWS_REGION: us-east-1
  AWS_IAM_ROLE: arn:aws:iam::<acct_id>:role/<name of role> # From OIDC Federation
  ECS_CLUSTER: dbsnapper
  ECS_SERVICE: github_runner
  ECS_TASK_DEFINITION: .aws/ecs_task_definition_github_runner.json # Initial task definition file in the repo for the ECS Task

jobs:
  register-runner:
    runs-on: ubuntu-latest
    outputs:
      registration_token: ${{ steps.create-token.outputs.registration_token }}
    steps:
      - name: Create Registration Token
        id: create-token
        # https://docs.github.com/en/rest/actions/self-hosted-runners?apiVersion=2022-11-28#about-self-hosted-runners-in-github-actions
        # use jq -r to output the raw version of the .token field
        run: |
          token=$(curl -L \
          -X POST \
          -H "Accept: application/vnd.github+json" \
          -H "Authorization: Bearer ${{ secrets.FG_PAT }}" \
          -H "X-GitHub-Api-Version: 2022-11-28" \
          "https://api.github.com/orgs/${{ env.ORGANIZATION }}/actions/runners/registration-token" | jq -r '.token')

          echo "registration_token=$token" >> $GITHUB_OUTPUT

  runner-ecs:
    runs-on: ubuntu-latest
    needs: register-runner
    steps:
        - name: Checkout for access to task definition
          uses: actions/checkout@v4

        # Uses OIDC connector for auth
        - name: Configure AWS credentials
          uses: aws-actions/configure-aws-credentials@v4
          with:
            aws-region: ${{ env.AWS_REGION }}
            role-to-assume: ${{ env.AWS_IAM_ROLE }}

        - name: Update Task Definition JSON
          id: update-task-def
          uses: restackio/update-json-file-action@main
          with:
            file: ${{ env.ECS_TASK_DEFINITION }}
            fields: |
            {
                "containerDefinitions[0].command": [
                "sh",
                "-c",
                "./config.sh --unattended --ephemeral --labels x64,linux,ecs --name ECS_Ephemeral_Runner --url ${{ env.ORGANIZATION_URL }} --token ${{ needs.register-runner.outputs.registration_token }} && ./run.sh"
                ]
            }

        - name: Deploy Amazon ECS task definition
          uses: aws-actions/amazon-ecs-deploy-task-definition@v1
          with:
            task-definition: ${{ env.ECS_TASK_DEFINITION }}
            service: ${{ env.ECS_SERVICE }}
            cluster: ${{ env.ECS_CLUSTER }}
            wait-for-service-stability: false

        - name: ECS Service Set desired tasks to 1
          run: |
            aws ecs update-service --cluster ${{ env.ECS_CLUSTER }} --service ${{ env.ECS_SERVICE }} --desired-count 1

  dbsnapper:
    runs-on: self-hosted
    needs: runner-ecs
    env:
      DBSNAPPER_SECRET_KEY: ${{ secrets.DBSNAPPER_SECRET_KEY }}
      DBSNAPPER_AUTHTOKEN: ${{ secrets.DBSNAPPER_AUTHTOKEN }}
    steps:
      - name: Download and install DBSnapper release
        run: |
          sudo apt-get update && sudo apt-get install -y curl && \
          curl -L -o dbsnapper.deb https://github.com/dbsnapper/dbsnapper/releases/download/v2.7.0/dbsnapper_linux_x86_64.deb && \
          sudo dpkg -i dbsnapper.deb
      - name: Run DBSnapper build command
        run: dbsnapper build dvdrental-prod

  deprovision:
    runs-on: ubuntu-latest
    if: ${{ always() }}
    needs: dbsnapper
    steps:
      # Uses OIDC connector for auth
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-region: ${{ env.AWS_REGION }}
          role-to-assume: ${{ env.AWS_IAM_ROLE }}

      - name: ECS Add desired tasks to 0
        run: |
          aws ecs update-service --cluster ${{ env.ECS_CLUSTER }} --service ${{ env.ECS_SERVICE }} --desired-count 0
```
