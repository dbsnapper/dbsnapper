# MySQL Local (mysql://)

<p align="center">
  <img alt="MySQL Logo" src="/static/mysql-icon.png"  />
</p>

## Overview

The MySQL Local engine allows you to create snapshots of your MySQL databases running in Docker containers.

## Requirements

The MySQL Local engine has been tested with Mysql 8.3 and uses the `mysqldump` and `mysql` command line tools to create, sanitize, and load database snapshots.

## Usage Notes

### Specifying the MySQL Local Engine

To use the MySQL Local engine to create or load a database snapshot, use the default `mysql://` scheme for the source or destination target url as needed. For example:

`mysql://username:password@host/database?sslmode=[require | disable]`

The `mysql` scheme is abbreviated as `myl` in the CLI and DBSnapper cloud
