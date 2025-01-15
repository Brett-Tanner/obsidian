---
title: High Performance Postgres for Rails
description: How to optimise Postgres for Rails
---

## Chapter 1 - An App to get you started

SQL is declarative, meaning the code you write tells Postgres what you want and Postgres decides how to get it. It does this by taking various factors into account to develop one or more 'query plans' for the data, then chooses among them based on statistics it collects and cost estimates of various operations. While you can't explicity select a plan, you can strongly influence which one is chosen.

## Migrations

Migrations generate **DDL** (Data Definition Language) statements like `CREATE TABLE`, `ALTER TABLE`, `DROP TABLE`, etc. You can also add **DML** (Data Manipulation Language) statements like `INSERT`, `UPDATE`, etc., but it's usually best to put these in a rake task so they don't slow down deployments.

Remember there's a table called `schema_migrations` rails uses to track which migrations you've run; `rails db:migrate` adds the versions of the migrations it runs to that table. If you're using an SQL schema running migrations cases `pg_dump` to add them to your `structure.sql` as `INSERT` statements, making them ready to run on another DB.

## Auth

Users are 'roles' with the `LOGIN` privilege in Postgres, and their access info can be stored in a `~/.pgpass` file using a semicolon as a delimiter.

## Terminology

- bloat: allocated but unused space, or space used on old tuples
- functions: enhance your database with new functionality, can be written in SQL or a procedural language
- pages: pre-allocated space filled with `INSERT`, `UPDATE`, etc. on table rows or indexes. 8KB by default.
- planner: also known as the 'optimizer', compares query plans and chooses the best one using a cost-based heuristic
- procedures: procedures expand on functions, granting more features like transactional control
- transactions: overloaded to mean many things, but commonly refers to a group of operations performed as one unit of work. To achieve this they have an 'isolated' view of the data at a point in time.
- tuples: immutable row versions created when a row is 'updated'

## Useful Tools

[rails-erd](https://github.com/voormedia/rails-erd) looks handy for generating Entity-Relationship Diagrams automatically. Not sure if it works with anything > 6 though.

[PostgresApp](https://postgresapp.com/) is an alternative to managing postgres versions with homebrew on Mac. It gives you a UI to switch between versions and start/stop/create servers.

`tee <file>` takes the output piped to it and redirects it to both stdout and the file.

# Chapter 2 - Administration Basics

## psql

The native Postgres client, ships with it by default.

You can stop and restart it with `pg_ctl restart` once you've set the `PG_DATA` environment variable found by `SHOW data_directory;`.

### Config

`~/.psqlrc` lets you customise `psql`, like in this [article](https://thoughtbot.com/blog/an-explained-psqlrc) from Thoughtbot.

You can find the path to your config file from `psql` with `SHOW config_file;`.

### Meta Commands

- `\c {database}` - connect to a database
  - A database is a logical collection of tables and other objects existing within your PG server.
- `\e` - launches your editor, saving and exiting runs the command you've edited
- `\l` - lists databases
- `\o {file}` - pipes the output of your queries to a file until toggled back with plain `\o`
- `\s` - command history
- `\timing` - toggles timing
- `\q` - quit
- `\x` - toggle vertical output

If you put `bind "^R" emc-inc-search-prev` in your `~/.editrc`, you can use `Ctrl+R` to search backwards in the command history. Just start typing and it'll display a match; execute with `\g`.

## Extensions

Stored in the `pg_extension` system catalog, a PG table or view tracking internal information.

Can be enabled in your config file as a comma-separated list for the `shared_preload_libraries` config option, then running a `CREATE EXTENSION` statement.

## Observability

The catalog `pg_stat_activity` shows current activity in your DB.

You can use it to find the `pid`s of running processes (among other things) and terminate them gently with `SELECT PG_CANCEL_BACKEND(pid);` or abruptly with `SELECT PG_TERMINATE_BACKEND(pid);`.

## Locking

Can be queried from the system catalog `pg_locks`.

[pglocks](https://pglocks.org/) is a handy resource for a deeper dive.

Generally uses pessimistic locking (resources locked upfront when modified), but can use optimistic.

Exclusive locks block all queries trying to access the selected resource, which can be a major problem on big tables. A deadlock is when 2 processes permanently block eachother.

Locks can be created explicitly, but are most often created implicitly be statements.

## Basic Commands

- `CREATE DATABASE {name}` - creates a new database, connect to it with `\c {name}`
- `CREATE INDEX {name} ON {table}({column})` - creates an index on a column
  - in prod you probably want the `CONCURRENTLY` keyword after `INDEX` to allow the table to continue to be queried
- `DROP INDEX {name}` - drops an index
