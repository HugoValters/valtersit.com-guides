> 📖 **Original article:** [Stop Paying for GitHub Actions: The Ultimate GitLab Self-Hosted Runner Migration Guide](https://www.valtersit.com/stop-paying-for-github-actions-gitlab-self-hosted-runner-migration/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

If you are a DevOps engineer or a HomeLab enthusiast, you likely felt the panic in late 2025 when GitHub announced massive billing changes for self-hosted GitHub Actions. While they backtracked slightly, the trust is broken. Relying entirely on a single platform for your CI/CD pipelines is a trap.

In this guide, I will show you how to take back control. We are going to build a blazing-fast, self-hosted GitLab Runner on a dedicated VPS and use a custom Python script to automatically sync all your GitHub repositories to GitLab. Full redundancy, zero vendor lock-in.


> 🖼️ **[Image: 'GitLab vs GitHub Migration' — view in full article](https://www.valtersit.com/stop-paying-for-github-actions-gitlab-self-hosted-runner-migration/)**


*(Prefer video? Watch my full setup process on YouTube right here!)*



## Step 1: The Infrastructure

Speed matters more than core count for CI/CD tasks like `npm install` or code compilation. For this setup, I am using **Zone.eu**, an Estonian-based hosting company. They offer incredibly fast VPS setups with NVMe drives, unmetered bandwidth, and human technical support.

*(If you want to support my content, you can grab your own Zone.eu server here using my referral link: [Zone CloudServer VPS](https://www.zone.eu/cloudserver-vps/?utm_source=youtube&utm_medium=referral&utm_campaign=valters))*

## Step 2: Installing the GitLab Runner

Once you have your VPS ready and you are logged in as `root`, it's time to install the GitLab Runner.

```bash
# Become root
sudo su

# Download the GitLab repository script
curl -L "[https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh](https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh)" | sudo bash

# Install the runner
sudo apt-get install gitlab-runner -y
```

## Step 3: Registering the Runner & Docker Setup

Head over to your GitLab account, navigate to **Settings > CI/CD > Runners**, and create a new group or project runner. Copy the registration token provided by GitLab.

Run the registration command in your terminal: *(Note: Choose `docker` as the executor and `docker:stable` as the default image).*

```bash
gitlab-runner register
```

Since we are using the Docker executor, we need to install Docker and ensure the `gitlab-runner` user has the correct permissions:

```bash
# Install Docker
sudo apt update
sudo apt install docker.io -y

# Start and enable Docker
sudo systemctl start docker
sudo systemctl enable docker

# Add the gitlab-runner user to the docker group
sudo usermod -aG docker gitlab-runner
```

## Step 4: The Automation Magic (GitHub to GitLab Sync)

We don't want to manually copy repositories. We want an automated fallback. First, generate a **Personal Access Token** in both GitHub and GitLab with the necessary read/write and repository API permissions.

Next, install Python and PIP on your server:

```bash
sudo apt-get update
sudo apt install python3 python3-pip python3-venv -y
```

Now, create the Python script (`git.py`) that will handle the synchronization:

```python
import os
import subprocess
import requests
import logging
import re
import shutil

# ==========================================
# CONFIG
# ==========================================
GITHUB_USER  = ""
GITHUB_TOKEN = ""
GITLAB_TOKEN = ""

# Group IDs from GitLab
PUBLIC_GROUP_ID  = 116797979
PRIVATE_GROUP_ID = 116798188
CACHE_DIR = "/home/python/git_cache"
```

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/stop-paying-for-github-actions-gitlab-self-hosted-runner-migration/](https://www.valtersit.com/stop-paying-for-github-actions-gitlab-self-hosted-runner-migration/)**
