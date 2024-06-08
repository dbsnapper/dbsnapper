---
title: Configuration Settings
description: DBSnapper configuration file and description of settings
---

The DBSnapper configuration file specifies your local targets along with system settings such as working directory, DBSnapper Cloud authentication token, defaults, overrides, and much more. This file is located at `~/.config/dbsnapper/dbsnapper.yml` by default.

## Sample Configuration

<!-- prettier-ignore-start -->
!!! example "`~/.config/dbsnapper/dbnsapper.yml`"

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
      
      # New for 2.3.0: Sharing Targets
      dvdrental-share:
        name: dvdrental-share
        share:
          dst_url: postgres://postgres:postgres@localhost:5432/dvdrental_share
          storage_profile_name: s3-from-awscli-profile

    # New for 2.6.0: Overrides - Sanitization Query and Destination Database     
    override:
      san_query: RFJPUCBUQUJMRSBJRiBFWElTVFMgZGJzbmFwcGVyX2luZm87CkNSRUFURSBUQUJMRSBkYnNuYXBwZXJfaW5mbyAoY3JlYXRlZF9hdCB0aW1lc3RhbXAsIHRhZ3MgdGV4dCBbXSk7CklOU0VSVCBJTlRPIGRic25hcHBlcl9pbmZvIChjcmVhdGVkX2F0LCB0YWdzKQpWQUxVRVMgKE5PVygpLCAne3F1ZXJ5OnNhbl9xdWVyeV9vdmVycmlkZSwgbG9jYXRpb246Y2xvdWR9Jyk7
      dst_db_url: postgres://postgres:postgres@localhost:15432/dbsnapper_dst_db_override

    # New for 2.7.0: Default Destination Database for Team Sharing
    defaults:
      shared_target_dst_db_url: postgres://postgres:postgres@localhost:15432/dbsnapper_dst_db_default



    # Storage profiles for sharing configurations
    storage_profiles:
      # All Profiles:
      # Leave access_key and secret_key to use a profile
      # Leave the awscli_profile empty to use the default profile
      # If the endpoint is specified in the profile it will override the
      # endpoint specified in this configuration.

      # S3
      s3-from-credentials:
        provider: s3
        awscli_profile:
        access_key: <access_key>
        secret_key: <secret_key>
        region: <region>
        bucket: dbsnapper-test-s3
        prefix:

      s3-from-awscli-profile:
        provider: s3
        awscli_profile: dbsnapper_credentials
        bucket: dbsnapper-test-s3
        prefix:

      # R2
      # Uses the account ID to create the endpoint url.
      # Can be omitted if the r2 endpoint_url is specified in the awscli config.
      # Or set the following Env Variables and leave the awscli_profile empty:
      # AWS_ACCESS_KEY
      # AWS_SECRET_KEY"
      # AWS_ENDPOINT_URL"
      r2-from-credentials:
        provider: r2
        awscli_profile:
        access_key: <access_key>
        secret_key: <secret_key>
        account_id: <account_id>
        bucket: dbsnapper-test-r2
        prefix:

      r2-from-awscli-profile:
        provider: r2
        awscli_profile: r2_production
        account_id: # optional if enpoint_url set in profile
        bucket: dbsnapper-test-r2
        prefix: 

      r2-from-env-or-default-awscli_profile:
        provider: r2
        bucket: dbsnapper-test-r2
        prefix: sanitized

      # Minio
      minio-from-credentials:
        provider: minio
        awscli_profile:
        access_key: <access_key>
        secret_key: <secret_key>
        endpoint: http://localhost:9000
        bucket: dbsnapper-test-minio
        prefix:

      # Digital Ocean Spaces
      # Endpoint shold be set to the endpoint of the spaces region i.e. nyc3 below
      dospaces-from-credentials:
        provider: dospaces
        awscli_profile:
        access_key: <access_key>
        secret_key: <secret_key>
        endpoint: https://nyc3.digitaloceanspaces.com
        bucket: dbsnapper-test-do
        prefix:
    ```
<!-- prettier-ignore-end -->

## Environment Variable Support

DBSnapper can be configured exclusively through environment variables if you don't want to rely on a configuration file. All configuration options can be represented as environment variables through a specific naming convention involving prefixing the environment variable with `DBSNAPPER` and replacing periods with two underscores `__`. Some examples include:

- `docker.images.postgres` -> `DBSNAPPER_DOCKER__IMAGES__POSTGRES: postgres:latest` # Sets the docker image to use for the postgres containers
- `defaults.shared_target_dst_db_url` -> `DBSNAPPER_DEFAULTS__SHARED_TARGET_DST_DB_URL: <connstring>` # Sets the default destination database URL for shared targets
- `override.san_query` -> `DBSNAPPER_OVERRIDE__SAN_QUERY: <base-64-encoded-value>` # Sets a query to use for sanitization overriding any existing queries.

## Configuration details by section

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

## Connection String Templates (v2.1.0)

All connection string URLs now support templating. This allows you to access environment variables in the connection string URLs. For example, you can now use the following connection string URL for a Postgres database:

```yaml
snapshot:
  src_url: postgres://{{`DB_USER` | env}}:{{`DB_PASSWORD` | env}}@localhost:5432/{{`DB_NAME` | env}}
