---
date: '2025-08-22T09:50:45+05:30'
draft: false
title: 'Ansible'
tags: ["Ansible", "yaml", "automation"]
---


### playbook
```yaml
---
- name: Install and configure Nginx   # Play name
  hosts: webservers                   # Target group from inventory
  become: true                        # Run with sudo/root privileges
  vars:                               # Variables (optional)
    http_port: 80

  pre_tasks:                          # Tasks to run before main tasks (optional)
    - name: Update apt cache
      apt:
        update_cache: yes

  tasks:                              # Main tasks
    - name: Install Nginx
      apt:
        name: nginx
        state: present

    - name: Start Nginx service
      service:
        name: nginx
        state: started
        enabled: true

  post_tasks:                         # Tasks to run after main tasks (optional)
    - name: Print completion message
      debug:
        msg: "Nginx is installed and running on {{ inventory_hostname }}"

  handlers:                           # Handlers (run when notified)
    - name: restart nginx
      service:
        name: nginx
        state: restarted
```


### hosts
```ini
[webservers]
192.168.1.10
192.168.1.11 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa

[dbservers]
192.168.1.20 ansible_user=ec2-user ansible_ssh_private_key_file=~/.ssh/aws.pem
```
