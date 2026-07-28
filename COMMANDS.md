# 📖 Commands Reference

Day-to-day commands for working with this lab. Assumes the environment is already bootstrapped — see [SETUP.md](./SETUP.md) if not.

---

## 🧪 Ad-hoc Commands

### Basic checks
```bash
# Check uptime
ansible all -a "uptime"

# Check disk space
ansible all -a "df -h"

# Check memory
ansible all -a "free -h"

# Verify sudo access
ansible all -b -a "whoami"
```

### Package install (example)
```bash
ansible webservers -b -m apt -a "name=vim state=present"
ansible webservers -b -m apt -a "name=tree state=present"
```

### Install Ansible Collections
```bash
ansible-galaxy collection install community.general
ansible-galaxy collection install ansible.posix
ansible-galaxy collection install community.mysql

# List collections
ansible-galaxy collection list
```

---

## 📝 First Playbook Example

`playbooks/system-info.yml`:
```yaml
---
- name: Gather System Information
  hosts: all
  become: yes
  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
        cache_valid_time: 3600

    - name: Display OS information
      debug:
        msg: |
          Hostname: {{ ansible_facts['hostname'] }}
          OS: {{ ansible_facts['distribution'] }} {{ ansible_facts['distribution_version'] }}
          IP: {{ ansible_facts['default_ipv4']['address'] }}

    - name: Check disk usage
      shell: df -h /
      register: disk_usage

    - name: Show disk usage
      debug:
        var: disk_usage.stdout_lines
```

Run it:
```bash
ansible-playbook playbooks/system-info.yml
```

---

## 📖 Quick Reference

| Command | Description |
|---------|-------------|
| `ansible all -m ping` | Test connectivity |
| `ansible all --list-hosts` | List managed hosts |
| `ansible-playbook playbook.yml` | Run playbook |
| `ansible-playbook playbook.yml --check` | Dry run |
| `ansible-playbook playbook.yml --check --diff` | Dry run with diff output |
| `ansible-playbook playbook.yml -v` / `-vvv` | Verbose output |
| `ansible-playbook playbook.yml --step` | Step through tasks interactively |
| `ansible-playbook playbook.yml --syntax-check` | Validate YAML/syntax only |
| `ansible-playbook playbook.yml --tags <tag>` | Run only tagged tasks |
| `ansible-playbook playbook.yml --skip-tags <tag>` | Skip tagged tasks |
| `ansible-playbook playbook.yml --limit <host/group>` | Restrict run to a subset of hosts |
| `ansible-inventory --list` | Show inventory |
| `ansible-doc <module>` | Module documentation |
| `ansible all -a "command"` | Run ad-hoc command |
| `ansible-lint playbook.yml --profile production` | Lint against strictest rule profile |
