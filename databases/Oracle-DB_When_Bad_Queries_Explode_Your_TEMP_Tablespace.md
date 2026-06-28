> 📖 **Original article:** [Oracle DB: When Bad Queries Explode Your TEMP Tablespace](https://www.valtersit.com/guides/databases/Oracle-DB_When_Bad_Queries_Explode_Your_TEMP_Tablespace/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

I have been an Oracle DBA since the days when we tuned 8i on Solaris boxes using nothing but Statspack and caffeine. If there is one thing that has remained constant across two decades and multiple versions of the Oracle Database, it is the sheer, unadulterated capacity for a "Self-Service Analyst" to take down a multi-million dollar production cluster with a single, poorly formatted SQL statement.

It usually happens on a Friday afternoon. The phone rings, and the monitoring team reports that the entire EBS or Data Warehouse instance has stopped processing. Every session is sitting in a `wait` state. You check the alerts and see the dreaded `ORA-01652: unable to extend temp segment by 128 in tablespace TEMP`.

Somewhere in the office, a "Data Scientist" is staring at a spinning wheel in their BI tool, wondering why their "simple report" is taking so long. They don't realize that by writing `SELECT * FROM Table_A, Table_B, Table_C` without any `JOIN` conditions, they have created a Cartesian product that is currently attempting to sort $500,000,000,000,000$ rows in memory.

### The Mechanics: The Spill to Disk

To understand why your database just committed suicide, you have to understand Oracle's memory architecture—specifically the relationship between the **PGA (Program Global Area)** and the **TEMP Tablespace**.

#### 1. The PGA Workarea
When a user executes a query that requires sorting (`ORDER BY`), grouping (`GROUP BY`), or a Hash Join, Oracle allocates a portion of the PGA called a **Workarea**. Ideally, this entire operation happens in RAM. This is "Optimal" execution. It’s fast, it’s efficient, and it doesn't touch the disk.

#### 2. The One-Pass and Multi-Pass Disaster
However, memory is finite. When the size of the data being sorted exceeds the available PGA (governed by `pga_aggregate_target`), Oracle can no longer perform the sort in RAM. It begins "spilling" the data to the TEMP tablespace.

This is where the performance cliff lives. The database starts writing "sort runs" to the physical datafiles of the TEMP tablespace. If the sort is massive (like a Cartesian product), it enters "Multi-Pass" mode. The database is now reading and writing the same blocks over and over again from the SAN just to figure out which row comes before another. 

#### 3. The Global Impact
The TEMP tablespace is a shared resource. Unlike a permanent tablespace that holds your tables, TEMP is used by every session. When one runaway query fills up the 100GB or 500GB of allocated TEMP files, *every other user* on the system who needs to perform a sort—even a tiny one—will fail. You have reached a total system-wide standstill because one person didn't know how to write a `WHERE` clause.

### The Mechanics of the Fix: DBA Defense

A Senior Oracle DBA does not wait for the `ORA-01652` error. You must be proactive in hunting these sessions down before they consume the last megabyte of storage.

#### 1. Monitoring Sort Usage
You need to be intimately familiar with `V$SORT_USAGE` (or `V$TEMPSEG_USAGE`). This view tells you exactly which session is holding onto TEMP segments and how much space they are consuming. If you see a session for a user named "Marketing_Intern" consuming 80GB of TEMP, you don't send an email; you kill the session.

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/databases/Oracle-DB_When_Bad_Queries_Explode_Your_TEMP_Tablespace/](https://www.valtersit.com/guides/databases/Oracle-DB_When_Bad_Queries_Explode_Your_TEMP_Tablespace/)**
