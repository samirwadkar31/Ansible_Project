# Ansible_Project

Provisioned 3 Ubuntu VM's on Azure cloud. One as a contole node and other two as managed nodes.<br>

Control node is where, I have configured the Ansible. We are going to run some Ansible-Playbooks on control node to perform few tasks on our managed nodes.

![image](https://github.com/samirwadkar31/Ansible_Project/assets/74359548/9daa5fa4-629a-4b80-a665-ca23324ddc0a)


#### Ansible configuration on control node:

![image](https://github.com/samirwadkar31/Ansible_Project/assets/74359548/9f9a7062-4b69-4f43-91d4-b1cc605c0d6a)

Also, make sure that python3 or latest is installed on both control and managed nodes.

![image](https://github.com/samirwadkar31/Ansible_Project/assets/74359548/14850aa2-2cdd-422d-ac25-db7a17273550)

![image](https://github.com/samirwadkar31/Ansible_Project/assets/74359548/7cca3806-7aa9-477c-9a5e-575cc3c5b5a8)


### Ansible PlayBook 1: Performing general Configuration Tasks on Target managed nodes.

```
---
- name: Configuration tasks
  hosts: my_servers
  become: true
  remote_user: samirwadkar31
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
       url: "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-{{ ansible_system | lower }}-{{ ansible_architecture }}"
       dest: /usr/local/bin/docker-compose
       mode: '0755'
   - name: Verify Docker Compose installation
     command: docker-compose --version

```

![image](https://github.com/samirwadkar31/Ansible_Project/assets/74359548/c1af5045-c3bb-44c6-92a4-5f4897b2f03c)
![image](https://github.com/samirwadkar31/Ansible_Project/assets/74359548/849fbb73-dd25-4ea9-8a86-589783021fc3)
![image](https://github.com/samirwadkar31/Ansible_Project/assets/74359548/735a44aa-f520-4671-ade8-7bc1daf8577b)
![image](https://github.com/samirwadkar31/Ansible_Project/assets/74359548/1bb374c7-8d91-4398-890b-9e4de2f4c1c2)

Verification:

sameer-managed-node-1:

docker is installed and new user devuser has been created,
![image](https://github.com/samirwadkar31/Ansible_Project/assets/74359548/31eefba2-5ec7-4214-bf66-2708fff91e73)


sameer-managed-node-2:

docker is installed and new user devuser has been created,
![image](https://github.com/samirwadkar31/Ansible_Project/assets/74359548/35f3f474-a46f-419a-9bed-da778d2d3911)


### Ansible PlayBook 2: Performing Security & Compliance Configuration tasks on Target managed nodes.

```
---
- name: Security & Complaince tasks
  hosts: my_servers
  become: true
  remote_user: samirwadkar31
  tasks:
   - name: Ensure password authentication is disabled under SSH folder
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

![image](https://github.com/samirwadkar31/Ansible_Project/assets/74359548/765f4153-fb7a-440d-8b63-2991946b855f)
![image](https://github.com/samirwadkar31/Ansible_Project/assets/74359548/df9b64b1-6ad3-4fb8-a364-225d829bdb1c)
![image](https://github.com/samirwadkar31/Ansible_Project/assets/74359548/a21493f8-a92d-471f-a124-d2240f2fd027)
![image](https://github.com/samirwadkar31/Ansible_Project/assets/74359548/a132505b-4b8c-46c9-b7c7-f0e2275ebf6d)




### Ansible PlayBook 3: Performing sample application deployment task on Target managed nodes.

```
---
- name: Simple Application Deployment
  hosts: my_servers
  become: true
  remote_user: samirwadkar31
  tasks:
    - name: Ensure apt cache is up-to-date
      apt:
        update_cache: yes
    - name: Upgrade all packages to the latest version
      apt:
        upgrade: dist
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

![image](https://github.com/samirwadkar31/Ansible_Project/assets/74359548/58d2ce45-720b-408c-a7af-8488dd1511be)
![image](https://github.com/samirwadkar31/Ansible_Project/assets/74359548/5396be0d-92e2-4670-98e9-01c5e106a245)

sameer-managed-node-1:
![image](https://github.com/samirwadkar31/Ansible_Project/assets/74359548/b038b52e-7b5b-4f74-b184-279c82915600)
sameer-managed-node-2:
![image](https://github.com/samirwadkar31/Ansible_Project/assets/74359548/2e243b61-f626-488d-ada3-bd0b0d9bf06b)


