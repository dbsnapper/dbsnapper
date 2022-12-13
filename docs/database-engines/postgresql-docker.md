# PostgreSQL Docker (pgdocker)

<p align="center">
  <img alt="PostgreSQL Logo" src="/static/postgres-icon.png"  />
</p>

## Overview

The PostgreSQL Docker engine allows you to create snapshots of your PostgreSQL databases running in Docker containers.

## Requirements

The PostgreSQL Docker engine has been tested with `postgres:latest` Docker image. But should be compatible with any recent version of PostgreSQL Docker image ( >= PostgreSQL 9.2 ) that includes `pg_dump`, `pg_restore`, and `psql` in the path.

## Usage Notes

### Specifying the PostgreSQL Docker Engine

To use the PostgreSQL Docker engine to create or load a database snapshot, use the `pgdocker://` scheme for the source or destination target url as needed. For example:

`pgdocker://username:password@host/database?sslmode=[require | disable]`

The `pgdocker` scheme is abbreviated as `pgd` in the CLI and DBSnapper cloud

### Permissions

Permissions for various operations:

- `build`
- `sanitize`
- `load`