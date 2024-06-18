---
title: Welcome
description: DBSnapper simplifies the process of creating de-identified database snapshots that can be used for real-world development, testing, and AI model training.
---

<p align="center">
  <img src="/static/dbs-docs.jpg" alt="DBSnapper" width="90%">
</p>

# Welcome to DBSnapper

<!-- prettier-ignore-start -->
!!! note "Latest News"

    - Our **Terraform Provider** is now available on the [Terraform Registry](https://registry.terraform.io/providers/dbsnapper/dbsnapper/latest). You can now manage DBSnapper resources using the Terraform platform and Infrastructure as Code (IaC).
    - We've released a **GitHub Action** to the GitHub Marketplace. The [Install DBSnapper Agent GitHub Action](https://github.com/marketplace/actions/install-dbsnapper-agent) simplifies downloading the DBSnapper Agent to the runner.
    - We've released new articles on using DBSnapper with GitHub Actions and Amazon ECS. Check out [Part 1](articles/dbsnapper-github-actions-amazon-ecs.md) and the simplified approach in [Part 2](articles/dbsnapper-github-actions-ecs-simplified.md) for more information. 
    - [Version 2.7.0 has been released](release-notes.md#v270---team-sharing-for-sso-groups) which supports SSO and Shared Team Targets.
    
<!-- prettier-ignore-end -->

<p class="img-box">
  <img src="/static/cloud/targets-with-shared-targets.jpg" alt="DBSnapper Cloud - All Targets Page" width="80%">
  <br/>
  <small>DBSnapper Cloud Targets page including Shared Team Targets.</small>
</p>

DBSnapper revolutionizes the way development teams handle database snapshotting, bringing de-identified production data into the heart of development and testing workflows. It stands as a robust alternative to the traditional, often cumbersome methods of creating development and test fixtures. With DBSnapper, you get to leverage real, production-grade data, stripped of its sensitive elements, to power your development and testing environments.

## Sign Up for DBSnapper Cloud

[Sign Up for the DBSnapper Cloud](https://app.dbsnapper.com/sign_up) and get started with a safer, simpler way to manage your database snapshots.

## Releases and Integrations

- [![GitHub Release](https://img.shields.io/github/v/release/dbsnapper/dbsnapper?label=DBSnapper%20Agent)](https://github.com/dbsnapper/dbsnapper/releases)
  The DBSnapper Agent interacts with your databases and communicates with the DBSnapper Cloud.

- [![Dynamic JSON Badge](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fregistry.terraform.io%2Fv2%2Fprovider-versions%2F55330&query=%24.data.attributes.tag&label=Terraform%20Provider)](https://registry.terraform.io/providers/dbsnapper/dbsnapper/latest)
  The DBSnapper Terraform Provider allows you to manage DBSnapper resources using the Terraform platform and Infrastucture as Code.

- [![GitHub Release](https://img.shields.io/github/v/release/dbsnapper/install-dbsnapper-agent-action?label=Github%20Action%20-%20Install%20DBSnapper%20Agent)](https://github.com/marketplace/actions/install-dbsnapper-agent)
  The DBSnapper Github Action allows you to include DBSnapper in your CI/CD pipelines for automated management of database snapshots.

- [![Static Badge](https://img.shields.io/badge/Okta_OIDC_SSO-Learn_More-blue)](./dbsnapper-cloud/sso/sso-okta-oidc.md)
  DBSnapper supports [Okta OIDC](./dbsnapper-cloud/sso/sso-okta-oidc.md) for Single Sign-On (SSO) authentication and group sharing.

## Features Overview

In addition to the main features of Subsetting, Snapshotting, Sanitization, and Sharing, DBSnapper offers a range of features that simplify database workload snapshotting and sharing. Here are some of the key features of DBSnapper:

- **Terraform Provider** - Manage DBSnapper resources using the Terraform platform and Infrastructure as Code.
- **Database Subsetting**: Create smaller, relationally-complete snapshots of your database, allowing you to work with a subset of the data for development or testing purposes.
- **Bring Your Own Storage**: Use your preferred cloud storage provider (i.e. Amazon S3 / Cloudflare R2) for storing snapshots securely in your private infrastructure.
- **SSO**: Single Sign-On (SSO) support enables seamless authentication and snapshot sharing with your team members.
- **GitHub Actions Compatible**: DBSnapper works well with GitHub Actions allowing you to integrate database snapshotting into your CI/CD pipelines. A GitHub Action is available on the Marketplace that will [Install the DBSnapper Agent](https://github.com/marketplace/actions/install-dbsnapper-agent) on your GitHub runner.
- **Presigned Upload and Downloads from Cloud Storage**: Securely upload and download snapshots from your cloud storage provider using presigned URLs.
- **Database Support**: DBSnapper supports Postgresql and MySQL databases with more database support coming soon.
- **Ephemeral Sanitization**: Run sanitization operations without the need to setup a temporary sanitization database or schema.
- **Templating Engine**: Use the built-in templating engine to allow sensitive data to be passed as environment variables.
- **Docker Enabled**: DBSnapper leverages Docker for access to database vendor tools and utilities, ephemeral sanitization, and other operations.
- **Zero-Config Operation**: Execute complex database workload operations in a single command without the need for complex configuration files and prerequisite setup steps.
- **Private-cloud First**: DBSnapper is designed to work in private cloud environments, ensuring that your data remains within your infrastructure.

## Main Capabilities

Now for a closer look at the main features of DBSnapper:

### Subsetting - Smaller snapshots, relationally complete

DBSnapper 2.0 introduces **Database Subsetting**. This feature allows you to create smaller, more manageable relationally-complete snapshots of your database. This is particularly useful when you want to create a snapshot of a large database, but only need a subset of the data for development or testing purposes.

- **Efficient Data Management**: Instead of working with the entire database, which can be cumbersome and resource-intensive, you can select just the relevant parts you need. This leads to quicker operations, lower resource usage, and more efficient data handling.

- **Maintains Data Integrity**: Despite working with a smaller dataset, the integrity and relational structure of the original database are preserved. This is crucial when developing or testing applications, as it ensures that any interactions with the subset reflect real-world scenarios.

- **Customizable to Specific Needs**: DBSnapper 2.0 provides flexibility in defining the criteria for subsetting. Whether it's specific tables, rows, or a particular set of data, you can tailor the subset to meet the exact needs of your project.

### Snapshotting - Simplified database backups

DBSnapper offers an efficient and powerful solution for snapshotting databases that simplifies the snapshotting process for different database platforms.

DBSnapper's snapshotting capability is not just about capturing data; it's a strategic tool that integrates into and enhances the entire software development lifecycle. From creating realistic test environments and aiding in AI model training to providing essential support in CI/CD pipelines, DBSnapper stands as an indispensable asset for any software development team aiming to streamline and improve their database management and utilization in a modern startup environment.

- **Real-world Test Cases**: Utilizing de-identified data snapshots, you can create more effective and realistic test cases. This helps in identifying potential issues in a more accurate production-like environment.

- **Seamless Integration with CI/CD Pipelines**: DBSnapper can be easily [intergrated into CI/CD pipelines such as GitHub Actions](articles/dbsnapper-github-actions-amazon-ecs.md), automating the process of generating snapshots for your team and ensuring the team is using the latest and most accurate data for testing.

- **Training AI Models**: For AI and machine learning initiatives, having access to diverse, real-world data sets is crucial. DBSnapper's ability to provide de-identified snapshots of real operational data can significantly enhance the training process of AI models, leading to more accurate models.

### Sanitization - De-identification and sensitive data removal

DBSnapper enables you to de-identify and sanitize your production data, removing sensitive information such as personal details, financial data, and other confidential information. This is crucial for ensuring compliance with data protection regulations and maintaining the privacy and security of your users.

- **Data Provenance During Sanitization**: The DBSnapper tools are designed to give you can full-control over your data, ensuring that no sensitive or proprietary data leaves your environment.

- **Adherence to GDPR and Other Regulations**: In the era of stringent data protection laws like the GDPR, CCPA, and others, DBSnapper's sanitization feature ensures that your data handling practices are compliant, reducing the risk of legal complications and hefty fines.

- **Maintaining Data Utility Post-Sanitization**: Despite the removal of sensitive data, the utility and integrity of the dataset are preserved, making it suitable for development, testing, and analysis without compromising privacy.

### Share - Securely distribute snapshots via DBSnapper Cloud

The sharing aspect of DBSnapper is made possible through the DBSnapper Cloud, a critical feature for secure storage and distribution of database snapshots. It's designed for seamless collaboration within your team or for integration with automated processes.

- **SSO-Aware Team Sharing**: DBSnapper Cloud supports Single Sign-On (SSO), and is SSO-Group aware, allowing you to easily share snapshots with your team members, using the groups you've already set up in your SSO provider.

- **Flexibility of Storage Choices**: With DBSnapper, you have the flexibility to 'Bring Your Own Cloud Storage Provider'. This means you can choose the cloud storage that best aligns with your company’s policies and data management strategies, ensuring that your data remains within your approved PaaS vendor.

- **Easy Access for Team Members**: Shared snapshots are easily accessible to authorized team members. This facilitates collaboration, as team members can work with the same data sets in a synchronized manner.

- **Integration with Automated Processes**: The DBSnapper Cloud is designed for integration with automated processes, such as CI/CD pipelines, making it simpler to incorporate database snapshots into your development and deployment workflows.
