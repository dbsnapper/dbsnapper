---
title: Sanitized Database Snapshots with DBSnapper
description: In this tutorial, we show how to create a database snapshot and then sanitize the sensitive data with DBSnapper.
---

# Creating Sanitized Database Snapshots with DBSnapper

In this example, we're going to create a simple database with User account information. Our goal is to be able to create sanitized snapshot of this database that we can use in our development

## Create a database and add some data

Let's create a database and call it `example_app`

```shell
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

!!! note
Yes, our passwords and pins are in the clear for simplicity in this example, of course you wouldn't do this in production, would you?

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

## Create a target

Our goal is to create a snapshot of the production `example_app` database that we can copy locally to our development database. Let's do that:

```yaml linenums="1" hl_lines="7 8 9 10" title="~/.config/dbsnapper/dbsnapper.yml"
docker:
  images:
    postgres: postgres:latest
secret_key: c614a689a559d1b517c28a5e4fcdc059
working_directory: /Users/joescharf/.dbsnapper
targets:
  example_app:
    src_url: postgresql://postgres:postgres@localhost:15432/example_app?sslmode=disable
    dst_url: postgresql://postgres:postgres@localhost:15432/example_app_dev?sslmode=disable
    query_file: ""
```

As you can see in the highlighted section, we're specifying `example_app` as our source datbase and `example_app_dev` as our destination

!!! danger
We can't stress this enough - the database `example_app_dev` in the example above will be automatically DROPPED and RECREATED when the `load` command is used!

### Checking our target

Let's make sure our target is configured properly and we can access it:

```shell
$ dbsnapper targets
```

```
DBSnapper Agent v0.11.0+2b8aaba4.2022-05-03T21:58:59Z

Listing all targets
+-------------+----------+--------+----------------------------------+--------+--------------------------------------+-------+----------+
|    NAME     | LOCATION | STATUS |               SRC                |  SIZE  |                 DST                  | QUERY | MESSAGES |
+-------------+----------+--------+----------------------------------+--------+--------------------------------------+-------+----------+
| example_app | local    | OK     | pg://localhost:15432/example_app | 8.3 MB | pg://localhost:15432/example_app_dev |       |          |
+-------------+----------+--------+----------------------------------+--------+--------------------------------------+-------+----------+
```

STATUS shows OK and no errors, so we're good to go.

## Build a snapshot

The first step in our journey is creating the snapshot. This can be done easily with the `build` command:

```shell
$ dbsnapper build example_app
```

```
DBSnapper Agent v0.11.0+2b8aaba4.2022-05-03T21:58:59Z

DB Source Host:  localhost:15432
START: Build Snapshot for target: example_app
--> Local target, localhost DB.
--> Using Docker image: postgres:latest
FINISH: Building DB Snapshot for target: example_app to file:///Users/joescharf/.dbsnapper/1651695547_example_app.dir
```

### Checking our snapshot

Now that we've successfully built a snapshot of our `example_app` database, let's check the snapshots for this target:

```shell
$ dbsnapper target example_app
```

```
DBSnapper Agent v0.11.0+2b8aaba4.2022-05-03T21:58:59Z

Listing snapshots for target: example_app
+-------+-------------------------+------------------------+----------------------------+--------+-------------+
| INDEX |         CREATED         |          NAME          |          FILENAME          |  SIZE  | SANITIZEDFN |
+-------+-------------------------+------------------------+----------------------------+--------+-------------+
|     0 | 2022-May-04 @ 14:19:07Z | 1651695547_example_app | 1651695547_example_app.zip | 1.4 kB |             |
+-------+-------------------------+------------------------+----------------------------+--------+-------------+
```

Great, we have a single snapshot available. Note that the `SANITIZEDFN` cell is empty, indicating that we don't have a sanitized version of this snapshot. That's ok, we'll deal with that later.

## Load our snapshot

Now that we have our snapshot, let's load it into our development database specified on `dst_url` of our target configuration.

```shell
$ dbsnapper load example_app 0
```

```
DBSnapper Agent v0.11.0+2b8aaba4.2022-05-03T21:58:59Z

