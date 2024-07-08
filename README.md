# Ansible_Project

Provisioned 3 Ubuntu VM's on Azure cloud. One as a contole node and other two as managed nodes.<br>
Control node is where, I have configured the Ansible. We are going to run ansible-playbooks on control node to perform some tasks on our managed nodes.

Ansible configuration on control node:

![image](https://github.com/samirwadkar31/Ansible_Project/assets/74359548/9f9a7062-4b69-4f43-91d4-b1cc605c0d6a)

Also, make sure that python3 or latest is installed on both control and managed nodes.

![image](https://github.com/samirwadkar31/Ansible_Project/assets/74359548/14850aa2-2cdd-422d-ac25-db7a17273550)

We are going to perform some configurations tasks on managed nodes using below ansible playbook:

```
---
- name: Basic server setup
  hosts: servers
  become: true
  tasks:
   - name: Ensure apt cache is up-to-date
     apt:
       update_cache: yes
   - name: Upgrade all packages to the latest version
     apt:
       upgrade: dist
   - name: Install basic packages
     apt:
       name:
         - vim
         - curl
         - git
       state: present
```
We are going to perform some security configuration tasks on managed nodes using below ansible playbook:

```
---
- name: Security & Complaince tasks
  hosts: servers
  become: true
  tasks:
   - name: Disable root SSH login
     lineinfile:
       path: /etc/ssh/sshd_config
       regexp: '^PermitRootLogin'
       line: 'PermitRootLogin no'
       state: present
   - name: Ensure SSH password authentication is disabled
     lineinfile:
       path: /etc/ssh/sshd_config
       regexp: '^PasswordAuthentication'
       line: 'PasswordAuthentication no'
       state: present
   - name: Restart SSH service to apply changes
     service:
       name: ssh
       state: restarted
   - name: Ensure UFW is installed
     apt:
       name: ufw
       state: present
   - name: Allow SSH through the firewall
     ufw:
       rule: allow
       port: 22
       proto: tcp
   - name: Enable the firewall
     ufw:
       state: enabled
       policy: allow
   - name: Ensure essential services are running
     service:
       name: "{{ item }}"
       state: started
       enabled: yes
     with_items:
       - ssh
       - ufw
```

