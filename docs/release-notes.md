---
title: Release Notes
description: DBSnapper release notes - notable changes and updates
---

<!-- prettier-ignore-start -->
!!! note "Full release notes"

    For a complete list of changes, see the [DBSnapper Releases](https://github.com/dbsnapper/dbsnapper/releases) Page which will include all changes, bug fixes, and enhancements.

<!-- prettier-ignore-start -->

## v2.1.0 - Connection String URL Templates

All connection string URLs now support templating. This allows you to access environment variables in the connection string URLs. For example, you can now use the following connection string URL for a Postgres database:

```yaml
snapshot:
  src_url: postgres://{{`DB_USER` | env}}:{{`DB_PASSWORD` | env}}@localhost:5432/{{`DB_NAME` | env}}
```

In this example we are indicating we want the username, password, and database name to be read from the `DB_USER`, `DB_PASSWORD`, and `DB_NAME` environment variables, respectively.

Templates conform to Go Templates syntax. Specify the `env` function to read the value from the environment.

```yaml
{{`ENV_VAR` | env}} # substitute the value of the ENV_VAR environment variable
{{`CONSTANT`}} # substitute the supplied `CONSTANT` value
```

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