START: Loading Target: example_app, ORIGINAL snapshot index: 0, Snapshot Name: 1651695547_example_app, Snapshot File: /Users/joescharf/.dbsnapper/1651695547_example_app.zip, Dest URL: postgresql://postgres:postgres@localhost:15432/example_app_dev?sslmode=disable
  Step 1: Drop and recreate the database
  Step 2: Loading snapshot with Docker image postgres:latest, DB url: postgresql://postgres:postgres@localhost:15432/example_app_dev?sslmode=disable
FINISH: Loading Snapshot for Target: example_app, Snapshot File: /Users/joescharf/.dbsnapper/1651695547_example_app.zip
```

### Checking our new database

Another success. Let's switch to the database and take a look at the data in the `users` table:

```
example_app_dev=# \c example_app_dev
psql (13.1, server 12.6 (Debian 12.6-1.pgdg100+1))
You are now connected to database "example_app_dev" as user "postgres".
example_app_dev=# select * from users;
 id | first_name | last_name |        email         |      password      | pin
----+------------+-----------+----------------------+--------------------+------
  1 | John       | Doe       | johndoe@example.com  | secretpassword     | 2468
  2 | Jane       | Doe       | janedoe@example.com  | ubersecretpassword | 1357
  3 | Fred       | Smith     | fsmith@dbsnapper.com | opensesame         | 7890
  4 | Sam        | Jackson   | sj@example.com       | iamsam             | 1234
(4 rows)
```

Great. So far we've made an exact copy of our `example_app` source database which now resides at `example_app_dev`.

But we have a problem. Passing around a database snapshot with all this personal information (PII) and sensitive authentication data (passwords, pins) is a security issue! Eventually something will git misplaced or misused, so we'll need to deal with that.

## Sanitizing the snapshot

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

Because we can, let's create a table and entry to record the time the sanitization was performed and the name of the query file used:

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

Now we need to update our target configuration and specify the `query_file` that will be used for sanitization:

```yaml linenums="1" hl_lines="10" title="~/.config/dbsnapper/dbsnapper.yml"
docker:
  images:
    postgres: postgres:latest
secret_key: c614a689a559d1b517c28a5e4fcdc059
working_directory: /Users/joescharf/.dbsnapper
targets:
  example_app:
    src_url: postgresql://postgres:postgres@localhost:15432/example_app?sslmode=disable
    dst_url: postgresql://postgres:postgres@localhost:15432/example_app_dev?sslmode=disable
    query_file: "example_app.san.sql"
```

### Perform the sanitization

Now we're ready to run the `sanitize` command against the `example_app` target:

```shell
$ dbsnapper sanitize example_app 0
```

```
DBSnapper Agent v0.11.0+2b8aaba4.2022-05-03T21:58:59Z

START: Sanitize example_app[0], Target: example_app, SnapshotName: 1651695547_example_app, Source: /Users/joescharf/.dbsnapper/1651695547_example_app.zip, Dest: /Users/joescharf/.dbsnapper/1651695547_example_app.san.zip
Step 1: Loading database to ephemeral DB url: postgres://circumvent:erythrodextrin-udometry-movement@forninst/rathite
Step 2: Sanitizing database with query:

... Query Removed for Brevity...

Step 3: Dumping sanitized database
Step 4: Cleaning up
FINISHED Sanitizing snapshot: /Users/joescharf/.dbsnapper/1651695547_example_app.san.zip
```

This output shows the steps taken to sanitize the database which involve:

1. Loading the database snapshot to an ephemeral Docker container DB located at `postgres://circumvent:erythrodextrin-udometry-movement@forninst/rathite`
2. Running the sanitization query against the ephemeral database
3. Dumping the sanitized database to a snapshot
4. Cleaning up any temporary directories and removing the ephemeral Docker container.

