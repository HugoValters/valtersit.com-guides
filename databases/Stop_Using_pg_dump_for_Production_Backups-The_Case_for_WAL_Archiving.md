> 📖 **Original article:** [Stop Using pg_dump for Production Backups: The Case for WAL Archiving](https://www.valtersit.com/guides/databases/Stop_Using_pg_dump_for_Production_Backups-The_Case_for_WAL_Archiving/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

I recently reviewed a "Disaster Recovery Plan" for a fintech startup that processed thousands of transactions an hour. Their entire strategy consisted of a single cron job that ran `pg_dump` every night at 2:00 AM and pushed the resulting `.sql` file to an S3 bucket.

I asked the lead dev: "If your database server's NVMe controller dies at 4:30 PM, what happens?"

The answer, after a long silence, was: "We lose everything since 2 AM." 

That is fourteen and a half hours of customer data, financial records, and audit logs gone forever. In the world of enterprise data, `pg_dump` is a utility for migrations and local development; it is not, and has never been, a professional backup strategy. If you are relying on it for production, you are just waiting for a catastrophic day at the office.

### The Mechanics: The Logical Snapshot Trap

To understand why `pg_dump` fails you, you have to understand the difference between a logical backup and a physical backup.

#### 1. The Recovery Point Objective (RPO) Failure
`pg_dump` creates a logical snapshot of your database at the exact moment the command started. If you run it once a day, your RPO is 24 hours. In 2026, a 24-hour data loss is a business-killing event. Professional systems aim for an RPO of seconds or minutes, not half a day.

#### 2. The Resource Tax
`pg_dump` has to read every single row of every single table. On a multi-terabyte database, this creates massive I/O pressure, bloats your cache, and can lead to transaction wrap-around issues if not managed correctly. It is a heavy, invasive process that slows down your production application while it's running.

#### 3. No Point-In-Time Recovery (PITR)
If a developer accidentally runs an `UPDATE` without a `WHERE` clause at 10:00 AM, `pg_dump` can't help you. Your only choice is to restore the 2 AM backup, losing eight hours of legitimate work to fix one mistake. You have no way to "roll back" to 9:59 AM.

### The Fix: WAL Archiving and PITR

The Senior DBA approach is to use **Physical Backups** combined with **Write-Ahead Log (WAL) Archiving**.

In PostgreSQL, every change made to the database is first written to a WAL file. These files are the "truth" of your database. If you take a base backup (a physical copy of the data files) and then archive every single WAL file generated after that backup, you can reconstruct the database state to any specific microsecond in history. This is Point-In-Time Recovery (PITR).

Instead of rolling your own scripts, use battle-tested enterprise tools like **pgBackRest** or **wal-g**. These tools handle the heavy lifting: parallel compression, delta restores, and managing the retention of your WAL archives.

### The Code: Enabling the Archive

You don't need to install a third-party tool to start the transition, but you do need to configure PostgreSQL to stop throwing away its history. 

Here are the essential parameters in `postgresql.conf` to enable WAL archiving. 

```text
# THE REAL ENGINEER'S WAY (Enabling WAL Archiving)
# /etc/postgresql/15/main/postgresql.conf

# 1. Enable Archive Mode (Requires a restart)
archive_mode = on

# 2. The command to execute whenever a WAL segment is filled
# This example uses a simple local copy, but in production, 
# this would be a pgBackRest or wal-g command.
archive_command = 'test ! -f /mnt/server_backups/pg_wal/%f && cp %p /mnt/server_backups/pg_wal/%f'

# 3. Ensure the logs contain enough information for recovery
wal_level = replica
```

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/databases/Stop_Using_pg_dump_for_Production_Backups-The_Case_for_WAL_Archiving/](https://www.valtersit.com/guides/databases/Stop_Using_pg_dump_for_Production_Backups-The_Case_for_WAL_Archiving/)**
