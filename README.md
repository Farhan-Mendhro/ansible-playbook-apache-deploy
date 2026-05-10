# ⚙️ Ansible Playbook — Apache Web Deploy

> A hands-on Ansible project that automates the installation of Apache2 and deploys a web page to AWS EC2 instances using a playbook — run from a WSL2 control node.

![Ansible](https://img.shields.io/badge/Ansible-2.x-EE0000?style=flat-square&logo=ansible&logoColor=white)
![Apache](https://img.shields.io/badge/Apache2-Webserver-D22128?style=flat-square&logo=apache&logoColor=white)
![AWS EC2](https://img.shields.io/badge/AWS-EC2-FF9900?style=flat-square&logo=amazon-aws&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04_LTS-E95420?style=flat-square&logo=ubuntu&logoColor=white)
![WSL2](https://img.shields.io/badge/WSL2-Control_Node-0078D6?style=flat-square&logo=windows&logoColor=white)

---

## 📋 Table of Contents

- [📁 Project Files](#-project-files)
- [🗺️ Architecture](#️-architecture)
- [📄 Inventory File](#-inventory-file)
- [📜 The Playbook](#-the-playbook)
- [▶️ How to Run](#️-how-to-run)
- [💡 Concepts Covered](#-concepts-covered)

---

## 📁 Project Files

```
ansible-playbook-apache-deploy/
├── first-playbook.yaml   # Ansible playbook — installs Apache2 and deploys the web page
├── inventory.ini         # Defines the EC2 managed nodes
└── index.html            # Web page deployed to the servers by the playbook
```

---

## 🗺️ Architecture

```
┌─────────────────────────────────────────┐
│            CONTROL NODE                  │
│         WSL2 / Ubuntu (Local PC)         │
│       ansible-playbook command           │
└──────────────────┬──────────────────────┘
                   │  SSH — Passwordless Auth
         ┌─────────┴──────────┐
         ▼                    ▼
┌─────────────────┐  ┌─────────────────┐
│   EC2 Server 1  │  │   EC2 Server 2  │
│  Ubuntu 24.04   │  │  Ubuntu 24.04   │
│    t2.micro     │  │    t2.micro     │
│   Apache2 ✓     │  │   Apache2 ✓     │
└─────────────────┘  └─────────────────┘
```

---

## 📄 Inventory File

```ini
[servers]
server1 ansible_host=<EC2-SERVER-1-IP> ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa
server2 ansible_host=<EC2-SERVER-2-IP> ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa
```

> ⚠️ Real IPs have been replaced with placeholders. Add your EC2 public IPs before running.

---

## 📜 The Playbook

```yaml
---

- host: all
  become: true
  tasks:

    - name: Install apache httpd
      ansible.builtin.apt:
        name: apache2
        state: present
        update_cache: yes

    - name: Copy file with owner and permissions
      ansible.builtin.copy:
        src: index.html
        dest: /var/www/html
        owner: root
        group: root
        mode: '0644'
```

**What each part does:**

| Key | Description |
|---|---|
| `host: all` | Targets every host in the inventory file |
| `become: true` | Runs tasks as `sudo` — required for installing packages |
| `ansible.builtin.apt` | Installs `apache2` and updates the package cache |
| `ansible.builtin.copy` | Copies `index.html` from WSL into `/var/www/html` on the EC2 server |
| `owner / group` | Sets file ownership to `root` on the remote server |
| `mode: '0644'` | Read/write for owner, read-only for everyone else |

---

## ▶️ How to Run

**Step 1 — Verify connectivity**
```bash
ansible all -i inventory.ini -m ping
```

Expected output:
```
server1 | SUCCESS => { "ping": "pong" }
server2 | SUCCESS => { "ping": "pong" }
```

**Step 2 — Dry run first (recommended)**
```bash
ansible-playbook -i inventory.ini first-playbook.yaml --check
```

**Step 3 — Run the playbook**
```bash
ansible-playbook -i inventory.ini first-playbook.yaml
```

**Step 4 — Verify in browser**
```
http://<EC2-PUBLIC-IP>
```

---

## 💡 Concepts Covered

| Concept | Description |
|---|---|
| **Ansible Playbook** | A YAML file defining automated tasks to run on remote hosts |
| **ansible.builtin.apt** | Fully-qualified module for installing packages on Debian/Ubuntu |
| **ansible.builtin.copy** | Transfers files from the control node to managed hosts |
| **become: true** | Privilege escalation — runs tasks as sudo on remote servers |
| **host: all** | Targets all hosts defined in the inventory without filtering by group |
| **File Permissions** | `mode: 0644` sets standard read/write permissions for web files |
| **Idempotency** | Running the playbook multiple times produces the same result — no duplicate installs |
| **Passwordless SSH** | RSA key authentication used so Ansible connects without a password |
| **WSL2** | Windows Subsystem for Linux used as the Ansible control node |
| **AWS EC2** | Cloud instances provisioned as managed nodes for the playbook |

---

<div align="center">
  <sub>Built for DevOps learning · Ansible · Apache2 · AWS EC2 · WSL2</sub>
</div>
