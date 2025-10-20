# Ansible — Notes, Rationale, Configuration & Ad‑hoc Commands



---

## 1) Why Ansible — short, professional answer

* **Agentless & simple:** Ansible uses SSH (or WinRM for Windows) and does not require an agent on managed nodes, simplifying maintenance and security exposure.
* **YAML playbooks:** Uses human‑readable YAML for playbooks (easy to learn, review, and version control).
* **Declarative + procedural:** You can write declarative state (install package) plus imperative tasks where needed.
* **Large ecosystem & modules:** Rich module library for cloud, networking, containers, package managers, users, files, templates, etc.
* **Idempotent by design:** Modules implement desired‑state behavior (see section on idempotence).
* **Good fit for configuration management and application deployment:** Especially when combined with other IaC tools.

### Why not Puppet / Chef today (short reasons)

* **Architecture & complexity:** Puppet and Chef historically use a client‑server model or agent on nodes — more moving parts, more maintenance.
* **Learning curve and DSL:** Puppet and Chef use their own DSLs (or Ruby for Chef), while Ansible uses YAML which is more familiar to many engineers.
* **Operational agility:** Ansible's push model and ad‑hoc command style are faster for many operational tasks and troubleshooting.
* **Community & momentum:** Modern cloud projects and many DevOps teams prefer Ansible for ease of integration and speed of iteration.

> Note: Puppet and Chef are still used in some enterprises — not extinct — but Ansible is currently more common in many cloud‑native and agile teams.

---

## 2) Idempotence — what it is and why it matters

**Definition:** A task is idempotent if running it multiple times results in the same state as running it once (no unintended side effects after the first run).

**Why it matters:** Reliable automation — you can safely re‑run playbooks in CI/CD or production without accidentally creating duplicate users, reinstalling packages unnecessarily, or corrupting state.

**Example:** `ansible.builtin.apt: name=vim state=present` — running this repeatedly will install `vim` once and then report `changed: false` on subsequent runs if already present.

---

## 3) Ansible vs Terraform — both are IaC but different responsibilities

* **Terraform**

  * Purpose: *Provisioning* infrastructure (cloud resources like VMs, networks, load balancers, DNS records).
  * Model: Declarative desired state (HCL) with a focus on resource graph, lifecycle, dependency graph, state file, and drift detection.
  * Example: Create an AWS EC2 instance, VPC, subnets, and security groups.

* **Ansible**

  * Purpose: *Configuration management* and application deployment across provisioned machines. Also supports some provisioning modules but not meant to replace Terraform's stateful approach.
  * Model: Procedural/declarative tasks in YAML playbooks, push‑based. Good at configuring OS, installing packages, templating configs, deploying apps.
  * Example: Install nginx on an EC2 instance, deploy app code, configure systemd service.

**Clear comparative example**

* **Terraform**: create `aws_instance.web` + `aws_security_group.sg_http` (opens port 80) + `aws_eip` (associate public IP)
* **Ansible**: after Terraform creates the VM and SSH key, use Ansible to `apt install nginx`, copy `/etc/nginx/sites-enabled/site.conf`, start `nginx` service, deploy web assets.

**Bottom line:** Use Terraform for *infrastructure provisioning*, Ansible for *config & app deployment*. Many teams use both in tandem.

---

## 4) Ansible uses a push model — what that means

* **Push model:** Control node (where you run Ansible) opens SSH connections to managed nodes and pushes tasks/playbooks to them.
* **Contrast with pull model:** Tools like Puppet/Chef can run an agent on the node which pulls configuration from a central server on schedule.

**Implication:** Instant execution, great for one‑off changes and ad‑hoc tasks. Requires network/SSH access from control node to targets.

---

## 5) Interview‑relevant Ansible topics (short checklist)

