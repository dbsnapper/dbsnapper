---
title: Sanitized Database Snapshots with DBSnapper
description: In this tutorial, we show how to create a database snapshot and then sanitize the sensitive data with DBSnapper.
updated: 2024-02-21
---

# Creating Sanitized Database Snapshots with DBSnapper

<!-- prettier-ignore-start -->
!!! note "Updated Feb. 21 2024 for v2.0"

    This tutorial has been updated to reflect the changes in DBSnapper v2.0.

<!-- prettier-ignore-end -->

In this example, we're going to create a simple database with User account information. Our goal is to be able to create sanitized snapshot of this database that we can use in our development

## Create a database and add some data

Let's create a database and call it `example_app`

```sh
psql -d 'postgres://postgres:postgres@localhost:15432?sslmode=disable' -c 'create database example_app;'
```

We'll create a simple users table with some basic user and authentication fields

```sql
DROP TABLE IF EXISTS users;
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    first_name text,
    last_name text,
    email character varying(110) unique not null,
    password character varying(50) not null,
    pin character varying(8)
);
```

And we'll load some sample data

```sql
INSERT INTO users (first_name, last_name, email, password, pin) VALUES
('John', 'Doe', 'johndoe@example.com', 'secretpassword', '2468'),
('Jane', 'Doe', 'janedoe@example.com', 'ubersecretpassword', '1357'),
('Fred', 'Smith', 'fsmith@dbsnapper.com', 'opensesame', '7890'),
('Sam', 'Jackson', 'sj@example.com', 'iamsam', '1234');
```

Our database so far

```
example_app=# select * from users;
 id | first_name | last_name |        email         |      password      | pin
----+------------+-----------+----------------------+--------------------+------
  1 | John       | Doe       | johndoe@example.com  | secretpassword     | 2468
  2 | Jane       | Doe       | janedoe@example.com  | ubersecretpassword | 1357
  3 | Fred       | Smith     | fsmith@dbsnapper.com | opensesame         | 7890
  4 | Sam        | Jackson   | sj@example.com       | iamsam             | 1234
```

## Create a target configuration

Our goal is to create a snapshot of the production `example_app` database that we can copy locally to our development database. Let's do that:

```yaml linenums="1" hl_lines="8-14" title="~/.config/dbsnapper/dbsnapper.yml"
docker:
  images:
    mysql: mysql:8-oracle
    postgres: postgres:latest
secret_key: c614a689a559d1b517c28a5e4fcdc059
working_directory: /Users/joescharf/.dbsnapper
targets:
  example_app:
    name: example_app
    snapshot:
      src_url: postgres://postgres:postgres@localhost:15432/example_app?sslmode=disable
      dst_url: postgres://postgres:postgres@localhost:15432/example_app_snap?sslmode=disable
    sanitize:
      dst_url:
      query_file:
```

As you can see in the highlighted section, we're specifying `example_app` as our source datbase and `example_app_snap` as our destination

<!-- prettier-ignore-start -->
!!! danger "Danger: Destination database `dst_url` will be DROPPED and RECREATED"

    Remember, the any database specified on a `dst_url` attribute will be DROPPED and a new empty database with the same name will be CREATED prior to loading the data from the snapshot!
<!-- prettier-ignore-end -->

### Checking our target

Let's make sure our target is configured properly and we can access it:

```sh
dbsnapper targets
```

**_Output:_**

````
DBSnapper Agent - Version: 2.0.0-alpha-dev Build Date: 2024-02-21T16:20:40-07:00
DBSnapper Cloud: Standalone Mode

Listing all targets
+-------------+----------+--------+-----------------------------------+--------+----------------------------------------+-----------------+-------+----------+
|    NAME     | LOCATION | STATUS |                SRC                |  SIZE  |                  DST                   | STORAGE PROFILE | QUERY | MESSAGES |
+-------------+----------+--------+-----------------------------------+--------+----------------------------------------+-----------------+-------+----------+
| example_app | local    | OK     | pgl://localhost:15432/example_app | 7.8 MB | pgl://localhost:15432/example_app_snap |                 | No    |          |
+-------------+----------+--------+-----------------------------------+--------+----------------------------------------+-----------------+-------+----------+```
````

STATUS shows OK and no errors, so we're good to go.

## Build a snapshot

The first step in our journey is creating the snapshot. This can be done easily with the `build` command:

```sh
$ dbsnapper build example_app
```

**_Output:_**

```
DBSnapper Agent - Version: 2.0.0-alpha.2 (d534e0fcfacd632e5d117bed05a4d44520b6d388) Build Date: 2024-02-21T20:55:39Z
DBSnapper Cloud: Standalone Mode

