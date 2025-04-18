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
- `\watch` - will re-run a query every 2 seconds by default, until stopped with `ctrl-c`
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

## Chapter 6 - Optimising ActiveRecord

Can use the `query_log_tags_enabled` option to give you query logs linking SQL to the call site in Rails, as long as you're not using prepared statements . Will also show the logs in Rails console.

- `#load_async` can prevent long running queries blocking others which don't depend on them.
- the `returning` option can specify return values you want from an insert (only an insert in AR)
  - but if you [use SQL](https://www.postgresql.org/docs/current/dml-returning.html) it can be used with any modification
  - useful for avoiding an extra query to fetch the updated data, like with `#reload`

### Common Problems

- N+1 queries
  - can use joins (`joins`, `left_joins`)
  - can use `preload(association)` to preload associations if you don't need conditions on the association
  - `includes(association)` automatically uses `IN` like preload or a `LEFT OUTER JOIN` if you add conditions on the association
  - for a nuclear option, can enable strict loading with `Model.strict_loading.all`, which prevents lazy loading completely
- Not using counter_cache
- Not using PG aggregate functions
  - It's possible to define your own in PG, and there are more than AR gives you access to
- Allocating too many objects by relying on AR when the primitives returned by raw queries like `find_by_sql`/`select_all`/`select_one` would be fine

### CTEs (Common Table Expressions)

Basically named sub-queries added as of AR 7.1, you can use `outer_query.with(subquery_name: subquery)` to create them and `#from(subquery).chained_query` to use them.

```ruby
Trip.with(
    recently_rated:
        Trip.where.not(rating: nil)
            .where("completed_at > ?", 30.days.ago)
).from(recently_rated).count
```

### Database Views

Encapsulate a query, giving it a name and storing the query text as an object in Postgres. No native support in Rails but can be added with a gem like [Scenic](https://github.com/scenic-views/scenic) which adds the ability to generate view migrations. You can make a view-backed model, which you should probably make read-only to be safe.

Can be defined and executed like:

```SQL
CREATE VIEW my_view AS
  SELECT * FROM my_table
  WHERE my_column = 'some value';

SELECT * FROM my_view;
```

While they don't have performance advantages themselves, 'Materialized Views' which pre-calculate and store results do.

- these are only useful if some amount of staleness is acceptable, similar to caching
- You can refresh a materialized view with `REFRESH`, can be done concurrently if there's a unique index
- You can add indexes to materialized views to improve performance even further
- No partial refreshes, if a single row changes the whole view needs to be re-fetched, however some extensions attempt to address this

### Caching

The query cache keeps results around for the duration of a controller action.

Prepared statements store queries without their parameters, allowing some time to be saved on parsing by plugging new parameters into them for subsequent queries. Rails creates them automatically by default and can store up to a thousand per connection.

### Useful Gems

- [bullet](https://github.com/flyerhzm/bullet)
- [prosopite](https://github.com/charkost/prosopite) - claims to avoid false positives/negatives you can experience with `bullet`

## Chapter 7 - Improving Query Performance

- Selectivity: How narrow or wide a selection is
- Cardinality: How many unique values there are
- Sequential scan: reading all rows for a table
- Index scan: Fetching values from an index
- Index-only scan: Fetching values *only* from the index, without needing to access table data

You can listen for slow queries usin `ActiveSupport::Notifications` on `sql.active_record` by measuring their duration and logging if they exceed a set time.

Internally, `pg_stat_statements` collects extensive stats for (up to 5000 by default) groups of normalized queries (grouped if they're the same with params stripped out).

### Query Execution Plans

`EXPLAIN {query}` - Gives you the predicted cost, rows and width of the most efficient execution plan for that query. Adding the `ANALYZE` option actually runs the query, and gives you the actual cost to compare with the estimate.

`ANALYZE` can also be used alone to manually gather statistics, for example `ANALYZE VERBOSE trips` will gather statistics for the `trips` table.

## Chapter 8 - Optimized Indexes

- Index definition: The columns covered by the index
- Partial indexes: Only cover some rows, decided by a condition
- Covering indexes: cover all columns needed for a query
- Operator classes: Operators to be used by an index, specific to index types
- Heap scan: Retrieve the rows identified by an index to get the unindexed column values. Can be lossy (get full pages which contain an indexed row, if `work_mem` is an issue) or exact.
- High cardinality: A large number of unique values in the column (makes B-tree indexes especially efficient)

Indexes default to B-tree, but you can use other types like gin for json, GIST or hash.

Since data is stored in 'pages' on disk, indexes are useful for big tables to potentially avoid reading a bunch of unnecessary pages.

However for smaller tables the planner might decide a sequential scan is faster, for reasons like:

- reading the few available pages is faster than accessing the index
- `SELECT *` or requesting many un-indexed columns, as even after filtering on indexes a second 'heap scan' is needed to retrieve the un-indexed column values
- Newer versions of PG can parallelize sequential scans, potentially making them even less costly (in some ways).
- Data distribution (skewed or uniform)
- Costs associated with sequential/random access

The `Buffers` line in `EXPLAIN` output contains `shared hit` (read from memory) and `read` (required disk access).

### Multi-Column Indexes

Can have a huge impact on performance for specific queries which only need the columns in the index, but take up extra space and can lead to redundant indexes. The [PG docs](https://www.postgresql.org/docs/current/indexes-multicolumn.html) recommend using them sparingly as single-column indexes are usually sufficient.

The leading column in a multi-column index should (generally) cut the list down as much as possible. Queries using only the non-leading columns from a multi-column index may even skip the index entirely and do a sequential scan.

They sound pretty useless with all those drawbacks, but can still be very effective for specific queries which might do something like only filter on the first indexed column then sort by the others.

### Partial Indexes

For columns with low cardinality like booleans, having a normal index is unlikely to be efficient as the small number of possible values mean many rows will need to be loaded regardless of the value filtered by.

Partial indexes can help with this, and also offering other benefits like decreased size & write latency. For example if you add a partial index on only the least common boolean value it can significantly speed up queries for rows with that value. In addition to boolean columns, they can also be useful for excluding `NULL` values from searches filtered on a nullable column.

### Expression Indexes

Have an expression in their definition, transforming a value before it's stored. Can also be referred to as 'functional indexes'. They're useful for normalizing data in unique indexes, for example lowercasing all email addresses to prevent duplicates (do we/does Devise do this?).

Especially important to check these are actually used, as the query conditions must match the expression *exactly*.

### Other Index Types

#### GIN

Work well for nested data like `jsonb` columns. If you want to use one for containment queries (e.g. rows where the json data has a 'pizza': true k/v pair) then you need to set the index up differently.

```sql
-- Regular GIN index``
CREATE INDEX ON trips USING GIN(data);

-- Containment GIN index
CREATE INDEX ON trips USING GIN(data JSONB_PATH_OPTS);
```

It's also possible to use a B-tree index with jsonb columns using expression indexes, but that seems more limited/verbose.

The `postgres-json-schema` extension allows adding and enforcing a schema for your JSON columns using check constraints. You need to generate a JSON Schema compatible schema definition string, then create a check constraint using it.

#### BRIN

- Block: synonym for page
- Block range: A group of pages physically adjacent in the table

BRIN is a Block Range INdex, which points to a page/block and stores the min/max values from that block for the indexed column. As such, they're best suited for data where the physical layout matches how the data is queried, like timestamps.

Since only pages are stored, it takes up very little space/has a small impact on insert performance compared to a B-tree index and can even outperform it on the right data.

```sql
CREATE INDEX ON trips USING BRIN(created_at);
```

#### Hash

Store computed hashes from source columns rather than the actual value.

Hash indexes only offer equality comparisons and cannot be used for unique constraints, in addition to potentially being *much* slower for new entries. However they use less space compared to B-tree indexes and entries have a uniform size (the size of the hash code). So potentially useful if indexing a column with large values.

### Indexes for Sorting

Since B-tree indexes are stored in sorted order by definition (ascending by default), they're very useful for sorting.

`NULL` values are at the start by default, can be changed in SQL using `NULLS LAST` or AR using something like `.order(Table.arel_table[:completed_at].desc.nulls_last)`.

### Covering Indexes

Support a specific query by indexing all the columns necessary for that query.

PG has the `INCLUDE` keyword to help with these, by specifying columns which can't be filtered by but supply data from the index to the `SELECT` clause. you append `INCLUDE` when creating your index and specify the data-only columns you want included.

### Useful Resources

- [Index maintenance queries](https://wiki.postgresql.org/wiki/Index_Maintenance)


## Chapter 9 - High Impact Database Maintenance

- Visibility map: Tracks which tuples are visible to transactions
- Autovacuum: Kinda like garbage collection for PG

3 most important maintenance operations are
- `VACUUM`: Is triggered automatically based on certain conditions
  - regular: marks space from dead tuples as reusable, updates visibility map
  - `FULL`: reclaims space from dead tuples. Takes out heavy exclusive locks, generally can't be run online (in maintenance?)
- `ANALYZE`: Recalculates statistics
- `REINDEX`: Rebuilds indexes

### Tuples

Can be queried like `SELECT ctid, id FROM users WHERE id = 1;` `ctid` has two values, the page number and the tuple number (which increments when the original row is changed).

### VACUUM

Pages aren't filled completely, they leave space to store new tuples when rows are updated. You can pass `ANALYZE` to `VACUUM` to recalculate statistics at the same time like `VACUUM (ANALYZE, VERBOSE) users`, and `SKIP_LOCKED` to skip locked tables.

You can also run paralell vacuum workers, and configure the maximum number of paralell workers.

You can query the history of `VACUUM` and `ANALYZE` runs from `pg_stat_all_tables` as `last_autoanalyze`/`last_analyze` etc.

#### Autovacuum

Should generally assign more resources as load increases, especially `UPDATE` and `DELETE` statements. Otherwise the rate dead tuples accumulate might exceed the amount autovacuum can deal with, leading to runaway bloat.

How do we view vacuum times in Crunchy? Do they do the tuning for us?

`autovacuum_vacuum_threshold` and `autovacuum_vacuum_scale_factor` determine when autovacuum should run based on the total number of dead tuples and their percentage of total tuples. They can be changed for individual tables or globally.

`autovacuum/_vacuum_cost_limit` decides how much work the process can do before going back to sleep.

### REINDEX

Used to remove references in the index to dead tuples, can be done `CONCURRENTLY`. You can provide a table name rather than the index name to reindex all indexes on a table.

### Unused Indexes

- Can prevent 'Heap only' tuple updates, which are much faster
  - HOT updates occur when the update doesn't touch an indexed column, and there's free space in the page
  - Can be made more likely by reducing the `fill_factor`
- Increase `VACUUM` runtime
- Increase query planning time
- Slow down backup & restore operations
- Obviously increase insert time & take up space

May not need indexes on append-only tables. Why?

## Chapter 10 - Reaching Greater Concurrency

- Connection Pooler: 'Middleware'-like software between Rails & PG to allow more efficient use of connections
- Client connections: Connections between ActiveRecord and the pooler
- Server connections: Connections between the pooler and the database

Database connections objects created in PG, and are used by Rails, `pgsql` and other tools like `pg_bouncer`. The total number of possible concurrent connections is limited by server resources, and establishing new connections adds latency due to authentication and SSL/TLS negotiation.

### From Rails to PG & back

1. AR generates a SQl query
2. The `postgresqladapter` gem ensures the query is compatible with PG
3. The `pg` gem acts a client interface from Ruby to PG
4. AR uses a connection from the pool (as opposed to creating a new one) to send the query to PG
5. PG gets the result and sends it back along the same connection. The connection stays open, but is idle.

AR is responsible for closing the connection, if not closed it'll be left in an idle state until `idle_timeout` is reached. It's important to balance keeping idle connections around for latency reduction with not having idle connections consuming all your possible connections.

Transactions can generally be in `idle` or `active` states. `idle` can include a connection doing work in a transaction but not actually using the connection. The `idle in transaction` state means a transaction was opened but is not completing any work, these should be terminated as errors.

### Managing Idle Connections

If idle connections take up all the available connections to your DB, any further attempts to connect will fail with `FATAL: sorry, too many clients already`.

`idle_timeout` defaults to 5min in Rails, and PG 14 or later allows you to set `idle_session_timeout` on the server.

PG has a `max_connections` value which defaults to 100, considered low for modern instances. Likely to be able to increase to something more like 500, but a tool like `pgtune` will give you a better idea for your DB.

How would you ever get to that many connections? Remember Puma spawns multiple workers, which can each have multiple threads. Same with Sidekiq. And if you have `n` application servers connecting to a single database, multiply that by `n * connection pool size`.

### `pg_bouncer`

- allows connections to exceed the `max_connections` value (good?)
- can hold connections open even if DB is temporarily unavailable, preventing them being lost
- re-uses connections more efficiently and offers different 'pool modes' with different trade-offs
  - 'aggressive' pool modes re-use connections more aggressively, offering latency reductions
  - but potentially prevents the use of some features like prepared statements or query logging

An alternative is [pgcat](https://github.com/postgresml/pgcat).

### Connection Errors

Be sure to set `statement_timeout`, `lock_timeout`, and `idle_in_transaction_session_timeout` to prevent long running queries. You can also set `idle_session_timeout`, but that might cause issues with legitimate sessions so tweak it more conservatively. Lower values will result in more errors, but increase your resiliency by ensuring long-running queries don't monopolize the DB.

### Locks

Setting `log_min_error_statement` will log queries to `postgresql.log` if blocked, and `log_lock_waits` provides a more targeted/detailed option.

`SKIP LOCKED` in a query skips locked rows, useful for a job that can come back later and try again.

Locks can be acquired pessimistically (upfront) or optimistically. Rails implements optimistic locking by adding a `lock_version` column to the table, which AR manages. If a modification to the row notices a newer version than the version it's modifying exists, it'll raise an `ActiveRecord::StaleObject` exception and need to be retried.

PG can also do optimistic locking with `SERIALIZABLE`.

'Advisory locks' are a weaker but faster type of lock, which is up to Rails to create and enforce. They avoid table bloat and are automatically cleaned up at the end of the session. They can be at the session or transaction level.


