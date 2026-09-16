> 📖 **Original article:** [PostgreSQL Backup Strategies: WAL, PITR & Restore Testing](https://www.valtersit.com/guides/databases/postgresql-backup-strategies-wal-pitr-restore-testing/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

A team I worked with ran a nightly `pg_dump` to S3 for eight months. Every night, `pg_dump` exited `1` on a lock timeout, and the shell script swallowed stderr into `/dev/null`. The S3 objects were 0 bytes. Nobody noticed because nobody was looking — the cron job "succeeded," the bucket had objects, the dashboard was green. They found out on the Tuesday their primary's `pg_wal` volume filled up after a `DELETE` went sideways. There was no backup. There was a hope, and a cron entry.

This article is for the SRE, DBA, or platform engineer who owns a production Postgres cluster and has *probably never completed a timed restore*. It covers three things: WAL archiving (the continuous stream), PITR (the time machine), and restore testing (the part everyone skips). It's Postgres-centric, but the principles map cleanly to MySQL binlogs + XtraBackup and MongoDB oplog + snapshots — the FAQ touches both.

Before any of it, one uncomfortable framing: **RPO and RTO are business decisions, not DBA preferences.** If you can't state your RPO in seconds, you don't have a backup strategy. You have a backup habit.

:::note[TL;DR]
- A replica is not a backup — it replicates your `DROP TABLE` in milliseconds.
- `archive_command` must return non-zero on failure and must never block. `cp` to NFS is a footgun that has PANIC'd production clusters.
- `recovery_target_time` is interpreted in the *server's* timezone. Always write an explicit offset or you'll overshoot by hours.
- Test restores on a schedule, on a separate host, with an automated harness. If your last restore test was "the day we set it up," it doesn't count.
:::

## Prerequisites

- PostgreSQL 14–17 (differences from 9.6 and 11/12 are called out inline).
- `pgbackrest` installed on the primary and on your restore host.
- An object store bucket (S3, GCS, or MinIO) with its own credentials — *not* shared with production app roles.
- A restore target: a dedicated VM, a container, or an isolated namespace with no route to prod.

## Replication Is Not a Backup, and Other Things You Should Already Know

The single most expensive misunderstanding in Postgres operations is that a hot standby is a disaster recovery strategy. It is not. It is a *high availability* strategy. The difference is not pedantic — it's the whole point.

### The Four Ways Backups Fail

**Silent failure.** The job returns exit 0 and the artifact is garbage. This is the `pg_dump` story above, and it's more common than anyone admits. Anything that produces an artifact without validating it is a coin flip.

**Corruption.** Bit rot on cold storage, a truncated S3 multipart upload from a killed `aws s3 cp`, or a missing WAL segment in the middle of your archive. This is why `pgbackrest` checksums every file and why `pg_verifybackup` exists.

**Inaccessible.** Credentials rotated and the backup role lost `s3:GetObject`. The KMS key lives in a vault nobody can unseal at 3 AM. The bucket policy was tightened for a compliance audit. Ransomware encrypted the backup share because it was mounted on the same host.

**Untested.** "We've never had to restore" is the most expensive sentence in this article.

Rank them by how often they actually bite: **untested first, inaccessible second, silent third, corruption last.** None of those four show up on a Grafana dashboard unless you build the panel.

### The RPO/RTO Math You Do Before You Pick a Tool

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/databases/postgresql-backup-strategies-wal-pitr-restore-testing/](https://www.valtersit.com/guides/databases/postgresql-backup-strategies-wal-pitr-restore-testing/)**
