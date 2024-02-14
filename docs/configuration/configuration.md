# Configuration Settings

The config file specifies your targets along with system settings such as working directory and secret encryption key.

<!-- prettier-ignore-start -->
!!! warn "DBSnapper 2.0 Changes"

    DBSnapper 2.0 has introduced some backwards-incompatible changes to the configuration file to support new and future features. The new configuration file is a YAML file with the following structure:
<!-- prettier-ignore-end -->

<!-- prettier-ignore-start -->
!!! example "Example configuration file"

    ```yaml
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
        query_file: ""
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

- `authtoken` (`string` optional) - Auth token used to authenticate with the DBSnapper Cloud. Can also be provided via the `DBSNAPPER_AUTHTOKEN` environment variable..
- `secret_key` (`string[32]`) - 16 byte hexadecimal string (32 characters) that acts as your encryption key. Created on `init`. Can be provided via the `DBSNAPPER_SECRET_KEY` environment variable instead of in configuration file.
- `working_directory` (`string` optional) - Working directory used to store database snapshots. Defaults to `~/.dbsnapper`

- `docker`

  - `images`
    - `<image_name>: <value>` - specifies the docker image to use for a given database engine. default is `postgres: postgres:latest` see [Database Engines](database-engines/introduction) for more information.

- `targets` - One or more entries specifying a snapshot workflow
  - `<target_name>` - specifies the name of the target definition
    - `name` - specifies the name of the target definition
    - `snapshot`
      - `src_url` - connection string of source database
      - `dst_url` - connection string of destination database
    - `sanitize:`
      - `query_file` - specifies the filename of the query to be used for sanitization. This query should be located in the `working_directory` specified in the configuration.

<!-- prettier-ignore-start -->
!!! danger

    A database specified on the `dst_url` will be DROPPED and RECREATED when the `load` command is used
<!-- prettier-ignore-end -->
