> 📖 **Original article:** [SQLite in Docker Containers: A Masterclass in Data Deletion](https://www.valtersit.com/guides/databases/SQLite-in-Docker-Containers-A-Masterclass-in-Data-Deletion/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

I see it at least once a month. A junior developer builds a sleek little Python or Go app, decides that a full PostgreSQL instance is "overkill," and bundles everything with SQLite. They build the image, push it to production, and for a week, everything is glorious. 

Then comes the first update. 

The developer runs `docker stop`, `docker rm`, and pulls the new image. They start the new container and—to their absolute horror—find a completely empty database. Six days of production customer data, gone. No backups, no recovery, just an empty `app.db` file and a very long, very painful meeting with the CTO.

Congratulations. You just learned the hard way that Docker containers are designed to be disposable, not persistent.

### The Mechanics: The Ephemeral Layer Trap

Docker uses what's called a Union File System (usually `overlay2` on modern Linux). When you start a container, Docker takes your read-only image and slaps a thin, writable layer on top of it. This is where your application lives, breathes, and—crucially—writes its data.

The problem is that this writable layer is tied to the specific lifecycle of that individual container instance. 

When you stop and remove a container, Docker ruthlessly deletes that writable layer. It doesn't care that your `app.db` was sitting in `/app/data/`. To Docker, that file was just temporary noise. If you didn't explicitly map that path to a persistent location outside the container, your data lived and died with that specific process. 

### The Fix: Persistent Volumes and WAL Mode

The Senior DevOps approach is simple: **Nothing of value stays inside the container.** You must use Docker Volumes or Bind Mounts. This maps a directory on your host machine (or a managed Docker volume) directly into the container's filesystem. When the container dies, the data stays safely on the host's disk, waiting for the next container to pick it up.

Furthermore, if you are running SQLite in a containerized environment, you should almost always enable **WAL (Write-Ahead Logging) mode**. By default, SQLite is quite "chatty" with the filesystem, which can cause performance bottlenecks on some Docker storage drivers. WAL mode moves the concurrency handling to a separate log file, allowing multiple readers and one writer without locking the entire database—essential when your app starts scaling or handling concurrent API requests.

### The Code: From Fatal to Bulletproof

Here is the "one-way ticket to unemployment" command that many beginners use:

```bash
# THE BAD WAY (Data will be deleted on next update)
docker run -d --name my-app my-app-image:latest
```

And here is the Senior Engineer's approach. We use a `docker-compose.yml` to define persistent storage and a simple SQL command to optimize the database for a high-concurrency container environment.

```yaml
# THE REAL ENGINEER'S WAY (Persistent and Optimized)
services:
  webapp:
    image: my-app:v1.2.4
    volumes:
      # Map a persistent host directory to the container's data path
      - /opt/my-app/data:/app/data
    environment:
      - DATABASE_URL=/app/data/prod.db
    restart: unless-stopped
```

Once your application connects to that database, run these PRAGMA commands to ensure it doesn't choke under load:

```sql
-- SQLite Optimization for Production
-- Run this once when your app initializes the connection

-- 1. THE FIX: Enable Write-Ahead Logging for concurrency
PRAGMA journal_mode = WAL;
```

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/databases/SQLite-in-Docker-Containers-A-Masterclass-in-Data-Deletion/](https://www.valtersit.com/guides/databases/SQLite-in-Docker-Containers-A-Masterclass-in-Data-Deletion/)**
