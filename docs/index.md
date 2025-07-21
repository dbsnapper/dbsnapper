---
title: Welcome to DBSnapper
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

## System Prerequisites

Before getting started with DBSnapper, ensure you have:

- **Operating System**: macOS, Linux, or Windows (WSL2 recommended)
- **Database Access**: PostgreSQL (9.6+) or MySQL (5.7+) with appropriate credentials
- **Storage Space**: Minimum 2x your database size for snapshot operations
- **Network Access**: Outbound HTTPS for DBSnapper Cloud features (optional)
- **Memory**: 512MB RAM minimum, 2GB+ recommended for large databases

[View detailed system requirements →](requirements.md)

## Choose Your User Journey

### 🚀 **Beginner: First Database Snapshot**
*Perfect for developers new to database snapshots*

**What You'll Learn**: Create, sanitize, and load your first database snapshot  
**Time Required**: ~15 minutes  
**Prerequisites**: Basic database knowledge

<div class="grid cards" markdown>

- :material-download:{ .lg .middle } **Step 1: Install DBSnapper**
  
    ---
    
    Download and install the CLI tool for your operating system
    
    [:octicons-arrow-right-24: Installation Guide](installation.md){ .md-button .md-button--primary }

- :material-rocket-launch:{ .lg .middle } **Step 2: Quick Start Tutorial**
  
    ---
    
    Create your first snapshot with our guided tutorial
    
    [:octicons-arrow-right-24: Start Tutorial](quick-start.md){ .md-button .md-button--primary }

- :material-shield-check:{ .lg .middle } **Step 3: Basic Sanitization**
  
    ---
    
    Learn to remove sensitive data from snapshots
    
    [:octicons-arrow-right-24: Sanitization Basics](sanitize/introduction.md){ .md-button }

