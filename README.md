# 🚀 Azure VM – Ansible Automation Setup (3 Linux VMs)

This document explains the **end-to-end setup of Ansible** using **3 Linux Virtual Machines** on Azure:

* **1 Ansible Control Node (Server)**
* **2 Managed Host Nodes**
* **SSH Key-based authentication**
* **Apache, Nginx, MySQL automation using playbooks**

---

## 🏗️ Architecture Overview

```
               ┌───────────────────────┐
               │  Ansible Control Node  │
               │  (Ubuntu Linux VM)     │
               │  - ansible.cfg         │
               │  - inventory/hosts     │
               │  - playbooks/          │
               │  - SSH key (pem)       │
               └───────────┬───────────┘
                           │ SSH (22)
          ┌────────────────┴───────────────┐
          │                                │
┌───────────────────┐        ┌───────────────────┐
│ Managed Host VM 1 │        │ Managed Host VM 2 │
│ Ubuntu Linux      │        │ Ubuntu Linux      │
│ Web / DB Server   │        │ Web / DB Server   │
└───────────────────┘        └───────────────────┘
```

---

## 🔐 SSH Key Setup

* Single private key used by Ansible server
* Stored securely inside the project

```
azure/ansible.pem
```

Permissions:

```bash
chmod 400 azure/ansible.pem
```

---

## 📦 Ansible Installation Methods (Tested)

You tried **multiple supported methods** (good practice 👌).
**Recommended:** Use **APT repository method** (Method 3).

---

### 🔹 Method 1: Python + Pip (Not Recommended for Production)

```bash
sudo apt update -y
sudo apt install python -y
sudo apt install pip -y
sudo pip install ansible -y
```

---

### 🔹 Method 2: ansible-core (Minimal)

```bash
sudo apt install ansible-core -y
```

---

### ✅ Method 3: Official Ansible PPA (Recommended)

```bash
sudo apt update
sudo apt install software-properties-common -y
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible -y
```

Verify:

```bash
ansible --version
```

---

## 📂 Project Directory Structure

Created inside `/etc/ansible` and later copied to GitHub repo.

```
AzureVM-Ansible/
├── ansible.cfg
├── inventory/
│   └── hosts
├── playbooks/
│   ├── ping.yml
│   ├── apache.yml
│   ├── mysql.yml
│   ├── nginx.yml
│   └── vars.yml
├── files/
│   └── index.html
└── azure/
    └── ansible.pem
```

---

## ⚙️ ansible.cfg

```ini
[defaults]
inventory = ./inventory
remote_user = ubuntu
private_key_file= azure/ansible.pem
host_key_checking=False
retry_files_enabled=False

[privilege_escalation]
become = true
become_method = sudo
become_user = root
become_ask_pass = false
```

---

## 🧾 Inventory File

`inventory/hosts`

```ini
[web]
10.0.0.5
10.0.0.6

[db]
10.0.0.6

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

---

## 🔍 Connectivity Test

### Ping all hosts

```bash
ansible -m ping all
```

### Ping specific group

```bash
ansible -m ping web
```

---

## 📜 Playbooks

---

### ✅ Ping Test Playbook

`playbooks/ping.yml`

```yaml
- name: Ping Test
  hosts: all
  become: true
  tasks:
    - name: Ping hosts
      ping:
```

Run:

```bash
ansible-playbook -i inventory/hosts playbooks/ping.yml
```

---

### 🌐 Apache Installation

`playbooks/apache.yml`

```yaml
- name: Install Apache
  hosts: web
  become: true
  tasks:
    - name: Install Apache
      apt:
        name: apache2
        state: present
        update_cache: yes

    - name: Start Apache
      service:
        name: apache2
        state: started
        enabled: yes
```

Run:

```bash
ansible-playbook -i inventory/hosts playbooks/apache.yml
```

---

### 🗄️ MySQL Installation (With Variables)

`playbooks/vars.yml`

```yaml
mysql_package: mysql-server
```

`playbooks/mysql.yml`

```yaml
- name: Install MySQL
  hosts: db
  become: true
  vars_files:
    - vars.yml
  tasks:
    - name: Install MySQL
      apt:
        name: "{{ mysql_package }}"
        state: present
        update_cache: yes
```

Run:

```bash
ansible-playbook -i inventory/hosts playbooks/mysql.yml
```

---

### 🚦 Nginx + Custom HTML Page

`files/index.html`

```html
<h1>Welcome from Ansible Nginx Server</h1>
```

`playbooks/nginx.yml`

```yaml
- name: Install Nginx
  hosts: web
  become: true
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present
        update_cache: yes

    - name: Copy index.html
      copy:
        src: files/index.html
        dest: /var/www/html/index.html

    - name: Start Nginx
      service:
        name: nginx
        state: started
        enabled: yes
```

Run:

```bash
ansible-playbook -i inventory/hosts playbooks/nginx.yml
```

---

## 🧪 Verification

* Apache: `http://<VM-IP>`
* Nginx: `http://<VM-IP>`
* MySQL:

```bash
sudo systemctl status mysql
```

---

## 📤 GitHub Upload

```bash
git clone https://github.com/atulkamble/AzureVM-Ansible.git
cd AzureVM-Ansible
git add .
git commit -m "Ansible Azure VM automation"
git push origin main
```

---

## ✅ Best Practices Followed

✔ SSH key-based auth
✔ Inventory grouping
✔ Variables usage
✔ Role-ready folder structure
✔ Idempotent playbooks
✔ GitHub version control

---
