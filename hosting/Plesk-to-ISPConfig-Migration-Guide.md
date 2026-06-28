> 📖 **Original article:** [Stop Paying for Plesk: How to Migrate to ISPConfig (Free Hosting Panel)](https://www.valtersit.com/guides/hosting/Plesk-to-ISPConfig-Migration-Guide/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

## The "Plesk Tax" is Getting Out of Hand

I don't have anything bad to say about Plesk from a purely technical standpoint. It is a great web hosting panel with tons of features, and it has been one of my favorite industry leaders for a long time.

However, something is fundamentally broken in their billing department. Recently, prices have skyrocketed. I got to the point where they increased the price twice in a single year. When the math no longer makes logical sense—especially when paying multiple middlemen (Scrum Masters, Project Managers, etc.) inflating the corporate price—it is time to stop the bleeding.

For professional administrators who want efficiency, full control, and zero licensing fees, **ISPConfig** is the way to go. It is a 100% free, open-source hosting control panel with a robust Web UI.

In this guide, I will show you how to install ISPConfig on a fresh Ubuntu VPS and configure your first email domain with DKIM.

**Infrastructure Note:** For this tutorial, I am using a fresh Ubuntu 24.04 LTS VPS provided by our infrastructure partner, [Zone.eu](https://zone.eu).

## Step 1: Pre-Flight Checklist (Hostname & DNS)

Before you run any installation scripts, you must ensure your server's hostname and DNS records are set up correctly. If you want email delivery to actually work, this is non-negotiable.

1. **Set your A Records**: Point your domain (e.g., `ispconfig.yourdomain.com`) to your VPS IP address.
2. **Set the Hostname**: Ensure your Ubuntu machine knows its own FQDN (Fully Qualified Domain Name).

```bash
sudo hostnamectl set-hostname ispconfig.yourdomain.com
```

> **Tip from the video:** In my video run-through, I accidentally forgot to set the hostname first, and the auto-installer politely yelled at me. Save yourself the 30 seconds and do it before executing the script!

## Step 2: The ISPConfig Auto-Installer

In the old days, installing ISPConfig required reading a 20-page manual and spending two hours configuring packages. Today, we use the official auto-installer.

Run the following command as root (or with `sudo`):

```bash
wget -O - https://get.ispconfig.org | sh -s -- --use-ftp-ports=21-22 --lang=en
```

When prompted, type `yes` to confirm the installation. The script will automatically install Apache/Nginx, MariaDB, Postfix, Dovecot, Rspamd, Let's Encrypt, and the ISPConfig interface.

Grab a coffee. This will take a few minutes.

## Step 3: Accessing the Web Panel

Once the script finishes, it will output your **Admin Password** and **MySQL Root Password** in the terminal.

**CRITICAL SECURITY STEP:** Copy these passwords to a secure password manager immediately. Then, clear your terminal and delete the setup log file located at `/root/ispconfig-install-log/setup.log` so no one can find your plaintext passwords if your server is ever breached.

Navigate to your new panel in your web browser using Port 8080:

```
https://[YOUR-SERVER-IP]:8080
```

Since this is a fresh install, your browser will warn you about an untrusted SSL certificate. Proceed past the warning (**Accept the risk**)—you can secure the admin panel with a valid Let's Encrypt certificate later.

Log in using the username `admin` and the password provided in the terminal.

## Step 4: Configuring an Email Domain & DKIM

While ISPConfig might not look as flashy as Plesk out of the box, its functionality is rock solid. Let's set up an email inbox.

### Add Email Domain

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/hosting/Plesk-to-ISPConfig-Migration-Guide/](https://www.valtersit.com/guides/hosting/Plesk-to-ISPConfig-Migration-Guide/)**
