# Ansible Industry Demo

## Overview

This repository demonstrates a real-world industry use case of Ansible for automated web server deployment and configuration management.

## Scenario

**Business Problem**: Deploy and configure web applications across AWS EC2 servers consistently and efficiently.

**Solution**: Automated deployment pipeline using Ansible that:
- Configures AWS EC2 servers
- Installs and configures Nginx web server
- Deploys static web application
- Manages web services lifecycle

## Quick Start

### Prerequisites
- AWS EC2 instance with SSH access
- Private key file (`.pem`)
- Ansible installed on control machine

### One-Command Deployment
```bash
ansible-playbook -i inventory/hosts playbooks/deploy.yml
```

## Project Structure

```
ansible-industry-demo/
├── README.md              # Project documentation
├── ansible.cfg             # Ansible configuration
├── inventory/
│   └── hosts              # Target servers definition
├── playbooks/
│   └── deploy.yml         # Main deployment orchestration
└── roles/
    └── web/               # Web server role
        ├── tasks/main.yml     # Installation & configuration tasks
        ├── handlers/main.yml  # Service management handlers
        └── files/index.html   # Application artifacts
```

## Architecture Flow

```
Configuration → Targets → Playbook → Role → Tasks → Result
```

## Component Breakdown

### 1. Configuration (`ansible.cfg`)

```ini
[defaults]
inventory = inventory/hosts
host_key_checking = False
remote_user = ec2-user
```

**Purpose**: Standardizes Ansible behavior across environments

| Setting | Purpose | Industry Benefit |
|---------|---------|------------------|
| `inventory` | Auto-loads inventory file | Eliminates manual `-i` parameter |
| `host_key_checking = False` | Bypasses SSH verification prompts | Enables automated CI/CD pipelines |
| `remote_user = ec2-user` | Sets default SSH user | Consistent across EC2 instances |

### 2. Inventory (`inventory/hosts`)

```ini
[web]
3.109.201.228 ansible_user=ec2-user ansible_ssh_private_key_file=~/mykey.pem
```

**Purpose**: Environment mapping and connection details

- **Group**: `web` - Logical grouping of web servers
- **Target**: `3.109.201.228` - Server IP address
- **Authentication**: SSH key-based authentication
- **Industry Context**: Represents environment configuration (dev/staging/prod)

### 3. Playbook (`playbooks/deploy.yml`)

```yaml
- hosts: web
  become: yes
  roles:
    - web
```

**Purpose**: Deployment orchestration

| Directive | Function | Business Value |
|-----------|----------|----------------|
| `hosts: web` | Target server group | Environment-specific deployments |
| `become: yes` | Enable sudo privileges | System-level configurations |
| `roles: - web` | Apply web server role | Modular, reusable components |

### 4. Role Structure (`roles/web/`)

#### Tasks (`roles/web/tasks/main.yml`)

```yaml
# Install nginx web server
- name: Install nginx
  package:
    name: nginx
    state: present

# Deploy application files
- name: Copy application file
  copy:
    src: index.html
    dest: /usr/share/nginx/html/index.html
  notify: restart nginx

# Ensure service is running
- name: Start nginx service
  service:
    name: nginx
    state: started
    enabled: yes
```

**Key Principles**:
- **Idempotency**: Safe to run multiple times
- **Change Detection**: Triggers handlers only when needed
- **Service Management**: Ensures availability and auto-start

#### Handlers (`roles/web/handlers/main.yml`)

```yaml
- name: restart nginx
  service:
    name: nginx
    state: restarted
```

**Purpose**: Controlled service restarts
- **Efficiency**: Only restarts when configuration changes
- **Industry Standard**: Minimizes service downtime

#### Files (`roles/web/files/index.html`)

```html
<h1>Deployment Successful!</h1>
```

**Purpose**: Application artifacts
- **Scalability**: In production, could be JAR, WAR, or Docker images
- **Version Control**: Artifacts managed alongside infrastructure code

## Execution Flow

When running `ansible-playbook playbooks/deploy.yml`:

1. **Configuration Loading**
   - Reads `ansible.cfg` for default settings
   - Loads `inventory/hosts` for target servers

2. **Connection Establishment**
   - SSH connection: `Control Machine → EC2 Server`
   - Authentication via private key

3. **Playbook Execution**
   - Applies `web` role to `web` group servers

4. **Task Execution**
   - Installs Nginx (if not present)
   - Deploys application files
   - Starts/enables Nginx service
   - Triggers restart handler (if changes detected)

5. **Verification**
   - Access application: `http://<EC2_PUBLIC_IP>`

## Industry Mapping

| Component | Industry Equivalent | Purpose |
|-----------|-------------------|---------|
| `inventory` | Environment Configuration | Dev/Staging/Prod server definitions |
| `ansible.cfg` | Automation Standards | Consistent behavior across teams |
| `playbook` | Deployment Pipeline | Orchestration and workflow |
| `roles` | Service Modules | Reusable, testable components |
| `handlers` | Change Management | Controlled service operations |
| `files` | Artifacts Repository | Deployable application components |

## Benefits in Industry Context

### 1. **Consistency**
- Eliminates configuration drift
- Standardized deployments across environments

### 2. **Repeatability** 
- Idempotent operations
- Version-controlled infrastructure

### 3. **Scalability**
- Single command deploys to multiple servers
- Role-based architecture supports complex applications

### 4. **Maintainability**
- Clear separation of concerns
- Self-documenting infrastructure code

### 5. **Reliability**
- Automated testing and validation
- Rollback capabilities

## Next Steps for Production

1. **Multi-Environment Support**: Separate inventory files for dev/staging/prod
2. **Security Hardening**: Vault for secrets management
3. **Testing Integration**: Molecule for role testing
4. **CI/CD Integration**: Jenkins/GitLab CI pipeline integration
5. **Monitoring**: Application and infrastructure monitoring setup

## Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| SSH connection failed | Verify private key permissions (`chmod 600`) |
| Permission denied | Check `ansible_user` and key file path |
| Nginx fails to start | Check port 80 availability and firewall rules |
| Files not copying | Verify file paths and permissions |

### Validation Commands

```bash
# Test connectivity
ansible web -m ping

# Check nginx status
ansible web -m shell -a "systemctl status nginx"

# Verify deployment
curl http://<EC2_PUBLIC_IP>
```