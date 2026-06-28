> 📖 **Original article:** [MySQL's Fake UTF-8: Why Your Database is Truncating Emojis and Text](https://www.valtersit.com/guides/databases/MySQLS-Fake-UTF-8-Why-Your-Database-is-Truncating-Emojis-and-Text/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

I’ve lost count of how many times I’ve seen a junior developer or a frazzled DevOps engineer staring at a stack trace, trying to figure out why their Node.js or Python app just threw a 500 Server Error. They dig into the logs and find a "General error: 1366 Incorrect string value" for a field that should be a simple comment. 

The culprit? A user typed a smiling poop emoji. Or a specific Chinese character. Or a mathematical symbol. 

In almost every other software ecosystem, UTF-8 just works. In MySQL, however, the developers made a historical decision that has wasted thousands of collective engineering hours: they named a 3-byte encoding `utf8`. 

If you are using `utf8` in MySQL, you are actually using `utf8mb3`. And in the world of modern text, that is a data-loss disaster waiting to happen.

### The Mechanics: The 3-Byte Lie

Standard UTF-8 is a variable-length encoding that uses between one and four bytes per character. This covers everything from basic English ASCII to complex emojis and ancient scripts. 

When MySQL first implemented UTF-8 back in the early 2000s, they decided to cap it at 3 bytes to save space and improve performance on the hardware of the time. They called this character set `utf8`. 

The problem is that a significant portion of the modern web—specifically emojis and certain CJK characters—requires exactly 4 bytes. When MySQL encounters a 4-byte character and your column is set to the legacy `utf8`, it doesn't just "handle it." Depending on your SQL mode, it will either:
1. **Silently truncate the data:** It will save everything up to the 4-byte character and throw the rest away. 
2. **Throw a fatal error:** Your application crashes because the database refuses to store the "invalid" string.

This isn't just about emojis. It’s about data integrity. If your database can't handle the full UTF-8 specification, it isn't a reliable data store.

### The Fix: UTF8MB4 or Bust

The Senior DBA fix is non-negotiable: you must always, without exception, use `utf8mb4`. 

`utf8mb4` is the "Most Bytes 4" implementation that actually follows the UTF-8 standard. If you are on MySQL 8.0, you should be using `utf8mb4` with the `utf8mb4_0900_ai_ci` collation (which is faster and more accurate for modern sorting). If you’re still on 5.7 or using MariaDB, `utf8mb4_unicode_ci` is your baseline.

### The Code: The Migration Path

Updating a production database isn't as simple as clicking a button. You have to convert the database, the tables, and the individual columns to ensure the change actually propagates.

```sql
-- THE REAL ENGINEER'S WAY (True UTF-8 Migration)

-- 1. Update the Database defaults
ALTER DATABASE my_database 
CHARACTER SET = utf8mb4 
COLLATE = utf8mb4_0900_ai_ci;

-- 2. Update the Table
-- This will rebuild the table, so be careful on multi-gigabyte tables
ALTER TABLE my_table 
CONVERT TO CHARACTER SET utf8mb4 
COLLATE utf8mb4_0900_ai_ci;

-- 3. Update individual columns (if CONVERT TO didn't catch them)
ALTER TABLE my_table 
MODIFY my_column TEXT 
CHARACTER SET utf8mb4 
COLLATE utf8mb4_0900_ai_ci;

-- 4. Check your work
SELECT 
    TABLE_NAME, 
    COLUMN_NAME, 
    CHARACTER_SET_NAME, 
    COLLATION_NAME 
FROM information_schema.COLUMNS 
WHERE TABLE_SCHEMA = 'my_database' 
AND CHARACTER_SET_NAME = 'utf8';
```

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/databases/MySQLS-Fake-UTF-8-Why-Your-Database-is-Truncating-Emojis-and-Text/](https://www.valtersit.com/guides/databases/MySQLS-Fake-UTF-8-Why-Your-Database-is-Truncating-Emojis-and-Text/)**
