> 📖 **Original article:** [Kubernetes Cluster Autoscaler: Scaling Nodes, Not Pods](https://www.valtersit.com/guides/kubernetes/kubernetes-cluster-autoscaler-scaling-nodes-not-pods/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

# Kubernetes Cluster Autoscaler: Scaling Nodes Instead of Just Pods

The Slack message said "HPA is working perfectly, we're at 58 replicas." That was at 09:14 on Black Friday. By 09:22, 44 of those replicas had been `Pending` for eight minutes. By 09:51, the checkout API was returning 503s, the incident channel had 30 people in it, and someone was asking why the "autoscaling" they'd configured six months ago wasn't autoscaling.

It was working. It was scaling exactly the thing it was told to scale: pods. The HorizontalPodAutoscaler creates more replicas. It does not, cannot, and will never create more nodes. When the scheduler runs out of room, the ReplicaSet controller keeps dutifully incrementing a counter while the scheduler quietly rejects every new pod. The dashboards look green because HPA reports the desired count it *wanted*, not the count that actually got scheduled.

This guide is for platform engineers, SREs, and anyone who owns a Kubernetes cluster's bill or its availability. By the end you'll know how the Cluster Autoscaler's control loop actually works, how to install it properly across AWS, GCP, and Azure, how to tune the scale-down flags that default to values that cost you money, and how to debug the six failure modes that account for nearly every "CA isn't scaling" ticket.

:::note[TL;DR]
- HPA scales pods; Cluster Autoscaler (CA) scales nodes. Configuring one without the other is a production incident waiting for a traffic spike.
- CA works by *simulating the scheduler*: it replays unschedulable pods against node groups and picks an expansion plan. Break its assumptions (custom schedulers, extended resources) and it goes blind.
- CA's blast radius includes terminating nodes. Auth it with IRSA/Workload Identity, never a node-wide instance role, and never a 2019 copy-paste manifest.
- The default scale-down thresholds are conservative. In bursty production traffic they churn nodes; in steady traffic they leave money on the table. Tune them deliberately.
- Read the `cluster-autoscaler-status` ConfigMap before you page anyone. It answers 80% of "why isn't it scaling" questions in one command.
:::

## Prerequisites

- A Kubernetes cluster (1.28+ as of this writing) with `kubectl` access and permission to edit `kube-system`
- Cloud provider CLI configured (`aws`, `gcloud`, or `az`) with read access to your node group construct
- Cluster nodes already provisioned via an autoscaling group / node pool / VMSS — CA cannot create capacity out of nothing
- Metrics Server installed if you also plan to run HPA
- Helm 3 if you take the recommended install path

---

## The Pod-Scaling Delusion

HPA is a feedback controller on a metric. It reads CPU utilization, compares it to a target, and adjusts a replica count. That's it. It has no concept of physical capacity. In its mental model, the cluster is infinite and the scheduler is instantaneous. Both assumptions are false in every real environment.

Here's how you detect the gap. This command is the single most useful thing you can run during an autoscaling incident:

```bash
kubectl get pods -A --field-selector=status.phase=Pending -o wide
```

If that returns rows while HPA is reporting healthy replica counts, you don't have a scaling problem — you have a *capacity* problem. Confirm it with the scheduler's own words:

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/kubernetes/kubernetes-cluster-autoscaler-scaling-nodes-not-pods/](https://www.valtersit.com/guides/kubernetes/kubernetes-cluster-autoscaler-scaling-nodes-not-pods/)**