* Idempotence and why it matters
* Inventory formats (INI, YAML, dynamic inventories) and host patterns
* `ansible` vs `ansible-playbook` vs `ansible-console`
* Modules vs raw commands; common modules: `apt`/`yum`, `package`, `service`, `copy`, `template`, `file`, `user`, `git`, `uri`, `lineinfile`, `blockinfile`, `cron`, `docker_container`, `k8s`, `uri`
* Roles, tasks, handlers, vars, defaults, templates, files, and meta
* Vault (encrypting secrets) and best practices
* Check mode (`--check`) and dry‑runs
* Handlers and `notify` semantics
* Tags and selective runs (`--tags`, `--skip-tags`)
* Error handling: `failed_when`, `ignore_errors`, `rescue`, `block` constructs
* Fact gathering and controlling it (`gather_facts: false/true`)
* Ansible Galaxy and reuse of community roles
* Performance: forks, pipelining, and connection optimizations
* Dynamic inventory (cloud provider plugins) and inventory caching

---

## 6) Relevance of Ansible in Docker & Kubernetes world

* **Docker:** Ansible can build images, push to registries, manage containers (via `docker_container` and `community.docker` modules), and orchestrate multi‑container deployment for VMs.
* **Kubernetes:** For day‑to‑day cluster config (e.g., change configmaps, apply manifests), teams often use `kubectl`/Helm/ArgoCD. Ansible has `k8s` and `k8s_info` modules and can manage manifests, but for GitOps workflows, tools like ArgoCD or Flux are more common.
* **Where Ansible shines:** hybrid environments, configuring the host OS that runs containers, CI/CD tasks, and bootstrapping clusters. It’s complementary, not obsolete.

---

## 7) Ansible configuration — files, location & precedence

### Where Ansible looks for configuration (precedence high → low)

1. **Command line options** (e.g., `ANSIBLE_CONFIG` env var overrides below) — highest precedence when specified.
2. **Environment variable** `ANSIBLE_CONFIG` — specify exact config file path.
3. **Current working directory** `./ansible.cfg` — if present, used next.
4. **Home directory** `~/.ansible.cfg` — user level config.
5. **System default** `/etc/ansible/ansible.cfg` — global default shipped with system packages.

> Ansible uses the first `ansible.cfg` it finds in that order.

### Common config options (meaning and why you might use them)

```ini
[defaults]
inventory = ./inventory          # default inventory path used by `ansible` and `ansible-playbook`
host_key_checking = False       # disable strict host key checking for dynamic infra/dev (use cautiously)
retry_files_enabled = False     # disable creating retry files (avoid clutter in CI)
log_path = ./ansible.log        # enable logging for audit/debug
deprecation_warnings = False    # suppress some warnings if desired
forks = 50                      # parallelism, increase for many hosts
remote_user = ubuntu            # default remote user if not provided per host
private_key_file = ~/.ssh/id_rsa # default ssh key if you reuse one

# connection specific
[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s  # speed up connections
pipelining = True                # reduce overhead for module execution

# vault
[vault]
# vault options
```

**Why set these?**: to optimize performance (forks, pipelining), control SSH behavior, centralize inventory, and ensure predictable behavior across environments.

---

## 8) Inventory — what it contains and patterns

### Formats

* **INI style** (simple, common)
* **YAML style** (structured)
* **Dynamic inventory scripts/plugins** (aws_ec2, azure_rm, gcp, etc.)

### Example INI inventory (enhanced)

```ini
[all]
13.233.225.159 ansible_user=ubuntu ansible_ssh_private_key_file=/home/asad/.ssh/id_rsa_ubuntuserver

[ubuntuserver]
13.233.225.159 ansible_user=ubuntu ansible_ssh_private_key_file=/home/asad/.ssh/id_rsa_ubuntuserver

[prod:children]
ubuntuserver

[webservers]
13.233.225.159 ansible_user=ubuntu ansible_ssh_private_key_file=/home/asad/.ssh/id_rsa_ubuntuserver

[vars]
# group vars can also be in group_vars/ folder
```

### What inventory entries can include

* Hostname or IP
* `ansible_user`, `ansible_port`, `ansible_ssh_private_key_file`
* Host variables and group variables (via `host_vars/` and `group_vars/` directories)
* Connection variables (`ansible_connection=ssh`, `ansible_python_interpreter=/usr/bin/python3`)

---

## 9) `ansible --version` — what it tells you

