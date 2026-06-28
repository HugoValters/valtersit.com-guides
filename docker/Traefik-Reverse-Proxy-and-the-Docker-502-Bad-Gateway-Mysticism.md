> 📖 **Original article:** [Traefik Reverse Proxy and the Docker '502 Bad Gateway' Mysticism](https://www.valtersit.com/guides/docker/Traefik-Reverse-Proxy-and-the-Docker-502-Bad-Gateway-Mysticism/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

It’s Sunday morning. You’ve just poured your first cup of coffee, and PagerDuty starts screaming. Your core web application is returning a blank white screen with "502 Bad Gateway". 

You SSH into the server. The application container is running. Traefik is running. CPU is idling. Everything looks fine, except they’ve suddenly stopped talking to each other. You check the logs and realize Watchtower pulled a minor patch update overnight, recreated the container, and completely broke your routing. 

Welcome to the absolute fragility of Docker's "auto-magic" service discovery. 

"Magic" in infrastructure just means "behavior that will predictably fail when you are not looking." Traefik is a fantastic reverse proxy, but if you rely on its default behavior for container networking, you are playing Russian roulette with your uptime.

### The Mechanics: Why Traefik Loses Its Mind

Traefik works by listening to the Docker daemon socket. When a container spins up with the `traefik.enable=true` label, Traefik queries Docker for that container's internal IP address to start routing traffic to it. 

Here is where the magic breaks down. 

A well-architected backend container doesn't just sit on one network. It sits on a public-facing proxy network (to talk to Traefik) and an isolated backend network (to talk to the database or Redis). 

When Traefik asks Docker, "Hey, what is the IP for the `api-backend` container?", Docker replies with *all* the IP addresses attached to that container. Traefik looks at the list and simply guesses. 

If it guesses the IP from the `frontend_proxy` network, your app works perfectly. If it guesses the IP from the `backend_db` network—a subnet Traefik explicitly has no route to—you get an instant 502 Bad Gateway. Because Docker assigns IPs dynamically upon container creation, a simple restart reshuffles the deck. Yesterday's working config is today's outage.

### The Fix: Stop The Guessing Game

The solution is to stop letting Traefik think for itself. As an engineer, your job is to be explicit. You must tell Traefik exactly which network to use for routing, completely eliminating the guess-work.

First, create a dedicated, external Docker network for your proxy traffic. Never rely on the default `docker-compose_default` bridge.

```bash
docker network create web_proxy
```

Next, lock down your `docker-compose.yml`. You need to use the absolute lifesaver label: `traefik.docker.network`.

### The Bulletproof Setup

Here is how your backend service should actually look in production. We are attaching it to two networks, but explicitly pinning Traefik's routing logic to the `web_proxy` network.

```yaml
services:
  api-backend:
    image: my-company/api:v2.4.1
    restart: unless-stopped
    networks:
      # Traefik lives here
      - web_proxy
      # Database lives here, completely hidden from Traefik
      - internal_db
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.api.rule=Host(`api.example.com`)"
      - "traefik.http.routers.api.entrypoints=websecure"
      - "traefik.http.routers.api.tls.certresolver=letsencrypt"
      
      # 1. THE FIX: Explicitly tell Traefik which network to route through
      - "traefik.docker.network=web_proxy"
      
      # 2. BONUS FIX: Tell Traefik exactly which port to hit
      # Don't let it guess if your container exposes multiple ports!
      - "traefik.http.services.api.loadbalancer.server.port=8080"
```

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/docker/Traefik-Reverse-Proxy-and-the-Docker-502-Bad-Gateway-Mysticism/](https://www.valtersit.com/guides/docker/Traefik-Reverse-Proxy-and-the-Docker-502-Bad-Gateway-Mysticism/)**
