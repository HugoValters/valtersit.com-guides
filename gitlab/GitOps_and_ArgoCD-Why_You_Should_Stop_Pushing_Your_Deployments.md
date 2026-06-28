> 📖 **Original article:** [GitOps and ArgoCD: Why You Should Stop Pushing Your Deployments](https://www.valtersit.com/guides/gitlab/GitOps_and_ArgoCD-Why_You_Should_Stop_Pushing_Your_Deployments/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

If you have spent fifteen years watching Jenkins or GitHub Actions runners blindly shove YAML into a Kubernetes cluster, you have seen the primary source of infrastructure entropy. The traditional 'Push' model of CD is an architectural relic of the VM era, forced into a containerized world. It relies on a 'Fire and Forget' philosophy where a CI runner—often sitting outside your cluster's security boundary—holds a high-privilege [KUBECONFIG] and executes 'kubectl apply' or 'helm upgrade'. This is not orchestration; it is a series of uncoordinated commands that assume the cluster's state matches the runner's ephemeral view of reality.

In a 'Push' model, the CI runner is the 'God Actor'. It needs network line-of-sight to your API server and long-lived credentials to modify resources. From a DevSecOps perspective, this is a massive attack surface. If your CI/CD provider is compromised, the attacker doesn't just get your code; they get a direct, authenticated tunnel into your production control plane. Furthermore, 'Push' CD has no concept of 'State Reconciliation'. If a developer manually edits a Deployment using 'kubectl edit' to fix a production outage, the CI runner has no idea that 'Drift' has occurred. The next time the pipeline runs, it might overwrite the fix, or worse, fail because the live state has diverged too far from the template.

GitOps, championed by tools like ArgoCD, flips this geometry on its head. It moves from 'Push' to a 'Pull' or 'Reconciliation' model. Your Kubernetes cluster becomes self-aware. An agent—ArgoCD—runs inside the cluster and treats a Git repository as the 'Single Source of Truth'. Instead of a runner pushing changes, ArgoCD continuously monitors the Git repo and compares the desired state [the YAML in Git] with the live state [the resources in K8s]. If they don't match, ArgoCD detects 'OutOfSync' status. Depending on your configuration, it can automatically 'Sync' the cluster back to the desired state, effectively nuking any manual 'ClickOps' or 'kubectl' interventions.

The 'Pull' model provides a clean 'Control Plane Isolation'. The only entity that needs 'cluster-admin' permissions is the ArgoCD instance running inside the cluster. Your CI pipeline's job ends at the 'Git Commit'. The CI runner doesn't need to know where the cluster is, what its IP is, or how to authenticate with its OIDC provider. It simply updates a tag in a YAML file and pushes to a 'Manifest Repo'. This decoupling ensures that even if your GitHub Actions runner is compromised, the attacker is trapped in the Git layer. They can propose changes, but they cannot execute arbitrary commands against your running pods. This 'Asynchronous' deployment pipeline is the only way to achieve true 'Least Privilege' at scale.

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/gitlab/GitOps_and_ArgoCD-Why_You_Should_Stop_Pushing_Your_Deployments/](https://www.valtersit.com/guides/gitlab/GitOps_and_ArgoCD-Why_You_Should_Stop_Pushing_Your_Deployments/)**
