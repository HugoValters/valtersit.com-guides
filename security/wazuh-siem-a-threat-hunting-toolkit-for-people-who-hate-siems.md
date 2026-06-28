> 📖 **Original article:** [Wazuh SIEM: A Threat Hunting Toolkit for People Who Hate SIEMs](https://www.valtersit.com/guides/security/wazuh-siem-a-threat-hunting-toolkit-for-people-who-hate-siems/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

### 1. Introduction: So You Need a SIEM. My Condolences.

Let's get one thing straight. You're here because someone—a manager, an auditor, or that little voice of dread in your head—told you that you need a Security Information and Event Management (SIEM) system. My condolences. Most SIEMs are expensive, proprietary black boxes designed to do one thing exceptionally well: generate invoices. They are compliance checkboxes, not functional security tools. They drown you in so many false positives that a real attack would look like just another Tuesday.

We're not here to install a "next-gen" magic quadrant leader. We're here to build a data analysis engine.

#### Why Commercial SIEMs Are (Mostly) Expensive Alert Cannons

The commercial SIEM market is built on a foundation of terrible ideas. The "per-GB-per-day" licensing model is the most fundamentally broken of them all. It actively punishes you for collecting more data, which is the entire point of security monitoring. The more visibility you want, the more it costs, until you inevitably start making compromises, like not logging DNS queries because it'll blow the budget.

Then there's the vendor lock-in. Their detection logic is an opaque, proprietary secret sauce. You can't see it, you can't easily modify it, and you're entirely at the mercy of their release cycle to detect the latest threat. The default configuration is a firehose of useless "Severity: Medium - User logged in" noise, designed to make the dashboard look busy and justify its existence. Alert fatigue isn't a side effect; it's a feature.

#### Enter Wazuh: The Box of Sharp Parts

Wazuh isn't a polished, shrink-wrapped product. It's a framework, a collection of sharp, powerful parts that you have to assemble yourself. It grew out of the OSSEC HIDS (Host-based Intrusion Detection System) and has been bolted onto the Elastic Stack—or OpenSearch, or whatever they're calling the fork this week. The point is, it's a battle-tested agent-based HIDS connected to a modern, scalable data backend.

The power of Wazuh isn't in the pre-canned dashboard widgets. It's in the absolute, granular control you have over data collection, rule logic, and automated response. It's free, as in beer. Your only cost is the hardware (or cloud bill) and your own time. If you're not willing to invest the time to learn its quirks and tune it properly, stop reading now. Go call your Splunk sales rep and prepare your purchase order. For everyone else, let's build something useful.

### 2. Architecture: Don't Screw This Up from the Start

Your initial architectural decisions will determine whether this project is a success or a slow, painful failure. Get this part right, and the rest is just tuning.

#### The Three Horsemen

Wazuh is composed of three primary services. Understand what each one does and its limitations.

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/security/wazuh-siem-a-threat-hunting-toolkit-for-people-who-hate-siems/](https://www.valtersit.com/guides/security/wazuh-siem-a-threat-hunting-toolkit-for-people-who-hate-siems/)**