START: Build Snapshot for target: example_app with engine: postgres-local
--> Local target, Local storage, non-localhost DB.
--> Zipping snapshot 1708557745_example_app to /Users/joescharf/.dbsnapper/1708557745_example_app.zip
FINISH: Building DB Snapshot for target: example_app
```

### Checking our snapshot

Now that we've successfully built a snapshot of our `example_app` database, let's check the snapshots for this target:

```sh
$ dbsnapper target example_app
```

**_Output:_**

```
DBSnapper Agent - Version: 2.0.0-alpha.2 (d534e0fcfacd632e5d117bed05a4d44520b6d388) Build Date: 2024-02-21T20:55:39Z
DBSnapper Cloud: Standalone Mode

Listing ALL snapshots for target: example_app
+-------+-------------------------+------------------------+----------------------------+--------+------------+-------+
| INDEX |         CREATED         |          NAME          |          FILENAME          |  SIZE  | SANITIZED? | SANFN |
+-------+-------------------------+------------------------+----------------------------+--------+------------+-------+
|     0 | 2024-Feb-21 @ 16:32:11Z | 1708558331_example_app | 1708558331_example_app.zip | 1.4 kB | false      |       |
+-------+-------------------------+------------------------+----------------------------+--------+------------+-------+
```

Great, we have a single snapshot available. Note that the `SANFN` (Sanitized FileName) cell is empty, indicating that we don't have a sanitized version of this snapshot. That's ok, we'll deal with that later.

## Load our snapshot

Now that we have our snapshot, let's load it into our development database specified on `dst_url` of our target configuration.

```sh
$ dbsnapper load example_app 0
```

**_Output:_**

```
DBSnapper Agent - Version: 2.0.0-alpha.2 (d534e0fcfacd632e5d117bed05a4d44520b6d388) Build Date: 2024-02-21T20:55:39Z
DBSnapper Cloud: Standalone Mode

START: Loading Snapshot
-> LOADING original Snapshot #0: Name: 1708558331_example_app, Snapshot File: 1708558331_example_app.zip, Dest DB URL: postgres://postgres:postgres@localhost:15432/example_app_snap?sslmode=disable
--> Using engine: postgres-local
--> Using Target: example_app
--> Pulling to local file: /Users/joescharf/.dbsnapper/1708558331_example_app.zip
--> Local snapshot already exists at /Users/joescharf/.dbsnapper/1708558331_example_app.zip
--> Pulled snapshot 1708558331_example_app to /Users/joescharf/.dbsnapper/1708558331_example_app.zip
--> Unzipping snapshot /Users/joescharf/.dbsnapper/1708558331_example_app.zip to /var/folders/z5/n821ctqx34nb__xp15r69p9h0000gp/T/dbsnapper-1871306128
--> Dropping and recreating database pgl://localhost:15432/example_app_snap
-> LOADING Snapshot Completed for Target: example_app
```

### Checking our new database

Another success. Let's switch to the database and take a look at the data in the `users` table:

```sql
postgres=# \l
                                                          List of databases
             Name              |  Owner   | Encoding |  Collate   |   Ctype    | ICU Locale | Locale Provider |   Access privileges
-------------------------------+----------+----------+------------+------------+------------+-----------------+-----------------------
 example_app                   | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            |
 example_app_snap              | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            |

postgres=# \c example_app_snap
psql (15.6 (Postgres.app), server 15.2 (Debian 15.2-1.pgdg110+1))
You are now connected to database "example_app_snap" as user "postgres".
example_app_snap=# select * from users;
 id | first_name | last_name |        email         |      password      | pin
----+------------+-----------+----------------------+--------------------+------
  1 | John       | Doe       | johndoe@example.com  | secretpassword     | 2468
  2 | Jane       | Doe       | janedoe@example.com  | ubersecretpassword | 1357
  3 | Fred       | Smith     | fsmith@dbsnapper.com | opensesame         | 7890
  4 | Sam        | Jackson   | sj@example.com       | iamsam             | 1234
(4 rows)
```

Great. So far we've made an exact copy of our `example_app` source database which is now available in the `example_app_snap` database.

But we have a problem. Passing around a database snapshot with all this personal information (PII) and sensitive authentication data (passwords, pins) is a security issue! Eventually something will git misplaced or misused, so we'll need to deal with that.

## Sanitizing and de-identifying the snapshot

The sanitization process takes a query and executes it against a database snapshot. The resulting changes to the snapshot are exported and stored in a sanitized snapshot file.

### Create the query

Let's create a sanitization query that will

1. Obfuscate `first_name` and `last_name`
2. Change the `email` to match the `first_name` and `last_name`
3. Change the `password` and `pin` for all users to a common password used for development

```sql linenums="1" title="example_app.san.sql"
-- sanitize the first_name, last_name
update users u
set first_name = 'User',
last_name = 'Id' || id;

