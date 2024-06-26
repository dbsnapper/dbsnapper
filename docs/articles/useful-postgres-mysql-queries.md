---
title: "Useful Postgres and MySQL Queries"
description: "Here are some of the queries we use in DBSnapper to get database schema and other information."
---

Here are some of the queries we use in DBSnapper to get database schema and other information.

## Connect and List Databases & Tables

<!-- prettier-ignore-start -->
!!! example "Connect and list databases and tables."
    
    === "Postgres"

        ```sh
        psql -d postgres://postgres:postgres@localhost:15432
        \l # list databases
        \c database_name # connect to database
        \d # list tables
        \d+ table_name # list columns
        ```

    === "MySQL"

        ```sh
        mysql -u root -P13306 -hlocalhost --protocol tcp -pmysql
        show databases;
        use database_name;
        show tables;
        describe table_name;
        ```

<!-- prettier-ignore-end -->

## List Tables

<!-- prettier-ignore-start -->
!!! example "Tables"
    
    === "Postgres"

        ```sql title="Query"
        SELECT concat(concat(nsp.nspname, '.'), cls.relname) as table_name
        FROM pg_class cls
          JOIN pg_namespace nsp ON nsp.oid = cls.relnamespace
        WHERE nsp.nspname NOT IN ('information_schema', 'pg_catalog')
          AND cls.relkind = 'r';
        ```

        ```sql title="Output"
              table_name
        -----------------------
        public.store
        public.payment
        public.film_category
        public.actor
        public.staff
        public.dbsnapper_info
        public.category
        public.country
        public.language
        public.customer
        public.users
        public.rental
        public.inventory
        public.film
        public.address
        public.film_actor
        public.city
        (17 rows)
        ```

    === "MySQL"

        ```sql title="Query"
        SELECT t.table_schema as schema_name,
          t.table_name as table_name,
          t.table_rows AS row_count,
          ROUND(data_length) AS data_size,
          ROUND(index_length) AS index_size,
          ROUND(data_length + index_length) AS total_size
        FROM information_schema.TABLES t
        WHERE t.table_schema = 'sakila' -- replace sakila with your database name
          AND t.table_type = 'BASE TABLE';
        ```

        ```sql title="Output"
        +-------------+-----------------+-----------+-----------+------------+------------+
        | schema_name | table_name      | row_count | data_size | index_size | total_size |
        +-------------+-----------------+-----------+-----------+------------+------------+
        | sakila      | actor           |       200 |     16384 |      16384 |      32768 |
        | sakila      | address         |       603 |     98304 |      16384 |     114688 |
        | sakila      | category        |        16 |     16384 |          0 |      16384 |
        | sakila      | city            |       600 |     49152 |      16384 |      65536 |
        | sakila      | country         |       109 |     16384 |          0 |      16384 |
        | sakila      | customer        |       599 |     81920 |      49152 |     131072 |
        | sakila      | dbs_rental_copy |     15831 |   1589248 |          0 |    1589248 |
        | sakila      | film            |      1000 |    196608 |      81920 |     278528 |
        | sakila      | film_actor      |      5462 |    196608 |      81920 |     278528 |
        | sakila      | film_category   |      1000 |     65536 |      16384 |      81920 |
        | sakila      | film_text       |      1000 |    180224 |      16384 |     196608 |
        | sakila      | inventory       |      4581 |    180224 |     196608 |     376832 |
        | sakila      | language        |         6 |     16384 |          0 |      16384 |
        | sakila      | payment         |     16086 |   1589248 |     638976 |    2228224 |
        | sakila      | rental          |     16419 |   1589248 |    1196032 |    2785280 |
        | sakila      | staff           |         2 |     65536 |      32768 |      98304 |
        | sakila      | store           |         2 |     16384 |      32768 |      49152 |
        +-------------+-----------------+-----------+-----------+------------+------------+
        17 rows in set (0.01 sec)
        ```

<!-- prettier-ignore-end -->

## List Table Columns

