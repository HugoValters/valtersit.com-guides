> 📖 **Original article:** [Beyond .env: Managing Production Secrets Without Getting Hacked](https://www.valtersit.com/guides/gitlab/Beyond_env-Managing_Production_Secrets_Without_Getting_Hacked/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

If you have spent fifteen years watching developers treat their .env files like a secure vault, you have likely witnessed at least one catastrophic credential leak that started with an "oops, I forgot to add that to .gitignore" commit. The .env file is the most pervasive anti-pattern in modern software development. It creates a false sense of isolation while practically begging for accidental exposure. I have seen "senior" architects argue that encrypting secrets within the Git repository using tools like git-crypt or ansible-vault is a valid strategy. It isn't. It is an admission that you don't understand the permanence of Git history or the inevitability of key compromise.

The fundamental flaw with Git-based secrets—even encrypted ones—is that the repository is designed to be distributed. Every developer who clones the repo has a full copy of the encrypted blobs. If your master encryption key is leaked, rotated poorly, or if the underlying algorithm is found to have a weakness five years from now, your entire historical record of production passwords, API keys, and database credentials is laid bare. You are essentially carrying around a ticking time bomb of technical debt that can be detonated by anyone who manages to exfiltrate a single developer's workstation. 

Furthermore, the .env model fails at scale. It offers zero auditability, no automatic rotation, and no "Least Privilege" controls. You have a single file containing every secret the application might ever need, which is then injected into the environment where it sits in memory, visible to any process that can inspect the proc filesystem. In a professional DevSecOps environment, we move beyond these amateurish patterns and treat secrets as short-lived, dynamic entities that are fetched on-demand from a hardened, centralized authority like HashiCorp Vault or AWS Secrets Manager.

Integrating a real secrets manager into your CI/CD pipeline is the first step toward sanity. Instead of passing a bucket of variables through your GitHub Actions or GitLab CI runners, you use the runner's identity—often via OIDC [OpenID Connect]—to authenticate with the secrets manager. The pipeline fetches only the specific keys it needs for that specific build or deployment step. This means your CI/CD logs don't become a graveyard of obfuscated [SECRET] masks that can be easily bypassed by a clever developer using a "base64" or "char-split" echo command. 

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/gitlab/Beyond_env-Managing_Production_Secrets_Without_Getting_Hacked/](https://www.valtersit.com/guides/gitlab/Beyond_env-Managing_Production_Secrets_Without_Getting_Hacked/)**
