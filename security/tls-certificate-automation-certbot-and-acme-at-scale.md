> 📖 **Original article:** [TLS Certificate Automation: Certbot and ACME at Scale](https://www.valtersit.com/guides/security/tls-certificate-automation-certbot-and-acme-at-scale/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

03:14. `payments-api` starts returning 502s. Four hundred and forty-seven seconds later someone notices the dashboard is red. Forty-seven minutes later, someone finds the postmortem bullet nobody wants to write: the "automation" was a cron job on a host that had been decommissioned eight months earlier. The cron job didn't fail. It was never going to run again. Nobody owned the cert, nobody monitored it, and the first signal was customer-facing downtime.

Certificate management is the cheapest layer of your stack to automate and the most expensive one to get wrong. It is also the layer where "we renew manually" stopped being a defensible statement somewhere around 2019 and became, in 2026, a statement about your on-call rotation rather than your security posture. If a leaf certificate expires and your service degrades, that is not bad luck — it is an unowned dependency.

This guide is for engineers running real traffic: sysadmins, SREs, and platform teams who already have Certbot installed and want to know why it keeps biting them anyway. You'll get ACME internals, Certbot patterns that survive contact with production, challenge-selection tradeoffs, renewal hook discipline, hardening, and the rate limits that will ruin your Thursday. What you won't get is a "five clicks in a vendor dashboard" walkthrough — that content exists, it's just not useful to anyone operating at scale.

:::note[TL;DR]
- Automating issuance is easy; automating **deployment** is where outages live. If your reload path isn't idempotent and monitored, you haven't automated anything.
- The CAB Forum baseline keeps shrinking certificate lifetimes — plan for 200 days now, 100 days in 2027, and short-lived certificates sooner than you think.
- HTTP-01 is the default for a reason; DNS-01 is the only path to wildcards and it turns your DNS API key into a certificate-issuance credential. Scope it accordingly.
- Trust the packaged `certbot.timer`, write a real deploy hook, and monitor the certificate **as served**, not the file on disk.
:::

## Prerequisites

- Certbot 2.x (or newer) installed from your distro, snap, or the official container image — check the docs for the current release line
- Root or sudo on the host that terminates TLS
- Port 80 reachable from the public internet if you're using HTTP-01 (including over IPv6)
- A DNS provider with a supported API plugin if you need wildcards
- `openssl` and `jq` for the inventory and monitoring snippets below

## 1. Certificate automation is the layer you actually skipped

### The 90-day treadmill is over

The CAB Forum ballot SC-081 phases down the maximum lifetime of publicly-trusted certificates: 200 days from March 2026, 100 days from March 2027, and 47 days from March 2029. Let's Encrypt has been piloting short-lived (roughly six-day) certificates for years and is pushing the ecosystem toward them. Check the current Baseline Requirements before you bake a number into an SLA — the schedule has been revised before and will be again.

What that fixes: revocation. Revocation has been broken since OCSP was invented. A six-day certificate doesn't need to be revoked reliably because it expires before anyone can care. What it breaks: every assumption built on a 60-day renewal window, every cron job with a monthly cadence, and every monitoring rule that alerts at 30 days remaining.

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/security/tls-certificate-automation-certbot-and-acme-at-scale/](https://www.valtersit.com/guides/security/tls-certificate-automation-certbot-and-acme-at-scale/)**
