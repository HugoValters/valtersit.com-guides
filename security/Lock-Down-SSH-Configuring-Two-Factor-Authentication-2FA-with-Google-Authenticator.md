> 📖 **Original article:** [Lock Down SSH: Configuring Two-Factor Authentication (2FA) with Google Authenticator](https://www.valtersit.com/guides/security/Lock-Down-SSH-Configuring-Two-Factor-Authentication-2FA-with-Google-Authenticator/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

# Secure Your Server: Forcing 2FA on SSH Connections


> 🖼️ **[Image: 'Hacker trying to bypass SSH with Google Authenticator shield' — view in full article](https://www.valtersit.com/guides/security/Lock-Down-SSH-Configuring-Two-Factor-Authentication-2FA-with-Google-Authenticator/)**


Every single day, automated bots and bad actors scan the internet for open `Port 22` configurations, attempting to brute-force their way into vulnerable Linux servers. While using SSH Keys is the standard best practice, adding an additional layer of Time-based One-Time Passwords (TOTP) ensures that even if your private key is compromised, the attacker cannot gain access.

In this guide, we will configure `libpam-google-authenticator` on an Ubuntu/Debian server to require both an SSH Key **and** a 6-digit Google Authenticator code.

## Prerequisites
* A Linux Server (Ubuntu/Debian). *For this lab, we are using a highly secured VPS provided by our infrastructure partner, [Zone.eu](https:/zone.valtersit.com).*
* Root or `sudo` privileges.
* The Google Authenticator app (or Authy/Bitwarden) installed on your smartphone.

## Valters IT YouTube Tutorial

---

## Step 1: Install the PAM Authenticator Module

First, update your package list and install the required Google Authenticator PAM (Pluggable Authentication Module).

`sudo apt update`
`sudo apt install libpam-google-authenticator -y`

## Step 2: Generate the TOTP Secret

Run the authenticator command as the user you use to log in via SSH (do not run this as root unless you explicitly log in as root).

`google-authenticator`

The terminal will prompt you with a series of questions:
1. **Make tokens time-based?** Press `y`.
2. A giant QR code will appear in your terminal. **Scan this with your authenticator app.**
3. **Save your emergency scratch codes** in a secure password manager. If you lose your phone, these are your only way back in!
4. **Update your ~/.google_authenticator file?** Press `y`.
5. **Disallow multiple uses of the same token?** Press `y` (Protects against replay attacks).
6. **Increase time skew window?** Press `n` (Unless you have severe clock drift issues).
7. **Enable rate-limiting?** Press `y` (Limits to 3 logins every 30 seconds).

---

## Step 3: Configure PAM for SSH

Now, we need to tell the PAM system to require the Google Authenticator module for SSH connections.

`sudo nano /etc/pam.d/sshd`

Scroll to the bottom of the file and add the following line:

`# Require Google Authenticator 2FA`
`auth required pam_google_authenticator.so`

Save and exit the file (CTRL+O, Enter, CTRL+X).

---

## Step 4: Force SSH Daemon to Require 2FA

By default, SSH might skip 2FA if you are using an SSH Private Key. We need to explicitly tell the SSH daemon to require **both** the public key and the interactive keyboard prompt (which will ask for the 2FA code).

`sudo nano /etc/ssh/sshd_config`

Find and modify (or add) the following directives in the file:

`# Ensure PAM is enabled`
`UsePAM yes`

`# Allow interactive keyboard authentication`
`KbdInteractiveAuthentication yes`
`# Note: On older versions of Ubuntu, this might be called 'ChallengeResponseAuthentication yes'`

`# Force BOTH SSH Key AND the Verification Code`
`AuthenticationMethods publickey,keyboard-interactive`

Save and exit the file.

---

## Step 5: Restart SSH and Test

Restart the SSH service to apply the changes.

`sudo systemctl restart ssh`

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/security/Lock-Down-SSH-Configuring-Two-Factor-Authentication-2FA-with-Google-Authenticator/](https://www.valtersit.com/guides/security/Lock-Down-SSH-Configuring-Two-Factor-Authentication-2FA-with-Google-Authenticator/)**