### Check the snapshot

Now that we've sanitized the database, let's take another look at the snapshots available for the `example_app` target

```shell
$ dbsnapper target example_app
```

```
DBSnapper Agent v0.11.0+2b8aaba4.2022-05-03T21:58:59Z

Listing snapshots for target: example_app
+-------+-------------------------+------------------------+----------------------------+--------+--------------------------------+
| INDEX |         CREATED         |          NAME          |          FILENAME          |  SIZE  |          SANITIZEDFN           |
+-------+-------------------------+------------------------+----------------------------+--------+--------------------------------+
|     0 | 2022-May-04 @ 14:19:07Z | 1651695547_example_app | 1651695547_example_app.zip | 1.4 kB | 1651695547_example_app.san.zip |
+-------+-------------------------+------------------------+----------------------------+--------+--------------------------------+
```

The big difference here is that `1651695547_example_app.san.zip` is listed in the `SANITIZEDFN` column, which indicates that we have a sanitized database snapshot available for this snapshot index.

### Load the snapshot

Like we did above, we'll once again, load the snapshot to our `example_app_dev` development database:

```shell
$ dbsnapper load example_app 0
```

```linenums="1" hl_lines="3"
DBSnapper Agent v0.11.0+2b8aaba4.2022-05-03T21:58:59Z

START: Loading Target: example_app, SANITIZED snapshot index: 0, Snapshot Name: 1651695547_example_app.san, Snapshot File: /Users/joescharf/.dbsnapper/1651695547_example_app.san.zip, Dest URL: postgresql://postgres:postgres@localhost:15432/example_app_dev?sslmode=disable
  Step 1: Drop and recreate the database
  Step 2: Loading snapshot with Docker image postgres:latest, DB url: postgresql://postgres:postgres@localhost:15432/example_app_dev?sslmode=disable
FINISH: Loading Snapshot for Target: example_app, Snapshot File: /Users/joescharf/.dbsnapper/1651695547_example_app.san.zip`
```

In line 3 above we're loading the **SANITIZED** snapshot at index 0. The `load` command automatically loads the sanitized snapshot for an index if one exists.

!!! note
If a sanitized snapshot exists, but you'd like to load the original snapshot, you can use the `--original` flag to force this behavior:
`dbsnapper load example_app 0 --original`

### Checking the sanitized database

Our sanitized database is now loaded, let's check it to see if it worked:

```hl_lines="8"
postgres=# \c example_app_dev
psql (13.1, server 12.6 (Debian 12.6-1.pgdg100+1))
You are now connected to database "example_app_dev" as user "postgres".
example_app_dev=# \dt
             List of relations
 Schema |      Name      | Type  |  Owner
--------+----------------+-------+----------
 public | dbsnapper_info | table | postgres
 public | users          | table | postgres
(2 rows)
```

There are our tables, with the new `dbsnapper_info` table that holds the sanitization timestamp entry.

```
example_app_dev=# select * from users;
 id | first_name | last_name |        email         |    password     | pin
----+------------+-----------+----------------------+-----------------+------
  1 | User       | Id1       | User_Id1@example.com | genericpassword | 0000
  2 | User       | Id2       | User_Id2@example.com | genericpassword | 0000
  3 | User       | Id3       | User_Id3@example.com | genericpassword | 0000
  4 | User       | Id4       | User_Id4@example.com | genericpassword | 0000
(4 rows)
```

And here is our user table with the sanitized `first_name`, `last_name`, `email`, `password` and `pin` fields.

## Conclusion

In this example, we created a simple database and demonstrated how you can use DBSnapper to _copy_ it with the `build` and `load` commands, _sanitize_ it with the `sanitize` command, and then `load` the sanitized version to your development database.

We hope you find this example useful. If you have any questions about dbsnapper, you can contact us via email at [info@dbsnapper.com](mailto:info@dbsnapper.com), or follow us on [Twitter @dbsnapper](https://twitter.com/dbsnapper).