When you run `ansible --version`, you get: the Ansible version, Python version, configured location of `ansible.cfg` (which file is being used), and the module location. This helps debug which config is active and environment details.

---

## 10) Ad‑hoc commands — format and many examples (explained)

**Ad‑hoc command format**

```
ansible <host-pattern> [-i inventory] -m <module> -a "<module-args>" [flags]
```

* `<host-pattern>`: `all`, group name, host name, pattern expressions (`webservers:dbservers`, `prod:!centos`), or glob/wildcard `*`.
* `-i inventory`: path to inventory if not using `ansible.cfg` default.
* `-m`: module name (`ping`, `command`, `shell`, `apt`, `yum`, `copy`, etc.).
* `-a`: module arguments or raw command.
* Common flags: `-b` (become), `--limit`, `--forks`, `-u` (remote user), `--private-key`.

### Host pattern variants to know

* `all` — every host in inventory
* `webservers` — group named `webservers`
* `webservers:dbservers` — intersection (hosts in both groups)
* `prod:!centosserver` — hosts in `prod` but not in `centosserver`
* `host[1:3]` or `web[1:3]` — numeric ranges (if your inventory uses those hostnames)
* `*` or `"*"` — wildcard (note shell quoting)

---

### Essential ad‑hoc commands (with explanation)

1. **Check Ansible version & active config**

```bash
ansible --version
```

*Explains which `ansible.cfg` is used, ansible version, python version.*

2. **Ping (reachability)**

```bash
ansible all -m ping
ansible -i inventory all -m ping
```

*Uses the `ping` module (not ICMP). Ensures Python & connectivity.*

3. **Ping with patterns**

```bash
ansible "*" -m ping
ansible prod -m ping
ansible 'prod:!centosserver' -m ping
```

4. **Limit execution**

```bash
ansible all -m ping --limit ubuntuserver
```

5. **Run a raw shell/command (no module)**

```bash
ansible all -a "hostname"
ansible all -a "uptime"
ansible all -a "free -m"
ansible all -a "whoami"
ansible all -a "lsb_release -a"
ansible all -a "cat /etc/os-release"
```

*These run the remote shell command via the default connection plugin. Use sparingly — prefer modules.*

6. **Install package (module way — idempotent)**

```bash
ansible all -m apt -a "name=vim state=present"  # Debian/Ubuntu
ansible all -m yum -a "name=vim state=present"  # CentOS/RHEL
```

*Module handles idempotence and package manager specifics.*

7. **Become (run as root) for privileged actions**

```bash
ansible all -m apt -a "name=vim state=present" -b
```

8. **Create user (idempotent via user module)**

```bash
ansible all -m user -a "name=deploy_user state=present groups=www-data" -b
```

9. **Copy a file to remote**

```bash
ansible all -m copy -a "src=./index.html dest=/tmp/index.html mode=0644" -b
```

10. **Template rendering (ad‑hoc using template module is rare; prefer playbook)**

```bash
ansible all -m template -a "src=templates/site.j2 dest=/etc/nginx/sites-available/site1" -b
```

11. **Run a command and register output (ad‑hoc with callback to controller)**
    *Ad‑hoc registration is not practical; use playbooks to register variables.*

12. **List modules and search**

```bash
ansible-doc -l
ansible-doc -l | grep apt
ansible-doc user
ansible-doc -s user  # short usage
```

13. **Count modules**

```bash
ansible-doc -l | wc -l
```

14. **Run command on hosts with async**

```bash
ansible all -m shell -a "sleep 300" -B 600 -P 0  # run async task (controller returns immediately)
```

15. **Run commands using specific inventory file**

```bash
ansible -i ./inventory webservers -m ping
```

16. **Check facts gathering**

```bash
ansible all -m setup | head -n 50  # gather facts
```

17. **Check listening ports via ss**

```bash
ansible all -m shell -a "ss -tulnp | grep :80" -b
```

18. **Copy SSH key for passwordless access**

```bash
ansible all -m authorized_key -a "user=ubuntu key='{{ lookup("file","~/.ssh/id_rsa.pub") }}'" -b
```

