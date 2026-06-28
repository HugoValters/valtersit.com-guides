> 📖 **Original article:** [ZFS Deduplication: The Fastest Way to Murder Your Server's RAM](https://www.valtersit.com/guides/databases/ZFS-Deduplication-The-Fastest-Way-to-Murder-Your-Servers-RAM/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

I see this all the time in homelab forums and junior sysadmin slack channels. Someone builds a new Proxmox or TrueNAS box, sees the "Deduplication" checkbox, and thinks they’ve discovered a magical cheat code to save 50% of their disk space. They click it, feel like a genius for about an hour, and then wake up the next day to a server that is completely frozen, with IO wait times through the roof.

Congratulations. You just murdered your server’s performance because you wanted to save a few bucks on hard drives. 

ZFS Deduplication is not a "set it and forget it" feature for general-purpose storage. It is a specialized tool that requires a massive, expensive sacrifice in the form of Error Correction Code (ECC) RAM. If you don't have the hardware to back it up, you're not saving space—you're just building a digital brick.

### The Mechanics: The DDT Death Spiral

When you enable deduplication, ZFS has to keep track of every single block written to the pool. It does this using a Deduplication Table (DDT). For every unique block, ZFS stores a cryptographic hash. Before writing any new data, ZFS calculates the hash of the incoming block and checks it against the DDT to see if it already exists.

Here is where the math kills you. The DDT is massive. A common rule of thumb is that you need between 1GB and 5GB of RAM for every 1TB of deduplicated data just to keep the DDT in memory (ARC). 

The second your DDT outgrows your available RAM, it spills over to the physical disks. Now, for every single write operation, ZFS has to perform multiple random reads from the slow-as-hell spinning disks just to check if a block is a duplicate. Your IOPS plummet to zero. Your system enters a "death spiral" where even deleting files takes hours because ZFS has to update the DDT for every block it removes.

### The Fix: Compression Over Deduplication

The Senior Storage Engineer's approach is simple: **Turn it off.** Unless you are running a specialized backup target holding 10,000 identical Windows VM templates, the space savings of deduplication will never justify the performance hit. 

If you want to save space without killing your server, use ZFS Compression. Unlike deduplication, ZFS compression (using `lz4` or `zstd`) is virtually free in terms of CPU overhead and requires zero extra RAM. In many cases, `lz4` actually *increases* your performance because it's faster for the CPU to compress a block than it is for the mechanical disk to write the full, uncompressed data.

### The Code: Inspect and Fix

First, check your pool to see if the Deduplication Table is already choking your system. Look for the "dedup" ratio and the size of the DDT.

```bash
# THE REAL ENGINEER'S WAY (Checking Dedup Stats)

# View the deduplication statistics for your pool
zpool list -D mypool
```

If the `dedup` ratio is less than 2.0x, you are definitely wasting your time and RAM. To stop the bleeding, disable deduplication on the dataset. Note: This will not undeduplicate existing data (you'd have to rewrite it), but it stops new writes from entering the DDT.

```bash
# 1. Disable deduplication immediately
zfs set dedup=off mypool/dataset

# 2. THE FIX: Enable zstd compression instead. 
# It's smarter, faster, and won't crash your kernel.
zfs set compression=zstd mypool/dataset

# 3. Verify the changes
zfs get compression,dedup mypool/dataset
```

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/databases/ZFS-Deduplication-The-Fastest-Way-to-Murder-Your-Servers-RAM/](https://www.valtersit.com/guides/databases/ZFS-Deduplication-The-Fastest-Way-to-Murder-Your-Servers-RAM/)**
