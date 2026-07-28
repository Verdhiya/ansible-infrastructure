# 🛠️ Environment Setup Guide

Step-by-step instructions to replicate this Ansible control node + managed-node lab from zero. This is a one-time bootstrap doc — for day-to-day commands (ad-hoc, playbook runs, quick reference), see [COMMANDS.md](./COMMANDS.md).

---

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

At this point the lab is bootstrapped — control node can reach every managed node, sudo works without a password prompt, and fact caching/pipelining are configured. From here, see [COMMANDS.md](./COMMANDS.md) for ad-hoc usage and playbook execution.

---

## 🔒 Security Notes
- ⚠️ Real inventory file (`inventories/production/hosts`) is gitignored
- 🔑 Never commit private SSH keys
- 🔐 Never commit passwords or sensitive data
- 🔰 Use Ansible Vault for secrets
- 🔀 Use separate SSH keys for Ansible and Git
