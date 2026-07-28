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
| `03-install_package_when-1.yml` | Early version — demonstrates a category error (distro-name `when` + wrong module) triggering a genuine task `failed` state |
| `04-install_package_when-2.yml` | Corrected version — branches `apt`/`dnf` tasks explicitly via `ansible_facts['os_family']` |

> **Note:** CentOS service enablement (`systemd`) and firewall rules (`ansible.posix.firewalld`) are intentionally left out of `04-install_package_when-2.yml` for now, to keep the Debian-vs-RedHat post-install differences visible and hand-run for learning purposes.

---

## 🧩 Variable-Driven Installs (`group_vars`)

`06-install_package_when-4.yml` uses the generic `ansible.builtin.package` module with OS-specific package names sourced from `group_vars`, instead of branching per task:

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

`07-remove_package.yml` uses the same pattern with `state: absent`, scoped via `hosts: webservers:centos` for targeted cleanup.

---

## 📁 Project Structure
```
.
├── ansible.cfg
├── docs/
│   └── screenshots/
│       ├── pb-09/
│       ├── pb-10/
│       └── pb-11/
├── files/
│   └── site-1.html
├── inventories/
│   └── production/
│       ├── hosts.sample
│       ├── group_vars/
│       └── host_vars/
├── playbooks/
│   ├── 01-system-info.yml
│   ├── 02-install_package.yml
│   ├── 03-install_package_when-1.yml
│   ├── 04-install_package_when-2.yml
│   ├── 05-install_package_when-3.yml
│   ├── 06-install_package_when-4.yml
│   ├── 07-remove_package.yml
│   ├── 08-install_package_when-tags-5.yml
│   ├── 09-install_package_when-tags-site-1.yml
│   ├── 10-install_package_template-site-1.yml
│   └── 11-apache-virtual-host-1.yml
├── roles/
└── templates/
    ├── dashboard.html.j2
    └── infrastructure-dashboard.conf.j2
```

---

## 🧭 How to Follow This Project

Playbooks are numbered in build order — each one builds on a concept introduced by the last. Read/run them in sequence to follow the actual learning path:

| # | Playbook | Concept introduced |
|---|----------|---------------------|
| 01 | `system-info.yml` | First playbook — `apt` cache update, fact-based `debug`, `register` + `shell` |
| 02 | `install_package.yml` | Basic package management with `apt`/`package` |
| 03 | `install_package_when-1.yml` | `when` conditionals — intentional category error (distro-name match + wrong module) to observe a real task `failed` state |
| 04 | `install_package_when-2.yml` | Corrected multi-OS branching via `ansible_facts['os_family']` |
| 05 | `install_package_when-3.yml` | Consolidated apt/dnf tasks per OS family (list-form package names) |
| 06 | `install_package_when-4.yml` | Generic `ansible.builtin.package` module driven by `group_vars` |
| 07 | `remove_package.yml` | Same `group_vars` pattern with `state: absent` — cleanup for a host-scope leak (see Known Weak Spots in progress log) |
| 08 | `install_package_when-tags-5.yml` | Tags — `tags:`, `--tags`, `--skip-tags`, multi-play host scoping |
| 09 | `install_package_when-tags-site-1.yml` | Static file deployment via `ansible.builtin.copy` |
| 10 | `install_package_template-site-1.yml` | `ansible.builtin.template` — Jinja2 rendering driven by live `ansible_facts` |
| 11 | `apache-virtual-host-1.yml` | Full vhost deployment — `handlers`/`notify`, `file` module (`directory`/`link`/`absent`), config validation (`apache2ctl configtest` / `httpd -t`), cross-OS `group_vars` for log paths |

**Supporting directories** (foundational, not sequential steps):
- `files/` — static assets deployed as-is (`site-1.html`, used by playbook 09)
- `templates/` — Jinja2 templates rendered per-host (`dashboard.html.j2` used by 10; `infrastructure-dashboard.conf.j2` used by 11)
- `inventories/production/group_vars/` — per-group variables (package names, log paths, vhost paths) driving playbooks 06 and 11
- `inventories/production/host_vars/` — scaffolded, not yet in active use
- `roles/` — scaffolded, not yet in active use

---

## 🖥️ Live Output

Screenshots below live in `docs/screenshots/pb-09/`, `pb-10/`, `pb-11/`. Content is identical across all four webservers per stage (confirms correct multi-host deployment, not just one lucky host) — one host is shown inline per stage, the rest are in the collapsed section.

### `09-install_package_when-tags-site-1.yml` — Static deploy (`copy`)
Static `site-1.html` pushed as-is via `ansible.builtin.copy` — same bytes on every host, only the hostname differs because it's baked into the file per the group it targets.

<img src="docs/screenshots/pb-09/files-web-01.png" width="600" alt="Static site output on web-01"/>

<details>
<summary>web-02, web-03, centos-01</summary>

<img src="docs/screenshots/pb-09/files-web-02.png" width="600" alt="Static site output on web-02"/>
<img src="docs/screenshots/pb-09/files-web-03.png" width="600" alt="Static site output on web-03"/>
<img src="docs/screenshots/pb-09/files-centos-01.png" width="600" alt="Static site output on centos-01"/>

</details>

### `10-install_package_template-site-1.yml` — Jinja2 rendering (`template`)
`dashboard.html.j2` rendered per-host from live `ansible_facts` — hostname, OS, kernel, memory, and vCPU count are all pulled from the actual target, not hardcoded.

<img src="docs/screenshots/pb-10/template-web-01.png" width="600" alt="Rendered dashboard on web-01"/>

<details>
<summary>web-02, web-03, centos-01</summary>

<img src="docs/screenshots/pb-10/template-web-02.png" width="600" alt="Rendered dashboard on web-02"/>
<img src="docs/screenshots/pb-10/template-web-03.png" width="600" alt="Rendered dashboard on web-03"/>
<img src="docs/screenshots/pb-10/template-centos-01.png" width="600" alt="Rendered dashboard on centos-01"/>

</details>

### `11-apache-virtual-host-1.yml` — Virtual host deploy
Same dashboard content, now served through a dedicated Apache vhost (own document root, symlinked config, validated via `apache2ctl configtest`/`httpd -t` before reload) instead of the default site. Visually identical to stage 10 by design — this confirms the vhost migration didn't break anything, which is exactly what you want a config-validation step to prove.

<img src="docs/screenshots/pb-11/apache-vhosts-web-01.png" width="600" alt="Vhost-served dashboard on web-01"/>

<details>
<summary>web-02, web-03, centos-01</summary>

<img src="docs/screenshots/pb-11/apache-vhosts-web-02.png" width="600" alt="Vhost-served dashboard on web-02"/>
<img src="docs/screenshots/pb-11/apache-vhosts-web-03.png" width="600" alt="Vhost-served dashboard on web-03"/>
<img src="docs/screenshots/pb-11/apache-vhosts-centos-01.png" width="600" alt="Vhost-served dashboard on centos-01"/>

</details>

---

## 📚 Documentation

- **[SETUP.md](./SETUP.md)** — one-time environment bootstrap: SSH keys, Git config, Ansible install, inventory, `ansible.cfg`, passwordless sudo, connectivity check
- **[COMMANDS.md](./COMMANDS.md)** — day-to-day command reference: ad-hoc commands, collections, playbook execution flags, quick reference table

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