> 📖 **Original article:** [Blue-Green Deployments Without Kubernetes: Zero Downtime Is Boring (and That's the Point)](https://www.valtersit.com/guides/ci_cd/blue-green-deployments-without-kubernetes-zero-downtime-is-boring-and-that/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

---

## Before We Start: The 2 AM Wake-Up Call

It's 2:14 AM on a Tuesday. Your phone buzzes. The on-call rotation page says `checkout-api-03` is returning HTTP 502s. You SSH in, heart sinking, and see it: the deploy script ran at 2:00 AM, systemd sent SIGTERM to the old app process, and the app — which never implemented a graceful drain handler — died mid-request. Two hundred keep-alive connections were still bound to it. Nginx had already marked it down and routed traffic to the new instance, but the damage was done: failed checkouts, retry storms, and a Slack channel full of "is anyone looking at this?"

The worst part? The deploy **was** successful. The new version was healthy. The mechanism was the failure.

This guide is for senior engineers and sysadmins maintaining production systems that don't run Kubernetes — bare metal, plain VMs, or a handful of cloud instances. You deserve clean cutovers, and you don't need a cluster to get them. After reading this, you'll be able to implement true blue-green deployments using systemd, Nginx or HAProxy, and scripts boring enough that you won't dread running them at 2 AM — because you won't have to.

:::note[TL;DR]

- Rolling deploys are only zero-downtime if every layer implements graceful drain. Most don't. That's why you get 502s.
- Blue-green is a binary traffic switch between two fully-provisioned environments. If your state lives on the app instance, you can't do it. Fix state first.
- systemd gives you everything you need to manage release versions as immutable units — no orchestrator required.
- The proxy layer is where cutover happens. HAProxy beats Nginx for this job because it can switch backends without a reload that risks dropping in-flight connections.
- Database migrations are the real risk. Code cutover is easy; schema cutover is where careers go to die.

:::

## Prerequisites

- A Linux VM or bare-metal server running systemd (any modern distro — Debian 12, Ubuntu 22.04+, RHEL 9+)
- Nginx or HAProxy already installed and serving traffic
- A stashed release artifact: tar.gz built and tested in CI, not a git checkout
- Root access and the willingness to read `man systemd.unit` before arguing with me
- The "old" version of your app currently in production

## Section 1 — Why "We Do Rolling Deploys" Is a Four-Letter Word

### 1.1 The Connection-Draining Myth

Rolling deployments are the default answer for "zero-downtime releases" because they _sound_ reasonable: replace instances one at a time, and traffic shifts to healthy ones. The dirty secret is that rolling deploys only work if every layer — the proxy, the process manager, and the application itself — cooperates on connection draining. In my experience, at least one of those three is broken**.

Here's the "bad" version first so you feel the pain before the fix:

```nginx
# nginx.conf — the "cargo-cult" upstream block
upstream app_backend {
    server 127.0.0.1:8001;  # old version — "we'll swap this later"
    server 127.0.0.1:8002;  # new version, just added
}
```

Then what happens during the deploy:

```bash
# deploy.sh — the "I yanked the upstream" pattern
# Step 1: Mark old server as down in nginx
sed -i 's/server 127.0.0.1:8001;/server 127.0.0.1:8001 down;/' /etc/nginx/nginx.conf
nginx -s reload

# Step 2: Kill the old app
systemctl stop myapp-old
```

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/ci_cd/blue-green-deployments-without-kubernetes-zero-downtime-is-boring-and-that/](https://www.valtersit.com/guides/ci_cd/blue-green-deployments-without-kubernetes-zero-downtime-is-boring-and-that/)**
