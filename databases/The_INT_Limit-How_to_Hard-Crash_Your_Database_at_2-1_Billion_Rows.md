> 📖 **Original article:** [The INT Limit: How to Hard-Crash Your Database at 2.1 Billion Rows](https://www.valtersit.com/guides/databases/The_INT_Limit-How_to_Hard-Crash_Your_Database_at_2-1_Billion_Rows/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

It’s a quiet Tuesday at 3:00 PM. Your application has been running flawlessly for four years. Suddenly, the monitoring dashboard turns into a sea of red. Every single `INSERT` statement is failing with a cryptic "Out of range value for column" error. Users can’t sign up, orders can’t be placed, and the logs are screaming into the void.

You check the database. The disk space is fine. The CPU is idling. The memory is healthy. Then you look at the `id` column of your central `transactions` table. 

`2,147,483,647`. 

Congratulations. You just hit the 32-bit signed integer limit. Your primary key has run out of numbers, and because you didn't plan for success five years ago, your entire business is now effectively offline. 

In the world of database administration, this isn't just a bug; it's a "Resume-Generating Event." You are about to find out that fixing this on a live, multi-gigabyte table takes hours of downtime that you don't have.

### The Mechanics: The 32-Bit Wall

To understand why your database just committed suicide, you have to look at the underlying math of data types. Most junior developers, following "Getting Started" guides, define their primary keys as a standard `INT`. 

In MySQL and MariaDB, a standard `INT` is a 4-byte (32-bit) signed integer. Because it is signed, it uses one bit to represent whether the number is positive or negative. This leaves 31 bits for the actual value. 
The math is simple: $2^{31} - 1 = 2,147,483,647$.

The second your auto-increment counter tries to hit `2,147,483,648`, the database engine looks at the 4-byte slot you provided and realizes the new number won't fit. It doesn't "roll over" like an odometer; it simply refuses to perform the write. 

#### The "Signed" Stupidity
Here is the real kicker: unless you are planning to store negative IDs (which you aren't), using a *signed* `INT` for an auto-incrementing primary key is a complete waste of half your numerical space. If you had at least used `INT UNSIGNED`, you would have had $2^{32} - 1$, or roughly 4.2 billion rows. You’d still hit the wall eventually, but you’d have bought yourself a few more years of ignorance.

### The Migration from Hell

When this crash happens at 3 PM, your first instinct is to run `ALTER TABLE orders MODIFY id BIGINT UNSIGNED;`. 

On a table with 2.1 billion rows, this command is a death sentence. 

In most older versions of MySQL and MariaDB, `ALTER TABLE` is not an "online" operation for data type changes. The database engine has to:
1. Create a brand new temporary table with the new schema.
2. Copy every single one of those 2.1 billion rows from the old table to the new one.
3. Rebuild every single index associated with that table.
4. Swap the old table for the new one.

This process will likely take 6 to 12 hours. During this time, the table is often locked for writes. Your business remains down for the entire duration of the rebuild. If you don't have enough free disk space to hold a second copy of your largest table, the `ALTER` will fail halfway through, leaving you in an even deeper circle of hell.

### The Fix: BIGINT by Default

The Senior DBA fix is to stop being stingy with bytes. 

In 2026, storage is cheap. The difference between a 4-byte `INT` and an 8-byte `BIGINT` is negligible compared to the cost of a 10-hour outage. 

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/databases/The_INT_Limit-How_to_Hard-Crash_Your_Database_at_2-1_Billion_Rows/](https://www.valtersit.com/guides/databases/The_INT_Limit-How_to_Hard-Crash_Your_Database_at_2-1_Billion_Rows/)**
