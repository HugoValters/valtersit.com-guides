> 📖 **Original article:** [Revolutionizing Notifications with NTFY.sh](https://www.valtersit.com/guides/docker/ntfy/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

*By: Hugo Valters*

In today's fast-paced digital world, effective communication is the backbone of productivity and success. Whether you're managing a complex DevSecOps pipeline, running a small business, or organizing personal tasks, timely and efficient notifications are critical. 

Enter **NTFY.sh**, a lightweight, open-source notification tool that offers simplicity, flexibility, and power.

---

## 📺 Video Demo & Tutorial

The demo and Installation Guide you will find in this article and also in my YouTube video here: 


  
  


---


  {/* ValtersIT_MDX_Raksti */}
  

  


## What is NTFY.sh?

NTFY.sh is a simple, HTTP-based notification service that allows you to send and receive notifications via REST API, Python, CURL or command-line tools. Think of it as a highly customizable and platform-independent tool for managing alerts and updates, without the overhead of more complex notification systems.

### Key Features
* **Easy-to-use HTTP API:** Send notifications with a single command or HTTP request.
* **Cross-platform compatibility:** Works on Windows, Linux, macOS, and mobile devices.
* **Self-hosting capability:** Secure your data by running NTFY.sh on your own server.
* **Integration-ready:** Works with tools like Docker, Home Assistant, and shell scripts.

## Why Use NTFY.sh?

* **1. Minimal Setup:** With no heavy dependencies or complex configurations, NTFY.sh can be up and running in minutes.
* **2. Versatile Applications:** From alerting you about a failing server to reminding you about your next coffee break, NTFY.sh's applications are virtually endless.
* **3. Privacy and Security:** For sensitive environments, the option to self-host NTFY.sh ensures complete control over your data.
* **4. Cost-Effective:** Being open-source, it eliminates the licensing costs associated with commercial notification systems.

## Top Use Cases for NTFY.sh

1. **DevOps and Monitoring Alerts:** Notify team members about failing builds in a CI/CD pipeline or alert admins about anomalies detected by tools like Prometheus, Zabbix or Nagios.
2. **IoT and Smart Home:** Receive alerts when your smart doorbell detects motion or the temperature in your greenhouse exceeds a threshold.
3. **Personal Productivity:** Schedule reminders for meetings, deadlines, or get updates on the status of long-running scripts.

---

## Installation Guide (Docker)

To start the installation we will need a VPS. In our tutorial, we will be using ZONE.EU. 

### 1. Update and Install Docker
First, become root and update your repositories:

```bash
sudo su
apt-get update && apt-get upgrade -y
```

Install required certificates and add Docker's repository keys:

```bash
sudo apt-get install ca-certificates curl -y
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
```

Install Docker and Docker Compose:

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin
apt-get install docker-compose -y
```

### 2. Docker Network and Nginx Proxy

Create a network for your containers:

```bash
docker network create valtersit
```

*Note: For the full Nginx Reverse Proxy and Portainer setup commands, refer to the video guide or the GitHub repository.*

### 3. Deploy NTFY with Docker Compose

Create a directory and your `docker-compose.yml` file:

```bash
mkdir -p /srv/docker/startup
cd /srv/docker/startup
nano docker-compose.yml
```

Example config for NTFY:

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/docker/ntfy/](https://www.valtersit.com/guides/docker/ntfy/)**
