> 📖 **Original article:** [Kubernetes RBAC Nightmare: Why Your Default ServiceAccount is a Backdoor](https://www.valtersit.com/guides/kubernetes/Kubernetes-RBAC-Nightmare-Why-Your-Default-ServiceAccount-is-a-Backdoor/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

Here is a terrifying fact about Kubernetes that most developers don't realize until their cluster is actively mining Bitcoin: by default, Kubernetes is entirely too helpful. 

When you deploy a Pod and don't explicitly specify a ServiceAccount, Kubernetes automatically assigns the `default` ServiceAccount to it. But it doesn't stop there. It forcefully injects a highly privileged JWT token directly into your container's filesystem at `/var/run/secrets/kubernetes.io/serviceaccount/token`. 

Unless your application is specifically designed to talk to the Kubernetes API, it has absolutely zero business possessing this token. Leaving it there is the infrastructure equivalent of taping the datacenter's master keys to the front door because you didn't want to bother carrying them.

### The Mechanics: The Exploit Path

Let's walk through how this destroys your weekend. 

You deploy a standard Node.js, Python, or Java web application. Three months later, a zero-day drops, or someone fat-fingers a dependency update, and an attacker gains Remote Code Execution (RCE) inside your container. 

The very first thing a competent attacker or automated exploit script does after getting a shell in a Kubernetes environment is run:
`cat /var/run/secrets/kubernetes.io/serviceaccount/token`

They now have a valid credential to authenticate against your internal Kubernetes API. 

Here is where lazy administration turns a minor application compromise into a catastrophic breach. If a cluster admin got frustrated with RBAC errors in the past and blindly bound a broad Role (like `edit` or, God forbid, `cluster-admin`) to the `default` ServiceAccount in that namespace, the attacker now owns the cluster. They can spin up privileged pods, mount the underlying worker node's root filesystem, read your secrets, and pivot anywhere they want. 

### The Fix: Zero Trust Kubernetes

As a Senior Engineer, you have to operate under the assumption that your application code *will* be compromised. Your infrastructure must contain the blast radius.

The fix requires two non-negotiable rules for your deployments:

1. **Disable Auto-mounting globally:** You must explicitly tell Kubernetes *not* to mount that token. If your app just serves web traffic or talks to a database, it doesn't need to know it's running in Kubernetes.
2. **Explicit, Least-Privilege Accounts:** For the rare pods that *do* need to talk to the API (like monitoring agents or CI/CD runners), create a dedicated `ServiceAccount` and a highly restricted `RoleBinding` just for them. 

### The Code

Here is the standard, vulnerable garbage you see in basic deployment templates:

```yaml
# THE BAD WAY (A Backdoor by Default)
# The default SA token is automatically mounted into this container.
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vulnerable-webapp
spec:
  template:
    spec:
      containers:
      - name: app
        image: my-app:latest
        ports:
        - containerPort: 8080
```

Here is the production-ready, DevSecOps-approved approach. We explicitly disable the token mount and assign a specific, unprivileged ServiceAccount (which you should create without any RoleBindings attached).

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/kubernetes/Kubernetes-RBAC-Nightmare-Why-Your-Default-ServiceAccount-is-a-Backdoor/](https://www.valtersit.com/guides/kubernetes/Kubernetes-RBAC-Nightmare-Why-Your-Default-ServiceAccount-is-a-Backdoor/)**
