---
title: Sanitize a Database
description: Detailed steps to sanitize a database using DBSnapper.
---

# Sanitize a Database

Given the example configuration for the `sakila` target, we can sanitize the datbase with the snapshot we created in the [Snapshot Build and Load](../snapshot/build_and_load.md) example.

<!-- prettier-ignore-start -->
!!! example "Target: sakila - sanitize configuration"

    ```yaml linenums="1"
        sanitize:
          dst_url: mysql://root:mysql@localhost:3306/sakila_sanitized?tls=false
          query_file: "sakila.san.sql"
    ```
<!-- prettier-ignore-end -->

Here is our snapshot from the prior example:

```sh linenums="1"
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

Now we're ready to sanitize this snapshot of the `sakila` target which can be done with the [`sanitize` command](../commands/sanitize.md):

```sh
dbsnapper sanitize sakila
```

**_Output:_**

```sh
DBSnapper Agent - Version: 2.0.0-alpha-dev Build Date: 2024-02-20T07:46:10-07:00
DBSnapper Cloud: Enabled

START: Sanitize Snapshot #0, Name: 1708236890_sakila
-> LOADING original Snapshot #0: Name: 1708236890_sakila, Snapshot File: 1708236890_sakila.zip, Dest DB URL: mysql://root:mysql@localhost:3306/sakila_sanitized?tls=false
--> Using engine: mysql-local
--> Using Target: sakila
--> Pulling to local file: /Users/snappy/.dbsnapper/1708236890_sakila.zip
--> Local snapshot already exists at /Users/snappy/.dbsnapper/1708236890_sakila.zip
--> Pulled snapshot 1708236890_sakila to /Users/snappy/.dbsnapper/1708236890_sakila.zip
--> Unzipping snapshot /Users/snappy/.dbsnapper/1708236890_sakila.zip to /var/folders/z5/n821ctqx34nb__xp15r69p9h0000gp/T/dbsnapper-1815162460
--> Dropping and recreating database myl://localhost:3306/sakila_sanitized
-> LOADING Snapshot Completed for Target: sakila
--> Executing sanitization query
--> Building sanitized snapshot
--> Zipping snapshot 1708236890_sakila to /Users/snappy/.dbsnapper/1708236890_sakila.san.zip
--> Sanitized snapshot '1708236890_sakila' created at /Users/snappy/.dbsnapper/1708236890_sakila.san.zip
FINISHED Sanitizing snapshot
```

When this is finished you can list the `sakila` target snapshot:

```sh
dbsnapper target sakila
```

**_Output:_**

```sh linenums="1"
Listing ALL snapshots for target: sakila
+-------+-------------------------+-------------------+-----------------------+--------+------------+---------------------------+
| INDEX |         CREATED         |       NAME        |       FILENAME        |  SIZE  | SANITIZED? |           SANFN           |
+-------+-------------------------+-------------------+-----------------------+--------+------------+---------------------------+
|     0 | 2024-Feb-17 @ 23:14:50Z | 1708236890_sakila | 1708236890_sakila.zip | 981 kB | true       | 1708236890_sakila.san.zip |
+-------+-------------------------+-------------------+-----------------------+--------+------------+---------------------------+
```

In this output, we now see that `SANITIZED?` column is `true` and the `SANFN` column shows the name of the sanitized snapshot file that is located in the working directory.