<!-- prettier-ignore-start -->
!!! example "Table Columns"
    
    === "Postgres"

        ```sql title="Query"
        SELECT att.attname,
          ty.typname,
          NULL as typmaxlen,
          att.attgenerated,
          att.attidentity
        FROM pg_attribute att
          JOIN pg_class cl ON cl.oid = att.attrelid
          JOIN pg_type ty ON ty.oid = att.atttypid
          JOIN pg_namespace ns ON ns.oid = cl.relnamespace
        WHERE (
            ('public' = '') IS NOT FALSE -- replace with your schema
            OR ns.nspname = 'public' -- replace with your schema
          )
          AND cl.relname = 'customer' -- replace with your table name
          AND att.attnum > 0
          AND NOT att.attisdropped
        ORDER BY att.attnum;
        ```

        ```sql title="Output"
          attname   |  typname  | typmaxlen | attgenerated | attidentity
        -------------+-----------+-----------+--------------+-------------
        customer_id | int4      |           |              |
        store_id    | int2      |           |              |
        first_name  | varchar   |           |              |
        last_name   | varchar   |           |              |
        email       | varchar   |           |              |
        address_id  | int2      |           |              |
        activebool  | bool      |           |              |
        create_date | date      |           |              |
        last_update | timestamp |           |              |
        active      | int4      |           |              |
        (10 rows)
        ```

    === "MySQL"

        ```sql title="Query"
        SELECT c.table_schema as table_schema,
          c.table_name as table_name,
          c.column_name AS attname,
          c.data_type AS typname,
          c.character_maximum_length AS typmaxlen,
          k.constraint_name AS con_type
        FROM information_schema.columns c
          LEFT JOIN information_schema.key_column_usage k ON c.table_schema = k.table_schema
          AND c.table_name = k.table_name
          AND c.column_name = k.column_name
        WHERE c.table_schema = 'sakila' -- replace with your schema
          AND c.table_name = 'customer' -- replace with your table name
        ORDER BY c.ordinal_position;
        ```

        ```sql title="Output"
        +--------------+------------+-------------+-----------+-----------+---------------------+
        | table_schema | table_name | attname     | typname   | typmaxlen | con_type            |
        +--------------+------------+-------------+-----------+-----------+---------------------+
        | sakila       | customer   | customer_id | smallint  |      NULL | PRIMARY             |
        | sakila       | customer   | store_id    | tinyint   |      NULL | fk_customer_store   |
        | sakila       | customer   | first_name  | varchar   |        45 | NULL                |
        | sakila       | customer   | last_name   | varchar   |        45 | NULL                |
        | sakila       | customer   | email       | varchar   |        50 | NULL                |
        | sakila       | customer   | address_id  | smallint  |      NULL | fk_customer_address |
        | sakila       | customer   | active      | tinyint   |      NULL | NULL                |
        | sakila       | customer   | create_date | datetime  |      NULL | NULL                |
        | sakila       | customer   | last_update | timestamp |      NULL | NULL                |
        +--------------+------------+-------------+-----------+-----------+---------------------+
        ```

<!-- prettier-ignore-end -->

## Show Primary Keys

<!-- prettier-ignore-start -->
!!! example "Primary Keys"
    
    === "Postgres"

        ```sql title="Query"
        SELECT cl.relname as table_name,
          a.attname AS column_name
        FROM pg_constraint c
          JOIN pg_namespace ns ON ns.oid = c.connamespace
          JOIN pg_class cl ON cl.oid = c.conrelid
          JOIN pg_attribute a ON a.attnum = ANY(c.conkey)
          AND a.attrelid = c.conrelid
        WHERE (
            ('public' = '') IS NOT FALSE -- replace with your schema
            OR ns.nspname = 'public' -- replace with your schema
          )
          AND cl.relname = 'address' -- replace with your table name
          AND c.contype = 'p'
        ORDER BY a.attnum;
        ```

        ```sql title="Output"
        table_name | column_name
        ------------+-------------
        address    | address_id
        (1 row)
        ```

    === "MySQL"

        ```sql title="Query"
        SELECT DISTINCT(column_name) as attname,
          table_name as table_name,
          table_schema as table_schema,
          constraint_name as constraint_name
        FROM information_schema.key_column_usage
        WHERE table_schema = 'sakila' -- replace with your schema
          AND table_name = 'customer' -- replace with your table name
          AND constraint_name = 'PRIMARY';
        ```

        ```sql title="Output"
        +-------------+------------+--------------+-----------------+
        | attname     | table_name | table_schema | constraint_name |
        +-------------+------------+--------------+-----------------+
        | customer_id | customer   | sakila       | PRIMARY         |
        +-------------+------------+--------------+-----------------+
        1 row in set (0.00 sec)
        ```
