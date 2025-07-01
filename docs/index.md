---
title: Welcome
description: DBSnapper simplifies the process of creating de-identified database snapshots that can be used for real-world development, testing, and AI model training.
---

<p align="center">
  <img src="/static/dbs-docs.jpg" alt="DBSnapper" width="90%">
</p>

# Welcome to DBSnapper

**DBSnapper revolutionizes database snapshot workflows** by creating sanitized, development-ready snapshots of production databases. Get real-world data for development and testing while maintaining security and compliance.

<!-- prettier-ignore-start -->
!!! tip "Latest Updates"

    - **VSCode Extension** - Available on the [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=dbsnapper.vscode-dbsnapper) for in-editor database snapshot management
    - **Terraform Provider** - Manage DBSnapper resources with [Infrastructure as Code](https://registry.terraform.io/providers/dbsnapper/dbsnapper/latest)
    - **GitHub Actions** - Automate database snapshots in your CI/CD pipelines
    
<!-- prettier-ignore-end -->

## Choose Your Path

### 🚀 **New to DBSnapper?**
Perfect for developers getting started with database snapshots.

**Prerequisites:** Access to a PostgreSQL or MySQL database  
**Time Required:** ~15 minutes

1. **[System Requirements](requirements.md)** - Check your setup
2. **[Installation Guide](installation.md)** - Get the CLI tool
3. **[Quick Start Tutorial](quick-start.md)** - Create your first snapshot

### ⚡ **Experienced Developer?**
Jump straight to advanced configurations and integrations.

**Prerequisites:** Familiarity with database operations  
**Best For:** Teams ready to implement snapshots in production workflows

1. **[How DBSnapper Works](how-it-works.md)** - Architecture overview
2. **[Configuration Reference](configuration.md)** - Advanced setup options
3. **[Integration Examples](articles/dbsnapper-github-actions-ecs-simplified.md)** - CI/CD workflows

### 🏢 **Enterprise Team?**
Scale database snapshots across your organization with team features.

**Prerequisites:** Team database management needs  
**Features:** SSO, team sharing, compliance tools

1. **[DBSnapper Cloud Overview](dbsnapper-cloud/introduction.md)** - Platform features
2. **[SSO Setup (Okta)](dbsnapper-cloud/sso/sso-okta-oidc.md)** - Enterprise authentication
3. **[Team Management](dbsnapper-cloud/targets.md)** - Shared resources

## Quick Migration Guides

### From Traditional Database Dumps
- **pg_dump/mysqldump users**: See [Database Engines](database-engines/introduction.md)
- **Docker database setups**: Check [Docker Integration](database-engines/postgresql-docker.md)

### From Competitor Tools
- **From Anonymize.io**: Review [Sanitization Strategies](sanitize/introduction.md)
- **From custom scripts**: See [Configuration Templates](configuration.md#common-patterns)

<p class="img-box">
  <img src="/static/dbs-architecture.jpg" alt="DBSnapper Architecture" width="90%">
  <br/>
  <small>DBSnapper Architecture Overview</small>
</p>

## Why DBSnapper?

### ✅ **The DBSnapper Advantage**
- **Real Production Data**: Work with actual data patterns, not synthetic fixtures
- **Privacy Compliant**: Built-in sanitization meets GDPR, HIPAA, and other regulations
- **Zero Infrastructure**: No databases to manage - ephemeral processing only
- **Team Ready**: Share snapshots securely with SSO and access controls

### ❌ **Traditional Challenges Solved**
- **No More Stale Test Data**: Always work with fresh, relevant datasets
- **End Synthetic Data Limitations**: Get realistic edge cases and data distributions  
- **Eliminate Security Risks**: Automatic PII removal and data masking
- **Stop Complex Setup**: One command replaces hours of database configuration

## Quick Help & Troubleshooting

### 🔧 **Common Issues**
- **Installation Problems**: [Installation Troubleshooting](installation.md#troubleshooting)
- **Connection Errors**: [Database Connection Guide](database-engines/introduction.md#common-connection-issues)
- **Configuration Issues**: [Configuration Validation](configuration.md#validation-and-testing)
- **Performance Optimization**: [Performance Best Practices](snapshot/configuration.md#performance-optimization)

### 📞 **Get Support**
- **Community**: [GitHub Discussions](https://github.com/dbsnapper/dbsnapper/discussions)
- **Enterprise Support**: [Contact Us](https://dbsnapper.com/contact)
- **Bug Reports**: [GitHub Issues](https://github.com/dbsnapper/dbsnapper/issues)

## Platform Integrations

### 🛠️ **Development Tools**
[![GitHub Release](https://img.shields.io/github/v/release/dbsnapper/dbsnapper?label=DBSnapper%20Agent)](https://github.com/dbsnapper/dbsnapper/releases) **DBSnapper Agent** - Core CLI tool for database operations

![Visual Studio Marketplace Version](https://img.shields.io/visual-studio-marketplace/v/dbsnapper.vscode-dbsnapper?label=VSCode%20Extension) **VSCode Extension** - Load snapshots directly from your editor

### ☁️ **Infrastructure**
[![Dynamic JSON Badge](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fregistry.terraform.io%2Fv2%2Fprovider-versions%2F55330&query=%24.data.attributes.tag&label=Terraform%20Provider)](https://registry.terraform.io/providers/dbsnapper/dbsnapper/latest) **Terraform Provider** - Infrastructure as Code resource management

[![GitHub Release](https://img.shields.io/github/v/release/dbsnapper/install-dbsnapper-agent-action?label=Github%20Action%20-%20Install%20DBSnapper%20Agent)](https://github.com/marketplace/actions/install-dbsnapper-agent) **GitHub Actions** - CI/CD pipeline automation

### 🔐 **Enterprise Security**
[![Static Badge](https://img.shields.io/badge/Okta_OIDC_SSO-Learn_More-blue)](./dbsnapper-cloud/sso/sso-okta-oidc.md) **SSO Integration** - Okta OIDC authentication and group-based sharing

## DBSnapper Cloud

[**Start Your Free Trial**](https://app.dbsnapper.com/sign_up) - Get team sharing, SSO, and managed storage in minutes.

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
