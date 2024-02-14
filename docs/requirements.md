# Requirements

## Supported Platforms

[DBSnapper CLI builds](https://github.com/dbsnapper/dbsnapper/releases) are available for the following platforms:

- MacOS
- Linux

## Supported Databases

DBSnapper uses the vendor-supplied database utilities to create and restore snapshots. DBsnapper accesses these tools via the host system path or optionally by executing them from a Docker container image.

DBSnapper Database Engines are available for the following databases:

- PostgreSQL Local (postgres://)
  - Uses `pg_dump`, `pg_restore`, and `psql` found in the path of the host system.
  - Tested with [Postgres.app](https://postgresapp.com/)
- MySQL Local (mysql://)
  - Uses `mysqldump` and `mysql` found in the path of the host system.
- PostgreSQL Docker Image (pgdocker://)
  - Uses `pg_dump`, `pg_restore`, and `psql` in the docker image
- MySQL Docker image (mydocker://)
  - Uses [MySQL Shell (`mysqlsh`)](https://dev.mysql.com/doc/mysql-shell/8.0/en/) client
