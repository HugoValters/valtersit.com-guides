> 📖 **Original article:** [IaC Security: Checkov and tfsec for Terraform and Helm](https://www.valtersit.com/guides/devsecops/iac-security-checkov-and-tfsec-for-terraform-and-helm/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

It was a Friday deploy. The PR looked fine — clean diff, two reviewers approved, CI was green. A single line in the `main.tf`: `acl = "public-read"` on an S3 bucket that was supposed to hold staging logs. It shipped at 4:47 PM. By Sunday morning, a scraper had found the bucket, a script had enumerated it, and the company was looking at a $40,000 AWS bill plus a data-breach notification to customers. The team didn't have a misconfigured bucket. They had a missing firewall — an IaC security gate.

If you're a working DevOps/SRE or platform engineer, you already know Terraform and Helm. You're now being told "security is your problem too" by people who couldn't spell "etag" last year. This guide is for you. We're going to cover two tools in depth — Checkov, the multi-framework policy-as-code scanner, and tfsec, the fast Terraform-focused scanner now part of Trivy. We'll also tackle a task that makes grown engineers weep: scanning Helm charts for misconfigurations before they hit your cluster.

What we're **not** going to do: pretend either tool is a silver bullet, or tell you to "shift left" without explaining what that costs you in build latency and engineer trust. Security scanning is a trade-off. This article sits at the intersection of developer experience and security gates — and you have to respect both, or neither works.

:::note[TL;DR]
- IaC misconfigurations are the modern firewall breach: automated, cheap to detect, expensive to ignore.
- Checkov scans Terraform, Helm, CloudFormation, and more with customizable policies; tfsec is a fast Terraform-only scanner (now Trivy-adjacent).
- Helm chart scanning is tricky — charts are templates, and the vulnerable values usually live in your environments, not the chart repo.
- Hard-fail CI gates on day one, and engineers will route around you. Log, triage, then block.
:::

**Prerequisites:**
- Terraform CLI installed (v1.5+ recommended, but the tools work with older versions).
- A Helm chart — either your own or a public one like `prometheus-community/kube-prometheus-stack`.
- Docker installed (for Trivy) or a local Go/Python environment.
- A GitHub/GitLab CI pipeline you can experiment on without angering the platform team. Ideally both.

---

## Why Your Terraform Is a Security Incident Waiting to Happen

Before we talk tools, we need to be honest about what we're dealing with. Terraform doesn't create security problems — it catalogs them. Every resource block in your HCL files is a promise to your cloud provider. If that promise says "world-readable," the provider delivers. The code review process, with three people rubber-stamping a 400-line diff on a Friday, is not a security control.

### The IaC Security Failure-Modes Catalog

Here's the list of issues I see in production Terraform code, ranked by how many breaches they've caused:

**Public-by-default resources.** S3 buckets with `acl = "public-read"`. Security groups with `cidr_blocks = ["0.0.0.0/0"]`. RDS instances with `publicly_accessible = true`. Each one is a five-second fix in code and a potential multi-million-dollar cleanup if it ships.

**Hardcoded secrets in HCL.** I've seen AWS access keys, database passwords, and Stripe tokens committed directly to main branches. They're in git history, forever, regardless of whether you delete them from the next commit. The only response is rotation. Not "we'll rotate next quarter." Immediately.

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/devsecops/iac-security-checkov-and-tfsec-for-terraform-and-helm/](https://www.valtersit.com/guides/devsecops/iac-security-checkov-and-tfsec-for-terraform-and-helm/)**
