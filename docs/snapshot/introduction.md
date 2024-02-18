---
title: Snapshot Introduction
description: Introduction to the database snapshotting capability of DBSnapper.
---

# Introduction

Snapshotting your database is a fundamental feature of DBSnapper, allowing you to create a point-in-time snapshot or backup of your database. Snapshotting takes a full copy of your database, including the schema and data, and stores it in a compressed file. This snapshot can then be used to restore your database to the state it was in when the snapshot was taken.

## Overview

DBSnapper provides the [`build`](/cmd/dbsnapper_build) and [`load`](/cmd/dbsnapper_load) commands to create and restore a snapshot of your database.

### `build` command steps

1. Connect to database specified by the `src_url` in the target definition
2. Invoke the database vendor's database dump command to create a snapshot of the database
3. Compress the snapshot file and store it in the working directory

### `load` command steps

1. Find the requested snapshot file in the working directory
2. Decompress the snapshot file
3. Connect to the database specified by the `dst_url` in the target definition
4. Drop and recreate the database
5. Invoke the database vendor's database restore command to restore the snapshot to the database

## Required Dependencies

DBSnapper uses database vendor tools to perform snapshot operations.

| Database Vendor | `build` command | `load` command | queries |
| --------------- | --------------- | -------------- | ------- |
| PostgreSQL      | `pg_dump`       | `pg_restore`   | `psql`  |
| MySQL           | `mysqldump`     | `mysql`        | `mysql` |
