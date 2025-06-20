---
title: Philosophy
description: The principles and values that guide DBSnapper's design and development approach to database snapshot management.
---

## Why DBSnapper Exists

Development teams across the industry face a common challenge: how to work with realistic data for development and testing without compromising security or privacy. Traditional approaches often fall short:

- **Brittle test fixtures** that don't reflect real-world data complexity
- **Manual database copying processes** that are error-prone and time-consuming
- **Compliance violations** when production data is used without proper sanitization
- **Development bottlenecks** when teams can't access quality test data

DBSnapper was created to solve these fundamental problems by providing an automated, secure, and compliant way to create de-identified database snapshots. Our goal is to make realistic development data accessible to every team, regardless of their infrastructure or compliance requirements.

## Core Principles

DBSnapper is built on fundamental principles that prioritize security, privacy, and user control. These principles guide every architectural decision and feature development.

### Security and Privacy First

Data security and privacy form the foundation of everything we build:

**Data Sovereignty**: Your data remains under your complete control at all times. DBSnapper never requires access to your databases outside your approved infrastructure, and we don't store copies of your sensitive data.

**Encryption Everywhere**: When using DBSnapper Cloud, all sensitive information is encrypted both at rest and in transit. Connection string templating allows you to retrieve credentials from environment variables, avoiding the need to provide sensitive credentials to external services.

**Compliance Ready**: Built-in data sanitization and subsetting features help teams maintain compliance with data protection regulations like GDPR, HIPAA, and SOX.

### Bring Your Own Infrastructure

Rather than locking you into proprietary solutions, DBSnapper embraces a "bring your own" philosophy:

**Your Cloud Storage**: Use your preferred cloud storage provider (Amazon S3, Cloudflare R2, or others) to ensure data stays within your approved infrastructure and reduces overall costs.

**Your Environment**: The DBSnapper Agent works equally well in on-premises environments, cloud infrastructure, or local development machines.

**Your Workflow**: Integrate with existing CI/CD pipelines, development tools, and team processes without disruption.

### Simplicity Without Compromise

Complex problems don't always require complex solutions. DBSnapper prioritizes simplicity while maintaining powerful capabilities:

**Minimal Dependencies**: Built with Go to create statically linked binaries that are easy to distribute and run across different operating systems and platforms.

**Smart Containerization**: Optional Docker-based database engines eliminate the need to install database tools locally while supporting different database versions and platforms.

**Configuration as Code**: YAML-based configuration files make snapshot processes reproducible and version-controllable.

## Design Decisions

These principles translate into specific architectural choices that make DBSnapper both powerful and accessible:

### Agent-Based Architecture

The DBSnapper Agent runs in your environment, maintaining complete control over your data while providing secure communication with DBSnapper Cloud for orchestration and sharing features.

### Database Agnostic Approach

Support for multiple database engines (PostgreSQL, MySQL) with a pluggable architecture that allows for future expansion without breaking existing workflows.

### Storage Flexibility

Integration with major cloud storage providers ensures you can use existing infrastructure investments while maintaining data governance requirements.

### Automation Ready

Built-in support for CI/CD integration, Infrastructure as Code (Terraform), and development environment integration (VS Code extension) enables automated workflows from day one.

## The Result: Better Development Workflows

These principles and design decisions work together to deliver tangible benefits for development teams:

- **Faster Development**: Realistic test data accelerates feature development and bug identification
- **Improved Quality**: Testing against production-like data improves application reliability
- **Enhanced Security**: Built-in sanitization and encryption protect sensitive information
- **Reduced Costs**: Use existing infrastructure and avoid vendor lock-in
- **Compliance Confidence**: Automated de-identification supports regulatory requirements

## Getting Started

Ready to experience these principles in action?

1. **[Install DBSnapper](installation.md)** to get started with the CLI tool
2. **[Follow the Quick Start Guide](quick-start.md)** to create your first sanitized snapshot
3. **[Explore DBSnapper Cloud](https://app.dbsnapper.com/sign_up)** for team collaboration features

Learn more about [how DBSnapper works](how-it-works.md) or dive into the [configuration options](configuration.md) to customize the tool for your specific needs.
