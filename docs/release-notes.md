---
title: Release Notes
description: The latest updates and changes to the DBSnapper Platform.
---

## v2.0.0 - Subsetting!

We're excited to announce the release of **DBSnapper v2.0**, which introduces a major new feature: **Database Subsetting**. This feature allows you to create a relationally consitent copy of your database that contains only a subset of the data. This is useful for creating smaller, more manageable datasets for development and testing.

<!-- prettier-ignore-start -->
!!! warning "Backwards Compatibility"
    This release introduces a new configuration file format and options. If you are upgrading from a previous version, you will need to update your configuration file to the new format. See the [Configuration Settings](configuration.md) documentation for more information.
    
<!-- prettier-ignore-end -->

### Additional Improvements

- Improved support for MySQL databases.
- Support for PostgreSQL COPY protocol for fast data copy operations.
- Simplified the sanitization command, eliminating the use of ephemeral database containers,
- Released [Docker images](https://github.com/dbsnapper/dbsnapper/pkgs/container/dbsnapper) for easier installation and use.
- An extensive refactoring and testing of the codebase to improve performance, quality, and maintainability.
- Improved documentation and examples.
