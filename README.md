# Ansible Industry Demo (Simple)

This repository demonstrates a simple real-world use case of Ansible in industry.

Scenario:
- Configure AWS EC2 servers
- Install Nginx
- Deploy a static web application
- Start and manage the web service

One command deployment:
ansible-playbook -i inventory/hosts playbooks/deploy.yml

This shows how Ansible is used in companies for application deployment and server configuration.




Let’s go line by line and understand how the whole flow works.

Folder structure:

ansible-industry-demo/
├── README.md
├── ansible.cfg
├── inventory/
│   └── hosts
├── playbooks/
│   └── deploy.yml
└── roles/
    └── web/
        ├── tasks/main.yml
        ├── handlers/main.yml
        └── files/index.html


Think of this like:

Configuration → Targets → Playbook → Role → Tasks → Result


README.md
This is just documentation for humans.

It explains:

What the project does

How to run it

What problem it solves

In industry:
README is mandatory so any engineer can understand your automation quickly.

ansible.cfg

[defaults]
inventory = inventory/hosts
host_key_checking = False
remote_user = ec2-user


Meaning:

inventory = inventory/hosts
You don’t need to pass -i inventory/hosts every time.
Ansible automatically reads it.

host_key_checking = False
Avoids SSH prompt:

Are you sure you want to continue connecting?

Used in automation pipelines.

remote_user = ec2-user
Default SSH user for EC2.
(Ubuntu users will change this to ubuntu)

This file makes Ansible behave consistently like in companies.

inventory/hosts

[web]
3.109.201.228 ansible_user=ec2-user ansible_ssh_private_key_file=~/mykey.pem


This tells Ansible:

Group name: web

Target server IP: 3.109.201.228

SSH user: ec2-user

Private key: mykey.pem

So Ansible now knows:

“To reach this server, SSH using ec2-user and this key.”

This is your target environment definition.

Industry meaning:
Inventory = environment map.

playbooks/deploy.yml

- hosts: web
  become: yes
  roles:
    - web


Line by line:

hosts: web
Run this playbook on all servers in [web] group.

become: yes
Use sudo (root permissions).

roles:
Instead of writing tasks directly, use a role called web.

Meaning:

“Apply the web server configuration on all web servers.”

This is exactly how real teams work:
Playbooks = orchestration
Roles = implementation

roles/web/tasks/main.yml

- name: Install nginx
  package:
    name: nginx
    state: present


Ansible:

Make sure nginx is installed.

Idempotent:

If already installed → nothing happens

If not installed → install it

- name: Copy application file
  copy:
    src: index.html
    dest: /usr/share/nginx/html/index.html
  notify: restart nginx


Meaning:

Copy your application file (website)

If the file changes → trigger handler

This is deployment in real companies:
Code changes → service restart.

- name: Start nginx service
  service:
    name: nginx
    state: started
    enabled: yes


Meaning:

Start nginx

Make sure it starts on reboot

roles/web/handlers/main.yml

- name: restart nginx
  service:
    name: nginx
    state: restarted


Handler logic:

Only runs if something changes.

In industry:
Handlers avoid unnecessary service restarts.

roles/web/files/index.html

This is your application:

<h1>Deployment Successful!</h1>


In real companies:

Instead of HTML

It could be Java, Node, Python app files

Ansible doesn’t care. It just deploys artifacts.

Now the FULL FLOW when you run:

ansible-playbook playbooks/deploy.yml


Step-by-step:

Ansible reads:

ansible.cfg

inventory/hosts

Finds:

Group = web

Server IP

SSH key

Connects via SSH:

Your Laptop → AWS EC2


Runs playbook:

deploy.yml


Playbook applies role:

web


Role executes:

Install nginx

Copy website

Start service

Restart if needed

Result:
Open browser:

http://<EC2_PUBLIC_IP>


You see your deployed app.

This is exactly how Ansible is used in industry:

Component	Industry Meaning
inventory	Environment configuration
ansible.cfg	Automation standards
playbook	Deployment pipeline
roles	Reusable service modules
handlers	Controlled restarts
files	Application artifacts