- :material-arrow-right:{ .lg .middle } **Step 4: What's Next?**
  
    ---
    
    Explore subsetting, sharing, and automation
    
    [:octicons-arrow-right-24: Next Steps](#beginners-next-steps){ .md-button }

</div>

### ⚡ **Experienced: Production Workflows**
*For teams ready to implement snapshots in production*

**What You'll Learn**: Advanced configurations, team workflows, CI/CD integration  
**Prerequisites**: Familiarity with database operations and DevOps practices

<div class="grid cards" markdown>

- :material-shield-lock:{ .lg .middle } **Architecture & Security**
  
    ---
    
    Understand DBSnapper's architecture and security model
    
    [:octicons-arrow-right-24: How It Works](how-it-works.md){ .md-button .md-button--primary }

- :material-cog:{ .lg .middle } **Advanced Configuration**
  
    ---
    
    Master complex sanitization rules and subsetting strategies
    
    [:octicons-arrow-right-24: Configuration Reference](configuration.md){ .md-button .md-button--primary }

- :material-account-group:{ .lg .middle } **Team Collaboration**
  
    ---
    
    Set up shared snapshots and access controls
    
    [:octicons-arrow-right-24: Team Workflows](share/introduction.md){ .md-button }

- :material-robot:{ .lg .middle } **Automation & CI/CD**
  
    ---
    
    Integrate with GitHub Actions, Jenkins, and more
    
    [:octicons-arrow-right-24: Automation Guide](articles/dbsnapper-github-actions-ecs-simplified.md){ .md-button }

</div>

### 🏢 **Enterprise: Organization-Wide Deployment**
*Scale database snapshots across your organization*

**What You'll Learn**: SSO integration, compliance, team management  
**Features**: Enterprise authentication, audit logs, priority support

<div class="grid cards" markdown>

- :material-cloud:{ .lg .middle } **DBSnapper Cloud Setup**
  
    ---
    
    Deploy the cloud platform for your organization
    
    [:octicons-arrow-right-24: Cloud Overview](dbsnapper-cloud/introduction.md){ .md-button .md-button--primary }

- :material-login:{ .lg .middle } **SSO Configuration**
  
    ---
    
    Integrate with Okta, Azure AD, or Google Workspace
    
    [:octicons-arrow-right-24: SSO Setup Guide](dbsnapper-cloud/sso/sso-okta-oidc.md){ .md-button .md-button--primary }

- :material-certificate:{ .lg .middle } **Compliance & Security**
  
    ---
    
    GDPR, HIPAA, and SOC 2 compliance strategies
    
    [:octicons-arrow-right-24: Compliance Guide](sanitize/configuration.md){ .md-button }

- :material-account-cog:{ .lg .middle } **Team Management**
  
    ---
    
    Manage users, permissions, and shared resources
    
    [:octicons-arrow-right-24: Team Administration](dbsnapper-cloud/targets.md){ .md-button }

</div>

## Migration Guides

### From Competitor Tools

<div class="grid cards" markdown>

- :material-database:{ .lg .middle } **From pg_dump/mysqldump**
  
    ---
    
    Migrate from native database tools to DBSnapper
    
    - [PostgreSQL Migration](database-engines/postgresql-local.md)
    - [MySQL Migration](database-engines/mysql-local.md)

- :material-shield-account:{ .lg .middle } **From Anonymize.io**
  
    ---
    
    Transition your anonymization rules to DBSnapper
    
    - [Sanitization Migration](sanitize/introduction.md)
    - [Rule Conversion Guide](sanitize/configuration.md)

- :material-script:{ .lg .middle } **From Custom Scripts**
  
    ---
    
    Replace bash scripts with DBSnapper workflows
    
    - [Script Migration Patterns](configuration.md)
    - [Automation Examples](cmd/dbsnapper.md)

- :material-docker:{ .lg .middle } **From Docker Setups**
  
    ---
    
    Integrate DBSnapper with containerized databases
    
    - [Docker Integration](database-engines/postgresql-docker.md)
    - [MySQL Docker Setup](database-engines/mysql-docker.md)

</div>

## Quick Troubleshooting

<!-- prettier-ignore-start -->
!!! warning "Common Issues & Solutions"

    **Installation Problems**
    
    - [Binary not found errors](installation.md#troubleshooting)
    - [Permission issues](installation.md#permission-errors)
    - [Version compatibility](installation.md#version-requirements)
    
    **Connection Errors**
    
    - [Database authentication failures](database-engines/introduction.md#authentication-issues)
    - [Network connectivity problems](database-engines/introduction.md#network-troubleshooting)
    - [SSL/TLS configuration](database-engines/introduction.md#ssl-configuration)
    
    **Snapshot Issues**
    
    - [Storage space problems](snapshot/configuration.md#storage-requirements)
    - [Performance optimization](snapshot/configuration.md#performance-tuning)
    - [Large database handling](subset/introduction.md#large-databases)
    
    **Configuration Problems**
    
    - [YAML syntax errors](configuration.md#validation-and-testing)
    - [Environment variables](configuration.md#environment-variables)
    - [Path resolution issues](configuration.md#path-configuration)
<!-- prettier-ignore-end -->

[View Complete Troubleshooting Guide →](troubleshooting.md)

## Why DBSnapper?

<div class="grid cards" markdown>

- :material-database-check:{ .lg .middle } **Real Production Data**
  
    ---
    
    Work with actual data patterns and edge cases, not synthetic fixtures that miss real-world complexity

- :material-shield-lock:{ .lg .middle } **Privacy Compliant**
  
    ---
    
    Built-in sanitization meets GDPR, HIPAA, and other regulations with automated PII removal

- :material-cloud-off:{ .lg .middle } **Zero Infrastructure**
  
    ---
    
    No databases to manage - ephemeral processing means no persistent sensitive data

- :material-account-group:{ .lg .middle } **Team Ready**
  
    ---
    
    Share snapshots securely with SSO integration and granular access controls

</div>

## Platform Integrations

### Development Tools
[![GitHub Release](https://img.shields.io/github/v/release/dbsnapper/dbsnapper?label=DBSnapper%20Agent)](https://github.com/dbsnapper/dbsnapper/releases) **DBSnapper Agent** - Core CLI tool

![Visual Studio Marketplace Version](https://img.shields.io/visual-studio-marketplace/v/dbsnapper.vscode-dbsnapper?label=VSCode%20Extension) **VSCode Extension** - In-editor snapshot management

### Infrastructure & Automation
[![Dynamic JSON Badge](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fregistry.terraform.io%2Fv2%2Fprovider-versions%2F55330&query=%24.data.attributes.tag&label=Terraform%20Provider)](https://registry.terraform.io/providers/dbsnapper/dbsnapper/latest) **Terraform Provider** - Infrastructure as Code

[![GitHub Release](https://img.shields.io/github/v/release/dbsnapper/install-dbsnapper-agent-action?label=Github%20Action%20-%20Install%20DBSnapper%20Agent)](https://github.com/marketplace/actions/install-dbsnapper-agent) **GitHub Actions** - CI/CD automation

### Enterprise Security
[![Static Badge](https://img.shields.io/badge/Okta_OIDC_SSO-Learn_More-blue)](./dbsnapper-cloud/sso/sso-okta-oidc.md) **SSO Integration** - Enterprise authentication

## Get Started Now

<div class="grid cards" markdown>

- :material-rocket-launch:{ .lg .middle } **Try DBSnapper Free**
  
    ---
    
    Download the CLI and create your first snapshot in minutes
    
    [:octicons-arrow-right-24: Download DBSnapper](installation.md){ .md-button .md-button--primary }

- :material-cloud:{ .lg .middle } **Start Cloud Trial**
  
    ---
    
    Get team sharing, SSO, and managed storage with DBSnapper Cloud
    
    [:octicons-arrow-right-24: Start Free Trial](https://app.dbsnapper.com/sign_up){ .md-button .md-button--primary }

</div>

## Beginner's Next Steps

After completing the quick start tutorial, explore these features:

1. **[Data Subsetting](subset/introduction.md)** - Create smaller, focused snapshots
2. **[Advanced Sanitization](sanitize/configuration.md)** - Complex data masking rules
3. **[Snapshot Sharing](share/introduction.md)** - Collaborate with your team
4. **[Automation Setup](articles/dbsnapper-github-actions-ecs-simplified.md)** - CI/CD integration
5. **[Performance Tuning](snapshot/configuration.md#performance-optimization)** - Optimize for large databases

## Get Support

- **Community**: [GitHub Discussions](https://github.com/dbsnapper/dbsnapper/discussions)
- **Bug Reports**: [GitHub Issues](https://github.com/dbsnapper/dbsnapper/issues)
- **Enterprise Support**: [Contact Us](https://dbsnapper.com/contact)
- **Documentation Feedback**: [Docs Issues](https://github.com/dbsnapper/dbsnapper-docs/issues)