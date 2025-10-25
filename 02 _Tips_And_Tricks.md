### Advanced Ansible Tips & Tricks for Production

---

## 1️⃣ Best Practices for Playbooks & Roles

* **Use Roles:** Modularize tasks, handlers, templates, vars, and defaults.
* **Use vars_files for secrets:** Combine with Ansible Vault.
* **Separate inventories:** dev, staging, prod with group_vars and host_vars.
* **Use pre_tasks and post_tasks:** For setup and cleanup.
* **Use handlers:** Restart or reload services only when necessary.
* **Avoid hard-coded values:** Use variables and templates.
* **Use meaningful names** for tasks and roles for clarity.

---

## 2️⃣ Environment Strategies

* **Multiple Environments:** dev, staging, production.

  * Folder structure:

    ```
    inventory/
      dev/
      staging/
      prod/
    ```
* **Vault per environment:** Encrypt credentials separately for staging/prod.
* **Testing:** Always test playbooks in staging before production.

---

## 3️⃣ Deployment & Rollback Strategies

### Blue-Green Deployment

* Maintain two environments: **Blue (current), Green (new)**.
* Switch traffic to Green after deployment.
* Rollback by redirecting traffic back to Blue.

### Canary Deployment

* Deploy new version to a **small percentage of servers or users** (e.g., 20-30%).
* Monitor logs, metrics, errors.
* Gradually increase traffic if stable.

### Rolling Updates

* Update servers in batches (e.g., 2-3 servers at a time).
* Use `serial` keyword in playbook:

```yaml
- hosts: web
  serial: 2
  tasks:
    - name: Deploy app
      copy: src=app dest=/var/www/app
```

* Reduces downtime and risk.

### Rollback Implementation

* Maintain **previous version of application** on separate server or folder.
* Use **control playbooks** for rollback tasks:

  * Stop new version
  * Start previous version
  * Update configs if needed

---

## 4️⃣ Error Handling & Reliability

* **Blocks & Rescue:** Handle errors gracefully.

```yaml
- block:
    - name: Task that might fail
      command: /bin/false
  rescue:
    - debug: msg="Task failed, running rescue"
```

* **ignore_errors:** Use for non-critical tasks.
* **register variables:** Capture output and handle logic accordingly.
* **wait_for:** Ensure services are up before next task.

---

## 5️⃣ Advanced Variables & Vault Usage

* **Variable precedence:** extra vars > task vars > role vars > inventory vars > defaults.
* **Vault IDs:** Manage multiple encrypted credentials.
* **Encrypt only sensitive data:** Use `encrypt_string` for inline secrets.
* **Combine vaults with roles:** Reference vars_files in playbooks.

---

## 6️⃣ Templates & Dynamic Configurations

* Use **Jinja2 templates** for configs:

```jinja2
{% for site in websites %}
<VirtualHost *:{{ site.port }}>
    ServerName {{ site.name }}
    DocumentRoot {{ site.doc_root }}
</VirtualHost>
{% endfor %}
```

* Avoid hard-coding server names, ports, or paths.
* Use loops for repetitive tasks like directories or user creation.

---

## 7️⃣ Loops & Task Optimization

* Loops: `loop`, `with_dict`, `with_fileglob`, `with_nested`, `with_sequence`
* Reduce task duplication.
* Use handlers instead of repeating service restarts.

---

## 8️⃣ Delegation & Local Actions

* Delegate tasks to **localhost** or control node if needed:

```yaml
- command: whoami
  delegate_to: localhost
```

* Useful for gathering information or generating files locally.

---

## 9️⃣ Tips for Production-Ready Playbooks

* Always **test in staging first**.
* Use **tags** to run only specific parts of playbooks.
* Monitor logs and output for failures.
* Version control everything, including roles and vaults.
* Automate with CI/CD for consistency.
* Document playbooks, roles, and variables for maintainability.

---

