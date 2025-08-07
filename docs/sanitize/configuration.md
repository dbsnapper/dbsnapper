---
title: Sanitize Configuration
description: How to configure the DBSnapper Agent to sanitize a database.
---

# Sanitize Configuration

The DBSnapper Agent is configured using a YAML file, which is created when you run `dbsnapper config init` In this file you can specify multiple target configurations, each target being a set of options for a database you want to sanitize.

Referring to our sample configuration file, the highlighted lines show the configuration options for a database sanitization:

<!-- prettier-ignore-start -->
!!! example "`~/.config/dbsnapper/dbnsapper.yml` example"

    ```yaml linenums="1" hl_lines="16-19"
    authtoken: 1234567890abcdef1234567890abcdef....
    working_directory: /Users/snappy/.dbsnapper
    docker:
      images:
        mysql: mysql:8-oracle
        postgres: postgres:16-alpine
    secret_key: 1234567890abcdef1234567890abcdef
    # Target configurations
    targets:
      sakila:
        name: sakila
        # Snapshot configuration
        snapshot:
          src_url: mysql://root:mysql@localhost:13306/sakila?tls=false
          dst_url: mysql://root:mysql@localhost:3306/sakila_snap?tls=false
        # Sanitization configuration
        sanitize:
          dst_url: mysql://root:mysql@localhost:3306/sakila_sanitized?tls=false
          query_file: sakila-sanitize.sql
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
    ```
<!-- prettier-ignore-end -->

## Configuration options

Sanitization configuration options consist of a destination url `dst_url` that will be used to load a snapshot for sanitization and a query file `query_file` that contains the sanitization queries. When the sanitization is complete, a snapshot of the sanitized database will be created and stored in the working directory.

<!-- prettier-ignore-start -->
!!! info "Subset configuration options"

    `dst_url`
    
    :   Connection string for the database where you will sanitize the snapshot (will be overwritten)

    `query_file`

    :   Name of the file containing the sanitization queries, located in your working directory (default: `~/.dbsnapper`)

<!-- prettier-ignore-end -->

<!-- prettier-ignore-start -->

!!! danger "Danger: Destination database `dst_url` will be DROPPED and RECREATED"

    Any connection string provided in the `dst_url` attribute will be overwritten when certain commands are used such as [`load`](../commands/load.md) and `sanitize` 

<!-- prettier-ignore-end -->
