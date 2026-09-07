> 📖 **Original article:** [Blue-Green Deployments: Zero-Downtime Without Kubernetes](https://www.valtersit.com/guides/ci_cd/blue-green-deployments-zero-downtime-without-kubernetes/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

You are at 2 AM, SSHed into a VM that invoices half your company's customers, and you just ran `git pull` in `/opt/app` followed by `systemctl restart myapp`. The app comes back. The boss's phone starts buzzing anyway — 17 stuck transactions, a `502` window, and one very loud customer. You spend the next hour explaining that "deploys are risky." They aren't risky. Your deploy method is risky.

I stopped avoiding this exact problem when I realized the industry was gaslighting me: zero-downtime releases do not require Kubernetes, a service mesh, or a 17-page runbook. I was doing this in 2008 with `rsync`, an HAProxy reload, and a symlink that cost less brainpower than a cron job. This piece is for the senior engineer who still owns actual VMs, who wants releases measured in milliseconds, not in "let's drain the node and hope."

After you finish, you will have a production-grade, Nginx-centric blue-green deployment script, know exactly how to roll back in two seconds, and understand why the database is the only place you are allowed to be scared.

:::note[TL;DR]
- Kubernetes is a control plane, not a deployment strategy. You don't need it for simple zero-downtime releases.
- Blue-green means two static versions behind a reverse proxy and an atomic symlink flip.
- The entire deploy is: rsync to the inactive side, smoke test, flip symlink, reload Nginx.
- Rolling back is re-pointing the symlink and reloading — never a git revert in production.
- It only works if your app is stateless and your database migrations tolerate two live code versions.
:::

## Prerequisites

Before we start, you need the following on a Linux host (systemd, not SysV init — no one should be writing init scripts in 2026):

- Nginx 1.20+ (or HAProxy 2.x; examples here use Nginx because it is the lingua franca).
- A reverse proxy listening on port 443 and terminating TLS. This is non-negotiable.
- systemd service unit for your app.
- A Linux user `deploy` in group `deploy`, with write access to `/opt/app/releases`.
- A CI/CD system that can push an artifact to the VM over SSH.

If you do not have these, stop reading and fix that first. Good engineers do not hand a web framework the responsibility for routing traffic.

## Why Your Boss Is Wrong — Or, The Kubernetes Fallacy

The phrase "we need Kubernetes for zero-downtime deployments" is how engineering teams launder a desire to look modern into a six-month infrastructure project. Let's be precise: zero-downtime is a property of *traffic routing*, not of *orchestration*. Orchestration manages containers, nodes, and a control plane that can — and will — eat your cluster if you ignore certificate rotation, CNI configuration, and API server audit logs.

In 2008, people ran blue-green deployments on a single VM with two filesystem paths and a reverse proxy. You can still do that in 2026, and the philosophical simplicity is a security control. Every API server you run is an IAM policy you must write, a firewall rule you must maintain, and a target you must patch. "Control plane" should trigger your fight-or-flight response, not comfort.

Kubernetes is justified when you need horizontal autoscaling, self-healing, multi-tenant isolation, or you operate at a scale where a lost node is business as usual. For 90 percent of startups running a CRUD app, it's a tax. Your app needs two directories and a reload signal, not a `kube-apiserver`.

### The Cost of Complexity

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/ci_cd/blue-green-deployments-zero-downtime-without-kubernetes/](https://www.valtersit.com/guides/ci_cd/blue-green-deployments-zero-downtime-without-kubernetes/)**
