---
title: High Performance Postgres for Rails
description: How to optimise Postgres for Rails
---

## Chapter 1 - An App to get you started

SQL is declarative, meaning the code you write tells Postgres what you want and Postgres decides how to get it. It does this by taking various factors into account to develop one or more 'query plans' for the data, then chooses among them based on statistics it collects and cost estimates of various operations. While you can't explicity select a plan, you can strongly influence which one is chosen.

### Migrations

Migrations generate **DDL** (Data Definition Language) statements like `CREATE TABLE`, `ALTER TABLE`, `DROP TABLE`, etc. You can also add **DML** (Data Manipulation Language) statements like `INSERT`, `UPDATE`, etc., but it's usually best to put these in a rake task so they don't slow down deployments.

Remember there's a table called `schema_migrations` rails uses to track which migrations you've run; `rails db:migrate` adds the versions of the migrations it runs to that table. If you're using an SQL schema running migrations cases `pg_dump` to add them to your `structure.sql` as `INSERT` statements, making them ready to run on another DB.

### Auth

Users are 'roles' with the `LOGIN` privilege in Postgres, and their access info can be stored in a `~/.pgpass` file using a semicolon as a delimiter.

You can create a role without the `LOGIN` privilege and use it as a 'group' to store privileges in by creating the role you want to add to the group `IN` the group role.

### Terminology

- bloat: allocated but unused space, or space used on old tuples
- functions: enhance your database with new functionality, can be written in SQL or a procedural language
- pages: pre-allocated space filled with `INSERT`, `UPDATE`, etc. on table rows or indexes. 8KB by default.
- planner: also known as the 'optimizer', compares query plans and chooses the best one using a cost-based heuristic
- procedures: procedures expand on functions, granting more features like transactional control
- transactions: overloaded to mean many things, but commonly refers to a group of operations performed as one unit of work. To achieve this they have an 'isolated' view of the data at a point in time.
- tuples: immutable row versions created when a row is 'updated'

### Useful Tools

