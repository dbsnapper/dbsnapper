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
!!! example "`~/.config/dbsnapper/dbnsapper.yml` example"

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
        # Sanitization configuration
        sanitize:
          dst_url: mysql://root:mysql@localhost:3306/sakila_sanitized?tls=false
          query_file: "sakila.san.sql"
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

## Supported configuration attributes

```yaml linenums="1"
authtoken: 1234567890abcdef1234567890abcdef....
working_directory: /Users/snappy/.dbsnapper
secret_key: 1234567890abcdef1234567890abcdef
```

`authtoken` (`string` optional)
: Auth token used to authenticate with the DBSnapper Cloud. Can also be provided via the `DBSNAPPER_AUTHTOKEN` environment variable.

`working_directory` (`string` optional)
: Working directory used to store database snapshots. Defaults to `~/.dbsnapper`

`secret_key` (`string[32]`)
: 16 byte hexadecimal string (32 characters) that acts as your encryption key. Created on `init`. Can be provided via the `DBSNAPPER_SECRET_KEY` environment variable instead of in configuration file.

### Docker

```yaml linenums="4"
docker:
  images:
    mysql: mysql:8-oracle
    postgres: postgres:16-alpine
```

`docker`
: Docker specific configuration setings

docker: `images`
: Docker images to use for database engines

docker: images: `<image_name>: <value>`
: specifies the docker image to use for a given database engine. In the example above the `mysql` and `postgres` docker images have been defined. See [Database Engines](database-engines/introduction.md) for more information.

### Targets

```yaml linenums="9"
targets:
  sakila:
    name: sakila
```

`targets`
: One or more target definitions to be used for snapshotting, subsetting, and sanitization.

targets: `<target_name>`
: Identifier for the target definition, must be unique.

targets: <target_name\>: `name`
: The name of the target - can be different from the target identifier, but the target identifier is used in the agent.

### Target Snapshot

```yaml linenums="13"
snapshot:
  src_url: mysql://root:mysql@localhost:13306/sakila?tls=false
  dst_url: mysql://root:mysql@localhost:3306/sakila_snap?tls=false
```

`snapshot`
: Snapshot configuration for the target

`src_url`
: Connection string of source database

`dst_url`
: Connection string of destination database

<!-- prettier-ignore-start -->
!!! danger

    A database specified on the `dst_url` will be DROPPED and RECREATED when the `load`, `sanitize`, and `subset` commands are used
<!-- prettier-ignore-end -->

### Target Sanitization

```yaml linenums="17"
sanitize:
  dst_url: mysql://root:mysql@localhost:3306/sakila_sanitized?tls=false
  query_file: "sakila.san.sql"
```

<!-- prettier-ignore-start -->
`sanitize`
:   Sanitization configuration for the target

`dst_url`
:   Connection string for the database where you will sanitize the snapshot (will be overwritten)

`query_file`
:   Specifies the filename of the query to be used for sanitization. This query should be located in the `working_directory` specified in the configuration.

<!-- prettier-ignore-end -->

### Target Subsetting

```yaml linenums="21"
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

Let's look at each section of this configuration in detail.

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

#### Initial Tables to Subset

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

````yaml linenums="27"

```yaml linenums="29"
copy_tables:
  - sakila.store
excluded_tables:
  - sakila.staff
````

`copy_tables`
: List of tables to be copied in whole to the subset database (`dst_url`). These tables are copied as-is from the source database to the subset database.

`excluded_tables`
: List of tables to be excluded from the subset database (`dst_url`). These tables are not copied to the subset database.

#### Relationships

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
