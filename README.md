# Ansible Industry Demo

## Overview

This repository demonstrates a simple real-world use case of Ansible in industry for automated web server deployment and configuration management.

## Scenario

**Objective**: Deploy and configure web applications across AWS EC2 servers consistently and efficiently.

**Tasks**:
- Configure AWS EC2 servers
- Install Nginx web server
- Deploy a static web application
- Start and manage the web service

## Quick Deployment

### One-Command Deployment
```bash
ansible-playbook -i inventory/hosts playbooks/deploy.yml
```

This demonstrates how Ansible is used in companies for application deployment and server configuration.

## Project Structure

```
ansible-industry-demo/
├── README.md              # Project documentation
├── ansible.cfg             # Ansible configuration settings
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

---

## File-by-File Breakdown

### 1. README.md (Documentation)


**Industry Standard**: 
- Explains what the project does
- How to run it
- What problem it solves
- Mandatory in industry so any engineer can understand your automation quickly

### 2. ansible.cfg (Configuration)

```ini
[defaults]
inventory = inventory/hosts
host_key_checking = False
remote_user = ec2-user
```

**Configuration Breakdown**:

| Setting | Purpose | Industry Benefit |
|---------|---------|------------------|
| `inventory = inventory/hosts` | Auto-loads inventory file | No need for `-i inventory/hosts` parameter |
| `host_key_checking = False` | Bypasses SSH verification prompt | Enables automated CI/CD pipelines |
| `remote_user = ec2-user` | Default SSH user for EC2 | Consistent across EC2 instances (Ubuntu uses `ubuntu`) |

**Industry Impact**: Makes Ansible behave consistently across company environments.

### 3. inventory/hosts (Environment Map)

```ini
[web]
3.109.201.228 ansible_user=ec2-user ansible_ssh_private_key_file=~/mykey.pem
```

**Configuration Details**:
- **Group**: `web` - Logical server grouping
- **Target**: `3.109.201.228` - Server IP address  
- **SSH User**: `ec2-user` - Connection username
- **Private Key**: `~/mykey.pem` - Authentication method

**Industry Context**: Inventory = environment map (dev/staging/prod server definitions)

### 4. playbooks/deploy.yml (Orchestration)

```yaml
- hosts: web
  become: yes
  roles:
    - web
```

**Playbook Structure**:

| Directive | Function | Business Value |
|-----------|----------|----------------|
| `hosts: web` | Target server group | Environment-specific deployments |
| `become: yes` | Enable sudo privileges | System-level configurations |
| `roles: - web` | Apply web server role | Modular, reusable components |

**Industry Practice**:
- **Playbooks** = Orchestration layer
- **Roles** = Implementation layer

### 5. roles/web/tasks/main.yml (Implementation)

#### Task 1: Install Nginx
```yaml
- name: Install nginx
  package:
    name: nginx
    state: present
```

**Idempotent Behavior**:
- ✅ Already installed → Nothing happens
- ⚡ Not installed → Installs nginx

#### Task 2: Deploy Application
```yaml
- name: Copy application file
  copy:
    src: index.html
    dest: /usr/share/nginx/html/index.html
  notify: restart nginx
```

**Key Features**:
- Copies application files (website)
- Triggers handler if file changes
- **Industry Pattern**: Code changes → Service restart

#### Task 3: Service Management
```yaml
- name: Start nginx service
  service:
    name: nginx
    state: started
    enabled: yes
```

**Service Configuration**:
- Starts nginx service
- Enables auto-start on reboot

### 6. roles/web/handlers/main.yml (Change Management)

```yaml
- name: restart nginx
  service:
    name: nginx
    state: restarted
```

**Handler Logic**:
- **Trigger**: Only runs when something changes
- **Industry Benefit**: Avoids unnecessary service restarts

### 7. roles/web/files/index.html (Application Artifacts)

```html
<h1>Deployment Successful!</h1>
```

**Production Context**:
- **Demo**: Simple HTML file
- **Real Companies**: Java JAR, Node.js, Python packages, Docker images
- **Ansible Flexibility**: Deploys any type of artifact

---

## Execution Flow

When running `ansible-playbook playbooks/deploy.yml`:

### Step 1: Configuration Loading
- Reads `ansible.cfg` for settings
- Loads `inventory/hosts` for target definitions

### Step 2: Target Discovery
- **Group**: `web`
- **Server IP**: `3.109.201.228`  
- **SSH Key**: Private key authentication

### Step 3: Connection Establishment
```
Your Laptop → SSH → AWS EC2 Server
```

### Step 4: Playbook Execution
- Applies `web` role to `web` group servers

### Step 5: Role Tasks Execution
1. **Install nginx** (if not present)
2. **Copy website** files
3. **Start nginx** service  
4. **Restart handler** (if changes detected)

### Step 6: Verification
```bash
# Access deployed application
curl http://<EC2_PUBLIC_IP>
# Expected: "Deployment Successful!"
```

---

## Industry Mapping

| Component | Industry Equivalent | Real-World Purpose |
|-----------|-------------------|-------------------|
| `inventory` | Environment Configuration | Dev/Staging/Prod server definitions |
| `ansible.cfg` | Automation Standards | Consistent behavior across teams |
| `playbook` | Deployment Pipeline | CI/CD orchestration workflow |
| `roles` | Service Modules | Reusable, testable components |
| `handlers` | Change Management | Controlled service operations |
| `files` | Artifacts Repository | Deployable application components |

## Benefits in Production

### ✅ **Consistency**
- Eliminates configuration drift
- Standardized deployments

### ✅ **Reliability** 
- Idempotent operations
- Predictable outcomes

### ✅ **Scalability**
- Single command → Multiple servers
- Role-based architecture

### ✅ **Maintainability**
- Version-controlled infrastructure
- Self-documenting code

---

## Next Steps for Enterprise

1. **Multi-Environment**: Separate inventories for dev/staging/prod
2. **Security**: Ansible Vault for secrets management
3. **Testing**: Molecule for role validation
4. **CI/CD Integration**: Jenkins/GitLab pipeline integration
5. **Monitoring**: Infrastructure and application monitoring

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


