---
title: Release Notes
description: DBSnapper release notes - notable changes and updates
---

<!-- prettier-ignore-start -->
!!! note "Full release notes"

    For a complete list of changes, see the [DBSnapper Releases](https://github.com/dbsnapper/dbsnapper/releases) Page which will include all changes, bug fixes, and enhancements.

<!-- prettier-ignore-start -->

## v2.4.0 - Ephemeral Sanitization Support

This release is bringing back the ability to use ephemeral containers for sanitization. This streamlines the sanitization process, leveraging containers to spin up a temporary database that can be used to sanitize the unsanitized snapshot data.

The `sanitize` command now behaves as follows:
1. It will create a new unsanitized and sanitized snapshot set if no snapshots exist for a target or the `-n` flag is set.
2. Will use an ephemeral container to sanitize the data if the `sanitize: dst_url` is not specified in the configuration file, or the `-e` flag is set. 

You can combine both the `-n` and `-e` flags to create a new snapshot set and use an ephemeral container for sanitization.

## v2.3.0 - New User Interface, Share Targets, Storage Engines Improvements, and More

A Terminal User Interface (TUI) has been added to the DBSnapper Agent, making it even easier to use. See all your targets, drill down into their snapshots, and load them all from the new UI.

<p align="center">
  <img src="/static/tui/dbs-ui-all-targets.png" alt="DBSnapper Agent User Interface" width="90%">
  <br/>
  <small>DBSnapper Agent User Interface - All Targets</small>
</p>

[Sharing Targets](configuration.md/#sharing-target) have been added to DBSnapper. Leveraging the ability to specify different storage profiles for original and sanitized snapshots, you can now create a _share target_ in your configuration file, that will allow you to list and load sanitized snapshots from a shared storage location. This is useful for sharing sanitized snapshots with developers, testers, and other stakeholders.

[New Storage Engines](cloud-storage-engines/introduction.md) have been added. In addition to our support for AWS S3 and CloudFlare R2, we have added support for <a href="https://min.io/" target="_blank">Minio</a> and <a href="https://www.digitalocean.com/products/spaces" target="_blank">Digital Ocean Spaces</a>

Storage Engines now support retrieving credentials from <a href="https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html" target="_blank"> the AWS CLI shared configuration</a>. It is now possible to retrieve S3 compatible storage engine credentials from environment variables, or you can specify an `awscli_profile` in your storage profile configuration to use the credentials from the specified AWS CLI profile. More information on this can be found in the [Storage Engine Configuration](cloud-storage-engines/configuration.md) documentation.

## v2.2.0 - Separate Storage Profiles for Unsanitized and Sanitized Snapshots

You can now specify different storage profiles for unsanitized (original) and sanitized snapshots, allowing you to store them in different buckets or cloud providers if desired. 
This will allow sharing only the sanitized snapshot cloud storage buckets with developers, while keeping the unsanitized snapshots private.

Up next is additional sharing functionality for accessing and loading the sanitized snapshots.

[Download the v2.2.0 release](https://github.com/dbsnapper/dbsnapper/releases/tag/v2.2.0) for your platform.

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
