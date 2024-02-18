---
title: Configuration Settings
description: DBSnapper configuration file and description of settings
---

# Configuration Settings

The config file specifies your targets along with system settings such as working directory and secret encryption key.

<!-- prettier-ignore-start -->
!!! warning "DBSnapper 2.0 Changes"

    DBSnapper 2.0 has introduced some backwards-incompatible changes to the configuration file to support new and future features. The new configuration file is a YAML file with the following structure:
<!-- prettier-ignore-end -->

<!-- prettier-ignore-start -->
!!! example "Example configuration file"

    ```yaml linenums="1"
    authtoken: 1234567890abcdef1234567890abcdef....
    working_directory: /Users/snappy/.dbsnapper
    secret_key: 1234567890abcdef1234567890abcdef
    docker:
      images:
        mysql: mysql:8-oracle
        postgres: postgres:16-alpine
    # Target configurations
    targets:
      sakila:
        name: sakila
        # Snapshot configuration
        snapshot:
          src_url: mysql://root:mysql@localhost:13306/sakila?tls=false
          dst_url: mysql://root:mysql@localhost:3306/sakila_snap?tls=false
        # Subsetting configuration
        subset:
          src_url: mysql://root:mysql@localhost:13306/sakila?tls=false
          dst_url: mysql://root:mysql@localhost:3306/sakila_subset?tls=false
          subset_tables:
            - table: sakila.film
              where: "film_id < 20"
            - table: sakila.actor
              percent: 20
          copy_tables:
            - sakila.store
          excluded_tables:
            - sakila.staff
          added_relationships:
            - fk_table: sakila.address
              fk_columns: city_id
              ref_table: sakila.city
              ref_columns: id
          excluded_relationships:
            - fk_table: sakila.store
              ref_table: sakila.staff
        # Sanitization configuration
        sanitize:
          query_file: sakila-sanitize.sql
    ```
<!-- prettier-ignore-end -->

## Supported configuration attributes

<!-- prettier-ignore-start -->
```yaml linenums="1"
authtoken: 1234567890abcdef1234567890abcdef....
working_directory: /Users/snappy/.dbsnapper
secret_key: 1234567890abcdef1234567890abcdef
```

`authtoken` (`string` optional)
:   Auth token used to authenticate with the DBSnapper Cloud. Can also be provided via the `DBSNAPPER_AUTHTOKEN` environment variable.

`working_directory` (`string` optional) 
:   Working directory used to store database snapshots. Defaults to `~/.dbsnapper`

`secret_key` (`string[32]`)
:   16 byte hexadecimal string (32 characters) that acts as your encryption key. Created on `init`. Can be provided via the `DBSNAPPER_SECRET_KEY` environment variable instead of in configuration file.

### Docker

```yaml linenums="4"
docker:
  images:
    mysql: mysql:8-oracle
    postgres: postgres:16-alpine
```

`docker`
:   Docker specific configuration setings

docker: `images`
:   Docker images to use for database engines

docker: images: `<image_name>: <value>`
:   specifies the docker image to use for a given database engine. In the example above the `mysql` and `postgres` docker images have been defined. See [Database Engines](/database-engines/introduction) for more information.

### Targets

```yaml linenums="9"
    targets:
      sakila:
        name: sakila
```
`targets`
:   One or more target definitions to be used for snapshotting, subsetting, and sanitization.

targets: `<target_name>`
:   Identifier for the target definition, must be unique.

targets: <target_name\>: `name`
:   The name of the target - can be different from the target identifier, but the target identifier is used in the agent.

### Target Snapshot

```yaml linenums="13"
        snapshot:
          src_url: mysql://root:mysql@localhost:13306/sakila?tls=false
          dst_url: mysql://root:mysql@localhost:3306/sakila_snap?tls=false
```
`snapshot`
:   Snapshot configuration for the target

`src_url`
:   Connection string of source database

`dst_url`
:   Connection string of destination database

### Target Sanitization

```yaml linenums="38"
        sanitize:
          query_file: sakila-sanitize.sql
```
`sanitize:`
:   Sanitization configuration for the target

`query_file`
:   specifies the filename of the query to be used for sanitization. This query should be located in the `working_directory` specified in the configuration.

<!-- prettier-ignore-end -->

<!-- prettier-ignore-start -->
!!! danger

    A database specified on the `dst_url` will be DROPPED and RECREATED when the `load` command is used
<!-- prettier-ignore-end -->
