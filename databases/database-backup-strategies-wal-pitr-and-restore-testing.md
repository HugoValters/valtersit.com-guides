> 📖 **Original article:** [Database Backup Strategies: WAL, PITR, and Restore Testing](https://www.valtersit.com/guides/databases/database-backup-strategies-wal-pitr-and-restore-testing/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

Your backup is not real until you've restored it. Everything else is just a hope and a prayer.

I've watched a production database die at 2:47 AM on a Sunday. The team had backups — beautiful, automated, monitored backups. They also had a recovery procedure that had never been tested. The restore failed because the backup server's disk was full. The backup tool reported success because it was writing to a buffer, and nobody checked the actual file sizes. The silence from the monitoring alerts was deafening.

Most database outages don't kill you. The *recovery* from them does. This guide covers WAL archiving (PostgreSQL-focused, but the concepts apply to MySQL/MariaDB), Point-in-Time Recovery (PITR), and the non-negotiable discipline of restore testing. By the end, you'll have a production-grade backup strategy that survives contact with reality.

The "full backup once a week" mentality is dead. If you're still doing that, stop reading and go fix your life. We aren't just backing up data; we are backing up *time*.

:::note[TL;DR]
- A backup is a recovery path, not a file copy — design accordingly
- WAL archiving is non-negotiable for any database where data loss matters
- PITR requires both a base backup and continuous WAL archives
- Restore testing must be automated, scheduled, and treated as production-critical
- Encryption at rest and in transit is mandatory, not optional
:::

## Prerequisites

Before we dive in, you need:

- PostgreSQL 12+ (the examples use PG 15/16 syntax)
- Root or sudo access to your database server
- A separate storage location for WAL archives (local disk, NFS, or S3)
- Docker or a VM for restore testing
- Basic familiarity with `psql` and Linux command line

## The Fundamentals — Why Your Backup Strategy is Probably Wrong

The core misconception most teams have: a backup is a copy of the data. It's not. A backup is a *recovery path* — a documented, tested sequence of steps that takes you from disaster to operational database. If you can't restore, you don't have a backup. You have a data hoarding habit.

### The 3-2-1 Rule (and Why It's Just the Starting Line)

Three copies, two media types, one offsite. That's the minimum bar for not being a complete amateur. If you think this is the endgame, you're a junior. The 3-2-1 rule protects against hardware failure and site loss. It does nothing for logical corruption, ransomware, or a bad `DELETE` statement.

### RPO and RTO — The Metrics That Actually Matter

**RPO (Recovery Point Objective):** How much data loss can you tolerate? If your RPO is 5 minutes, you can afford to lose the last 5 minutes of transactions.

**RTO (Recovery Time Objective):** How long can you be down? If your RTO is 4 hours, you have 4 hours from disaster to full operation.

If your RPO is "zero," you need synchronous replication, not just backups. Don't conflate the two. Backups protect against data corruption and deletion. Replication protects against server failure. They serve different purposes and you need both.

### The "Full Backup" Fallacy

Weekly `pg_dump` runs are for small apps and people who don't value their jobs. They don't scale, and they don't protect against logical corruption. The worst part: a full backup from 7 days ago means you lose up to 7 days of data. That's not a backup strategy; that's a data loss strategy with extra steps.

Here's the difference between the two backup types:

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/databases/database-backup-strategies-wal-pitr-and-restore-testing/](https://www.valtersit.com/guides/databases/database-backup-strategies-wal-pitr-and-restore-testing/)**
