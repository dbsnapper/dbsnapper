---
title: Welcome
description: DBSnapper simplifies the process of creating de-identified database snapshots that can be used for real-world development, testing, and AI model training.
---

<p align="center">
  <img src="/static/dbs-docs.jpg" alt="DBSnapper" width="90%">
</p>

# Welcome to DBSnapper

<p align="center">
  <img src="/static/cloud/targets-with-shared-targets.jpg" alt="DBSnapper Cloud - All Targets Page" width="70%">
  <br/>
  <small>DBSnapper Cloud Targets page including Shared Team Targets.</small>
</p>

DBSnapper revolutionizes the way development teams handle database snapshotting, bringing de-identified production data into the heart of development and testing workflows. It stands as a robust alternative to the traditional, often cumbersome methods of creating development and test fixtures. With DBSnapper, you get to leverage real, production-grade data, stripped of its sensitive elements, to power your development and testing environments.

## Features

In addition to the main features of Subsetting, Snapshotting, Sanitization, and Sharing, DBSnapper offers a range of features that simplify database workload snapshotting and sharing. Here are some of the key features of DBSnapper:

- **SSO**: Single Sign-On (SSO) support enables seamless authentication and snapshot sharing with your team members.
- **Github Actions Compatible**: DBSnapper works well with Github Actions allowing you to integrate database snapshotting into your CI/CD pipelines. A GitHub Action is available on the Marketplace that will [Install the DBSnapper Agent](https://github.com/marketplace/actions/install-dbsnapper-agent) on your Github runner.
- **Bring Your Own Storage**: Use your preferred cloud storage provider (i.e. Amazon S3 / Cloudflare R2) for storing snapshots securely in your private infrastructure.
- **Presigned Upload and Downloads from Cloud Storage**: Securely upload and download snapshots from your cloud storage provider using presigned URLs.
- **Database Support**: DBSnapper supports Postgresql and MySQL databases with more database support coming soon.
- **Ephemeral Sanitization**: Run sanitization operations without the need to setup a temporary sanitization database or schema.
- **Templating Engine**: Use the built-in templating engine to allow sensitive data to be passed as environment variables.
- **Docker Enabled**: DBSnapper leverages Docker for access to database vendor tools and utilities, ephemeral sanitization, and other operations.
- **Zero-Config Operation**: Execute complex database workload operations in a single command without the need for complex configuration files and prerequisite setup steps.
- **Private-cloud First**: DBSnapper is designed to work in private cloud environments, ensuring that your data remains within your infrastructure.

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

- **Seamless Integration with CI/CD Pipelines**: DBSnapper can be easily [intergrated into CI/CD pipelines such as Github Actions](/docs/articles/dbsnapper-github-actions-amazon-ecs.md), automating the process of generating snapshots for your team and ensuring the team is using the latest and most accurate data for testing.

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

## Get Started

Ready to get started with DBSnapper? Head over to the [Installation](installation.md) guide to install DBSnapper on your system. Once installed, you can follow the [Quick Start](quick-start.md) guide to create your first snapshot.
