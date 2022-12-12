# Requirements


## Supported Platforms

[DBSnapper CLI builds](https://github.com/dbsnapper/dbsnapper/releases) are  available for the following platforms:

- Mac OS X
- Linux
- Windows - ⚠️ currently untested ⚠️

## Supported Databases

DBSnapper uses the vendor-supplied database utilities to create and restore snapshots. DBsnapper accesses these tools via Docker image or via a local installation on the host system. 

DBSnapper Database Engines are available for the following databases:

- PostgreSQL Docker Image
    - Uses `pg_dump`, `pg_restore`, and `psql` in the docker image
    - Tested with Docker image`postgres:latest`
- PostgreSQL Local
    - Uses `pg_dump`, `pg_restore`, and `psql` found in the path of the host system.
    - Tested with [Postgres.app](https://postgresapp.com/)
- MySQL Docker image
    - Uses [MySQL Shell (`mysqlsh`)](https://dev.mysql.com/doc/mysql-shell/8.0/en/) client
    - Tested with Docker image `mysql:8.0-oracle`
## Docker

DBSnapper uses Docker extensively for accessing datbase tools to create and restore snapshots. Additionally, DBSnapper uses Docker to sanitize data by spinning up ephemeral Docker containers.

- DBSnapper has been tested with Docker Desktop for Mac (v4.14.1)