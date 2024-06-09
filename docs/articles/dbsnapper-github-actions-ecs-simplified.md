---
title: Updated - Using DBSnapper, GitHub Actions, and ECS - A Simplified Approach
description: Following on our previous article, we've simplified the process to use DBSnapper with GitHub Actions and ECS.
---

## Overview

<!-- prettier-ignore-start -->
!!! note "Third Party GitHub Runners"

    This article provides an alternative approach to using DBSnapper with GitHub Actions and Amazon ECS. This article uses a GitHub runner built by a third party. There are actually several nice third-party runners that build upon the official GitHub Actions runner, and provide additional capabilities that the official runner doesn't yet support. You can find a list of these runners on the [Awesome-runners](https://github.com/jonico/awesome-runners) list. For this article we are using the [myoung34 GitHub Runner](https://hub.docker.com/r/myoung34/github-runner).

<!-- prettier-ignore-end -->

The previous version of this article used the official GitHub Actions runner to run the DBSnapper Agent in an Amazon ECS Fargate Task. We've found that there are third-party runners that provide additional capabilities that the official runner doesn't yet support. We will present the simplified approach using the [myoung34 GitHub Runner](https://hub.docker.com/r/myoung34/github-runner) in this article. We will be brief in our explanation of the steps, as they are similar to the [previous article](dbsnapper-github-actions-amazon-ecs.md).

### Using the myoung34 GitHub Runner

The [myoung34 GitHub Runner](https://hub.docker.com/r/myoung34/github-runner) is a third-party runner that builds upon the official GitHub Actions runner. This runner makes use of environment variables to configure the runner and and most importantly supports an `ACCESS_TOKEN` environment variable that will take care of registering the runner with the GitHub Actions service. This eliminates the need for the first step in the previous article where we had to get a registration token to register the runner with GitHub.

### Simplified GitHub Actions Workflow

In this workflow we only have three jobs instead of the four in the previous article. We describe some of the important changes below:

```yaml title="Simplified GitHub Actions Workflow" linenums="1"
name: "GHA-my-ECS-DBSnapper Build Snapshot"
on: [push]

# For github OIDC provider
# https://github.com/aws-actions/configure-aws-credentials?tab=readme-ov-file#oidc
permissions:
  id-token: write
  contents: read
env:
  ORGANIZATION: <Your_GitHub_Organization_Name>
  ORGANIZATION_URL: <Your_GitHub Organization URL>
  GHR_IMAGE: myoung34/github-runner:latest
  AWS_REGION: us-east-1
  AWS_IAM_ROLE: arn:aws:iam::<account_id>:<role_name>
  ECS_CLUSTER: <ECS_Cluster_Here>
  ECS_SERVICE: <ECS_Service_Here>
  ECS_TASK_DEFINITION: .aws/ecs_task_definition_my_github_runner.json

jobs:
  runner-ecs:
    runs-on: ubuntu-latest
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
              "containerDefinitions[0].image": "${{ env.GHR_IMAGE }}",
              "containerDefinitions[0].environment": [
                {
                  "name": "ACCESS_TOKEN",
                  "value": "${{ secrets.FG_PAT }}"
                },
                {
                  "name": "RUNNER_SCOPE",
                  "value": "org"
                },
                {
                  "name": "ORG_NAME",
                  "value": "${{ env.ORGANIZATION }}"
                },
                {
                  "name": "EPHEMERAL",
                  "value": "true"
                },
                {
                  "name": "LABELS",
                  "value": "ecs"
                },
                {
                  "name": "RUNNER_NAME_PREFIX",
                  "value": "ecs"
                }
              ]
            }

      - name: Deploy Amazon ECS task definition
        uses: aws-actions/amazon-ecs-deploy-task-definition@v1
        with:
          task-definition: ${{ env.ECS_TASK_DEFINITION }}
          service: ${{ env.ECS_SERVICE }}
          cluster: ${{ env.ECS_CLUSTER }}
          wait-for-service-stability: false

      - name: ECS Add desired tasks to 1
        run: |
          aws ecs update-service --cluster ${{ env.ECS_CLUSTER }} --service ${{ env.ECS_SERVICE }} --desired-count 1

  dbsnapper:
    runs-on: self-hosted
    needs: runner-ecs
    env:
      DBSNAPPER_SECRET_KEY: ${{ secrets.DBSNAPPER_SECRET_KEY }}
      DBSNAPPER_AUTHTOKEN: ${{ secrets.DBSNAPPER_AUTHTOKEN }}
    steps:
      - name: Install DBSnapper Agent
        uses: dbsnapper/install-dbsnapper-agent-action@v1
        with:
          version: latest
      - name: Run DBSnapper List Targets
        run: dbsnapper build rds_production_db

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

The first job in this workflow involves setting up and starting the ECS task definition that will launch the myoung34 GitHub Runner. The biggest difference here is in the **Update Task Definition JSON** step starting on line 33. Instead of creating a command to run the runner as we did in the previous article, we instead update the task definition JSON file to include the necessary environment variables for the runner as follows:

- **ACCESS_TOKEN**: We provide our Fine-Grained Personal Access Token (FG_PAT) as a secret to the runner. This token is used to authenticate the runner with GitHub Actions, saving us a step in our workflow.
- **RUNNER_SCOPE**: We set the runner scope to `org` to allow the runner to access all repositories in the organization.
- **ORG_NAME**: We provide the organization name to the runner.
- **EPHEMERAL**: We set the runner to ephemeral mode, so it will be deregistered from the GitHub Actions service when the task is stopped.
- **LABELS**: We provide a label to the runner to identify it as an ECS runner - customize this as you see fit.
- **RUNNER_NAME_PREFIX**: We provide a prefix for the runner name to identify it in the GitHub Actions service.

There are several other environment variables you can use to configure the runner. You can find more information on the [myoung34 GitHub Runner](https://hub.docker.com/r/myoung34/github-runner) page.

The rest of the workflow is similar to the previous article, with the `dbsnapper` job installing and running the DBSnapper Agent to build a snapshot and the `deprovision` job stopping the ECS task.

### Task Definition

The task definition JSON file is similar to the one in the previous article, we're including it here for completeness.

```json title=".aws/ecs_task_definition_my_github_runner.json" linenums="1"
{
  "containerDefinitions": [
    {
      "name": "my_github_runner",
      "image": "myoung34/github-runner:latest",
      "cpu": 0,
      "portMappings": [],
      "essential": true,
      "environment": [],
      "mountPoints": [],
      "volumesFrom": [],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/github_runner",
          "awslogs-create-group": "true",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs-my"
        }
      },
      "systemControls": []
    }
  ],
  "family": "my_github_runner",
  "taskRoleArn": "<task_role_arn>",
  "executionRoleArn": "<execution_role_arn>",
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
