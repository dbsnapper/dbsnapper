---
title: Snapshot Build and Load
description: Creating and restoring a database snapshot with the DBSnapper build and load commands.
---

# Build and Load

Given the example configuration for the `sakila` target, we can build a snapshot and load it to the destination database.

<!-- prettier-ignore-start -->
!!! example "Target: sakila - snapshot configuration"

    ```yaml linenums="1"
        snapshot:
          src_url: mysql://root:mysql@localhost:13306/sakila?tls=false
          dst_url: mysql://root:mysql@localhost:3306/sakila_snap?tls=false
    ```
<!-- prettier-ignore-end -->

## Build a snapshot

Now we're ready to create our first snapshot of the `sakila` target which can be done with the [`build` command](/cmd/dbsnapper_build):

```sh
dbsnapper build sakila
```

**_Output:_**

```sh
DBSnapper CLI v2.0.0+35dda9f9.2024-02-16T06:24:43Z
DBSnapper Cloud: Enabled

START: Build Snapshot for target: sakila with engine: mysql-local
--> Local target, Local storage, non-localhost DB.
--> Zipping snapshot 1708236890_sakila to /Users/snappy/.dbsnapper/1708236890_sakila.zip
FINISH: Building DB Snapshot for target: sakila
```

When this is finished you can list all snapshots for the `sakila` target with:

```sh
dbsnapper target sakila
```

**_Output:_**

```sh linenums="1" hl_lines="10"
DBSnapper CLI v2.0.0+35dda9f9.2024-02-16T06:24:43Z
DBSnapper Cloud: Enabled

Tables in target:  sakila
...
Listing ALL snapshots for target: sakila
+-------+-------------------------+-------------------+-----------------------+--------+------------+-------+
| INDEX |         CREATED         |       NAME        |       FILENAME        |  SIZE  | SANITIZED? | SANFN |
+-------+-------------------------+-------------------+-----------------------+--------+------------+-------+
|     0 | 2024-Feb-17 @ 23:14:50Z | 1708236890_sakila | 1708236890_sakila.zip | 981 kB | false      |       |
+-------+-------------------------+-------------------+-----------------------+--------+------------+-------+
```

And we see our new snapshot `1708236890_sakila` listed on line 10.

## Load a snapshot

If a `dst_url` is defined in the target snapshot definition, you can load a snapshot to the destination using the index on the snapshot list. The [`load` command](/cmd/dbsnapper_load) will drop and recreate the destination database and restore the snapshot to the destination.

Use the index of the snapshot to specify which snapshot to load. The default is snapshot index 0.

```sh
dbsnapper load sakila 0
```

**_Output:_**

```sh linenums="1"
DBSnapper CLI v2.0.0+35dda9f9.2024-02-16T06:24:43Z
DBSnapper Cloud: Enabled

START: Loading original Snapshot #0: Name: 1708236890_sakila, Snapshot File: 1708236890_sakila.zip, Dest DB URL: mysql://root:mysql@localhost:3306/sakila_snap?tls=false
--> Using engine: mysql-local
--> Using Target: sakila
--> Pulling to local file: /Users/snappy/.dbsnapper/1708236890_sakila.zip
--> Local snapshot already exists at /Users/snappy/.dbsnapper/1708236890_sakila.zip
--> Pulled snapshot 1708236890_sakila to /Users/snappy/.dbsnapper/1708236890_sakila.zip
--> Unzipping snapshot /Users/snappy/.dbsnapper/1708236890_sakila.zip to /var/folders/z5/n821ctqx34nb__xp15r69p9h0000gp/T/dbsnapper-2125850599
--> Dropping and recreating database myl://localhost:3306/sakila_snap
FINISH: Loading Snapshot for Target: sakila
```

Here we see:

- In lines 4-6, we identify which snapshot we're using
- In lines 7-10, the snapshot is retrieved and unzipped to a temporary directory. Since it is already in the working directory, we don't have to `pull` it from the cloud
- In line 11, the destination database is dropped, recreated, and loaded with the snapshot
