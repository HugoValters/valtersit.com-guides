> 📖 **Original article:** [Docker Secrets vs Env Vars: Credential Management](https://www.valtersit.com/guides/docker/docker-secrets-vs-env-vars-credential-management/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

Every second tutorial on the internet shows `docker run -e DB_PASSWORD=...` as if that is a reasonable way to run production software. It is not. It is how credentials end up in GitHub issues, log aggregators, and post-mortem slides. I have spent fifteen years cleaning up after that exact decision. There is a production PostgreSQL password floating around in a public issue tracker right now because one well-meaning engineer ran `docker inspect` and pasted the output into an issue titled "Help debugging connection refused." They thought they were being helpful. They were handing over the master key.

This is not an article about "best practices." It is a field guide to what actually happens when you default to environment variables for credentials: who can read them, why they keep leaking, and how to get Docker secrets deployed before your next incident review. 'It worked in staging' does not count as a security argument. By the end, you will know the mechanics of Docker secrets, the attack vectors you are closing, and the one place where env vars are still acceptable.

---

:::note[TL;DR]
- Environment variables are process configuration, not a credential store. Anyone with root or Docker socket access can dump them.
- Docker secrets use in-memory tmpfs mounts, so credentials are never written to disk and are removed when the container dies.
- Docker secrets are not encryption. They shrink the blast radius with access control and ephemerality, not magic.
- Use Compose secrets for single containers; use Swarm secrets for rolling rotation. `docker run --env` for credentials is indefensible once you know better.
:::

## Prerequisites
- Docker Engine 20.10+ with the Compose plugin
- A Swarm cluster if you want rolling secret rotation without downtime
- `jq` installed on your workstation for inspecting container metadata
- Root or a user with access to the Docker socket for the attack demos

---

## THE HEROKU HANGOVER: HOW ENV VARS BECAME THE DEFAULT

### 1.1 The 12-Factor App Did Not Tell You to Put Passwords in `ENV`

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/docker/docker-secrets-vs-env-vars-credential-management/](https://www.valtersit.com/guides/docker/docker-secrets-vs-env-vars-credential-management/)**