```

In this example we are indicating we want the username, password, and database name to be read from the `DB_USER`, `DB_PASSWORD`, and `DB_NAME` environment variables, respectively.

Templates conform to Go Templates syntax. Specify the `env` function to read the value from the environment.

```yaml
{{`ENV_VAR` | env}} # substitute the value of the ENV_VAR environment variable
{{`CONSTANT`}} # substitute the supplied `CONSTANT` value
```

## Sharing Target (v2.3.0)

Added in v2.3.0, a sharing target allows you to specify a target definition that can be used to access shared snapshots on a cloud storage provider.

### Sharing sanitized snapshots

In v.2.2.0, we added the ability to specify different cloud storage locations for original (unsanitized) and sanitized snapshots. This allows you to securely share sanitized snapshots with developers and other third parties while keeping the unsanitized snapshots private. You specify the different storage locations in the on the target settings page of the DBSnapper Cloud.

### Accessing the share

To access the shared snapshots, you need to create a new target definition in your configuration file. The target definition should include the `share` section with the `dst_url` and `storage_profile_name` attributes.

<!-- prettier-ignore-start -->
!!! example "Specifying a Sharing Target"

    ```yaml
    # New for 2.3.0: Sharing Targets
    dvdrental-share:
      name: dvdrental-share
      share:
        dst_url: postgres://postgres:postgres@localhost:5432/dvdrental_share
        storage_profile_name: s3-dbsnapper-sanitized
    ```
<!-- prettier-ignore-end -->

The example above shows a sharing target named `dvdrental-share`. The `dst_url` attribute specifies the connection string that will be used when loading any of the shared snapshots.

The `storage_profile_name` attribute specifies the name of a storage profile that will be used to access the shared snapshots. In the example above we are using the `s3-dbsnapper-sanitized` storage profile to access the shared snapshots. This storage profile must be defined in the `storage_profiles` section of the configuration file.

<!-- prettier-ignore-start -->
!!! example "Storage profile configuration"

    ```yaml
    storage_profiles:
      s3-dbsnapper-sanitized
        provider: s3
        awscli_profile:
        access_key: <access_key>
        secret_key: <secret_key>
        region: <region>
        bucket: dbsnapper-sanitized
        prefix:
    ```
<!-- prettier-ignore-end -->

The example above shows the configuration for the `s3-dbsnapper-sanitized` storage profile. This storage profile specifies the connection details for the cloud storage provider where the shared snapshots are stored. It also specifies the `bucket` and `prefix` where the shared snapshots are stored.

## Overrides (v2.6.0)

Added in v2.6.0, the `override` section allows you to specify a custom sanitization query and destination database for a target. This is useful when you want to override the default sanitization query and destination database for a target.

<!-- prettier-ignore-start -->
```yaml title="Override configuration settings" linenums="1"
# New for 2.6.0: Overrides - Sanitization Query and Destination Database     
override:
  san_query: RFJPUCBUQUJMRSBJRiBFWElTVFMgZGJzbmFwcGVyX2luZm87CkNSRUFURSBUQUJMRSBkYnNuYXBwZXJfaW5mbyAoY3JlYXRlZF9hdCB0aW1lc3RhbXAsIHRhZ3MgdGV4dCBbXSk7CklOU0VSVCBJTlRPIGRic25hcHBlcl9pbmZvIChjcmVhdGVkX2F0LCB0YWdzKQpWQUxVRVMgKE5PVygpLCAne3F1ZXJ5OnNhbl9xdWVyeV9vdmVycmlkZSwgbG9jYXRpb246Y2xvdWR9Jyk7
  dst_db_url: postgres://postgres:postgres@localhost:15432/dbsnapper_dst_db_override
```
<!-- prettier-ignore-end -->

### Sanitization Query Override

The `san_query` attribute allows you to specify a custom sanitization query for a target. This query will be used instead of the default sanitization query specified in the `sanitize` section of the target.

The `san_query` attribute must be base64 encoded.

### Destination Database Override

The `dst_db_url` attribute allows you to provide a destination database connection string that will be used for any operation that requires a destination database. This is useful when you want to override the destination database location configured in the target definition.

## Defaults (v2.7.0)

<!-- prettier-ignore-start -->
```yaml title="Default configuration settings" linenums="1"

    # New for 2.7.0: Default Destination Database for Team Sharing
    defaults:
      shared_target_dst_db_url: postgres://postgres:postgres@localhost:15432/dbsnapper_dst_db_default
```
<!-- prettier-ignore-end -->

Similar to the `override` section, the `defaults` section allows you to specify a default destination database for shared team snapshots. Snapshots shared with a team do not use the destination database specified in the target definition. Instead, each team member must set a default shared target destination database in their configuration file.