19. **Run ad‑hoc with extra vars**

```bash
ansible all -m shell -a "echo $MYVAR" -e "MYVAR=hello"
```

20. **Use forks (parallelism)**

```bash
ANSIBLE_FORKS=50 ansible all -m ping
ansible all -m ping --forks 50
```

21. **Use sudo as another user**

```bash
ansible all -m shell -a "whoami" -b -u ubuntu --become-user=root
```

22. **Debug / Verbose**

```bash
ansible all -m ping -v   # one -v for verbose
ansible-playbook site.yml -vvv  # increase verbosity
```

---

## 11) Best practices & production‑grade tips (short)

* Use **roles** to organize reusable functionality: `roles/<role_name>/{tasks,handlers,templates,files,vars,defaults}`.
* Use **templates** for config files, not `copy` when values vary per host.
* Keep secrets in **Ansible Vault** and don’t commit plaintext credentials.
* Use **handlers** for services and trigger them with `notify` to avoid unnecessary restarts.
* Use **tags** to run only parts of playbooks in CI or for quick changes.
* Use **`--check`** to run dry‑runs, but remember some modules can’t fully simulate changes.
* Use **CI pipelines** to lint and test playbooks (`ansible-lint`, `molecule`) before production runs.
* Use **dynamic inventory** for cloud environments; cache inventory for performance when needed.
* Keep inventory and secrets out of public repos; use private repos or secrets managers.

---

## 12) Rollback strategies

* **Idempotent tasks** usually make rollbacks simpler; implement reverse tasks (e.g., remove a config file, stop service) as separate playbooks or `--tags` that support rollback.
* Keep **versioned releases** of artifacts (git tags, docker image tags); to rollback, redeploy previous tag.
* Use **handlers + atomic switches** (write new config to file, test `nginx -t`, switch symlink to new config, reload), so you can revert symlink if test fails.
* Use **git** as source of truth for site code so rolling back is `git checkout <older-tag>` via `ansible.builtin.git`.

---

## 13) Useful directories & file layout for a repo (recommended)

```
ansible-repo/
├── inventory
├── group_vars/
│   └── all.yml
├── host_vars/
├── roles/
│   └── nginx_site/
│       ├── tasks/main.yml
│       ├── templates/site.j2
│       ├── handlers/main.yml
│       └── defaults/main.yml
├── playbooks/
│   └── deploy_site.yml
└── README.md
```

---

## 14) Quick checklist to include in your GitHub README (so an interviewer/ reviewer can run)

1. Prereqs: `ansible` installed on control node, SSH key with access to targets.
2. Example inventory and how to run:

   ```bash
   ansible-playbook -i inventory playbooks/deploy_site.yml
   ```
3. How to change variables: `group_vars/all.yml` or `vars/main.yml`.
4. How to rollback: steps or tag to checkout.

---

## 15) Final: Add more ad‑hoc commands you should memorize (concise list)

* `ansible all -m ping` — reachability
* `ansible all -m setup` — gather facts
* `ansible all -m command -a "uptime"` — run command
* `ansible all -m shell -a "df -h"` — run shell (shell features enabled)
* `ansible all -m copy -a "src=./file dest=/tmp/file" -b` — copy as root
* `ansible all -m yum -a "name=httpd state=present" -b` — install on RHEL
* `ansible all -m apt -a "name=nginx state=present" -b` — install on Debian/Ubuntu
* `ansible all -m service -a "name=nginx state=started enabled=yes" -b` — manage services
* `ansible all -m user -a "name=deploy state=present" -b` — user create
* `ansible all -m git -a "repo=https://... dest=/var/www/site"` — clone repo
* `ansible all -m unarchive -a "src=/tmp/file.tar.gz dest=/opt/" -b` — extract tarball
* `ansible all -m uri -a "url=http://localhost:80 return_content=yes"` — http check
* `ansible all -m authorized_key -a "user=ubuntu key='{{ lookup('file', '~/.ssh/id_rsa.pub') }}'" -b` — push public key
* `ansible all -m file -a "path=/tmp/test state=directory mode=0755"` — create directory

---

