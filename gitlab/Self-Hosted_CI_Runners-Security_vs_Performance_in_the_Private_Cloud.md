> 📖 **Original article:** [Self-Hosted CI Runners: Security vs. Performance in the Private Cloud](https://www.valtersit.com/guides/gitlab/Self-Hosted_CI_Runners-Security_vs_Performance_in_the_Private_Cloud/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

If you have spent fifteen years managing build infrastructure, you know that 'Convenience' is the primary vector for internal network compromise. GitHub-hosted runners are the industry standard for a reason: they provide a clean, isolated virtual machine for every single job. When the job finishes, the VM is nuked. It is the ultimate 'Defense in Depth' for CI/CD. But the moment your builds start taking thirty minutes because of IOPS throttling or you realize you are spending ten thousand dollars a month on 'Minutes,' the business side starts asking about self-hosted runners. This is where the real security nightmare begins.

The fundamental problem with self-hosted runners is Isolation—or the lack thereof. Most teams start by spinning up a 'persistent' Linux box, installing the GitHub Actions runner agent, and calling it a day. You have just created a 'Pet' that lives inside your private VPC, has a long-lived identity, and likely has access to internal resources like Artifactory, private databases, or your Kubernetes management API. If a developer unknowingly pulls in a malicious dependency—or if an attacker submits a clever Pull Request that executes a shell script—that code is now running on a machine with a direct line of sight to your crown jewels.

Unlike GitHub-hosted runners, which are ephemeral by design, many self-hosted setups are static. They accumulate state. They have local caches in [/home/runner/work] that persist between builds. An attacker doesn't even need to escape the runner process; they just need to drop a malicious binary in a shared cache or modify the [.bashrc] of the runner user to intercept the next build's environment variables. This is 'Lateral Movement' made easy. You aren't just running code; you are hosting a permanent residency for potential threats.

The Escape Risk is not theoretical. I have seen 'security audits' where a simple [curl] from a CI job revealed the internal metadata service of the cloud provider, exposing IAM roles that were far too permissive. From there, the attacker doesn't just own the build; they own the AWS account. If your runner is sitting in a 'Management' subnet because it 'needs to deploy to K8s,' you have effectively bypassed your entire firewall strategy. A compromised build can use the runner's identity to scan your internal network, pivot to other instances, or exfiltrate sensitive data from internal S3 buckets that aren't exposed to the public internet.

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/gitlab/Self-Hosted_CI_Runners-Security_vs_Performance_in_the_Private_Cloud/](https://www.valtersit.com/guides/gitlab/Self-Hosted_CI_Runners-Security_vs_Performance_in_the_Private_Cloud/)**