<!-- prettier-ignore-end -->

## Database Relationships

<!-- prettier-ignore-start -->
!!! example "All Database Relationships"
    
    === "Postgres"

        ```sql title="Query"
        SELECT fk_nsp.nspname || '.' || fk_table AS fk_table,
          array_agg(
            fk_att.attname
            ORDER BY fk_att.attnum
          ) AS fk_columns,
          tar_nsp.nspname || '.' || ref_table AS ref_table,
          array_agg(
            tar_att.attname
            ORDER BY fk_att.attnum
          ) AS ref_columns
        FROM (
            SELECT fk.oid AS fk_table_id,
              fk.relnamespace AS fk_schema_id,
              fk.relname AS fk_table,
              unnest(con.conkey) as fk_column_id,
              tar.oid AS target_table_id,
              tar.relnamespace AS target_schema_id,
              tar.relname AS ref_table,
              unnest(con.confkey) as target_column_id,
              con.connamespace AS constraint_nsp,
              con.conname AS constraint_name
            FROM pg_constraint con
              JOIN pg_class fk ON con.conrelid = fk.oid
              JOIN pg_class tar ON con.confrelid = tar.oid
            WHERE con.contype = 'f'
          ) sub
          JOIN pg_attribute fk_att ON fk_att.attrelid = fk_table_id
          AND fk_att.attnum = fk_column_id
          JOIN pg_attribute tar_att ON tar_att.attrelid = target_table_id
          AND tar_att.attnum = target_column_id
          JOIN pg_namespace fk_nsp ON fk_schema_id = fk_nsp.oid
          JOIN pg_namespace tar_nsp ON target_schema_id = tar_nsp.oid
        GROUP BY 1,
          3,
          sub.constraint_nsp,
          sub.constraint_name;
        ```

        ```sql title="Output"
              fk_table       |     fk_columns     |    ref_table     |  ref_columns
        ----------------------+--------------------+------------------+----------------
        public.address       | {city_id}          | public.city      | {city_id}
        public.city          | {country_id}       | public.country   | {country_id}
        public.customer      | {address_id}       | public.address   | {address_id}
        public.film          | {language_id}      | public.language  | {language_id}
        public.film_actor    | {actor_id}         | public.actor     | {actor_id}
        public.film_actor    | {film_id}          | public.film      | {film_id}
        public.film_category | {category_id}      | public.category  | {category_id}
        public.film_category | {film_id}          | public.film      | {film_id}
        public.inventory     | {film_id}          | public.film      | {film_id}
        public.payment       | {customer_id}      | public.customer  | {customer_id}
        public.payment       | {rental_id}        | public.rental    | {rental_id}
        public.payment       | {staff_id}         | public.staff     | {staff_id}
        public.rental        | {customer_id}      | public.customer  | {customer_id}
        public.rental        | {inventory_id}     | public.inventory | {inventory_id}
        public.rental        | {staff_id}         | public.staff     | {staff_id}
        public.staff         | {address_id}       | public.address   | {address_id}
        public.store         | {address_id}       | public.address   | {address_id}
        public.store         | {manager_staff_id} | public.staff     | {staff_id}
        (18 rows)
        ```

    === "MySQL"

        ```sql title="Query"
        SELECT concat(table_schema, '.', table_name) AS fk_table,
          group_concat(
            column_name
            ORDER BY ordinal_position
          ) AS fk_columns,
          concat(
            referenced_table_schema,
            '.',
            referenced_table_name
          ) AS ref_table,
          group_concat(
            referenced_column_name
            ORDER BY ordinal_position
          ) AS ref_columns
        FROM information_schema.key_column_usage
        WHERE referenced_table_schema NOT IN (
            'information_schema',
            'performance_schema',
            'sys',
            'mysql',
            'innodb',
            'tmp'
          ) AND table_schema = 'sakila' -- replace with your schema

        GROUP BY 1,
          3,
          constraint_schema,
          constraint_name;
        ```

        ```sql title="Output"
        +----------------------+----------------------+------------------+--------------+
        | fk_table             | fk_columns           | ref_table        | ref_columns  |
        +----------------------+----------------------+------------------+--------------+
        | sakila.address       | city_id              | sakila.city      | city_id      |
        | sakila.city          | country_id           | sakila.country   | country_id   |
        | sakila.customer      | address_id           | sakila.address   | address_id   |
        | sakila.customer      | store_id             | sakila.store     | store_id     |
        | sakila.film          | language_id          | sakila.language  | language_id  |
        | sakila.film          | original_language_id | sakila.language  | language_id  |
        | sakila.film_actor    | actor_id             | sakila.actor     | actor_id     |
        | sakila.film_actor    | film_id              | sakila.film      | film_id      |
        | sakila.film_category | category_id          | sakila.category  | category_id  |
        | sakila.film_category | film_id              | sakila.film      | film_id      |
        | sakila.inventory     | film_id              | sakila.film      | film_id      |
        | sakila.inventory     | store_id             | sakila.store     | store_id     |
        | sakila.payment       | customer_id          | sakila.customer  | customer_id  |
        | sakila.payment       | rental_id            | sakila.rental    | rental_id    |
        | sakila.payment       | staff_id             | sakila.staff     | staff_id     |
        | sakila.rental        | customer_id          | sakila.customer  | customer_id  |
        | sakila.rental        | inventory_id         | sakila.inventory | inventory_id |
        | sakila.rental        | staff_id             | sakila.staff     | staff_id     |
        | sakila.staff         | address_id           | sakila.address   | address_id   |
        | sakila.staff         | store_id             | sakila.store     | store_id     |
        | sakila.store         | address_id           | sakila.address   | address_id   |
        | sakila.store         | manager_staff_id     | sakila.staff     | staff_id     |
        +----------------------+----------------------+------------------+--------------+
        22 rows in set (0.02 sec)
        ```

