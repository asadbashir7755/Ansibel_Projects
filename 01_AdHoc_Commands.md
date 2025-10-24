# Ansible — Notes, Rationale, Configuration & Ad-hoc Commands (Complete & Simplified)

---

## 1) Why Ansible

* **Agentless & simple:** No agent needed; works via SSH.
* **Uses YAML:** Easy to read/write.
* **Idempotent:** Safe to re-run.
* **Modular:** Rich library of modules for everything.
* **Declarative + procedural:** Define state, but also automate steps.
* **Ideal for:** Configuration management, app deployment, and automation.

### Why not Puppet / Chef

* Require agents and central servers.
* Complex DSLs (Ruby-like syntax).
* Ansible easier, faster, YAML-based.

---

## 2) Idempotence

**Meaning:** Re-running the same task doesn’t change system state after first success.
**Example:** `state=present` for packages won’t reinstall again.
**Benefit:** Safe automation and repeatable deployments.

---

## 3) Ansible vs Terraform

| Tool          | Purpose                                   | Example Use                 |
| ------------- | ----------------------------------------- | --------------------------- |
| **Terraform** | Provision infrastructure (EC2, VPC, etc.) | Create cloud resources      |
| **Ansible**   | Configure systems and deploy apps         | Install Apache, deploy site |

**Use both:** Terraform creates infra → Ansible configures it.

---

## 4) Push Model

Ansible pushes configurations from the control node to managed nodes using SSH.
**No agent needed**, faster for ad-hoc and small environments.

---

## 5) Interview Topics

* Idempotence
* Inventory formats (INI/YAML)
* Modules vs raw commands
* Playbooks, roles, vars, handlers
* Vault (secret management)
* Handlers + notify
* Tags, check mode, forks
* Dynamic inventory (AWS, etc.)
* Error handling blocks (`rescue`, `ignore_errors`)

---

## 6) Docker & Kubernetes Relevance

* **Docker:** Build images, deploy containers, manage multi-host setups.
* **Kubernetes:** Can apply manifests with `k8s` module but not a full GitOps replacement.
* **Best use:** Host configuration + hybrid infra automation.

---

## 7) Ansible Configuration (ansible.cfg)

**File lookup order:**

1. Command-line `ANSIBLE_CONFIG`
2. `./ansible.cfg`
3. `~/.ansible.cfg`
4. `/etc/ansible/ansible.cfg`

**Common settings:**

```ini
[defaults]
inventory = ./inventory
host_key_checking = False
retry_files_enabled = False
forks = 50
remote_user = ubuntu
private_key_file = ~/.ssh/id_rsa
log_path = ./ansible.log

[ssh_connection]
pipelining = True
```

---

## 8) Inventory Basics

**INI Format:**

```ini
[ubuntuserver]
13.233.225.159 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa
```

**YAML Format:**

```yaml
all:
  hosts:
    13.233.225.159:
      ansible_user: ubuntu
      ansible_ssh_private_key_file: ~/.ssh/id_rsa
```

**Supports:** host_vars, group_vars, children, dynamic inventories.

---

## 9) ansible --version

Shows: version, python version, active `ansible.cfg` path, and module location.

---

## 10) Ad-hoc Commands (with Examples)

**Format:**

```bash
ansible <host-pattern> -m <module> -a "args" -b
```

### Common Examples

* Ping hosts → `ansible all -m ping`
* Check uptime → `ansible all -a "uptime"`
* Install package → `ansible all -m apt -a "name=nginx state=present" -b`
* Start service → `ansible all -m service -a "name=nginx state=started enabled=yes" -b`
* Copy file → `ansible all -m copy -a "src=index.html dest=/var/www/html/index.html" -b`
* Create user → `ansible all -m user -a "name=deploy state=present" -b`
* Gather facts → `ansible all -m setup | head -n 20`
* Async → `ansible all -m shell -a "sleep 300" -B 600 -P 0`

**Docs & modules:**

```bash
ansible-doc -l | grep apt
ansible-doc user
```

---

## 11) Best Practices

* Use **roles** for structure.
* Keep **vars** and **defaults** separated.
* Use **templates** for config files.
* Use **Vault** for secrets.
* Always use **handlers + notify**.
* Lint with `ansible-lint`.
* Test with `molecule`.
* Use **tags** to target tasks.

---

## 12) Rollback

* Maintain **idempotent** playbooks.
* Keep **versioned artifacts**.
* Use **separate rollback playbooks**.
* Validate before reloads (`nginx -t` pattern).

---

## 13) Recommended Project Layout

```
ansible-repo/
├── inventory
├── group_vars/
│   └── all.yml
├── roles/
│   └── apache_site/
│       ├── tasks/main.yml
│       ├── templates/site.j2
│       ├── handlers/main.yml
│       └── defaults/main.yml
├── playbooks/
│   └── deploy.yml
└── ansible.cfg
```

---

## 14) Quick Checklist for README

1. Install Ansible.
2. Add inventory.
3. Run: `ansible-playbook -i inventory playbooks/deploy.yml`
4. Change variables in `group_vars/all.yml`.

---

## 15) Ad-hoc Command Quick List

```
ansible all -m ping
ansible all -m setup
ansible all -m command -a "df -h"
ansible all -m apt -a "name=apache2 state=present" -b
ansible all -m service -a "name=apache2 state=started" -b
ansible all -m copy -a "src=index.html dest=/var/www/html/" -b
ansible all -m file -a "path=/tmp/test state=directory"
ansible all -m git -a "repo=https://... dest=/var/www/site" -b
ansible all -m uri -a "url=http://localhost:80 return_content=yes"
ansible all -m authorized_key -a "user=ubuntu key='{{ lookup('file', '~/.ssh/id_rsa.pub') }}'" -b
```

---

## Summary

**Ansible = Simple + Powerful + Idempotent.**
Perfect for automating server config, deployments, and integrations with tools like Docker, AWS, and Terraform.
