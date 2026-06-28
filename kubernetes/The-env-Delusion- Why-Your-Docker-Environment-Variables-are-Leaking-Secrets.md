> 📖 **Original article:** [The .env Delusion: Why Your Docker Environment Variables are Leaking Secrets](https://www.valtersit.com/guides/kubernetes/The-env-Delusion- Why-Your-Docker-Environment-Variables-are-Leaking-Secrets/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

There is a persistent, industry-wide delusion that a `.env` file is somehow a secure vault for your infrastructure's most sensitive credentials. 

I don't care how many Medium tutorials told you to map `DB_PASSWORD=SuperSecret` into your `docker-compose.yml`. I don't care if you excluded `.env` from your `.gitignore`. If you are passing passwords, API keys, or TLS certificates into your containers via environment variables, you are committing security malpractice. 

And if you are putting `ENV API_KEY=xyz` directly into your `Dockerfile`? You might as well just post your company's credit card on Reddit. Because that secret is now permanently baked into the image layers, available to literally anyone who runs `docker history `.

Environment variables were designed for configuration flags like `NODE_ENV=production` or `DEBUG=true`. They were never designed to hold the keys to the kingdom. 

### The Mechanics: How Your Secrets Leak

Let's break down exactly how an attacker reads your environment variables once they get even a tiny foothold in your system.

First, Docker treats runtime environment variables as metadata, not secrets. Anyone with access to the Docker daemon (or a compromised CI/CD pipeline) can simply run:
`docker inspect `
Scroll down a bit, and there is your production database password, sitting in plain text in the JSON output. 

But it gets worse at the Linux kernel level. Environment variables are attached to the process tree. If an attacker manages to get a limited shell on the host machine, or compromises another container running with shared PIDs, they don't even need Docker access. They just need to find your app's Process ID and run:
`cat /proc/[pid]/environ`

Boom. A null-byte separated list of every single "secret" your app was started with. 

### The Fix: File-Based Secrets

Senior security engineers don't use environment variables for sensitive data. We use files. Specifically, we use in-memory files (tmpfs) that never touch the physical disk and disappear the second the container stops.

Docker has a built-in `secrets` mechanism that handles this perfectly, even in basic Docker Compose setups without Swarm or Kubernetes. 

When you use Docker secrets, the credential is mounted as a read-only file directly into the container's memory at `/run/secrets/`. The fix requires a slight change in your application code: instead of reading `os.environ['DB_PASSWORD']`, your app needs to open the file `/run/secrets/db_password` and read the string. 

### The Code

Here is the amateur-hour configuration that will fail a basic security audit:

```yaml
# THE BAD WAY (Leaking everywhere)
# Passwords exposed in docker inspect and /proc/
services:
  database:
    image: postgres:15
    environment:
      - POSTGRES_USER=admin
      - POSTGRES_PASSWORD=SuperSecretAdminPassword123
```

Here is the hardened, DevSecOps-approved approach. Notice that we define the secret at the bottom, point it to a local file on the host, and inject it into the service. 

```yaml
# THE REAL ENGINEER'S WAY (File-based Secrets)
# Secret is mounted directly to /run/secrets/db_password in RAM
services:
  database:
    image: postgres:15
    # The official Postgres image knows to look for files ending in _FILE
    environment:
      - POSTGRES_USER=admin
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password
    secrets:
      - db_password
```

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/kubernetes/The-env-Delusion- Why-Your-Docker-Environment-Variables-are-Leaking-Secrets/](https://www.valtersit.com/guides/kubernetes/The-env-Delusion- Why-Your-Docker-Environment-Variables-are-Leaking-Secrets/)**