[rails-erd](https://github.com/voormedia/rails-erd) looks handy for generating Entity-Relationship Diagrams automatically. Not sure if it works with anything > 6 though.

[PostgresApp](https://postgresapp.com/) is an alternative to managing postgres versions with homebrew on Mac. It gives you a UI to switch between versions and start/stop/create servers.

`tee <file>` takes the output piped to it and redirects it to both stdout and the file.

## Chapter 2 - Administration Basics

### psql

The native Postgres client, ships with it by default.

You can stop and restart it with `pg_ctl restart` once you've set the `PG_DATA` environment variable found by `SHOW data_directory;`.

#### Config

`~/.psqlrc` lets you customise `psql`, like in this [article](https://thoughtbot.com/blog/an-explained-psqlrc) from Thoughtbot.

You can find the path to your config file from `psql` with `SHOW config_file;`.

#### Meta Commands

- `\c {database}` - connect to a database
  - A database is a logical collection of tables and other objects existing within your PG server.
- `\d {table}` - shows the columns, indexes and constraints on the table
  - `\dx` lists all installed extensions
  - `\dD` lists all domains
- `\e` - launches your editor, saving and exiting runs the command you've edited
- `\l` - lists databases
- `\o {file}` - pipes the output of your queries to a file until toggled back with plain `\o`
- `\s` - command history
- `\timing` - toggles timing
- `\q` - quit
- `\x` - toggle vertical output

If you put `bind "^R" emc-inc-search-prev` in your `~/.editrc`, you can use `Ctrl+R` to search backwards in the command history. Just start typing and it'll display a match; execute with `\g`.

### Extensions

Stored in the `pg_extension` system catalog, a PG table or view tracking internal information.

Can be enabled in your config file as a comma-separated list for the `shared_preload_libraries` config option, then running a `CREATE EXTENSION` statement.

### Observability

The catalog `pg_stat_activity` shows current activity in your DB.

You can use it to find the `pid`s of running processes (among other things) and terminate them gently with `SELECT PG_CANCEL_BACKEND(pid);` or abruptly with `SELECT PG_TERMINATE_BACKEND(pid);`.

### Locking

Can be queried from the system catalog `pg_locks`.

[pglocks](https://pglocks.org/) is a handy resource for a deeper dive.

Generally uses pessimistic locking (resources locked upfront when modified), but can use optimistic.

Exclusive locks block all queries trying to access the selected resource, which can be a major problem on big tables. A deadlock is when 2 processes permanently block eachother.

Locks can be created explicitly, but are most often created implicitly be statements.

### Basic Commands

- `CREATE DATABASE {name}` - creates a new database, connect to it with `\c {name}`
- `CREATE INDEX {name} ON {table}({column})` - creates an index on a column
  - in prod you probably want the `CONCURRENTLY` keyword after `INDEX` to allow the table to continue to be queried
- `DROP INDEX {name}` - drops an index

#### Modifiers

- `CASCADE`: Applies the command to all related tables as well
- `CONCURRENTLY`: Allows carrying out an operation which would normally acquire a lock without acquiring one.
  - Takes considerably longer as it must perform two scans of the table and create a temporary invalid index.
  - If it fails, will leave the index behind. Recommend to drop and restart, but could also rebuild with `REINDEX CONCURRENTLY`.
  - Adding/dropping indexes
  - Rebuilding an index
  - Adding/detaching a partition

## Chapter 4 - Data Correctness & Consistency

At the application level they're ActiveRecord validations, but at the DB level they're objects used to constrain values on columns, tables or even multiple tables.

You can find them in `pg_constraint`.

Adding a constraint automatically adds an index to make enforcement of the constraint fast, but when adding an index to a database in use you generally want to add the index first and with `CONCURRENTLY` to minimize locking. As the index can be added `CONCURRENTLY` it does not interfere unduly with normal operation, and then speeds up adding the constraint which does.

In a Rails migration you can add the index in a migration which calls `disable_ddl_transaction!` before the `change` block and passes the `algorithm: :concurrently` option to `add_index`.

It's possible to have duplicate constraints as long as they have different names, so watch out for that.

**Possible we have some? How would we check? Hopefully comes up later**

### Fixing Constraint Violations

A simplistic approach is to get all the duplicate rows using `GROUP BY` and delete all but the one with `MIN(id)` or `MAX(id)` however sometimes there is business logic to prioritise which rows should be kept, for example preferring a verified user over an unverified one.

A more customizable approach uses _Common Table Expressions_ (CTE) and the `ROW_NUMBER()` window function with `PARTITION_BY` to order duplicate records by given columns and delete records after a given cutoff in that order. An example is provided by [this blog post](https://sqlfordevs.com/delete-duplicate-rows).

### `CHECK` Constraints

Any condition you can express using SQL which evaluates to a boolean can be a check constraint. Similar to AR validations but in the DB.

Rails support was added in AR 6.1 (so we have it) with `add_check_constraint` in migrations, and `if_exists` was added later.

```ruby
# in an AR migration
def change
    add_check_constraint :trips,
      "completed_at > created_at",
      name: "trips_completed_at_check"
end
```

```SQL
-- In SQL
ALTER TABLE trips
  ADD CONSTRAINT trips_completed_at_check
  CHECK (completed_at > created_at);
```

`INTERVAL` can be useful for time based constraints, for example to require `completed_at` be at least 30m more than `created_at` you'd change the SQL above to:

```SQL
ALTER TABLE trips
  ADD CONSTRAINT trips_completed_at_check
  CHECK (completed_at > (created_at + INTERVAL '30 minutes'));
```

It's also possible to add check constraints which only enforce their conditions for new row changes using `NOT VALID` in SQL or the `validate: false` option in AR.

This is not possible for `UNIQUE` or `NULL` constraints, but you can use temporary `CHECK` constraints to prepare the table for a future `NOT NULL` or `UNIQUE` constraint by ensuring all new records meet the constraint.

It's also possible to defer some constraints (`UNIQUE`, `PRIMARY KEY`, `REFERENCES` and `EXCLUDE`) using `DEFERRABLE`.

You might want to do this to simplify operations which would otherwise violate constraints, like swapping the order of two items (like the school ordering back at KU for the setsu calendar). In that case, if you defer constraints until the transaction finishes they're never violated.

To enable constraints only at the end of a transaction you can define them with `DEFERRABLE INITIALLY DEFERRED`.

### `EXCLUDE` Constraints

These prevent overlaps between rows within a table, like two people reserving trips on the same taxi at overlapping times.

They use 'GiST' indexes and require an _operator_ class, as well as the `btree_gist` extension (or maybe a similar one) to be enabled.

### Casing

The `citext` extension gives you a `CITEXT` column type which is queried in a case-insensitive way.

This allows, for example, an email to be entered and displayed as "BTAN@gmail.com", but still not allow a duplicate email of "btan@gmail.com" and will match searches foe the downcased version to the upcased one.

Another approach is to use generated columns (virtual columns; `t.virtual`, in ActiveRecord), which PG will keep up to date for you by applying a transformation to the column they're derived from. Generated columns can be stored or not.

### Enums

- A PG type
- Values can be added to the beginning or end of an enum, but not in the middle
- Enum values cannot be dropped
- Can by found in `pg_type`
- Can be used as a column type across multiple tables

In an AR migration `create_enum` creates the type, then you can set a column to the `enum` type and specify `enum_type: :my_enum` in the options.

```ruby
create_enum :account_type, %w[personal business]

def change
  add_column :users, :enum, enum_type: :account_type
end
```

Or in SQL:

```SQL
CREATE TYPE account_type AS ENUM ('personal', 'business');
```

'Domains' are very similar, but can have additional constraints like `NOT NULL` attached and are defined using `CHECK` constraints like:

```SQL
CREATE DOMAIN account_type AS TEXT
CONSTRAINT account_type_check CHECK (
    VALUE IN ('personal', 'business')
);
```

### ActiveRecord Validations

To create custom validators extend `ActiveModel::Validator` and add a `validate(record)` method.

You can also validate individual attributes by extending `ActiveModel::EachValidator` and adding a `validate_each(record, attribute, value)` method.

### Useful Gems

- [active_record_doctor](https://github.com/gregnavis/active_record_doctor)
- [database_consistency](https://github.com/djezzzl/database_consistency)

## Chapter 5 - Modifying Busy Databases Without Downtime

- Multiversion Concurrency Control (MVCC): Mechanism for managing row changes and concurrent access
- ACID: A set of guarantees Postgres makes about Atomicity, Consistency, Isolation, Durability
- Isolation levels: configurable access level for transactions
- Denormalization: Duplicating some data for improved access speed
- Backfilling: Populating columns with new data
- Table Rewrites: Internal changes from schema migrations that cause a significant availability delay

`disable_ddl_transactions!` disables ALL transaction types, but only the one that wraps the whole migration. It doesn't apply to explicit transactions or transactions around individual operations in the migration.

You can add a lock timeout at a local, database or session level to prevent transactions waiting on locks forever. You can set `log_lock_waits` and provide a value for `deadlock_timeout` to log them to `postgresql.log` when they occur.

`statement_timeout` can be set from Rails in `database.yml` to limit the maximum time a statement can run before being cancelled. If you have some queries which are allowed/expected to be slow, you can route them through a separate DB config to the same database, but with a smaller connection pool and longer `statement_timeout`.

### Indicators of Dangerous Migrations

- Adding a constraint which is validated immediately to an existing column
  - Should instead add it as `NOT VALID` which applies only to new rows, then validate in second migration
  - Still does need a lock with `NOT VALID` but extremely brief
  - _Why is this better though? Applying the validation should still be slow, and still needs to happen._
- Removing or renaming a column or table which will cause issues with the 'Schema Cache'
- Changing the column type to an incompatible type

#### Schema Caching

When AR starts up it scans the DB tables for each model backed by one and stores the fields in the schema cache. If you drop a column without considering AR (_how?_), you might get errors when it tries to write to column that doesn't exist.

Can avoid by adding the column to `ignored_columns`, but at that point you're restarting/deploying anyway so why not just use AR for the migration/make code changes to not access the removed column? Seems to be something about stuff [waiting in memory](https://api.rubyonrails.org/classes/ActiveRecord/ModelSchema/ClassMethods.html#method-i-ignored_columns), but still not exactly sure of the usecase.

#### Backfilling large tables

- Application Defaults: provide application-level defaults for the missing values, in tandem with applying those defaults when the old rows are saved
- Over-provisioning: Temporarily over-provision the DB server and/or run during quiet periods.
- Double Writing: Write to two locations simultaneously and switch reads to one.
- Backfill indexes: Add temporary indexes to speed up backfilling.
- Specialized tables: Duplicate data from a source to a special table
  - unconnected to anything else
  - set to `UNLOGGED` (no crash protection)/with auto-vacuum disabled
  - you want as few rows as possible here, just enough to identify the record in the destination table and hold the values you're backfilling

### Useful Gems

- [Strong Migrations](https://github.com/ankane/strong_migrations)
- [pgBadger](https://github.com/darold/pgbadger)
  - organizes lock-related info from `postgresql.log`
