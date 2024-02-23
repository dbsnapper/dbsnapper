---
title: Subset Configuration
description: How to configure the DBSnapper Agent to subset a database.
---

# Subset Configuration

The DBSnapper Agent is configured using a YAML file, which is created when you run `dbsnapper config init` In this file you can specify multiple target configurations, each target being a set of options for a database you want to subset.

Referring to our sample configuration file, the highlighted lines show the configuration options for a database subsetting:

<!-- prettier-ignore-start -->
!!! example "`~/.config/dbsnapper/dbnsapper.yml` example"

    ```yaml linenums="1" hl_lines="20-40"
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

The subset configuration for a target is rather involved and provides a number of options to control the subsetting process. The following is a list of the subset configuration options:

#### Subset Connection Strings

```yaml linenums="21"
subset:
  src_url: mysql://root:mysql@localhost:13306/sakila?tls=false
  dst_url: mysql://root:mysql@localhost:3306/sakila_subset?tls=false
```

`subset`
: Subset configuration for the target

`src_url`
: Connection string of source database

`dst_url`
: Connection string for the database where the subset will be created (will be overwritten)

#### Subset Tables

```yaml linenums="24"
subset_tables:
  - table: sakila.film
    where: "film_id < 20"
  - table: sakila.actor
    percent: 20
```

`subset_tables`

: List of tables to be subsetted. The subset tables are the initial tables of interest, and the `where` and `percent` clauses can be used to control the portion of each table to be included in the subset. Rows from other tables are included as needed to maintain referential integrity with the subset tables.

`table`
: The name of the table to be subsetted. All tables are specified in the format `schema.table`.

`where`
: Providing a `where` clause will subset the table based on the condition specified.

`percent`
: Specifying the `percent` clause will take a random sample of the table based on the percentage specified.

<!-- prettier-ignore-start -->
!!! note "`subset_tables` clauses"

    One (and only one) of the `where` or `percent` clauses must be provided for each table in the `subset_tables` list.
<!-- prettier-ignore-end -->

#### Tables to Copy or Exclude

```yaml linenums="29"
copy_tables:
  - sakila.store
excluded_tables:
  - sakila.staff
```

`copy_tables`
: List of tables to be copied in whole to the subset database (`dst_url`). These tables are copied as-is from the source database to the subset database.

`excluded_tables`
: List of tables to be excluded from the subset database (`dst_url`). These tables are not copied to the subset database.

#### Defining Relationships

```yaml linenums="33"
added_relationships:
  - fk_table: sakila.address
    fk_columns: city_id
    ref_table: sakila.city
    ref_columns: id
excluded_relationships:
  - fk_table: sakila.store
    ref_table: sakila.staff
```

<!-- prettier-ignore-start -->

`added_relationships`
:   List of table relationships to be considered. These relationships are added to the configuration when a set of tables have a foreign key relationship that is not defined in the database schema via foreign key constraints.

    Each `added_relationships` entry is defined by the `fk_table`, `fk_columns`, `ref_table`, and `ref_columns` attributes.

`excluded_relationships` 
:   List of table relationships to be excluded. A relationship should be excluded when a **circular dependency** is detected in the database schema. This can occur when a table has a foreign key relationship to another table that also has a foreign key relationship back to the original table.

    Each `excluded_relationships` entry is defined by the `fk_table` and `ref_table` attributes.
<!-- prettier-ignore-end -->
