> 📖 **Original article:** [My Bulletproof Home Lab with ZimaBoard: Redundant Networks, Kasm & Tailscale VPN](https://www.valtersit.com/bulletproof-home-lab-zimaboard-redundant-networks-kasm-tailscale/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

Imagine this: Your primary internet connection drops dead in the middle of a critical task. No dedicated IP? No problem. What if you could seamlessly switch to a backup network, access your secure home lab from anywhere — even your iPad on the road — and do it all without exposing your entire setup to risks? 

That's the magic I'm unpacking today with a ZimaBoard, dual networks, Kasm workspaces, and Tailscale VPN. Stick around; by the end, you'll have the blueprint to make your home lab unbreakable. And trust me, the payoff is worth every step.


> 🖼️ **[Image: 'ZimaBoard Bulletproof Home Lab' — view in full article](https://www.valtersit.com/bulletproof-home-lab-zimaboard-redundant-networks-kasm-tailscale/)**




## Why This Setup Changes Everything

Not everyone has the luxury of a static IP or enterprise-grade redundancy. But in a world where downtime means lost productivity (or worse, missed opportunities), why settle? 

I set out to create a fully functional, redundant home lab using a compact ZimaBoard. In my case, it's powered by an optical fiber network (`192.168.1.0/24`) as primary and a 5G backup (`192.168.118.0/24`). If the optical line fails, 5G kicks in automatically — smooth as silk.

This isn't just theory. I've tested it during weekend blackouts, and the handover is flawless. Plus, with **Tailscale**, you get secure remote access without port-forwarding headaches. Open it to the public DNS for quick checks, or lock it down to your devices only. It's flexible, secure, and insanely convenient for travelers like me who need to vet suspicious links or keep Discord running from an iPad.

> **Warning:** All IPs, Tailscale accounts, and Kasm installs in this guide were created for demonstration. They've been wiped clean post-video. Always prioritize security — don't expose more than necessary.

If you're running a single network, skip ahead to the Kasm install. But if redundancy calls your name, let's dive in.

---

## Step 1: Prep Your ZimaBoard with Ubuntu

Out of the box, ZimaBoard ships with CasaOS. For this powerhouse setup, we need raw Ubuntu. If you haven't switched yet, check out my previous install guides on how to flash Ubuntu onto your board. Once Ubuntu is up, we're ready to rock.

## Step 2: Configuring Redundant Networks for Zero Downtime

Here's where the intrigue builds: Two independent networks, one smart failover.

First, inspect your interfaces:

```bash
ip a
```

You'll spot something like:
* `enp3s0`: Optical network (e.g., IP `192.168.1.135`)
* `enp2s0`: 5G network (e.g., IP `192.168.118.9`)

Install the tools for priority assignment:

```bash
sudo apt install -y ifmetric ifplugd
```

Assign metrics (lower number = higher priority):

```bash
sudo ifmetric enp2s0 200
sudo ifmetric enp3s0 100
```

Verify your new routes with:

```bash
ip r
```

You should see routes prioritizing the optical network:
```text
default via 192.168.1.254 dev enp3s0 proto dhcp src 192.168.1.135 metric 100
default via 192.168.118.1 dev enp2s0 proto dhcp src 192.168.118.9 metric 200
```

For faster switching, tweak `ifplugd`. Edit `/etc/default/ifplugd` with nano:

```bash
sudo nano /etc/default/ifplugd
```

Change the line:
`ARGS="-q -f -u0 -d10 -w -I"`

To:
`ARGS="-q -f -u0 -d2 -w -I"`

Save (Ctrl+X, Y, Enter), then enable and start the service:

```bash
sudo systemctl enable ifplugd && sudo systemctl start ifplugd
```

Boom — network swaps now happen in seconds. Test by yanking a cable; watch the magic.

---

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/bulletproof-home-lab-zimaboard-redundant-networks-kasm-tailscale/](https://www.valtersit.com/bulletproof-home-lab-zimaboard-redundant-networks-kasm-tailscale/)**
