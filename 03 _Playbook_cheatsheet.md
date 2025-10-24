# Ansible Playbook Cheat Sheet

## 📘 General Information

This document contains **all essential notes and examples** related to Ansible Playbooks — from writing, structuring, testing, and running them. Everything learned from the course or other resources will be continuously added here.

---

## 🧩 Why Playbooks?

Playbooks are YAML files in Ansible used to **automate configuration, deployment, and orchestration**. They describe *what* to do and *on which hosts*.

All playbooks are stored inside the directory:

```
04_Playbooks/
```

---

## 🧱 Writing a Playbook

* Playbooks are written in **YAML**.
* Must start with three dashes (`---`).
* YAML uses **indentation (2 spaces)** for structure — indentation is critical.
* A single playbook can have **one or more plays**.

### Anatomy of a Playbook

```
---
- name: Base system configuration
  hosts: webservers
  become: true
  vars:
    user_name: deploy_user
  tasks:
    - name: Ensure Nginx is installed
      ansible.builtin.package:
        name: nginx
        state: present

    - name: Start Nginx
      ansible.builtin.service:
        name: nginx
        state: started
```

Each play has:

* **name** → description of the play
* **hosts** → where tasks will run
* **become** → privilege escalation
* **vars** → variables section
* **tasks** → list of operations to perform

---

## 📖 How to Run a Playbook

Run the following command to explore options:

```
ansible-playbook --help
```

### Step 1: Syntax Check

```
ansible-playbook playbook_name.yaml --syntax-check
```

If syntax is correct, the return code will be **0** (verify using `echo $?`).

### Step 2: Dry Run (Check Mode)

```
ansible-playbook playbook_name.yaml --check
```

➡️ This will **simulate** what would happen without making any real changes.

### Step 3: Execute the Playbook

```
ansible-playbook playbook_name.yaml
```

This makes **actual changes** to the target hosts.

---

## 🧪 Example Output

```
PLAY [base system configuration] *************************************************
TASK [Gathering Facts] ***********************************************************
ok: [13.233.225.159]
TASK [ansible.builtin.user] ******************************************************
changed: [13.233.225.159]
TASK [ansible.builtin.package] ***************************************************
changed: [13.233.225.159]
PLAY RECAP ************************************************************************
13.233.225.159 : ok=5  changed=3  unreachable=0  failed=0  skipped=0
```

---

## 🧩 Useful Commands

### 🧾 List Hosts

Check which hosts the playbook will run on:

```
ansible-playbook playbook_name.yaml --list-hosts
```

Example output:

```
playbook: 01_First_Playbook.yaml

  play #1 (prod): base system configuration  TAGS: []
    pattern: ['prod']
    hosts (1):
      13.233.225.159
```

### 🎯 Run Specific Tasks with Tags

Assign tags to tasks, then run specific ones:

```
ansible-playbook playbook_name.yaml --tags adminuser
```

### ⚡ Disable Fact Gathering (Faster Execution)

Add this under play definition:

```
gather_facts: false
```

*(Only do this if your playbook doesn’t depend on facts.)*

### 🪜 Run Step by Step

```
ansible-playbook playbook_name.yaml --step
```

Prompts for confirmation before each task.

### ⏩ Start from a Specific Task

If you have multiple plays, start from a certain point:

```
ansible-playbook playbook_name.yaml --start-at-task="Task Name Here"
```

---

✅ **Tip:** For module details, always check the official documentation. No one memorizes everything — even senior engineers refer to docs. Learn by intention, not by copy-pasting.
