> 📖 **Original article:** [Schrödinger's Backup: If You Haven't Tested a Restore, You Don't Have a Backup](https://www.valtersit.com/guides/security/Schrodingers-Backup-If-You-Havent-Tested-a-Restore-You-Dont-Have-a-Backup/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

Let me introduce you to Schrödinger's Backup: the condition of your corporate data is simultaneously pristine and completely destroyed until you actually attempt a bare-metal restore. 

I have sat in entirely too many incident response war rooms where a company gets hit by LockBit or BlackCat. The CEO is panicking, but the IT Director smugly crosses his arms and says, "Don't worry, we use Veeam. We'll just restore from last night."

Ten minutes later, the blood completely drains from the IT Director's face. He realizes that the backup server was joined to the exact same Active Directory domain that just got compromised. The attacker used their stolen Domain Admin credentials to log into the backup repository and encrypted the backups, too. 

You didn't have a disaster recovery plan. You just had a really expensive secondary target.

### The Mechanics: Destroying the Safety Net

Modern ransomware is not a dumb script that just blindly encrypts `C:\Users`. It is a human-operated, highly targeted "living off the land" operation. 

The absolute *first* thing a competent threat actor does after escalating privileges is hunt down your safety net. They query Active Directory for servers with "backup", "veeam", "rubrik", or "datto" in the hostname. They log into your hypervisors. They run `vssadmin delete shadows /all /quiet` to nuke your local Volume Shadow Copies. They log into your network-attached storage (NAS) and format the volume. 

Only *after* they have systematically dismantled your ability to recover do they push the button to encrypt your production environment. If your backup system relies on the same authentication perimeter (Active Directory, shared local admin passwords) as your production system, it will fall the second your domain falls.

### The Fix: Immutability and the 3-2-1-1 Rule

The old 3-2-1 backup rule (3 copies, 2 media, 1 offsite) is dead. You need 3-2-1-1: the final "1" stands for **Immutable**.

Immutable storage is Write-Once-Read-Many (WORM). Once the data is written, it is physically and cryptographically impossible to delete, modify, or encrypt it for a specified retention period. It doesn't matter if the attacker gets Domain Admin. It doesn't matter if the attacker gets the literal AWS root credentials. The storage API will simply reject any delete or modify requests until the timer expires. 

The modern Senior Engineer approach is to push your secondary backups to a cloud bucket with strict Object Lock enabled in Compliance Mode. 

### The Code & Config

Here is how you actually build an immutable vault. This Terraform snippet creates an AWS S3 bucket and locks it down with a 30-day compliance retention policy.

```hcl
# THE REAL ENGINEER'S WAY (Immutable S3 Storage)
# If an attacker compromises your entire datacenter and AWS keys, 
# they STILL cannot delete these backups for 30 days.

resource "aws_s3_bucket" "immutable_backups" {
  bucket = "corp-airgapped-backups-2026"
}

# 1. Enable Object Lock (Must be done at bucket creation)
resource "aws_s3_bucket_versioning" "backup_versioning" {
  bucket = aws_s3_bucket.immutable_backups.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_object_lock_configuration" "backup_lock" {
  bucket = aws_s3_bucket.immutable_backups.id
```

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/security/Schrodingers-Backup-If-You-Havent-Tested-a-Restore-You-Dont-Have-a-Backup/](https://www.valtersit.com/guides/security/Schrodingers-Backup-If-You-Havent-Tested-a-Restore-You-Dont-Have-a-Backup/)**
