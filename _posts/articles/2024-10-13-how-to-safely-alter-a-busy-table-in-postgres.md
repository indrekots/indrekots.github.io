---
layout: post
title: "How to Safely Alter a Busy Table in Postgres"
excerpt: "When altering busy tables in Postgres, acquiring the required locks can lead to application downtime if not handled carefully. This post explains the potential pitfalls of using ALTER TABLE on high-traffic tables, and provides strategies for minimizing disruptions."
modified: 2024-10-13 23:17:18
categories: articles
tags: [postgres, database, sql, locks]
image:
  path: /images/2024-10-13-how-to-safely-alter-a-busy-table-in-postgres/cover.jpg
  thumbnail: /images/2024-10-13-how-to-safely-alter-a-busy-table-in-postgres/cover_thumb.jpg
  caption: "Photo by [Jan Antonin Kolar](https://unsplash.com/photos/brown-wooden-drawer-lRoX0shwjUQ)"
comments: true
share: true
published: true
aging: false
amazon_links: false
---

As you introduce new features to your application, you will almost certainly need to evolve your database schema.
However, in Postgres, there are a couple of things you need to understand before you start issuing `ALTER TABLE` commands if you want to avoid bringing your application to a grinding halt.

## Table Locks

Postgres provides [various modes for locking a table](https://www.postgresql.org/docs/current/explicit-locking.html#LOCKING-TABLES).
When it comes to the `ALTER TABLE` command, it acquires the strongest table lock available: `AccessExclusiveLock`.
This lock ensures that the holder is the only transaction accessing the table in any way.
No other transaction is allowed to read from or write to the table while the `ALTER TABLE` operation is executing.

For tables with low traffic, this usually isn't a problem.
A simple addition of a column is typically very fast and it doesn't hinder the application's performance.
However, things get more complicated when you need to alter a table that receives a lot of concurrent transactions.
To better understand the problem, let's look at the following example scenario.

## Example

The following simulates a query that an application might issue.
It starts a transaction and queries for accounts.
A select query acquires an `AccessShareLock` on the table.
Notice that the transaction isn't immediately committed.
This simulates a scenario where the application might be doing some work before committing the transaction.

```sql
begin;
select * from accounts;
```

Next, in another database session, let's simulate a schema migration.

```sql
alter table accounts add column age int;
```

When the `ALTER TABLE` command is issued, you'll see that the statement never finishes.
This is because the `ALTER TABLE` transaction tries to acquire an `AccessExclusiveLock` on the `accounts` table but is unable to do so because there's an existing read transaction on the table.
As mentioned earlier, the `AccessExclusiveLock` guarantees that the holder of the lock is the only transaction working on the table.
This also means that when there are existing locks on the table, the transaction has to wait until all existing locks on the table are released.

In this scenario, the `accounts` table receives a lot of transactions.
The following simulates the next query from the application.

```sql
select * from accounts;
```

Once we execute this query, we immediately notice that it doesn't return anything.
The query just hangs.
It tries to acquire an `AccessShareLock` lock on the table but is unable to do so because of the transaction created in the previous step.

The `ALTER TABLE` transaction isn't able to proceed.
It joined a wait queue.
The `AccessShareLock` required by the final select query is incompatible with `AccessExclusiveLock`.
Therefore the final select query also has to join the wait queue.

To sum up, in a busy table, it may be very difficult to acquire an `AccessExclusiveLock` that's required by `ALTER TABLE` commands.
It's surprisingly easy to accidentally bring your application to a grinding halt.
If the `ALTER TABLE` transaction cannot acquire the lock immediately, you're denying access to the table for everybody.

## How to Not Mess Up

An easy way to avoid accidental service disruptions is to define a [`lock_timeout`](https://www.postgresql.org/docs/current/runtime-config-client.html#GUC-LOCK-TIMEOUT) for the database migration.
This ensures that Postgres will abort the statement if it waits longer than the specified amount of time while attempting to acquire a lock on a table.

```sql
begin;
set lock_timeout = '1s'
alter table accounts add column age int;
commit;
```

If the `ALTER TABLE` statement cannot acquire a lock on the `accounts` table within one second, the statement is cancelled with the following error.

```
ERROR:  canceling statement due to lock timeout
```

The value of `lock_timeout` should be set to something small.
Keep in mind, in the worst case scenario, this is how long you're blocking all other transactions from accessing the table.

With busy tables, however, you might find that you need to perform the migration multiple times until it executes successfully.
Therefore, if you can and your tooling supports it, it makes sense to retry the migration automatically several times.

You might also want to consider when to perform the schema migration.
For example, your database might be processing fewer transactions during the night.
Performing the migration during low traffic periods increases the likelihood of success.

## Summary

Most schema migration operations in Postgres acquire exclusive locks.
In databases that process a lot of transactions, acquiring the lock can take a significant amount of time.
If the lock isn’t acquired quickly, it can result in blocked transactions and even application downtime.
To avoid any negative outcomes, it's strongly advised to configure a `lock_timeout`.