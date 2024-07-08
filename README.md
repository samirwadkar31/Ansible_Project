# Ansible_Project

Provisioned 3 Ubuntu VM's on Azure cloud. One as a contole node and other two as managed nodes.<br>
Control node is where, I have configured the Ansible. We are going to run ansible-playbooks on control node to perform some tasks on our managed nodes.

Ansible configuration on control node:

![image](https://github.com/samirwadkar31/Ansible_Project/assets/74359548/9f9a7062-4b69-4f43-91d4-b1cc605c0d6a)

Also, make sure that python3 or latest is installed on both control and managed nodes.

![image](https://github.com/samirwadkar31/Ansible_Project/assets/74359548/14850aa2-2cdd-422d-ac25-db7a17273550)

### Ansible PlayBook 1: Performing some general configuration tasks on Target managed nodes.

```
---
- name: Configuration tasks
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
   - name: Set timezone to UTC
     timezone:
       name: Etc/UTC
   - name: Create a new user with sudo privileges
     user:
       name: devuser
       comment: "Developer User"
       shell: /bin/bash
       groups: sudo
       append: yes
       state: present
   - name: Set password for the new user
     user:
       name: devuser
       password: "{{ 'password' | password_hash('sha512') }}"
       update_password: always
   - name: Install Docker
     apt:
       name: docker.io
       state: present
       update_cache: yes
   - name: Ensure Docker service is running
     service:
       name: docker
       state: started
       enabled: yes
   - name: Add devuser to the docker group
     user:
       name: devuser
       groups: docker
       append: yes
   - name: Install Docker Compose
     get_url:
       url: https://github.com/docker/compose/releases/download/1.29.2/docker-compose-`uname -s`-`uname -m`
       dest: /usr/local/bin/docker-compose
       mode: '0755'
   - name: Verify Docker Compose installation
     command: docker-compose --version

```
### Ansible PlayBook 2: Performing some Security & Compliance Configuration tasks on Target managed nodes.

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
   - name: Install Fail2Ban
     apt:
       name: fail2ban
       state: present
       update_cache: yes
   - name: Configure automatic security updates
     apt:
       name: unattended-upgrades
       state: present
       update_cache: yes
   - name: Enable automatic security updates
     lineinfile:
       path: /etc/apt/apt.conf.d/20auto-upgrades
       line: 'APT::Periodic::Unattended-Upgrade "1";'
       create: yes
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
   - name: Install Prometheus Node Exporter
     apt:
       name: prometheus-node-exporter
       state: present
       update_cache: yes
   - name: Ensure Prometheus Node Exporter is running
     service:
       name: prometheus-node-exporter
       state: started
       enabled: yes
```

### Ansible PlayBook 3: Performing sample application deployment task on Target managed nodes.

```
---
- hosts: all
  become: true
  tasks:
    - name: Install apache http server
      apt:
        name: apache2
        state: present
        update_cache: yes
    - name: Copy file with owner and permissions
      copy:
        src: index.html
        dest: /var/www/html
        owner: root
        group: root
        mode: '0644'
```
