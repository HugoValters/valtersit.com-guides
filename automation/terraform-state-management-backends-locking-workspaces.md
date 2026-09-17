> 📖 **Original article:** [Terraform State Management: Backends, Locking, Workspaces](https://www.valtersit.com/guides/automation/terraform-state-management-backends-locking-workspaces/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

# Terraform State Management: Remote Backends, Locking, and Workspaces

A platform engineer ran `terraform apply` from a laptop on hotel wifi. The connection dropped mid-write. Terraform had already destroyed 40 resources and recreated a third of them before the state file was truncated to zero bytes. The team spent six hours reconciling 340 resources by hand — reading AWS console pages, cross-referencing tags, and writing `terraform import` commands until the plan came back clean.

That is the cheapest possible version of this story. The expensive version involves a state file in a public GitHub repo.

Terraform state is the single source of truth for your cloud estate. It contains plaintext secrets. It is a mutable, lockable, versionable artifact that most teams treat like a build cache. This guide covers the three failure modes that actually hurt — state loss, state race, state exfiltration — and the operational runbook around backends, locking, and workspaces.

> Anyone with write access to your state bucket has root on your CI runner. —[Valter Sit](https://valtersit.com/)

:::note[TL;DR]
- State files store **plaintext** for everything you pass to a resource. `sensitive = true` masks CLI output, not the JSON.
- Use a remote backend with locking from day one. Local state is for sandboxes you destroy in the same session.
- On AWS, `use_lockfile = true` (Terraform 1.10+) makes the DynamoDB lock table obsolete. Stop creating one.
- Workspaces are a naming convention, not an isolation boundary. Use directories for prod/staging.
- `moved`, `import`, and `removed` blocks replace 90% of the `terraform state mv/rm/push` surgery you're doing.
:::

## Prerequisites

- Terraform 1.10 or newer (workspace-based locking via `use_lockfile` requires it), or OpenTofu 1.7+ if you want state encryption
- A cloud account with permission to create a storage bucket and an IAM role
- `aws-cli` v2, `jq`, and `terraform` on your PATH
- Basic familiarity with writing `terraform plan` and reading a diff

## What's Actually in a State File (And Why It Scares People)

The state JSON is not opaque. It has a `version`, a `terraform_version`, a `serial` that increments on every write, a `lineage` UUID that ties a state file to its backend key, `outputs`, and a `resources` array. The `serial` and `lineage` fields are what stop you from accidentally pointing two environments at the same key — the lineage mismatch error is a safety feature, not a bug.

The uncomfortable part is `attributes`. Everything you hand to a resource is serialized there verbatim.

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/automation/terraform-state-management-backends-locking-workspaces/](https://www.valtersit.com/guides/automation/terraform-state-management-backends-locking-workspaces/)**
