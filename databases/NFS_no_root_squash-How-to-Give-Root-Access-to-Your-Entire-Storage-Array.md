> 📖 **Original article:** [NFS no_root_squash: How to Give Root Access to Your Entire Storage Array](https://www.valtersit.com/guides/databases/NFS_no_root_squash-How-to-Give-Root-Access-to-Your-Entire-Storage-Array/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

I’ve seen it a thousand times in the last fifteen years. A sysadmin is trying to set up a shared volume for a cluster of Docker containers or a legacy web-hosting environment. They map the mount, try to write a file from the client, and—surprise—they get a `Permission Denied` error. 

Instead of doing the hard work of aligning UIDs and GIDs across their infrastructure or investigating why their application user can’t write to the export, they go to the storage server, open `/etc/exports`, and append `no_root_squash` to the line. They run `exportfs -ar`, the permission error disappears, and they walk away feeling like they’ve "solved" the problem.

What they’ve actually done is hand over the keys to the entire storage array—and likely the entire Linux environment—to anyone who manages to get a shell on the client machine. 

In the world of hardened Linux administration, `no_root_squash` is not a configuration flag; it is a giant, glowing "ENTER HERE" sign for attackers.

### The Mechanics: What root_squash Actually Does

To understand why `no_root_squash` is so dangerous, you have to understand the fundamental (and somewhat naive) trust model of the Network File System (NFS) protocol. 

NFS was designed in an era where we believed the network was a friendly place. When a client sends a request to an NFS server to read or write a file, it identifies the user by their **UID (User ID)**. If the user on the client is `uid=1001`, the NFS server checks if `uid=1001` has permissions on the local filesystem. 

#### The Root Problem
The problem arises when the user on the client is `root` (`uid=0`). By default, the NFS server knows that allowing a remote `root` user to have `root` privileges on its local disk is a recipe for disaster. 

This is where `root_squash` comes in. It is a security feature enabled by default on almost every modern NFS implementation. When `root_squash` is active, any request coming from a remote `root` user is automatically "squashed" (mapped) to a low-privileged user, typically `nobody` or `nfsnobody`. This ensures that even if you have root access on your laptop or your web server, you can’t just start deleting system files or changing permissions on the storage server.

### The Mechanics of the Exploit: Escalation via setuid

When you disable this protection by using `no_root_squash`, you create a direct path for local privilege escalation (LPE). 

Let’s walk through the "Senior Sysadmin’s Nightmare" scenario:

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/databases/NFS_no_root_squash-How-to-Give-Root-Access-to-Your-Entire-Storage-Array/](https://www.valtersit.com/guides/databases/NFS_no_root_squash-How-to-Give-Root-Access-to-Your-Entire-Storage-Array/)**