-- sanitize the email
update users u
set email = first_name || '_' || last_name || '@example.com';

-- sanitize password and pin
update users u
set password = 'genericpassword',
pin = '0000';
```

Let's also create a table that will record the time the sanitization was performed and the name of the query file used:

```sql linenums="14" title="example_app.san.sql"
-- Add a dbsnapper_info table to record timestamp of sanitizaiton

DROP TABLE IF EXISTS dbsnapper_info;

CREATE TABLE dbsnapper_info (
  created_at timestamp,
  tags text[]
);

INSERT INTO dbsnapper_info (created_at, tags)
  VALUES (NOW(), '{example_app.san.sql}');

```

And let's save this file as `example_app.san.sql` in our `working_directory` (which defaults to `~/.dbsnapper`)

### Update the target configuration

Now we need to update our target configuration and specify the `query_file` that will be used for sanitization along with the `dst_url` for the sanitized snapshot:

<!-- prettier-ignore-start -->
!!! tip "Destination URLs and DBSnapper"

    DBSnapper uses separate `dst_url` attributes for the `snapshot` `subset`, and `sanitize` operations. This allows you to specify different destination databases for the different DBSnapper operations.
<!-- prettier-ignore-end -->

```yaml linenums="1" hl_lines="12-14" title="~/.config/dbsnapper/dbsnapper.yml"
docker:
  images:
    postgres: postgres:latest
secret_key: c614a689a559d1b517c28a5e4fcdc059
working_directory: /Users/joescharf/.dbsnapper
targets:
  example_app:
    name: example_app
    snapshot:
      src_url: postgres://postgres:postgres@localhost:15432/example_app?sslmode=disable
      dst_url: postgres://postgres:postgres@localhost:15432/example_app_snap?sslmode=disable
    sanitize:
      dst_url: postgres://postgres:postgres@localhost:15432/example_app_sanitized?sslmode=disable
      query_file: "example_app.san.sql"
```

### Perform the sanitization

Now we're ready to run the `sanitize` command against the `example_app` target:

```sh
$ dbsnapper sanitize example_app 0
```

**_Output:_**

```sh
DBSnapper Agent - Version: 2.0.0-alpha.2 (d534e0fcfacd632e5d117bed05a4d44520b6d388) Build Date: 2024-02-21T20:55:39Z
DBSnapper Cloud: Standalone Mode

START: Sanitize Snapshot #0, Name: 1708558331_example_app
-> LOADING original Snapshot #0: Name: 1708558331_example_app, Snapshot File: 1708558331_example_app.zip, Dest DB URL: postgres://postgres:postgres@localhost:15432/example_app_sanitized?sslmode=disable
--> Using engine: postgres-local
--> Using Target: example_app
--> Pulling to local file: /Users/joescharf/.dbsnapper/1708558331_example_app.zip
--> Local snapshot already exists at /Users/joescharf/.dbsnapper/1708558331_example_app.zip
--> Pulled snapshot 1708558331_example_app to /Users/joescharf/.dbsnapper/1708558331_example_app.zip
--> Unzipping snapshot /Users/joescharf/.dbsnapper/1708558331_example_app.zip to /var/folders/z5/n821ctqx34nb__xp15r69p9h0000gp/T/dbsnapper-1231401554
--> Dropping and recreating database pgl://localhost:15432/example_app_sanitized
-> LOADING Snapshot Completed for Target: example_app
--> Executing sanitization query
--> Building sanitized snapshot
--> Zipping snapshot 1708558331_example_app to /Users/joescharf/.dbsnapper/1708558331_example_app.san.zip
--> Sanitized snapshot '1708558331_example_app' created at /Users/joescharf/.dbsnapper/1708558331_example_app.san.zip
```

This output shows the steps taken to sanitize the database which involve:

1. Pulling the snapshot from the cloud or finding it locally on disk.
2. Loading the snapshot into the database specified in the `dst_url` attribute.
3. Running the sanitization query specified in `query_file` against the `dst_url` database.
4. Dumping the sanitized database to a snapshot, compressing it, and uploading it to the cloud if configured.

### Check the sanitized snapshot

Now that we've sanitized the database, let's take another look at the snapshots available for the `example_app` target

```sh
$ dbsnapper target example_app
```

**_Output:_**

```sh
DBSnapper Agent - Version: 2.0.0-alpha.2 (d534e0fcfacd632e5d117bed05a4d44520b6d388) Build Date: 2024-02-21T20:55:39Z
DBSnapper Cloud: Standalone Mode

