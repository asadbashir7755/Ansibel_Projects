### Complete Ansible Roles & Advanced Concepts Cheat Sheet

---

## 1. Roles in Ansible

### What are Roles?

* Roles are **predefined, reusable sets of tasks** for organizing playbooks.
* They allow you to structure automation in a **modular and professional way**.

### Why Use Roles?

* **Reusability:** Write once, use anywhere.
* **Organization:** Separate tasks, templates, vars, handlers, and defaults.
* **Scalability:** Helps manage large projects in production.

### Anatomy of a Role

```
role_name/
├── tasks/       # main.yml and other task files
├── handlers/    # main.yml, handlers for services
├── templates/   # Jinja2 templates
├── files/       # static files to copy
├── vars/        # variables (role-specific)
├── defaults/    # default variables
├── meta/        # metadata, dependencies
└── tests/       # optional, test playbooks
```

### Role Folder Structure for Production

* `tasks/` -> split into multiple files for large deployments
* `handlers/` -> restart services, reload configs
* `vars/` -> sensitive variables if needed
* `defaults/` -> non-sensitive defaults
* `meta/` -> dependencies like `dependencies: [default-system-conf]`
* `templates/` -> Jinja2 templates for configs
* `files/` -> static files to copy
* `tests/` -> optional for QA testing

### Variables Precedence in Roles

**Highest to Lowest:**

1. Extra vars (`-e`)
2. Task vars
3. Block vars
4. Role vars (`vars/main.yml`)
5. Inventory vars (host_vars/group_vars)
6. Role defaults (`defaults/main.yml`)
7. Include vars

### Hash Behavior in Ansible

* **Default:** Replace
* **Merge:** Merge dictionaries instead of replacing

```yaml
vars:
  dict1:
    key1: value1
  dict2:
    key2: value2
```

* Merged result: `{ key1: value1, key2: value2 }`

---

## 2. Generating & Using Roles

### Generate Role

```bash
ansible-galaxy init role_name
```

### Nested Roles

* Roles can have **dependencies** defined in `meta/main.yml`

```yaml
dependencies:
  - role: default-system-conf
```

* Ansible runs dependencies **first**, then the main role.

### Example Tasks in Role with Dependencies

```yaml
TASK [default_system_conf : create deploy_user group] - ok
TASK [default_system_conf : create deploy user with uid 5004] - ok
```

---

## 3. Loops in Ansible

### Loop Syntax

```yaml
- name: Create multiple directories
  ansible.builtin.file:
    path: "/opt/{{ item }}"
    state: directory
  loop:
    - app1
    - app2
    - app3
```

### with_dict

```yaml
- name: Add users with comments
  ansible.builtin.user:
    name: "{{ item.key }}"
    comment: "{{ item.value }}"
  with_dict:
    asad: "DevOps Engineer"
    saqib: "Frontend Developer"
```

### with_fileglob

```yaml
- name: Copy all HTML files
  ansible.builtin.copy:
    src: "{{ item }}"
    dest: /var/www/html/
  with_fileglob:
    - "*.html"
```

### with_nested

```yaml
- name: Create user and group combinations
  debug:
    msg: "User {{ item[0] }} belongs to group {{ item[1] }}"
  with_nested:
    - ['asad', 'saqib']
    - ['devops', 'frontend']
```

### with_sequence

```yaml
- name: Create numbered directories
  ansible.builtin.file:
    path: "/data/{{ item }}"
    state: directory
  with_sequence: start=1 end=5
```

### Using Loops with Templates

```jinja2
{% for site in websites %}
<VirtualHost *:{{ site.port }}>
    ServerAdmin webmaster@localhost
    DocumentRoot {{ site.doc_root }}
    ServerName {{ site.name }}
    ErrorLog ${APACHE_LOG_DIR}/{{ site.name }}-error.log
    CustomLog ${APACHE_LOG_DIR}/{{ site.name }}-access.log combined
</VirtualHost>
{% endfor %}
```

---

## 4. Error Handling in Roles

### Using Block and Rescue

```yaml
- block:
    - name: Task that might fail
      command: /bin/false
  rescue:
    - name: Run alternative task on failure
      debug: msg="Task failed, running rescue"
```

### Ignore Errors

```yaml
- name: This task may fail
  command: /bin/false
  ignore_errors: yes
```

### Using Registers

```yaml
- name: Run a command
  command: /bin/true
  register: result

- name: Show result
  debug:
    var: result.stdout
```

---

## 5. Deployment Strategies

### Blue-Green Deployment

* Two identical environments: **Blue (current)** and **Green (new release)**
* Switch traffic to Green once ready. If failure occurs, revert to Blue.

### Canary Release

* Deploy new version to a **small subset of servers/users**
* Test stability before rolling out fully

### Rolling Updates Strategy

* Gradually update servers **one batch at a time**
* Reduces downtime and risk

---

## 6. Playbook Structure

### Pre-tasks, Tasks, Post-tasks

```yaml
pre_tasks:
  - name: Run tasks before main tasks
    debug: msg="This runs first"

tasks:
  - name: Main deployment tasks
    debug: msg="Deploy app"

post_tasks:
  - name: Cleanup or final steps
    debug: msg="Runs at end"
```

### Delegation

```yaml
- name: Run command on remote host but delegate to another
  command: whoami
  delegate_to: localhost
```

### Wait_for Testing

```yaml
- name: Wait for port 80 to be open
  wait_for:
    host: 127.0.0.1
    port: 80
    timeout: 300
```

---

