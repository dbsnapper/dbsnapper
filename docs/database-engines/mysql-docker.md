# MySQL Docker (mydocker://)

<p align="center">
  <img alt="MySQL Logo" src="/static/mysql-icon.png"  />
</p>

## Overview

The MySQL Docker engine allows you to create snapshots of your MySQL databases running in Docker containers.

## Requirements

The MySQL Docker engine has been tested with `mysql:8-oracle` Docker image. But should be compatible with any recent version of the MySQL Docker image that includes the `mysqlsh` MySQL Shell client.

## Usage Notes

### Specifying the MySQL Docker Engine

To use the MySQL Docker engine to create or load a database snapshot, use the `mydocker://` scheme for the source or destination target url as needed. For example:

`mydocker://username:password@host/database`

The `mydocker` scheme is abbreviated as `myd` in the CLI and DBSnapper cloud
