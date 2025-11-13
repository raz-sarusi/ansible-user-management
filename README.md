# Ansible User Management Project

This project automates Linux user management using Ansible roles.  
It includes the ability to:

- Create new Linux users
- Remove existing users
- Deploy SSH authorized keys
- Assign groups and shells
- Configure passwordless sudo for selected users

This project is part of my DevOps practice work on AWS EC2 instances.

---

## Project Structure

```
ansible/
├── ansible.cfg
├── inventory
├── site.yml
└── roles/
    └── users/
        ├── tasks/
        │   └── main.yml
        └── templates/
            └── sudoers_user.j2
```

The entire automation is executed through `site.yml`.

---

## ansible.cfg

```
[defaults]
inventory = /home/ubuntu/ansible/inventory
remote_user = ubuntu
private_key_file = /home/ubuntu/ansible.pem
host_key_checking = False
retry_files_enabled = False
timeout = 30
forks = 10
interpreter_python = auto

[privilege_escalation]
become = True
become_method = sudo
become_ask_pass = False
```

---

## Inventory Example

```
[group_a]
server1 ansible_host=1.2.3.4

[group_a:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=/home/ubuntu/ansible.pem
ansible_python_interpreter=/usr/bin/python3
```

---

## site.yml (Playbook Entry Point)

Below is an example of how your `site.yml` should look:

```
---
- name: Manage users on servers
  hosts: group_a
  become: true

  vars:
    users_to_create:
      - name: "deploy"
        shell: "/bin/bash"
        groups: ["sudo"]
        pubkey: "ssh-ed25519 AAAA...your-key..."
        sudo_nopasswd: true

    users_to_remove:
      - "olduser"

  roles:
    - users
```

This is the file you execute to apply user management to the servers.

---

## Role Breakdown

### roles/users/tasks/main.yml

```
---
- name: Create users
  ansible.builtin.user:
    name: "{{ item.name }}"
    shell: "{{ item.shell }}"
    groups: "{{ item.groups | join(',') }}"
    create_home: true
    state: present
  loop: "{{ users_to_create }}"
  when: users_to_create is defined

- name: Add SSH authorized key
  ansible.builtin.authorized_key:
    user: "{{ item.name }}"
    key: "{{ item.pubkey }}"
    state: present
  loop: "{{ users_to_create }}"
  when: item.pubkey is defined

- name: Configure passwordless sudo
  ansible.builtin.template:
    src: "sudoers_user.j2"
    dest: "/etc/sudoers.d/{{ item.name }}"
    mode: "0440"
    validate: "visudo -cf %s"
  loop: "{{ users_to_create }}"
  when: item.sudo_nopasswd | default(false)

- name: Remove unwanted users
  ansible.builtin.user:
    name: "{{ item }}"
    state: absent
    remove: true
    force: true
  loop: "{{ users_to_remove }}"
  when: users_to_remove is defined
```

---

## Template File

### roles/users/templates/sudoers_user.j2

```
{{ item.name }} ALL=(ALL) NOPASSWD:ALL
```

---

## How to Run

Execute the automation:

```
ansible-playbook site.yml
```

Simulate changes first:

```
ansible-playbook site.yml --check --diff
```

---

## Purpose

This project demonstrates practical DevOps skills:

- Ansible role development
- Automation of Linux system administration
- SSH and sudoers configuration
- Structured and reusable playbook architecture
- Real-world configuration management

---


