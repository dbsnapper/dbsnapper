---
title: Database Subsetting with DBSnapper
description: Introduction to database subsetting with DBSnapper
---

# Introduction

**Database Subsetting** is a new feature released in v2.0 of the DBSnapper Agent. Database subsetting is the process of creating a smaller, yet fully functional version of a large database. This smaller subset retains all the critical relationships and structures of the original database but is more manageable in size.

While this would seem to be a simple task involving a simple SQL query to select the rows of interest, the main challenge is to ensure that the subset of data is relationally consistent such that all the relationships between the database tables are maintained. For example, if you have a table of `racers` and a table of `race_results`, you need to ensure that the relevant `race_results` are included for each `racer` record in the subset.

## Overview

DBSnapper provides the `subset` command to create a subset of a database. The subset command relies on the subsetting configuration specified for the target database and will be dicussed in a following section.

<!-- prettier-ignore-start -->
!!! tip "Database Subsetting Terminology"
    
    **Database Subset**
    :   A smaller, yet fully functional version of a large database.

    **Subset Table** 
    :   The primary tables of interest in the subset. These are the tables that are the main focus of the subset and from which all other tables are derived.

    **Upstream Table**
    :  Also known as a `parent` table, the `upstream` table has a foreign_key column that references a primary key in a `downstream` or `child` table. Upstream tables, therefore have a dependency on the downstream table that they reference.

    **Downstream Table**
    :  Also known as a `child` table, the `downstream` table has a primary key column that is referenced by a foreign_key column in an `upstream` table.

    **Copy Table**
    :  A table that is copied in its entirety to the subset. This is useful for tables that are not directly related to the subset tables but are required for the subset to function correctly.

<!-- prettier-ignore-end -->

### Steps to Create a Database Subset:

The high-level steps to create a datbase subset are as follows:

0. **Analyze the Database** - DBSnapper will first analyze the database and determine the set of tables that should be included in the subset based on the selected Subset Tables and the relationships between the tables in the database schema as determined by the foreign key constraints defined in the database schema.

1. **Process the Subset Tables** - DBSnapper will first create copies of the subset tables based on the criteria specified in the `percent` or `where` attributes of the subsetting configuration. In the case of `percent` we take a random sample of the specified percentage of the table. In the case of `where` we take a subset of the table based on the specified `where` clause.

2. **Process the Upstream Tables** - For each upstream table that has a foreign key reference to a subset table, DBSnapper will copy the records from the upstream table that are related to the records copied in the "Process the Subset Tables" step. This process is then repeated for any new upstream tables that are created as a result of this step.

3. **Transfer the Copy Tables** DBSnapper provides a configuration option to specify tables that you would like to have copied in their entirety to the subset. This is useful for tables that are not directly related to the subset tables but are required for the subset to function correctly. These tables are copied to the destination subset at this time.

4. **Process the Downstream Tables** - For each `downstream` table that is refenced by a table in the subset, copy the records from the `downstream` table that are referenced by the upstream table into the destination subset.

## Table Constraints

Identifying the relationships between the tables in the database is a critical part of the subsetting process. DBSnapper can automatically determine relationships if they are defined in the table (relation) schema. Depending on the database, These relationships are not always defined, so it is also possible to define the relationships manually in the DBSnapper configuration.

The table relationship information is used to determine the order in which the tables are processed and to ensure that the subset is relationally consistent.

### Automatic Relationships

DBSnapper will automatically determine relationships between tables based on foreign key constraints defined in the table (relation) schema if they are defined.

### Specifying Relationships Explicitly

If the relationships are not defined in the table schema, you can define the relationships manually in the DBSnapper configuration in the `added_relationships` section.

```yaml linenums="33"
added_relationships:
  - fk_table: sakila.address
    fk_columns: city_id
    ref_table: sakila.city
    ref_columns: id
```

In this example we are definiing a relationship between the `sakila.address` and `sakila.city` tables.

The `sakila.address` table is the _upstream_ table with the foreign key reference to the `sakila.city` table. The `sakila.city` table is the _downstream_ table with the primary key.

### Circular References

Circular references are situations where a table has a foreign key relationship to another table that also has a foreign key relationship back to the original table, creating a cycle in the relationship graph and an infinite loop in the subsetting process.

<!-- prettier-ignore-start -->
!!! note "Self-Referencing Tables"

    Circular references are also present when a table has a reference to itself. 
<!-- prettier-ignore-end -->

If a circular reference is detected, the subsetting process will stop and an error will be generated. You must then identify where the circular reference should be broken and exclude the relationship from the subsetting process. You can specify the relationships to be excluded in the `excluded_relationships` section of the DBSnapper configuration:

```yaml linenums="38"
excluded_relationships:
  - fk_table: sakila.store
    ref_table: sakila.staff
```

In this example we are excluding the relationship between the `sakila.store` and `sakila.staff` tables. Any foreign_key values in the `sakila.store` table that reference the `sakila.staff` table will be set to `NULL` to break the circular reference.
