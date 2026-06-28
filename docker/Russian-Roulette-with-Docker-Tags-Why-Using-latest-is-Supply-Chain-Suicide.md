> 📖 **Original article:** [Russian Roulette with Docker Tags: Why Using :latest is Supply Chain Suicide](https://www.valtersit.com/guides/docker/Russian-Roulette-with-Docker-Tags-Why-Using-latest-is-Supply-Chain-Suicide/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

If I see `image: node:latest` in a production manifest during an architecture review, the pull request is instantly rejected. 

Relying on the `:latest` tag—or frankly, any mutable tag—is the infrastructure equivalent of playing Russian roulette. It is not a version. It is a moving target, an arbitrary pointer that someone else controls. You deploy on Friday, and everything works perfectly. On Monday morning, your application is either randomly throwing 500 errors because a major language update was pushed, or worse, it's silently exfiltrating your environment variables to a command-and-control server. And because you didn't pin your dependencies, you have absolutely zero historical record of what actually changed.

### The Mechanics: Supply Chain Poisoning

Docker tags are just aliases. They are not immutable. A repository maintainer can overwrite the `latest` or `v2` tag ten times a day if they want to. 

This makes your infrastructure highly vulnerable to supply chain poisoning. Let's say you use a popular third-party image for your API gateway or background worker. An attacker compromises the maintainer's Docker Hub account via credential stuffing or a stolen session token. The attacker builds a malicious version of the image containing a backdoor, and forcefully pushes it to the `:latest` tag.

If your CI/CD pipeline triggers a deployment, or if you run a tool like Watchtower that automatically pulls updates, your servers will blindly download the backdoored image and restart your containers. You just deployed malware into production, completely bypassing your own firewalls and security scans, simply because you trusted a mutable string.

### The Fix: Immutable Infrastructure via Digests

The absolute, non-negotiable rule of a Senior DevSecOps engineer is that production infrastructure must be immutable. 

To achieve this in Docker, you must stop pulling by tags and start pulling by SHA256 digests. A cryptographic digest guarantees that you are downloading the exact, byte-for-byte image you audited and tested in staging. If an attacker compromises the upstream repository and overwrites the image, the SHA256 hash changes. Your pipeline will attempt to pull the specific hash you defined, the registry will serve the untampered version (if it still exists) or fail safely. The attack is neutralized before the code even reaches your host.

### The Code

Here is the lazy, vulnerable configuration that guarantees an eventual outage or breach:

```yaml
# THE BAD WAY (Mutable and unpredictable)
services:
  frontend:
    # What version is this? Nobody knows. It changes daily.
    image: nginx:latest
    ports:
      - "80:80"
```

Here is the bulletproof, cryptographically verifiable approach. Notice we keep the human-readable tag for context, but the Docker daemon strictly enforces the SHA256 hash.

```yaml
# THE REAL ENGINEER'S WAY (Cryptographically pinned)
services:
  frontend:
    # Pinned to a specific, immutable SHA256 digest. 
    # (Visually tracking nginx:1.25.3-alpine)
    image: nginx:1.25.3-alpine@sha256:6e0172e0faaf5d376d8eab6cce9a71b1273946af9fb6d2216518db6cc3b17135
    ports:
      - "80:80"
```

Finding the digest is trivial. Run `docker inspect  | grep RepoDigests` on a known-good image, or grab it straight from the container registry UI. 

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/docker/Russian-Roulette-with-Docker-Tags-Why-Using-latest-is-Supply-Chain-Suicide/](https://www.valtersit.com/guides/docker/Russian-Roulette-with-Docker-Tags-Why-Using-latest-is-Supply-Chain-Suicide/)**
