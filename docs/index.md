---
title: Welcome
description: DBSnapper simplifies the process of creating de-identified database snapshots that can be used for real-world development, testing, and AI model training.
---

<p align="center">
  <img src="/static/dbs-docs.jpg" alt="DBSnapper" width="90%">
</p>

# Welcome to DBSnapper

<!-- prettier-ignore-start -->
!!! note "Latest Updates"

    - **VSCode Extension** - Available on the [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=dbsnapper.vscode-dbsnapper) for in-editor database snapshot management
    - **Terraform Provider** - Manage DBSnapper resources with [Infrastructure as Code](https://registry.terraform.io/providers/dbsnapper/dbsnapper/latest)
    - **GitHub Actions** - Automate database snapshots in your CI/CD pipelines
    
<!-- prettier-ignore-end -->

<p class="img-box">
  <img src="/static/dbs-architecture.jpg" alt="DBSnapper Architecture" width="90%">
  <br/>
  <small>DBSnapper Architecture Overview</small>
</p>

DBSnapper revolutionizes the way development teams handle database snapshotting, bringing de-identified production data into the heart of development and testing workflows. It stands as a robust alternative to traditional, often cumbersome methods for creating development and test fixtures. With DBSnapper, you get to leverage real, production-grade data, stripped of its sensitive elements, to power your development and testing environments.

## Quick Start

Ready to get started? Here's your path to creating your first database snapshot:

1. **[Install DBSnapper](installation.md)** - Get the CLI tool set up
2. **[Quick Start Guide](quick-start.md)** - Create your first snapshot in minutes
3. **[Sign up for DBSnapper Cloud](https://app.dbsnapper.com/sign_up)** - Share snapshots with your team

## Sign Up for DBSnapper Cloud

[Sign Up for the DBSnapper Cloud](https://app.dbsnapper.com/sign_up) and get started with a safer, simpler way to manage your database snapshots.

## Releases and Integrations

- [![GitHub Release](https://img.shields.io/github/v/release/dbsnapper/dbsnapper?label=DBSnapper%20Agent)](https://github.com/dbsnapper/dbsnapper/releases)
  The DBSnapper Agent interacts with your databases and communicates with the DBSnapper Cloud.

- ![Visual Studio Marketplace Version](https://img.shields.io/visual-studio-marketplace/v/dbsnapper.vscode-dbsnapper?label=VSCode%20Extension) - DBSnapper Extension for VSCode, allowing you to load database snapshots directly from your editor.

- [![Dynamic JSON Badge](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fregistry.terraform.io%2Fv2%2Fprovider-versions%2F55330&query=%24.data.attributes.tag&label=Terraform%20Provider)](https://registry.terraform.io/providers/dbsnapper/dbsnapper/latest)
  The DBSnapper Terraform Provider allows you to manage DBSnapper resources using the Terraform platform and Infrastructure as Code.

- [![GitHub Release](https://img.shields.io/github/v/release/dbsnapper/install-dbsnapper-agent-action?label=Github%20Action%20-%20Install%20DBSnapper%20Agent)](https://github.com/marketplace/actions/install-dbsnapper-agent)
  The DBSnapper GitHub Action allows you to include DBSnapper in your CI/CD pipelines for automated management of database snapshots.

- [![Static Badge](https://img.shields.io/badge/Okta_OIDC_SSO-Learn_More-blue)](./dbsnapper-cloud/sso/sso-okta-oidc.md)
  DBSnapper supports [Okta OIDC](./dbsnapper-cloud/sso/sso-okta-oidc.md) for Single Sign-On (SSO) authentication and group sharing.

## Key Features

DBSnapper provides comprehensive database snapshot management with these core capabilities:

### 🗄️ **Database Support & Storage**

- **PostgreSQL and MySQL** databases with more engines coming soon
- **Bring Your Own Storage** - Use Amazon S3, Cloudflare R2, or your preferred cloud storage
- **Presigned URLs** for secure upload and download operations

### 🔒 **Security & Compliance**

- **Data Sanitization** - Remove or mask sensitive information
- **SSO Integration** - Okta OIDC support with group-based sharing
- **Private-cloud First** - Your data stays in your infrastructure

### ⚙️ **Developer Experience**

- **Zero-Config Operation** - Complex operations in a single command
- **Terraform Provider** - Infrastructure as Code support
- **GitHub Actions** - CI/CD pipeline integration
- **VSCode Extension** - In-editor snapshot management
- **Docker-Enabled** - Leverages containerization for database tools

### 📊 **Advanced Capabilities**

- **Database Subsetting** - Create smaller, relationally-complete snapshots
- **Ephemeral Sanitization** - No need for temporary databases
- **Templating Engine** - Environment variable support for sensitive data

## Core Capabilities

### 📦 Subsetting - Smaller snapshots, relationally complete

Create smaller, more manageable relationally-complete snapshots of your database. Perfect for development and testing when you only need a subset of production data.

**Key Benefits:**

- **Efficient Data Management** - Work with relevant data only, reducing resource usage
- **Maintains Data Integrity** - Preserves relational structure and referential integrity
- **Customizable Criteria** - Define specific tables, rows, or data sets to include

### Snapshotting - Simplified database backups

DBSnapper offers an efficient and powerful solution for snapshotting databases that simplifies the snapshotting process for different database platforms.

DBSnapper's snapshotting capability is not just about capturing data; it's a strategic tool that integrates into and enhances the entire software development lifecycle. From creating realistic test environments and aiding in AI model training to providing essential support in CI/CD pipelines, DBSnapper stands as an indispensable asset for any software development team aiming to streamline and improve their database management and utilization in a modern startup environment.

- **Real-world Test Cases**: Utilizing de-identified data snapshots, you can create more effective and realistic test cases. This helps in identifying potential issues in a more accurate production-like environment.

- **Seamless Integration with CI/CD Pipelines**: DBSnapper can be easily [integrated into CI/CD pipelines such as GitHub Actions](articles/dbsnapper-github-actions-amazon-ecs.md), automating the process of generating snapshots for your team and ensuring the team is using the latest and most accurate data for testing.

- **Training AI Models**: For AI and machine learning initiatives, having access to diverse, real-world data sets is crucial. DBSnapper's ability to provide de-identified snapshots of real operational data can significantly enhance the training process of AI models, leading to more accurate models.

### Sanitization - De-identification and sensitive data removal

DBSnapper enables you to de-identify and sanitize your production data, removing sensitive information such as personal details, financial data, and other confidential information. This is crucial for ensuring compliance with data protection regulations and maintaining the privacy and security of your users.

- **Data Provenance During Sanitization**: The DBSnapper tools are designed to give you full control over your data, ensuring that no sensitive or proprietary data leaves your environment.

- **Adherence to GDPR and Other Regulations**: In the era of stringent data protection laws like the GDPR, CCPA, and others, DBSnapper's sanitization feature ensures that your data handling practices are compliant, reducing the risk of legal complications and hefty fines.

- **Maintaining Data Utility Post-Sanitization**: Despite the removal of sensitive data, the utility and integrity of the dataset are preserved, making it suitable for development, testing, and analysis without compromising privacy.

### Share - Securely distribute snapshots via DBSnapper Cloud

The sharing aspect of DBSnapper is made possible through the DBSnapper Cloud, a critical feature for secure storage and distribution of database snapshots. It's designed for seamless collaboration within your team or for integration with automated processes.

- **SSO-Aware Team Sharing**: DBSnapper Cloud supports Single Sign-On (SSO), and is SSO-Group aware, allowing you to easily share snapshots with your team members, using the groups you've already set up in your SSO provider.

- **Flexibility of Storage Choices**: With DBSnapper, you have the flexibility to 'Bring Your Own Cloud Storage Provider'. This means you can choose the cloud storage that best aligns with your company’s policies and data management strategies, ensuring that your data remains within your approved PaaS vendor.

- **Easy Access for Team Members**: Shared snapshots are easily accessible to authorized team members. This facilitates collaboration, as team members can work with the same datasets in a synchronized manner.

- **Integration with Automated Processes**: The DBSnapper Cloud is designed for integration with automated processes, such as CI/CD pipelines, making it simpler to incorporate database snapshots into your development and deployment workflows.
