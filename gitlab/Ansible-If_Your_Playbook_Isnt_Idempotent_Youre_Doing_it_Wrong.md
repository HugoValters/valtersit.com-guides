> 📖 **Original article:** [Ansible: If Your Playbook Isn't Idempotent, You're Doing it Wrong](https://www.valtersit.com/guides/gitlab/Ansible-If_Your_Playbook_Isnt_Idempotent_Youre_Doing_it_Wrong/)
> *Mirror of the full guide published on [valtersit.com](https://www.valtersit.com)*

---

If you have spent fifteen years managing fleets of servers, you have seen the horror of the 'run-once' automation script. Ansible is often marketed as the 'simple' alternative to Puppet or Chef, but that simplicity is a trap for the uninitiated. The core philosophy of any configuration management tool must be idempotency—the ability to apply the same operation multiple times and achieve the same result without changing the state after the first successful run. If your playbook shows 'changed=1' every time you run it, you haven't written automation; you’ve written a ticking time bomb.

A non-idempotent playbook is a liability because it breaks the feedback loop. When I run a 'terraform plan' or an 'ansible-playbook --check', I need to know exactly what is going to happen. If your tasks are constantly 'changing' files or restarting services regardless of their current state, I can no longer distinguish between a legitimate configuration drift and your poorly written code. This is why the 'shell' and 'command' modules are the primary indicators of a junior engineer. Using 'shell: systemctl restart nginx' at the end of every task is a crime against infrastructure. You are forcing a service disruption every time the automation runs, regardless of whether the configuration actually changed.

Native modules like 'apt', 'yum', or 'template' are designed with state-checking logic built into the Python code that runs on the managed node. When you use the 'apt' module to install 'nginx', Ansible doesn't just run 'apt-get install'. It queries the local DPKG database first. If Nginx is already there and at the correct version, the module returns 'ok' and moves on. This is the 'Desired State' model. You are declaring that Nginx [should] be present, not commanding the server to [install] it.

Let's talk about the performance tax. Ansible is notoriously slow compared to Go-based agents or even the old-school Ruby-based Puppet. Every single task in your playbook involves a series of high-latency operations: the master serializes a Python script and its arguments into a JSON blob, opens an SSH connection, transfers the blob to a temporary directory on the managed node [usually ~/.ansible/tmp], unpacks it, invokes the local Python interpreter, executes the logic, captures the JSON output from stdout, and sends it back over the wire. If you have 500 tasks, you are doing this 500 times. On a high-latency network or a box with a slow CPU, this Python overhead becomes the primary bottleneck. This is why we use 'pipelining' to keep the SSH connections open, but the CPU cycles spent on the managed node to parse Python bytecode are significant.

---

> **⚠️ TRUNCATED** — This is a shortened mirror.
> Full guide (with all configs, diagrams and examples): **[https://www.valtersit.com/guides/gitlab/Ansible-If_Your_Playbook_Isnt_Idempotent_Youre_Doing_it_Wrong/](https://www.valtersit.com/guides/gitlab/Ansible-If_Your_Playbook_Isnt_Idempotent_Youre_Doing_it_Wrong/)**
