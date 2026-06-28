> 📖 **Original article:** [How I Automated My Entire Server Setup Using GitHub Actions and Ansible](https://www.valtersit.com/how-i-automated-my-entire-server-setup-using-github-actions-and-ansible/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

## Intro: The Weekend That Changed Everything

It started with a single crash. My VPS unexpectedly went down at 2 AM on a Saturday. I was out of town. No access. No backups. That's when I promised myself — never again.

Fast-forward a month later, I've automated the entire setup of my production-ready server infrastructure using GitHub Actions, Ansible, and some clever scripting. From a fresh Hetzner VPS to a hardened, monitored, and SSL-secured node — all in one push.


> 🖼️ **[Image: 'Automated Server Setup' — view in full article](https://www.valtersit.com/how-i-automated-my-entire-server-setup-using-github-actions-and-ansible/)**


Here's how I did it, what worked, and what almost fried my brain.

---

## Step 1: Why GitHub Actions?

Most people use GitHub Actions for CI/CD pipelines. But I realized — why not use it to automate infrastructure setup? You can trigger provisioning, SSH into fresh hosts, and run Ansible playbooks automatically.

**The Benefits:**
* You don't need your laptop online to deploy.
* Reproducible environments for clients and projects.
* Full audit logs via GitHub history.

---

## Step 2: Laying the Groundwork (Hetzner + Cloud-init)

I started with Hetzner Cloud. You can deploy servers via API with predefined SSH keys and cloud-init scripts. I set up a GitHub Action that:

1. Calls Hetzner API to create a VPS with a tag (e.g., `rancher-node`).
2. Injects my SSH public key.
3. Waits for the server to become reachable.
4. Triggers the next job: **Ansible provisioning**.

---

## Step 3: Ansible Magic — From Zero to Production

My Ansible playbook handles the "heavy lifting." Instead of manually typing commands, YAML does it for me:

* User creation + SSH key setup.
* UFW firewall with ports 22 (or custom), 80, 443, 3306.
* Docker + Docker Compose installation.
* Fail2Ban and automatic security updates.
* Caddy reverse proxy with Cloudflare DNS plugin.
* Tailscale installation and auto-authentication.
* Daily backups to an external ZimaBlade over SSH.

YAML never looked so sexy.

---

## Step 4: Secrets and Safety

I store sensitive data (Cloudflare tokens, Tailscale auth keys) in **GitHub Encrypted Secrets**. 

**Pro Tips:**
* Use different SSH keys for Ansible than you use for your personal laptop.
* Limit token scopes to only what is necessary.
* Always test with `--check` and `--diff` in Ansible before deploying changes to production.

---

## Step 5: Reverse Proxy Glory (Caddy FTW)

Forget Nginx config hell. Caddy auto-generates HTTPS certs via Let's Encrypt or Cloudflare DNS. Here is a simple snippet from my setup:

```bash
myapp.domain.com {
    reverse_proxy localhost:3000
    tls {
        dns cloudflare {env.CLOUDFLARE_API_TOKEN}
    }
}
```

This is pure poetry in motion.

---

## Step 6: Backups That Actually Work

Each server connects via **Tailscale** to a home ZimaBlade node with a 1TB disk. Nightly `rsync` backups run automatically, and I get a notification if they fail. Restoring a server is now just one command away.

*Bonus:* I monitor everything with **Grafana + Prometheus**, also deployed via my Ansible scripts.

---

## Final Thoughts: Why This Matters

Since implementing this setup, I've:
* Launched 5+ client projects without touching a manual console.
* Recovered from server failures in under 15 minutes.
* Slept much better.

Automation isn't just for big companies. If you're a solo dev, freelancer, or indie hacker — this kind of setup gives you freedom, resilience, and peace of mind.

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/how-i-automated-my-entire-server-setup-using-github-actions-and-ansible/](https://www.valtersit.com/how-i-automated-my-entire-server-setup-using-github-actions-and-ansible/)**