Listing ALL snapshots for target: example_app
+-------+-------------------------+------------------------+----------------------------+--------+------------+--------------------------------+
| INDEX |         CREATED         |          NAME          |          FILENAME          |  SIZE  | SANITIZED? |             SANFN              |
+-------+-------------------------+------------------------+----------------------------+--------+------------+--------------------------------+
|     0 | 2024-Feb-21 @ 16:32:11Z | 1708558331_example_app | 1708558331_example_app.zip | 1.4 kB | true       | 1708558331_example_app.san.zip |
+-------+-------------------------+------------------------+----------------------------+--------+------------+--------------------------------+
```

The big difference here is that `1708558331_example_app.san.zip` is listed in the `SANFN` column, which indicates that we have a sanitized database snapshot available for this snapshot index.

### Load the sanitized snapshot

Like we did above, we'll once again, load the snapshot to our `example_app_snap` snapshot database:

```sh
$ dbsnapper load example_app 0
```

```sh linenums="1" hl_lines="5"
DBSnapper Agent - Version: 2.0.0-alpha.2 (d534e0fcfacd632e5d117bed05a4d44520b6d388) Build Date: 2024-02-21T20:55:39Z
DBSnapper Cloud: Standalone Mode

START: Loading Snapshot
-> LOADING sanitized Snapshot #0: Name: 1708558331_example_app, Snapshot File: 1708558331_example_app.san.zip, Dest DB URL: postgres://postgres:postgres@localhost:15432/example_app_snap?sslmode=disable
--> Using engine: postgres-local
--> Using Target: example_app
--> Pulling to local file: /Users/joescharf/.dbsnapper/1708558331_example_app.san.zip
--> Local snapshot already exists at /Users/joescharf/.dbsnapper/1708558331_example_app.san.zip
--> Pulled snapshot 1708558331_example_app to /Users/joescharf/.dbsnapper/1708558331_example_app.san.zip
--> Unzipping snapshot /Users/joescharf/.dbsnapper/1708558331_example_app.san.zip to /var/folders/z5/n821ctqx34nb__xp15r69p9h0000gp/T/dbsnapper-3818520887
--> Dropping and recreating database pgl://localhost:15432/example_app_snap
-> LOADING Snapshot Completed for Target: example_app
FINISH: Loading Snapshot
```

In line 5 above we're loading the **SANITIZED** snapshot at index 0. **The `load` command automatically loads the sanitized snapshot for an index if one exists.**

<!-- prettier-ignore-start -->

!!! note

    If a sanitized snapshot exists, but you'd like to load the original snapshot, you can use the `--original` flag to force this behavior:

    `dbsnapper load example_app 0 --original`

<!-- prettier-ignore-end -->

### Checking the sanitized database

Our sanitized database is now loaded, let's check it to see if it worked:

```sql hl_lines="8"

postgres=# \l
                                                          List of databases
             Name              |  Owner   | Encoding |  Collate   |   Ctype    | ICU Locale | Locale Provider |   Access privileges
-------------------------------+----------+----------+------------+------------+------------+-----------------+-----------------------
 example_app                   | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            |
 example_app_sanitized         | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            |
 example_app_snap              | postgres | UTF8     | en_US.utf8 | en_US.utf8 |            | libc            |


postgres=# \c example_app_snap
psql (15.6 (Postgres.app), server 15.2 (Debian 15.2-1.pgdg110+1))
You are now connected to database "example_app_snap" as user "postgres".
example_app_snap=#
example_app_snap=# \dt
             List of relations
 Schema |      Name      | Type  |  Owner
--------+----------------+-------+----------
 public | dbsnapper_info | table | postgres
 public | users          | table | postgres
(2 rows)

 example_app_snap=# select * from dbsnapper_info;
         created_at         |         tags
----------------------------+-----------------------
 2024-02-21 23:47:17.454871 | {example_app.san.sql}
(2 rows)

```

There are our tables, with the new `dbsnapper_info` table that holds the sanitization timestamp entry.

And here is our user table with the sanitized `first_name`, `last_name`, `email`, `password` and `pin` fields.

```sql
example_app_snap=# select * from users;
 id | first_name | last_name |        email         |    password     | pin
----+------------+-----------+----------------------+-----------------+------
  1 | User       | Id1       | User_Id1@example.com | genericpassword | 0000
  2 | User       | Id2       | User_Id2@example.com | genericpassword | 0000
  3 | User       | Id3       | User_Id3@example.com | genericpassword | 0000
  4 | User       | Id4       | User_Id4@example.com | genericpassword | 0000
(4 rows)
```

## Conclusion

In this example, we created a simple database and demonstrated how you can use DBSnapper to _copy_ it with the `build` and `load` commands, _sanitize_ it with the `sanitize` command, and then `load` the sanitized version to your development database.

We hope you find this example useful. If you have any questions about dbsnapper, you can contact us via email at [contact@dbsnapper.com](mailto:contact@dbsnapper.com).