<!-- prettier-ignore-end -->

### Forward Relationships

```mermaid
erDiagram

payment {
        numeric amount
        smallint customer_id FK
        timestamp_without_time_zone payment_date
        integer payment_id PK
        integer rental_id FK
        smallint staff_id FK
    }

customer {
        integer active
        boolean activebool
        smallint address_id FK
        date create_date
        integer customer_id PK
        character_varying email
        character_varying first_name
        character_varying last_name
        timestamp_without_time_zone last_update
        smallint store_id
    }

rental {
    smallint customer_id FK
    integer inventory_id FK
    timestamp_without_time_zone last_update
    timestamp_without_time_zone rental_date
    integer rental_id PK
    timestamp_without_time_zone return_date
    smallint staff_id FK
}

staff {
    boolean active
    smallint address_id FK
    character_varying email
    character_varying first_name
    character_varying last_name
    timestamp_without_time_zone last_update
    character_varying password
    bytea picture
    integer staff_id PK
    smallint store_id
    character_varying username
}

payment }o--|| customer : "customer_id"
rental }o--|| customer : "customer_id"
payment }o--|| rental : "rental_id"
payment }o--|| staff : "staff_id"
rental }o--|| staff : "staff_id"

```

<!-- prettier-ignore-start -->
!!! example "Forward Relationships"
    
    === "Postgres"

        ```sql title="Query"
        SELECT
          COALESCE(conname, '') AS constraint_name,
          conrelid::regclass AS table_name,
          a.attname AS column_name,
          confrelid::regclass AS referenced_table_name,
          COALESCE(af.attname, '') AS referenced_column_name
        FROM
          pg_constraint c
        JOIN
          pg_namespace n ON n.oid = c.connamespace
        JOIN
          pg_class cl ON cl.oid = c.conrelid
        JOIN
          pg_attribute a ON a.attnum = ANY(c.conkey) AND a.attrelid = c.conrelid
        JOIN
          pg_class conf ON conf.oid = c.confrelid
        JOIN
          pg_attribute af ON af.attnum = ANY(c.confkey) AND af.attrelid = c.confrelid
        WHERE
          n.nspname = 'public' -- replace with your schema
          AND cl.relname = 'payment' -- replace with your table name
          AND c.contype = 'f'
        ORDER BY
          constraint_name;
        ```

        ```sql title="Output"
            constraint_name      | table_name | column_name | referenced_table_name | referenced_column_name
        --------------------------+------------+-------------+-----------------------+------------------------
        payment_customer_id_fkey | payment    | customer_id | customer              | customer_id
        payment_rental_id_fkey   | payment    | rental_id   | rental                | rental_id
        payment_staff_id_fkey    | payment    | staff_id    | staff                 | staff_id
        (3 rows)
        ```

    === "MySQL"

        ```sql title="Query"
        SELECT coalesce(table_name, '') as table_name,
          coalesce (column_name, '') as column_name,
          coalesce(referenced_column_name, '') as referenced_column_name,
          coalesce(referenced_table_name, '') as referenced_table_name
        FROM information_schema.key_column_usage
        WHERE table_schema = 'sakila' -- replace with your schema
          AND table_name = 'payment' -- replace with your table name
          AND referenced_table_name != 'NULL';
        ```

        ```sql title="Output"
        +------------+-------------+------------------------+-----------------------+
        | table_name | column_name | referenced_column_name | referenced_table_name |
        +------------+-------------+------------------------+-----------------------+
        | payment    | customer_id | customer_id            | customer              |
        | payment    | rental_id   | rental_id              | rental                |
        | payment    | staff_id    | staff_id               | staff                 |
        +------------+-------------+------------------------+-----------------------+
        3 rows in set (0.01 sec)
        ```
