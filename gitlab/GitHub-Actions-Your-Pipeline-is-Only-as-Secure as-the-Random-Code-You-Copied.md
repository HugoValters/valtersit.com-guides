> 📖 **Original article:** [GitHub Actions: Your Pipeline is Only as Secure as the Random Code You Copied](https://www.valtersit.com/guides/gitlab/GitHub-Actions-Your-Pipeline-is-Only-as-Secure as-the-Random-Code-You-Copied/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

If you have spent fifteen years watching developers treat their CI/CD pipelines like a shared bucket for random scripts, you know exactly why the software supply chain is currently a smoking crater. GitHub Actions is the ultimate 'convenience' tool that has successfully convinced an entire generation of engineers that security is someone else's problem. The Marketplace is an architectural dumpster fire where amateurs blindly copy-paste YAML snippets from third-party actions maintained by people who probably don't even use two-factor authentication. 

Treating a GitHub Action like a safe, isolated sandbox is the first mistake. When you add a 'uses: someone/cool-action@v2' to your workflow, you aren't just calling a function. You are inviting an unvetted stranger to run arbitrary code on your runner, with access to your source code, your environment variables, and—depending on your incompetence—your production secrets. It is a Remote Code Execution [RCE] playground disguised as a productivity tool.

## The Marketplace Rant: Stop Trusting Strangers

The level of blind trust in the Marketplace is staggering. Developers who would normally spend three days debating a new dependency in their package.json will, without hesitation, add an action that has '1k stars' to their deployment pipeline. Stars are not a security audit. Most of these actions are poorly maintained wrappers around shell scripts or Node.js apps that haven't seen a dependency update since the runner images were on Ubuntu 18.04.

Every time you pull in a third-party action, you are expanding your attack surface exponentially. You are trusting that the maintainer's GitHub account isn't compromised, that their build process isn't hijacked, and that they haven't been social-engineered into adding a 'telemetry' feature that exfiltrates your [GITHUB_TOKEN] to a Discord webhook in Eastern Europe. If you haven't audited the source code in the [dist] folder of that action, you have no idea what is actually running on your infrastructure.

## Supply Chain Security: Tags are for Amateurs, SHAs are for Pros

The most common security failure in GitHub Actions is pinning by version tags. Using [@v2] or [@v2.3.4] is a security theater. In Git, tags are mutable. An attacker who gains write access to a popular action repository can simply delete the [@v2] tag and push it again, pointing to a malicious commit. Your pipeline will happily pull the 'new' version without a single warning.

The only way to achieve immutable, verifiable security is to pin actions by their full 40-character commit SHA. A SHA is a cryptographic hash of the repository's state at that exact moment. It cannot be spoofed or moved. 

[BAD EXAMPLE: Pinning by Tag]
uses: actions/checkout@v3

[GOOD EXAMPLE: Pinning by SHA]
uses: actions/checkout@8ade135a41bc03ea155e62e844d188df1ea18608 # v3.5.3

By using the SHA, you ensure that even if the maintainer goes rogue or their account is hijacked, your pipeline remains locked to the code you actually audited. Yes, it makes updates more tedious. Yes, it looks 'ugly' in your YAML. If you care about aesthetics over your production secrets, you shouldn't be in DevSecOps.

## The pull_request_target Disaster

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/gitlab/GitHub-Actions-Your-Pipeline-is-Only-as-Secure as-the-Random-Code-You-Copied/](https://www.valtersit.com/guides/gitlab/GitHub-Actions-Your-Pipeline-is-Only-as-Secure as-the-Random-Code-You-Copied/)**
