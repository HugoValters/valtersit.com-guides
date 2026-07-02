> 📖 **Original article:** [How to Protect Your Homelab and API with a Free Self-Hosted WAF (SafeLine Docker Install)](https://www.valtersit.com/guides/security/safeline-waf-docker-homelab-api-protection/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

### Why I Stopped Trusting Cloudflare Alone

My site recently crossed 5,000 daily visitors. Sounds like a celebration. What it actually means is that the volume of garbage hitting my server — automated scanners, credential stuffing bots, SQL injection probes — scaled up with the legitimate traffic.

Cloudflare catches most of it. Most. The remaining 5–10% that leaks through the CDN layer goes straight at your origin, and if you're running APIs or self-hosted services, that origin is the actual target. Enterprise WAFs from Fortinet or F5 would solve this, but they start at thousands of dollars per year. That's not a homelab budget. That's not even a small business budget.

Enter **SafeLine WAF** — a completely self-hosted, Docker-based Web Application Firewall with a legitimate free Community Edition. No call-home telemetry. No per-GB pricing that punishes visibility. Just a container that sits between the internet and your services and kills the bad traffic before it reaches your application layer.

Here is what changed after I deployed it, and exactly how you can do the same in under ten minutes.

---

### What SafeLine Actually Does (And What It Doesn't)

Before you deploy anything, understand what you're getting.

SafeLine is a **reverse proxy WAF** built on a modified Nginx engine called Tengine. All traffic goes through it first. It inspects each request against a rule engine, blocks anything that looks malicious, and forwards clean traffic to your backend service.

**It blocks:**
- SQL injection (`' OR 1=1`, `UNION SELECT`, etc.)
- Cross-site scripting (XSS) payloads in headers and query strings
- Path traversal attacks (`../../../etc/passwd`)
- Scanner and bot fingerprints (Nuclei, sqlmap, Nikto, Masscan)
- Brute force attacks via rate limiting
- HTTP protocol violations

**It does not replace:**
- A firewall at the network level (`ufw`, `iptables`, cloud security groups)
- Proper secrets management
- Input validation in your application code

SafeLine is one layer. Defense in depth means you still need the other layers.

---

### Community Edition vs Pro — What You Actually Get for Free

The free Community Edition covers everything a homelab or small deployment needs:

| Feature | Community | Pro |
|---|---|---|
| WAF protection | ✅ | ✅ |
| Custom block rules | ✅ | ✅ |
| Dashboard + analytics | ✅ | ✅ |
| Sites supported | Unlimited | Unlimited |
| Rate limiting | ✅ | ✅ |
| CAPTCHA challenges | ✅ | ✅ |
| Dynamic protection rules | ❌ | ✅ |
| Support SLA | Community | Priority |
| Authentication protection | Basic | Advanced |

For a homelab, personal API, or small production deployment: the free tier is enough.

---

### Prerequisites

You need:
- A Linux server (Debian/Ubuntu/Rocky — anything with Docker support)
- Docker Engine 20.10+ installed
- Docker Compose v2
- Ports `80`, `443`, and `9443` available
- At least 1 GB RAM free

If you don't have Docker installed yet:

```bash
curl -fsSL https://get.docker.com | sh
systemctl enable --now docker
```

---

### Installing SafeLine WAF — The 60-Second Method

SafeLine ships an automated setup script that handles everything: pulls images, creates the `docker-compose.yml`, sets up volumes, and starts the stack.

**Step 1 — Run the installer:**

```bash
bash -c "$(curl -fsSLk https://waf.chaitin.com/release/latest/setup.sh)"
```

The script will ask for an installation directory (default: `/data/safeline`). Accept the default unless you have a specific reason to change it.

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/security/safeline-waf-docker-homelab-api-protection/](https://www.valtersit.com/guides/security/safeline-waf-docker-homelab-api-protection/)**
