> 📖 **Original article:** [Self-Hosted Email with Postfix and Dovecot: Honest Sysadmin's Guide](https://www.valtersit.com/guides/hosting/self-hosted-email-with-postfix-and-dovecot-honest-sysadmin/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

I spent three days debugging a silent email blackhole. SPF, DKIM, DMARC all passing. Logs clean. Yet every message to Outlook.com vanished. The culprit? My PTR record pointed to `mail.example.com` but the reverse DNS zone for my IP had a trailing space in the hostname. A single invisible character cost me a week of reputation with Microsoft’s delivery team.

Self-hosting email is not a weekend project. It’s a career‑long commitment to DNS hygiene, reputation management, and knowing that one misconfiguration can land your IP on every RBL in existence. This guide is for the sysadmin who already runs a mail server and is tired of “just works” tutorials that omit the hard parts. By the end, you will have a production‑hardened Postfix + Dovecot setup—and the war stories to know why every decision matters.

:::note[TL;DR]
- Self‑hosted email is a ton of work; you must own a dedicated IP with a clean reputation.
- SPF, DKIM, and DMARC are non‑negotiable—skip them and your email will be spam.
- Use Maildir, not mbox. Use virtual users, not system accounts.
- TLS everywhere: require it for inbound and outbound. No exceptions.
- Monitor logs with pflogsumm and automate certificate renewal—your Saturday nights depend on it.
:::

## Prerequisites
- A Linux server (Debian 12 or RHEL 9) with root access and a public IP—not a shared residential IP.
- A domain name with DNS control. You’ll add A, MX, PTR, SPF, DKIM, DMARC records.
- Basic familiarity with systemd and firewalld or iptables.
- Open ports: 25 (SMTP), 465 (SMTPS), 587 (submission), 993 (IMAPS), 143 (STARTTLS), 995 (POP3S). Do not open 110 or 143 without TLS forced.

## Why Self‑Hosting Email Is (Usually) a Bad Idea — But Here We Are

### The True Cost of Control vs. Convenience

The illusion of full control evaporates the first time you realize deliverability is 90% of the problem. Google and Microsoft have spent billions on reputation systems; your $5 VM can’t compete on trust. You trade convenience for a constant fire‑fighting loop: SPF alignment, IP warming, feedback loop registration, and hourly dives into your RBL status.

**Comparison: Self‑Hosted vs. Google vs. Exchange**

| Factor | Self‑Hosted | Google Workspace | Exchange Online |
|--------|-------------|------------------|----------------|
| Monthly cost (10 users) | $5‑10 (VPS) | $120+ | $120+ |
| Control | Full (you or your pager) | Limited (UI policies) | Moderate (admin tools) |
| Deliverability | You own the reputation | Baked‑in reputation | Baked‑in reputation |
| Maintenance effort | 4‑8 hours/week | 30 min/month | 1 hour/month |
| IP reputation risk | Your entire IP’s neighbors matter | Immutable | Immutable |

I’ve run mail servers for 15 years. If you aren’t ready to read RBL listings on a Sunday morning, buy a service. This guide assumes you are that kind of masochist.

### Your Email Will End Up in Spam Without This Checklist

Start with SPF, DKIM, and DMARC or don’t start at all. I’ve seen perfectly configured Postfix boxes get flagged because the SPF record used `+all` or the DKIM selector was missing. These are not optional. They are the foundation of trust in 2026.

### What This Guide Covers (and What It Doesn’t)

This guide covers the minimal viable Postfix + Dovecot stack with SASL and TLS, plus the deliverability trinity. It explicitly excludes webmail clients (Roundcube, SnappyMail), antivirus/antispam (Rspamd, Amavis, postfwd)—those war stories come in a separate guide.

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/hosting/self-hosted-email-with-postfix-and-dovecot-honest-sysadmin/](https://www.valtersit.com/guides/hosting/self-hosted-email-with-postfix-and-dovecot-honest-sysadmin/)**
