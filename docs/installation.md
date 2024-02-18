# Installation

The DBSnapper Agent is available for Mac and Linux with several options to easliy install onto your system.

## MacOS Universal Binary

MacOS users can install the Universal Mac package available on the [DBSnapper Agent Releases Page](https://github.com/dbsnapper/dbsnapper/releases). This package is designed to be compatible with various MacOS versions and hardware architectures, including both Intel and Apple Silicon chips

## Homebrew tap

Get the latest release with <a href="https://brew.sh" target="_blank">Homebrew</a>

```sh
brew install dbsnapper/tap/dbsnapper
```

## Debian, RPM, and APK Packages

For Linux users, `dbsnapper` can be installed using the `.deb`, `.rpm`, or `.apk` packages, depending on your Linux distribution and architecture. You can download these packages from the <a href="https://github.com/dbsnapper/dbsnapper/releases" target="_blank">DBSnapper Agent Releases page</a>.

Here's an example of how you can install dbsnapper using a Debian package:

1. **Download the Release**: Open your terminal and use wget to download the desired release. Replace 1.2.1 with the version number you wish to install:

<!-- prettier-ignore-start -->
!!! note "Replace the following values accordingly:"

    `TAG` with the version number you wish to install,

    `ARCH` with your system architecture, and

    `PKG_MGR` with your package extension.
<!-- prettier-ignore-end -->

This following example installs the `dbsnapper` package version 1.2.1 for a Linux system with an AMD64 architecture using the Debian package manager.

```sh
TAG=1.2.1 && \
ARCH=Linux_amd64 && \
PKG_MGR=deb && \

wget https://github.com/dbsnapper/dbsnapper/releases/download/v"$TAG"/dbsnapper_"$TAG"_"$ARCH"."$PKG_MGR"
```

2. **Install with `dpkg`**: Once the download is complete, you can install the package using dpkg:

```sh
dpkg -i dbsnapper_"$TAG"_"$ARCH"."$PKG_MGR"
```

3. **Verify Installation**: After installation, you can verify that dbsnapper is installed by running the following command:

```sh
dbsnapper config check
```

Which will output some useful information about your environment and the dbsnapper installation.

```sh
root@snappy:/# dbsnapper config check
DBSnapper Agent v1.2.2+df088f77.2024-02-12T18:02:24Z
DBSnapper Cloud: Standalone Mode

Checking DBSnapper Configuration
  ❌ Error reading config file: Config File "dbsnapper" Not Found in "[/root/.config/dbsnapper /root]"
  🔵 Postgres Local Engine (pglocal)
    ✅ psql found at /usr/bin/psql
    ✅ pg_dump found at /usr/bin/pg_dump
    ✅ pg_restore found at /usr/bin/pg_restore
  🔵 MySQL Local Engine (mylocal)
    ✅ mysqldump found at /usr/bin/mysqldump
    ✅ mysql found at /usr/bin/mysql
  🔵 Postgres Docker Engine (pgdocker)
    ❌ Error connecting to docker: error during connect: Get "http://docker:2375/_ping": dial tcp: lookup docker on 127.0.0.11:53: no such host
    ❌ docker.images not set in config file
    ❌ docker.images.postgres not set in config file
  🔵 Mysql Docker Engine (mydocker)
    ❌ Error connecting to docker: error during connect: Get "http://docker:2375/_ping": dial tcp: lookup docker on 127.0.0.11:53: no such host
    ❌ docker.images not set in config file
    ❌ docker.images.mysql not set in config file
  ⚠️ One or more database engines are not configured
  ⚠️  DBSnapper Cloud not configured - get an account at https://dbsnapper.com

  ⚠️  Configuration has warnings

  root@snappy:/#
```

Continue to the [Quick Start](quick-start.md) guide to configure DBSnapper and create your first snapshot.
