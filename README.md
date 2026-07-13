<div align="center">

# 🚀 Ansible Infrastructure Project

![Ansible](https://img.shields.io/badge/Ansible-14.1.0-EE0000?style=for-the-badge&logo=ansible&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04_LTS-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![CentOS](https://img.shields.io/badge/CentOS_Stream_9-262577?style=for-the-badge&logo=centos&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.12.3-3776AB?style=for-the-badge&logo=python&logoColor=white)

**Automation for Managing Web Servers & Database Infrastructure**

</div>

---

## 📋 Overview
Ansible automation for managing web servers and database infrastructure across a mixed Ubuntu 24.04 LTS (Debian family) and CentOS (RedHat family) Oracle VM fleet.

## 🛠️ Tech Stack

<p align="center">
  <img src="https://img.shields.io/badge/Ansible-EE0000?style=for-the-badge&logo=ansible&logoColor=white" alt="Ansible"/>
  <img src="https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white" alt="Ubuntu"/>
  <img src="https://img.shields.io/badge/CentOS-262577?style=for-the-badge&logo=centos&logoColor=white" alt="CentOS"/>
  <img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python"/>
  <img src="https://img.shields.io/badge/Oracle_VM-F80000?style=for-the-badge&logo=oracle&logoColor=white" alt="Oracle VM"/>
  <img src="https://img.shields.io/badge/Git-F05032?style=for-the-badge&logo=git&logoColor=white" alt="Git"/>
  <img src="https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white" alt="GitHub"/>
  <img src="https://img.shields.io/badge/SSH-4EAA25?style=for-the-badge&logo=openssh&logoColor=white" alt="SSH"/>
  <img src="https://img.shields.io/badge/YAML-CB171E?style=for-the-badge&logo=yaml&logoColor=white" alt="YAML"/>
</p>

---

## 🏗️ Infrastructure
- **Control Node:** ansible-control
- **Web Servers:** 🌐 web-01, web-02, web-03
- **Database:** 🗄️ db-01
- **CentOS Server:** 🎩 centos-01 (RedHat family — multi-OS branching target)
- **OS:** 🐧 Ubuntu 24.04 LTS (Debian family) + CentOS (RedHat family)
- **Ansible Version:** ⚙️ 14.1.0 (core 2.21.1)
- **Python Version:** 🐍 3.12.3

---

## 🎯 Architecture Diagram

```mermaid
graph TB
    A[ansible-control<br/>Control Node<br/>Ubuntu 24.04 LTS] --> B[web-01<br/>Web Server]
    A --> C[web-02<br/>Web Server]
    A --> D[web-03<br/>Web Server]
    A --> E[db-01<br/>Database Server]
    A --> F[centos-01<br/>Web Server<br/>RedHat family]
    
    style A fill:#4CAF50,stroke:#333,stroke-width:2px,color:#fff
    style B fill:#2196F3,stroke:#333,stroke-width:2px,color:#fff
    style C fill:#2196F3,stroke:#333,stroke-width:2px,color:#fff
    style D fill:#2196F3,stroke:#333,stroke-width:2px,color:#fff
    style E fill:#FF9800,stroke:#333,stroke-width:2px,color:#fff
    style F fill:#9C27B0,stroke:#333,stroke-width:2px,color:#fff
```

---

## 🔄 Ansible Communication Flow

```mermaid
sequenceDiagram
    participant User
    participant Control as ansible-control
    participant Web as Web Servers
    participant DB as Database

    User->>Control: Run playbook/command
    Control->>Control: Read inventory & config
    Control->>Web: SSH connection (ed25519)
    Control->>DB: SSH connection (ed25519)
    Web-->>Control: Execute tasks
    DB-->>Control: Execute tasks
    Control-->>User: Return results
```

---

## 📊 Workflow: Running a Playbook

```mermaid
flowchart LR
    A[Write Playbook] --> B[ansible-playbook command]
    B --> C{Read ansible.cfg}
    C --> D[Load Inventory]
    D --> E[Connect via SSH]
    E --> F[Execute Tasks]
    F --> G[Gather Facts]
    G --> H[Apply Changes]
    H --> I[Return Status]
    
    style A fill:#e1f5ff
    style I fill:#c8e6c9
```

---

## 🐧 Multi-OS Package Management (`when` + `os_family`)

Playbooks branch package-manager logic by OS family rather than hardcoding distro names — scales cleanly to Rocky/Alma/Fedora without touching the `when` clause.

```mermaid
flowchart TD
    A[Gather Facts] --> B{ansible_facts os_family}
    B -->|Debian| C[apt: install apache2]
    B -->|RedHat| D[dnf: install httpd]
    C --> E[apt: update repo index]
    D --> F[dnf: install php]
    E --> G[apt: install libapache2-mod-php]
    G --> H[Play Recap]
    F --> H
    
    style B fill:#fff9c4
    style H fill:#c8e6c9
```


| File | Purpose |
|------|---------|
| `install_package_when.yml` | Early version — demonstrates a category error (distro-name `when` + wrong module) triggering a genuine task `failed` state |
| `install_package_when-2.yml` | Corrected version — branches `apt`/`dnf` tasks explicitly via `ansible_facts['os_family']` |

> **Note:** CentOS service enablement (`systemd`) and firewall rules (`ansible.posix.firewalld`) are intentionally left out of `install_package_when-2.yml` for now, to keep the Debian-vs-RedHat post-install differences visible and hand-run for learning purposes.

---

## 🧩 Variable-Driven Installs (`group_vars`)

`install_package_when-4.yml` uses the generic `ansible.builtin.package` module with OS-specific package names sourced from `group_vars`, instead of branching per task:

```yaml
- name: Install Apache and PHP
  ansible.builtin.package:
    name:
      - "{{ apache_package }}"
      - "{{ php_package }}"
    state: present
```

| Group | `apache_package` | `php_package` |
|-------|-------------------|----------------|
| webservers | apache2 | libapache2-mod-php |
| centos | httpd | php |

`remove_package.yml` uses the same pattern with `state: absent`, scoped via `--limit` for targeted cleanup.

---

## 📁 Project Structure
```
ansible/
├── .gitignore
├── ansible.cfg
├── inventories/
│   └── production/
│       ├── hosts.sample
│       ├── group_vars/
│       │   ├── webservers.yml
│       │   ├── centos.yml
│       │   └── database.yml
│       └── host_vars/
├── playbooks/
│   ├── system-info.yml
│   ├── install_package_when.yml
│   ├── install_package_when-2.yml
│   ├── install_package_when-3.yml
│   ├── install_package_when-4.yml
│   └── remove_package.yml
├── roles/
├── files/
└── templates/
```

---

# 📚 Setup Documentation

## 🔐 SSH Key Configuration for Ansible

### Create SSH Keys
```bash
ssh-keygen -t ed25519 -C "your_comment"

# Creates:
#    ~/.ssh/id_ed25519      # Private key
#    ~/.ssh/id_ed25519.pub  # Public key
```

### Copy Public Key to Managed Nodes
```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub <username>@<host>
```

Repeat for all managed nodes.

### Verify Password-less SSH
```bash
ssh <username>@<host>
```

---

## 📦 Git Configuration

### Install Git
```bash
sudo apt update && sudo apt install git -y
git --version
```

### Setup SSH Keys for Git
Copy existing Git SSH keys to `~/.ssh/` or create new ones (use different keys than Ansible SSH keys):

```bash
chmod 600 ~/.ssh/<your_git_private_key>
chmod 644 ~/.ssh/<your_git_public_key>
```

### Configure SSH for GitHub
```bash
vim ~/.ssh/config
```

Add:
```
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/<your_git_private_key>
```

### Configure Git Identity
```bash
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
```

### Verify Configuration
```bash
git config --list
ssh -T git@github.com
```

### Initialize Git Repository
```bash
mkdir ~/ansible && cd ~/ansible
git init
git status
```

### Create .gitignore
```bash
touch .gitignore
```

---

## ⚙️ Ansible Installation & Setup

### Step 1: Install Ansible (via pipx)
```bash
sudo apt update
sudo apt install pipx -y
pipx ensurepath
pipx install --include-deps ansible
source ~/.bashrc
ansible --version
```

### Step 2: Create Directory Structure
```bash
cd ~/ansible

mkdir -p inventories/production/{group_vars,host_vars}
mkdir -p roles playbooks files templates

# Verify
tree -L 3
```

### Step 3: Create Inventory File
```bash
vim inventories/production/hosts
```

Add (replace placeholders with your actual values):
```ini
[webservers]
web-01 ansible_host=<IP_ADDRESS>
web-02 ansible_host=<IP_ADDRESS>
web-03 ansible_host=<IP_ADDRESS>

[database]
db-01 ansible_host=<IP_ADDRESS>

[all:vars]
ansible_user=<your_username>
ansible_ssh_private_key_file=~/.ssh/id_ed25519
ansible_python_interpreter=/usr/bin/python3
```

### Step 4: Create Ansible Configuration
```bash
vim ansible.cfg
```

Add:
```ini
[defaults]
inventory = ./inventories/production/hosts
host_key_checking = False
remote_user = <your_username>
private_key_file = ~/.ssh/id_ed25519
retry_files_enabled = False
gathering = smart
fact_caching = jsonfile
fact_caching_connection = /tmp/ansible_facts
fact_caching_timeout = 3600
nocows = 1

[privilege_escalation]
become = True
become_method = sudo
become_user = root
become_ask_pass = False

[ssh_connection]
pipelining = True
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
```

### Step 5: Configure Passwordless Sudo on All Managed Hosts
SSH to each VM and run:
```bash
ssh <username>@<host>
echo "$USER ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/$USER
sudo chmod 0440 /etc/sudoers.d/$USER
exit
```
Repeat for all managed hosts.

### Step 6: Test Connectivity
```bash
# Verify configuration
ansible --version

# List hosts
ansible all --list-hosts

# Accept host keys
ssh-keyscan <host1-ip> <host2-ip> <host3-ip> <host4-ip> >> ~/.ssh/known_hosts

# Test ping
ansible all -m ping

# Test groups
ansible webservers -m ping
ansible database -m ping
```

### Step 7: Test Ad-hoc Commands
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

### Step 8: Install Packages
```bash
ansible webservers -b -m apt -a "name=vim state=present"
ansible webservers -b -m apt -a "name=tree state=present"
```

### Step 9: Install Ansible Collections (Optional)
```bash
ansible-galaxy collection install community.general
ansible-galaxy collection install ansible.posix
ansible-galaxy collection install community.mysql

# List collections
ansible-galaxy collection list
```

### Step 10: Create First Playbook
```bash
vim playbooks/system-info.yml
```

Add:
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

Run playbook:
```bash
ansible-playbook playbooks/system-info.yml
```

---

## 🔁 Playbook Execution Flow

```mermaid
flowchart TD
    A[ansible-playbook system-info.yml] --> B{Parse Playbook}
    B --> C[Target: all hosts]
    C --> D[Become: sudo]
    D --> E[Task 1: Update apt cache]
    E --> F[Task 2: Display OS info]
    F --> G[Task 3: Check disk usage]
    G --> H[Task 4: Show results]
    H --> I[Playbook Complete]
    
    style A fill:#e3f2fd
    style I fill:#c8e6c9
    style E fill:#fff9c4
    style F fill:#fff9c4
    style G fill:#fff9c4
    style H fill:#fff9c4
```

---

## 📖 Quick Reference

| Command | Description |
|---------|-------------|
| `ansible all -m ping` | Test connectivity |
| `ansible all --list-hosts` | List managed hosts |
| `ansible-playbook playbook.yml` | Run playbook |
| `ansible-playbook playbook.yml --check` | Dry run |
| `ansible-playbook playbook.yml -v` | Verbose output |
| `ansible-inventory --list` | Show inventory |
| `ansible-doc <module>` | Module documentation |
| `ansible all -a "command"` | Run ad-hoc command |
| `ansible-lint playbook.yml --profile production` | Lint against strictest rule profile |

---

## 🔒 Security Notes
- ⚠️ Real inventory file (`inventories/production/hosts`) is gitignored
- 🔑 Never commit private SSH keys
- 🔐 Never commit passwords or sensitive data
- 🔰 Use Ansible Vault for secrets
- 🔀 Use separate SSH keys for Ansible and Git

---

## 📝 Notes
Personal learning and development project

---

<div align="center">

**Made with ❤️ using Ansible**

![Status](https://img.shields.io/badge/Status-Active-success?style=flat-square)
![Maintained](https://img.shields.io/badge/Maintained-Yes-green?style=flat-square)

</div>

---

