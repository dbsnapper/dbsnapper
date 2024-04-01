# Installation

The DBSnapper Agent is available for Mac and Linux with several ways to quickly install the Agent on your system.

<p align="center">
  <img src="../static/tui/dbs-ui-all-targets.png" alt="DBSnapper Agent User Interface" width="90%">
  <br/>
  <small>DBSnapper Agent User Interface</small>
</p>

## Releases Page

Our releases are available on the [DBSnapper Agent Releases](https://github.com/dbsnapper/dbsnapper/releases) Page. You can download the latest release and view release notes from this page.

## Docker Images

One of the easiest ways to get started with DBSnapper is to use the Docker image. The DBSnapper Agent is available as a Docker image on the [DBSnapper Agent Packages](https://github.com/dbsnapper/dbsnapper/pkgs/container/dbsnapper) Page. You can pull the latest image using the following command:

```sh
docker pull ghcr.io/dbsnapper/dbsnapper:latest
```

And run an interactive shell with the following command:

```sh
docker run -it ghcr.io/dbsnapper/dbsnapper:latest /bin/bash
```

## MacOS Universal Binary

MacOS users can install the Universal Mac package available on the [DBSnapper Agent Releases](https://github.com/dbsnapper/dbsnapper/releases) Page. This package is designed to be compatible with various MacOS versions and hardware architectures, including both Intel and Apple Silicon chips

## Homebrew Tap

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

    `PKG_MGR_EXT` with your package extension.
<!-- prettier-ignore-end -->

This following example installs the `dbsnapper` package version 1.2.1 for a Linux system with an AMD64 architecture using the Debian package manager.

```sh
TAG=2.0.3 && \
ARCH=linux_x86_64 && \
PKG_MGR_EXT=deb && \

wget https://github.com/dbsnapper/dbsnapper/releases/download/v"$TAG"/dbsnapper_"$ARCH"."$PKG_MGR_EXT"
```

2. **Install with `dpkg`**: Once the download is complete, you can install the package using dpkg:

```sh
dpkg -i dbsnapper_"$ARCH"."$PKG_MGR"
```

3. **Verify Installation**: After installation, you can verify that dbsnapper is installed by running the following command:

```sh
dbsnapper config check
```

Which will output some useful information about your environment and the dbsnapper installation.

```sh
root@snappy:/# dbsnapper config check

DBSnapper Agent - Version: 2.0.3 (9e8b3abde4e0) Build Date: 2024-03-05T21:33:39Z
DBSnapper Cloud: Standalone Mode

Checking DBSnapper Configuration
  ✅ Config file ( /Users/joescharf/app/dbsnapper/agent/dbsnapper.yml ) found and loaded
  🔵 Postgres Local Engine (pglocal)
    ✅ psql found at /Applications/Postgres.app/Contents/Versions/latest/bin/psql
    ✅ pg_dump found at /Applications/Postgres.app/Contents/Versions/latest/bin/pg_dump
    ✅ pg_restore found at /Applications/Postgres.app/Contents/Versions/latest/bin/pg_restore
  🔵 MySQL Local Engine (mylocal)
    ✅ mysqldump found at /opt/homebrew/bin/mysqldump
    ✅ mysql found at /opt/homebrew/bin/mysql
  🔵 Postgres Docker Engine (pgdocker)
    ✅ Docker client connected
    ✅ docker.images set in config file
    ✅ docker.images.postgres set in config file
      ✅ Found Docker image: postgres:latest
  🔵 Mysql Docker Engine (mydocker)
    ✅ Docker client connected
    ✅ docker.images set in config file
    ✅ docker.images.mysql set in config file
      ✅ Found Docker image: mysql:8.0-oracle
  ✅ All supported database engines configured
  ⚠️  DBSnapper Cloud not configured - get an account at https://dbsnapper.com

  ✅ Configuration OK
```

Continue to the [Quick Start](quick-start.md) guide to configure DBSnapper and create your first snapshot.
