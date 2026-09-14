> 📖 **Original article:** [ntopng Traffic Analysis: Finding Bandwidth Hogs](https://www.valtersit.com/guides/networking/ntopng-traffic-analysis-finding-bandwidth-hogs/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

It's 03:14 and the WAN uplink is pinned at 940 Mbps. The NOC has already restarted the firewall once. You SSH into the monitoring box, run `iftop -i eth1`, and get a screen of scrolling IP pairs that changes every time you blink. You *think* it's `10.20.4.31` talking to something in Frankfurt. You are not sure. By the time you scroll back, the numbers have moved. Two hours later you find out it was a misconfigured vulnerability scanner on a guest VLAN, and you found it by asking a human, not by asking your tools.

This guide is for engineers who already know what a SPAN port is and are tired of scalar counters. You'll get deployment architectures that survive real link speeds, a config baseline that avoids the usual self-inflicted wounds, the REST API patterns for scripting the hunt, and alerting that fires before the link saturates rather than after the post-mortem. What you will not get: a GUI tour, screenshots, or a re-explanation of TCP.

:::note[TL;DR]
- ntopng wins because it correlates host × protocol × time — a dimension reduction problem that `iftop` and SNMP structurally cannot solve.
- Sizing is the failure mode. Mirroring a 40G uplink into a 1G NIC produces a shredder, not a monitor.
- The defaults that bite: `admin`/`admin`, wrong interface speed, undefined local networks, unbounded Redis.
- Alerts that terminate in the ntopng UI are not alerts. Ship them to syslog or a webhook.
:::

## Prerequisites

- A SPAN/mirror port or a TAP, already configured, on the link you care about
- Linux host (Debian/Ubuntu or RHEL-family) with a NIC you can put in promiscuous mode
- Redis reachable — local socket or a dedicated instance
- Root or `CAP_NET_RAW` on the capture interface
- `jq`, `curl` for the scripting sections; `python3` with `requests` for the automation section
- Decent familiarity with `ntopng --help` output — flag names shift between major releases, and I'd rather you verify than trust me on version-specific syntax

## Why ntopng and Not Another `iftop` Screenshot

### The structural problem with interface counters and packet sniffers

SNMP hands you one number per interface: `ifOutOctets`. It tells you the pipe is full and nothing else. `iftop` gives you a five-second rolling window of flows — great for "is something happening right now," useless for "which host, which application, and is this different from yesterday."

Bandwidth analysis is a dimension reduction problem. You start with a firehose of packets and need to collapse it to *per-host × per-protocol × per-time*. A scalar (interface counters) and a point-in-time snapshot (`iftop`) can't get you there. They're the wrong shape of data. ntopng is the right shape because it maintains per-host state in Redis, classifies at L7 with nDPI, and exposes the whole thing through a queryable API.

### What ntopng actually gives you that nothing else in the free tier does

Passive flow reconstruction from a mirror with no agent on the endpoints. nDPI classification that does signature matching, TLS SNI extraction, DNS correlation, and behavioural heuristics. Per-host historical statistics persisted in Redis across restarts. A REST API for everything the UI shows. Threshold alerting out of the box. That combination — free, single binary, no endpoint agents — is why it stays in the toolbox.

### What ntopng is not

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/networking/ntopng-traffic-analysis-finding-bandwidth-hogs/](https://www.valtersit.com/guides/networking/ntopng-traffic-analysis-finding-bandwidth-hogs/)**
