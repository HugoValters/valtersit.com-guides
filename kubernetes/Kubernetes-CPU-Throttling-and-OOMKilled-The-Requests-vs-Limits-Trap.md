> 📖 **Original article:** [Kubernetes CPU Throttling & OOMKilled: The Requests vs Limits Trap](https://www.valtersit.com/guides/kubernetes/Kubernetes-CPU-Throttling-and-OOMKilled-The-Requests-vs-Limits-Trap/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

There is a widespread epidemic of engineers copy-pasting Helm charts and Stack Overflow YAMLs without understanding what the `resources` block actually does. The standard playbook is to set a tiny CPU request "to save money" and an arbitrary CPU limit "because it feels safe." 

Then they deploy their Node.js API or heavy Java Spring monolith and stare in confusion at their Grafana dashboards. The app takes 45 seconds to boot, drops connections under minor load, and runs like a heavily medicated potato—even though the underlying worker node has 64 cores sitting completely idle. 

### The Mechanics: Under the Hood

Kubernetes doesn’t actually understand "CPU". It understands Linux cgroups. 

When you set a CPU *limit*, Kubelet translates that into a Linux CFS (Completely Fair Scheduler) quota. Here is the brutal reality of CFS quotas: they are calculated in 100-millisecond windows. If your application attempts to burst—say, to compile a regex, parse a large JSON payload, or run garbage collection—and hits that quota in the first 10ms of the window, the Linux kernel ruthlessly pauses the container thread for the remaining 90ms. 

Your application isn't slow. It is being actively strangled and put to sleep by the operating system hundreds of times a second.

Memory is even worse. If you set a memory request of 256Mi and a limit of 1Gi, Kubernetes assigns your pod to the `Burstable` Quality of Service (QoS) class. When the underlying node inevitably feels slight memory pressure, the kernel's Out Of Memory (OOM) killer wakes up and looks for victims. It explicitly targets `Burstable` pods that are using more memory than their baseline `requests`. Your pod gets slapped with an `OOMKilled` (Exit Code 137), your users get disconnected, and your pager goes off.

### The Fix: Guaranteed QoS

Senior engineers don't play the overcommit game. Stop treating your infrastructure like a greedy landlord packing 15 college students into a two-bedroom apartment. 

If your application needs 2 CPU cores to process requests smoothly without micro-stuttering, you give it 2 cores. If it needs 1GB of RAM, you give it 1GB of RAM.

You achieve this by entering the `Guaranteed` QoS class. To get this classification from Kubernetes, your `requests` must perfectly match your `limits` for both CPU and Memory.

### The Code

Here is the default garbage you see in almost every generic tutorial:

```yaml
# THE BAD WAY (Burstable QoS)
# This will throttle your CPU and get you OOMKilled.
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-potato
spec:
  template:
    spec:
      containers:
      - name: app
        image: my-app:latest
        resources:
          requests:
            cpu: "100m"
            memory: "256Mi"
          limits:
            cpu: "1000m"
            memory: "1Gi"
```

Here is the bulletproof, production-grade approach. By setting requests equal to limits, you guarantee the pod receives dedicated resources. Kubelet won't let the CFS quota throttle your baseline, and the OOM killer will look for other victims first.

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/kubernetes/Kubernetes-CPU-Throttling-and-OOMKilled-The-Requests-vs-Limits-Trap/](https://www.valtersit.com/guides/kubernetes/Kubernetes-CPU-Throttling-and-OOMKilled-The-Requests-vs-Limits-Trap/)**