<!-- prettier-ignore-end -->

### Reverse Relationships

ER Diagram

```mermaid
erDiagram

payment {
        numeric amount
        smallint customer_id FK
        timestamp_without_time_zone payment_date
        integer payment_id PK
        integer rental_id FK
        smallint staff_id FK
    }

customer {
        integer active
        boolean activebool
        smallint address_id FK
        date create_date
        integer customer_id PK
        character_varying email
        character_varying first_name
        character_varying last_name
        timestamp_without_time_zone last_update
        smallint store_id
    }

rental {
    smallint customer_id FK
    integer inventory_id FK
    timestamp_without_time_zone last_update
    timestamp_without_time_zone rental_date
    integer rental_id PK
    timestamp_without_time_zone return_date
    smallint staff_id FK
}


payment }o--|| customer : "customer_id"
rental }o--|| customer : "customer_id"
payment }o--|| rental : "rental_id"
```

<!-- prettier-ignore-start -->
!!! example "Reverse Relationships"
    
    === "Postgres"

        ```sql title="Query"
        SELECT
          COALESCE(conname, '') AS constraint_name,
          conrelid::regclass AS table_name,
          a.attname AS column_name,
          confrelid::regclass AS referenced_table_name,
          COALESCE(af.attname, '') AS referenced_column_name
        FROM
          pg_constraint c
        JOIN
          pg_namespace n ON n.oid = c.connamespace
        JOIN
          pg_class cl ON cl.oid = c.conrelid
        JOIN
          pg_attribute a ON a.attnum = ANY(c.conkey) AND a.attrelid = c.conrelid
        JOIN
          pg_class conf ON conf.oid = c.confrelid
        JOIN
          pg_attribute af ON af.attnum = ANY(c.confkey) AND af.attrelid = c.confrelid
        WHERE
          n.nspname = 'public' -- replace with your schema
          AND
          confrelid::regclass::text = 'customer' -- replace with your table name
        ORDER BY
          constraint_name;
        ```

        ```sql title="Output"
            constraint_name      | table_name | column_name | referenced_table_name | referenced_column_name
        --------------------------+------------+-------------+-----------------------+------------------------
        payment_customer_id_fkey | payment    | customer_id | customer              | customer_id
        rental_customer_id_fkey  | rental     | customer_id | customer              | customer_id
        (2 rows)
        ```
    === "MySQL"

        ```sql title="Query"
        SELECT coalesce(table_name, '') as table_name,
          coalesce (column_name, '') as column_name,
          coalesce(referenced_column_name, '') as referenced_column_name,
          coalesce(referenced_table_name, '') as referenced_table_name
        FROM information_schema.key_column_usage
        WHERE table_schema = 'sakila' -- replace with your schema
          AND referenced_table_name = 'customer'; -- replace with your table name
        ```

        ```sql title="Output"
        +------------+-------------+------------------------+-----------------------+
        | table_name | column_name | referenced_column_name | referenced_table_name |
        +------------+-------------+------------------------+-----------------------+
        | payment    | customer_id | customer_id            | customer              |
        | rental     | customer_id | customer_id            | customer              |
        +------------+-------------+------------------------+-----------------------+
        2 rows in set (0.00 sec)
        ```

<!-- prettier-ignore-end -->
