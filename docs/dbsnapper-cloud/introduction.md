---
title: DBSnapper Cloud Platform - Introduction
description: The DBSnapper Cloud Platform provides additional capabilities for building, managing, and sharing sanitized database snapshots.
---

# Introduction to DBSnapper Cloud

**DBSnapper Cloud** extends your database management capabilities beyond the local CLI, providing a secure, collaborative platform for managing and sharing database snapshots across your organization.

## Why DBSnapper Cloud?

### Key Benefits

- **Centralized Management**: Streamline database snapshot management across teams and environments
- **Enhanced Collaboration**: Share sanitized database snapshots securely with team members
- **Automated Workflows**: Seamlessly integrate with CI/CD pipelines and development processes
- **Enterprise-Grade Security**: Built-in SSO integration and granular access controls

### Feature Comparison

| Feature                   | CLI Only | DBSnapper Cloud |
| ------------------------- | :------: | :-------------: |
| Local Snapshots           |    ✅    |       ✅        |
| Database Subsetting       |    ✅    |       ✅        |
| Data Sanitization         |    ✅    |       ✅        |
| Team Sharing              |    ❌    |       ✅        |
| SSO Integration           |    ❌    |       ✅        |
| Cloud Storage Integration | Limited  |      Full       |
| Access Control            |  Basic   |    Advanced     |
| Audit Logging             |    ❌    |       ✅        |

## Common Use Cases

1.  **Development Environments**

    - Quickly provision development databases with sanitized production data
    - Maintain consistency across team environments

2.  **Testing and QA**

    - Create reproducible test environments
    - Share consistent datasets across QA teams

3.  **CI/CD Integration**

    - Automate database provisioning in automated CI/CD pipelines
    - Ensure consistent and up-to-date test data across builds

4.  **Compliance and Security**

    - Maintain GDPR and CCPA compliance with sanitized data
    - Control access through role-based SSO permissions

## Getting Started

1.  [Sign up for DBSnapper Cloud](https://app.dbsnapper.com/sign_up) for DBSnapper Cloud
2.  Install the [DBSnapper Agent](../installation.md) on your local system
3.  Configure Storage Profiles and Targets
4.  Set up your SSO provider for team access (optional)
5.  Create your first cloud-managed snapshot

## DBSnapper Cloud Processing Pipeline

<p align="center">
  <img alt="DBSnapper Cloud Processing Pipeline" src="/static/flow.png" width="90%" />
</p>

The DBSnapper Cloud brings everything together and provides a processing pipeline to create, store, and share sanitized database snapshots.
