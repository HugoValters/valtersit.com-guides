> 📖 **Original article:** [Your Ansible Playbook Isn't a Fancy Bash Script. Stop Writing It Like One.](https://www.valtersit.com/guides/automation/your-ansible-playbook-isnt-a-fancy-bash-script-stop-writing-it-like-one/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

### 1. Introduction: The Confession

#### Let's Be Honest With Ourselves

We've all done it. You're staring down a deadline, the task is simple enough: "ensure this line is in that config file." The Ansible module documentation is a click away, a pristine world of structured parameters and states. But the familiar comfort of a Bash one-liner calls to you. It whispers sweet nothings in your ear: `grep -q 'some_setting' /etc/config || echo 'some_setting=true' >> /etc/config`.

It feels fast. It feels clever. It's also a trap.

This approach is the automation equivalent of patching a server by yanking the power cord. It might work once, but it's a terrible habit that builds a mountain of technical debt. It's the mindset of a script-kiddie, not an infrastructure engineer. You’re treating Ansible as a glorified SSH-for-loop, a fancy way to run remote scripts.

The `shell` and `command` modules are crutches, not tools for daily use. Relying on them for routine configuration undermines the entire purpose of Ansible: declarative, idempotent, and auditable configuration management. When you lean on `shell`, you're not managing state; you're just executing a sequence of commands. You could have done that with a `for` loop over an inventory file in 1999. It's time to level up.

---

### 2. The Core Sin: Imperative vs. Declarative

#### You're Thinking in Verbs, Not Nouns

The fundamental mistake that leads to `shell`-littered playbooks is a failure to grasp the imperative vs. declarative paradigm shift.

*   **Imperative (Bash/Shell/Scripts):** You provide a precise sequence of commands to execute. You are describing *how* to achieve a result. `"Check if the file /etc/sysctl.conf contains the line 'vm.swappiness=10'. If it doesn't, append it. Then, run the command to reload the configuration."` This is a list of verbs: check, append, run.

*   **Declarative (Ansible):** You define the desired final state. You are describing *what* the end result should be, and you trust the tool to figure out the *how*. `"The system's configuration must have 'vm.swappiness' set to '10'.` This is a statement of nouns and their attributes: a configuration parameter, a value.

This distinction is not academic; it has massive practical implications.

##### Idempotency: The Holy Grail

This is the most critical concept you are violating. An operation is idempotent if running it once has the same effect as running it a hundred times. A well-written Ansible module is idempotent by design. You run `ansible.posix.sysctl` to set a value. The first time, it reports `changed=true` and sets the value. The next 99 times, it checks, sees the value is already correct, and reports `changed=false`, doing nothing.

Your naive `echo "..." >> /etc/config` shell command is the opposite of idempotent. Run it ten times, and you get ten duplicate lines in your config file. Your system is now in a broken, unintended state.

##### State Reporting: The Currency of Automation

Ansible's `ok`, `changed`, and `failed` statuses are gold. They are the auditable, machine-readable record of what your automation run actually did. In a CI/CD pipeline or a change management system, the difference between `ok` and `changed` is everything. It tells you if the system drifted from its desired state and was corrected.

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/automation/your-ansible-playbook-isnt-a-fancy-bash-script-stop-writing-it-like-one/](https://www.valtersit.com/guides/automation/your-ansible-playbook-isnt-a-fancy-bash-script-stop-writing-it-like-one/)**
