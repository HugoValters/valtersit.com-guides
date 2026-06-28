> 📖 **Original article:** [Read-Only Roles: How to Stop the Accidental DROP TABLE](https://www.valtersit.com/guides/databases/Read-Only_Roles-How_to_Stop_the_Accidental_DROP_TABLE/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

I’ve heard the same excuse in a dozen different post-mortem meetings: "I was just running a quick SELECT query to check the revenue numbers, I don't know how the table disappeared." 

Here is how it happens: A developer or data analyst connects to the production database using the `postgres` or `root` superuser account because they "didn't want to deal with permission errors." They open a GUI tool like DBeaver, DataGrip, or pgAdmin. They have a scratchpad with five different queries on it. They think they’ve highlighted the `SELECT` statement, but their mouse slips, or they hit a keyboard shortcut, and they accidentally execute the `DROP TABLE customers CASCADE;` line sitting right below it. 

Because they are connected as a superuser, the database doesn't ask for confirmation. It doesn't check if they have permission. It just obeys. Within 200 milliseconds, your primary customer table—and all its foreign key dependencies—is gone. 

If you are allowing humans to run routine queries using superuser credentials, you aren't practicing database administration; you are practicing a high-stakes form of digital Russian Roulette. 

### The Mechanics: The Superuser Delusion

To understand why this is a systemic failure, you have to understand the **Principle of Least Privilege (PoLP)**. In a secure environment, a user should have the absolute minimum level of access required to perform their job, and not a single bit more. 

#### The Superuser Bypass
The `postgres` user (or `root` in MySQL) is a special entity. It is designed for administrative tasks: vacuuming, WAL management, user creation, and schema migrations. Superusers bypass every single permission check in the database engine. They are the "God Mode" of your data. 

When you connect as a superuser, you have the power to execute Data Definition Language (DDL) commands. DDL commands—like `DROP`, `TRUNCATE`, and `ALTER`—alter the physical structure of the database. Most developers only ever need Data Manipulation Language (DML) rights—`SELECT`, `INSERT`, `UPDATE`—and even then, only on specific tables. For an analyst just "checking numbers," even `INSERT` and `UPDATE` are unnecessary risks. 

#### The GUI Threat Multiplier
Modern SQL editors make this worse. They often allow multiple tabs, "run all" buttons, and context menus that include "Delete Table" just two pixels away from "Refresh Data." If your credentials allow a `DROP` command, the GUI will happily send it. A read-only role acts as a physical safety on the trigger; even if the command is sent, the database engine rejects it with a `Permission Denied` error. 

### The Mechanics of the Fix: Group Roles and Default Privileges

The Senior DBA approach is to create a hierarchy of roles. You don't grant permissions to individuals; you grant permissions to **Group Roles**, and then you make individual users members of those groups.

#### 1. The Schema Usage Trap
In PostgreSQL, simply granting `SELECT` on a table isn't enough. You must also grant `USAGE` on the schema (e.g., `public`) that contains the table. Without `USAGE` on the schema, the user can't even "see" the objects inside it to query them.

#### 2. The "New Table" Gotcha
A common mistake is running `GRANT SELECT ON ALL TABLES`. This works perfectly for every table that exists *right now*. However, the second a developer runs a migration and creates a `new_orders` table tomorrow, your read-only user will get a `Permission Denied` error. 

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/databases/Read-Only_Roles-How_to_Stop_the_Accidental_DROP_TABLE/](https://www.valtersit.com/guides/databases/Read-Only_Roles-How_to_Stop_the_Accidental_DROP_TABLE/)**
