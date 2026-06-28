> 📖 **Original article:** [Consumer SSDs on ZFS/Databases: Why Your Drives Are Dying in 6 Months](https://www.valtersit.com/guides/databases/Consumer-SSDs-on-ZFS_Databases-Why-Your-Drives-Are-Dying-in-6-Months/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

I love the smell of burning NAND in the morning. Usually, it comes from a homelabber or a junior sysadmin who thought they were being "cost-effective" by buying four 2TB Samsung QVO or cheap Kingston drives for a Proxmox ZFS mirror. 

They load up a busy PostgreSQL database or a few dozen Docker containers, and for the first three weeks, everything is great. Then, the latency spikes. Suddenly, their "lightning-fast" SSDs are writing at 10MB/s—slower than a rusty spinning hard drive from 2008. By month six, the ZFS pool is degraded, and the SMART logs show the drives have 0% life remaining. 

Congratulations. You just learned the expensive way that "Gaming" SSDs have absolutely zero business being near a Copy-on-Write (CoW) filesystem or a production database.

### The Mechanics: The CoW Hammer and Write Amplification

ZFS is a Copy-on-Write filesystem. It never overwrites data in place; it always writes new data to a new block and updates the pointers. Combine this with a database like PostgreSQL or MariaDB, which constantly performs small, synchronous writes (the WAL or transaction logs), and you have a recipe for **Write Amplification**. 

Every 4KB write from your database can turn into much more data being physically written to the NAND due to ZFS record sizes, metadata updates, and SSD garbage collection. 

Consumer SSDs (especially QVO/QLC) are built for "bursty" workloads—opening Chrome, loading a game, or booting Windows. They use a tiny SLC (Single-Level Cell) cache to hide their true, pathetic speeds. The "Q" in QVO effectively stands for "Quit," because once that SLC cache fills up during a sustained database write, the drive has to write directly to the slow QLC NAND while simultaneously clearing out its own cache. Your IOPS plummet, and your wear-leveling count skywrites your drive's funeral date.

### The Missing Piece: Power Loss Protection (PLP)

The real killer, however, is the lack of Power Loss Protection (PLP). 

When a database sends a `fsync` command, it is telling the hardware: "Do not tell me this is finished until it is safely written to non-volatile storage." 

Consumer drives don't have the capacitors to flush their DRAM cache to NAND during a power failure. To prevent data corruption, they have to be extremely conservative. Every time ZFS or Postgres demands a synchronous write, the consumer drive stalls the entire bus to ensure the data is actually on the chips. Enterprise drives (like the Intel Optane or the Samsung PM/SM series) have physical capacitors. They can "lie" to the OS, say the write is finished instantly because they know they have enough stored juice to flush the cache even if the cord is pulled. 

This is why an "older" enterprise drive with lower "on-paper" sequential speeds will absolutely smoke a brand-new "990 Pro" in a real database workload.

### The Fix: Enterprise Gear and ZIL/SLOG

The Senior Storage Engineer's fix is simple: **Buy the right tool for the job.** If you are running a database or ZFS, you need enterprise-grade SSDs with high TBW (Terabytes Written) ratings and integrated PLP. Look for the Intel/Solidigm D7 series, Samsung PM893/PM1733, or Micron 7450. 

If you're stuck with slow main storage, at least add a dedicated **SLOG (ZFS Intent Log)** device. A small, high-endurance Intel Optane drive (like the P1600X) used as a SLOG will soak up those synchronous writes, protect your data, and let your main pool breathe.

### The Code: Checking the Damage

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/databases/Consumer-SSDs-on-ZFS_Databases-Why-Your-Drives-Are-Dying-in-6-Months/](https://www.valtersit.com/guides/databases/Consumer-SSDs-on-ZFS_Databases-Why-Your-Drives-Are-Dying-in-6-Months/)**
