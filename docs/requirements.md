# Requirements

Below are the requirements for the DBSnapper Agent.

## Supported Platforms

You can find the latest builds at the [DBSnapper Agent Releases page](https://github.com/dbsnapper/dbsnapper/releases).

- **MacOS**: Latest versions of MacOS are supported.

- **Linux**: Full compatibility with various distributions of Linux, catering to a wide range of Linux-based environments.

## Supported Databases

DBSnapper leverages the native database utilities for snapshot creation and restoration. It can access these tools either directly from the host system's path or through Docker container images, offering flexibility in how you manage your database environments.

The following databases are supported at this time:

- **PostgreSQL**: DBSnapper supports both local and Docker-based PostgreSQL databases.

- **MySQL**: DBSnapper supports both local and Docker-based MySQL databases.

See the [Database Engines](database-engines/introduction.md) documentation for more information on how to configure and use these database engines.

## Supported Cloud Storage Providers

The [DBSnapper Cloud](dbsnapper-cloud/introduction.md) extends the capabilities of the DBSnapper Agent by providing a support for cloud storage in a "Bring Your Own Cloud Storage" arrangement. The DBSnapper Agent supports the following cloud storage providers:

- **Amazon S3**: DBSnapper supports the [Amazon Simple Storage Service (S3)](https://aws.amazon.com/s3/) cloud storage platform.

- **Cloudflare R2**: DBSnapper supports [Cloudflare R2](https://developers.cloudflare.com/r2/) cloud storage platform.
