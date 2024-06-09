---
title: Quick start
description: How to quickly get started with the DBSnapper Agent and use it to build and load a database snapshot.
---

## Really Quick Start (update)

Just use the latest version of the DBSnapper container image, provide the minimum required `DBSNAPPER_SECRET_KEY` and `DBSNAPPER_AUTHTOKEN` environment variables necessary to run DBSnapper in Cloud mode, without a configuration file.

```sh
docker run -v /var/run/docker.sock:/var/run/docker.sock -e DBSNAPPER_SECRET_KEY=XXX -e DBSNAPPER_AUTHTOKEN=YYY --rm --network dbsnapper --pull always ghcr.io/dbsnapper/dbsnapper:latest dbsnapper build dvdrental-cloud
```

This command does the following:

1. It pulls the latest version of the [DBSnapper Agent container image](https://ghcr.io/dbsnapper/dbsnapper) from the GitHub Container Registry,
2. It mounts the host Docker socket. This is optional, but necessary for ephemeral sanitization operations requiring Docker-in-Docker (DinD) support.
3. Passes the `DBSNAPPER_SECRET_KEY` and `DBSNAPPER_AUTHTOKEN` environment variables to the container
4. Runs the `dbsnapper build dvdrental-cloud` command to build a snapshot of the `dvdrental-cloud` target defined in the DBSnapper Cloud.
5. If a storage profile is defined in the DBSnapper Cloud, the snapshot will be uploaded to the configured storage provider.

## Quick Start

In this example we will create and restore a database snapshot.

## Initialize the configuration file

Run the `config init` command to create an example configuration at `~/.config/dbsnapper/dbsnapper.yml`

```sh
dbsnapper config init
```

<!-- prettier-ignore-start -->
!!! example "Configuration file initialized to default values"

    ```yaml
    secret_key: d3d234bc83dd4efe7b7329855ba0acc2
    working_directory: /Users/snappy/.dbsnapper
    docker:
      images:
        postgres: postgres:latest
    ```
<!-- prettier-ignore-end -->

### Check the configuration and environment

Next, we can check our configuration and required dependencies. This runs some checks to verify the configuration file is valid and reports on the database tools found in the path as well as Docker engine and database image availability.

```sh
dbsnapper config check
```

<!-- prettier-ignore-start -->
!!! example "`dbsnapper config check` output"

    ```sh
    Checking DBSnapper Configuration
      ✅ Config file ( /Users/snappy/app/dbsnapper/cli/dbsnapper.yml ) found and loaded
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
      ✅ DBSnapper Cloud connected

      ✅ Configuration OK

    ```
<!-- prettier-ignore-end -->

### Add target definitions

Add one or more databse `targets` to configuration file. Here we define an `app` target with a `src_url` specifying the source database and a `dst_url` specifying the destination database

<!-- prettier-ignore-start -->
!!! example "Defining an app target definition"

    ```yaml
    targets:
      app:
        snapshot:
          src_url: postgresql://postgres:postgres@localhost:15432/app?sslmode=disable
          dst_url: postgresql://postgres:postgres@localhost:15432/app_snap?sslmode=disable
    ```
<!-- prettier-ignore-end -->

<!-- prettier-ignore-start -->
!!! danger "Danger: Destination database `dst_url` will be DROPPED and RECREATED"

    A database specified on the `dst_url` will be DROPPED and RECREATED when the `load` command is used
<!-- prettier-ignore-end -->

### List targets

Now that we have a target defined, we can list all targets with:

```sh
dbsnapper targets
```

This command will also check the size and connectivity status for each target defined in the configuration file.

<p class="img-box">
  <img src="/static/tui/dbs-ui-all-targets.png" alt="DBSnapper Agent UI - All Targets" width="90%">
  <br/>
  <small>DBSnapper Agent UI - All Targets</small>
</p>

### Build a snapshot

Now we're ready to create our first snapshot of the `app` target which can be done with the `build` command:

```sh
dbsnapper build app
```

This will connect to the database using the native dump utility and will create a dump of all data in the database.

### List snapshots for a target

Once you've successfully built a snapshot, you can list all the sanpshots for a target with the following command (note the singular `target` command):

```sh
dbsnapper target app
```

<p class="img-box crop">
  <img src="/static/tui/dbs-ui-target-snapshots.png" alt="DBSnapper Agent UI - Target Snapshots">
    <br/>
  <small>DBSnapper Agent UI - Target Snapshots Example</small>
</p>

### Load a snapshot

If a `dst_url` is defined in the target definition, you can load a snapshot to the destination using the index on the snapshot list.

```sh
dbsnapper load app 0
```

<!-- prettier-ignore-start -->
!!! danger "Danger: Destination database `dst_url` will be DROPPED and RECREATED"

    Remember, the database specified on `dst_url` will be DROPPED and a new empty database with the same name will be CREATED prior to loading the data from the snapshot!
<!-- prettier-ignore-end -->
