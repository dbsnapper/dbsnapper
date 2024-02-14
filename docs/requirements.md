# Requirements

Below are the requirements for the DBSnapper CLI.

## Supported Platforms

You can find the latest builds at the [DBSnapper CLI Releases page](https://github.com/dbsnapper/dbsnapper/releases).

- **MacOS**: Latest versions of MacOS are supported.

- **Linux**: Full compatibility with various distributions of Linux, catering to a wide range of Linux-based environments.

## Supported Databases

DBSnapper leverages the native database utilities for snapshot creation and restoration. It can access these tools either directly from the host system's path or through Docker container images, offering flexibility in how you manage your database environments.

The following databases are supported at this time:

- **PostgreSQL**: DBSnapper supports both local and Docker-based PostgreSQL databases.

- **MySQL**: DBSnapper supports both local and Docker-based MySQL databases.

See the [Database Engines](database-engines/introduction.md) documentation for more information on how to configure and use these database engines.
