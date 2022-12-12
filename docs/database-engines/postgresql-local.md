# PostgreSQL Local (pglocal)

<p align="center">
  <img alt="Postgres.app Logo" src="/static/postgresapp-icon.png"  />
</p>

## Overview

The PostgreSQL Local engine allows you to create snapshots of your PostgreSQL databases running in Docker containers.

## Requirements

The PostgreSQL Local engine has been tested with [Postgres.app](https://postgresapp.com/) and  should be compatible with any recent version of PostgreSQL installed on a host machine ( >= PostgreSQL 9.2 ) that includes `pg_dump`, `pg_restore`, and `psql` in the path.

## Usage Notes

### Specifying the PostgreSQL Local Engine

To use the PostgreSQL Local engine to create or load a database snapshot, use the `pglocal://` scheme for the source or destination target url as needed. For example:

`pglocal://username:password@host/database?sslmode=[require | disable]`

The `pglocal` scheme is abbreviated as `pgl` in the CLI and DBSnapper cloud

### Permissions

Permissions for various operations:

- `build`
- `sanitize`
- `load`