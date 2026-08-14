> 📖 **Original article:** [Ansible Idempotency: Writing Playbooks That Never Fail Twice](https://www.valtersit.com/guides/automation/ansible-idempotency-writing-playbooks-that-never-fail-twice/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

# Ansible Idempotency in Practice: Writing Playbooks That Never Fail Twice

## Introduction — The Lie We Tell Ourselves About Ansible

It's 3:17 AM. Your phone buzzes with a PagerDuty alert. The production database replica is down, and the on-call engineer just ran your "emergency fix" playbook for the second time. It worked at 2:45 AM. Now it's hanging on a task that's trying to add a user to sudoers — except that user already exists, and the `command` module just appended a duplicate line that's causing a syntax error, which broke sudo entirely.

This is the reality of Ansible idempotency. Ansible markets itself as idempotent, but that's a half-truth. The modules are idempotent. Your playbooks are only as idempotent as the discipline you apply when writing them. The `command: curl | bash` incident above? That's not a hypothetical — it's a pattern I've seen destroy production environments more times than I care to count.

The real definition of idempotency in configuration management is: **same input → same state**. It's not "same input → same output." A playbook that runs successfully twice but leaves the system in a different state on the second run has failed its idempotency test.

This guide covers the six most common idempotency violations I've encountered running over 500 playbooks in production, and the exact patterns that fix them. You'll learn how to write playbooks that converge to a desired state — safely, predictably, and without requiring a rollback script every time you need to rerun them.

This is for teams running Ansible in production, not lab environments. If you're running playbooks against systems that matter, this guide will save you from the 3 AM wake-up calls.

---

## The Anatomy of Idempotency Failure — Why Playbooks Break on Second Run

### The Three Categories of Idempotency Violations

Every idempotency failure I've seen falls into one of three categories:

**Category 1: Non-deterministic operations** — Commands that produce different results each run. Think timestamps, random passwords, or operations that depend on external state.

**Category 2: Stateful operations with no state tracking** — Installing, copying, or modifying without checking current state. The `command` module is the prime offender here.

**Category 3: Order-dependent operations** — Tasks that assume a certain execution sequence. Run in a different order, and everything breaks.

### The "Works on My Machine" Paradox

Your playbook passes in staging. It passes in CI. It passes in production — once. Then someone manually changes a config file, or a package gets updated out-of-band, and suddenly the playbook that "always worked" is failing.

Environment drift is the #1 cause of "it worked yesterday" syndrome. Your playbook assumed a clean slate, but production is never clean. Here's a classic example:

```yaml
# BAD: This will fail on second run
- name: Add user to sudoers
  command: echo "{{ item }} ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
  loop: "{{ users }}"
```

The first run works. The second run appends duplicate entries. The third run breaks sudo entirely. And you've now locked yourself out of every server that ran this playbook.

### The Cost of Broken Idempotency

From my experience running hundreds of production playbooks, **40% of Ansible failures in production are idempotency-related**. That's not a rounding error — that's a systemic problem.

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/automation/ansible-idempotency-writing-playbooks-that-never-fail-twice/](https://www.valtersit.com/guides/automation/ansible-idempotency-writing-playbooks-that-never-fail-twice/)**
