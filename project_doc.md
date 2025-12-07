# Complete Ansible Infrastructure Project - File Listing

## 📦 All Files You Need to Create

### Root Directory Files

```bash
ignition/
├── config.yaml              # Create from config.example.yaml (GITIGNORED)
├── config.example.yaml      # Template - copy from artifact
├── inventory.ini            # Server inventory - copy from artifact
├── ansible.cfg              # Ansible config - copy from artifact
├── .gitignore              # Protects secrets - copy from artifact
└── README.md               # Documentation - copy from artifact
```

---

## 📂 Directory Structure

```bash
ignition/
├── playbooks/               # Create this directory
│   ├── provision.yml       # Copy from artifact
│   ├── deploy.yml          # Copy from artifact
│   ├── nginx.yml           # Copy from artifact
│   ├── backup.yml          # Copy from artifact
│   └── restore.yml         # Copy from artifact
│
└── templates/              # Create this directory
    ├── systemd-service.j2  # Copy from artifact
    └── nginx-site.conf.j2  # Copy from artifact
```

---

## 📄 File-by-File Breakdown

### 1. **config.example.yaml** (Template - commit to Git)

Contains:
- Server connection settings
- GitHub organization and deploy key placeholder
- Domain configuration
- Database credentials placeholders
- 25 example services configuration
- Backup settings

**Purpose:** Template for others to create their own `config.yaml`

---

### 2. **config.yaml** (Your secrets - NEVER commit)

Contains:
- YOUR actual server IP
- YOUR GitHub deploy key (private)
- YOUR database passwords
- YOUR 25+ service configurations
- YOUR backup encryption key

**Purpose:** Your actual configuration with real secrets

**How to create:**
```bash
cp config.example.yaml config.yaml
nano config.yaml  # Fill in your values
```

---

### 3. **inventory.ini** (Can commit if no sensitive IPs)

Contains:
- Server group definitions
- Connection settings

**Purpose:** Tells Ansible which servers to manage

---

### 4. **ansible.cfg** (Commit to Git)

Contains:
- Ansible behavior configuration
- SSH settings
- Performance optimizations

**Purpose:** Configures Ansible behavior

---

### 5. **.gitignore** (Commit to Git)

Contains:
- config.yaml (most important!)
- SSH keys
- Environment files
- Backup files
- IDE files
- OS files

**Purpose:** Prevents committing secrets

---

### 6. **README.md** (Commit to Git)

Contains:
- Complete documentation
- Quick start guide
- Configuration reference
- Troubleshooting
- Operations guide

**Purpose:** Project documentation

---

### 7. **playbooks/provision.yml** (Commit to Git)

Contains:
- System update tasks
- PostgreSQL 17 installation
- Redis installation
- Go installation
- Node.js installation
- NGINX installation
- User creation
- SSH key deployment
- Firewall configuration

**Purpose:** One-time server setup

**Usage:**
```bash
ansible-playbook -i inventory.ini playbooks/provision.yml -e @config.yaml
```

---

### 8. **playbooks/deploy.yml** (Commit to Git)

Contains:
- Git clone/update for all services
- Database creation
- Go service builds
- Node.js service builds
- Migration execution
- Systemd service creation
- Service start/restart

**Purpose:** Deploy/update all services

**Usage:**
```bash
ansible-playbook -i inventory.ini playbooks/deploy.yml -e @config.yaml
```

---

### 9. **playbooks/nginx.yml** (Commit to Git)

Contains:
- NGINX configuration generation
- Site enabling
- SSL certificate generation
- Certbot auto-renewal setup

**Purpose:** Configure reverse proxy and SSL

**Usage:**
```bash
ansible-playbook -i inventory.ini playbooks/nginx.yml -e @config.yaml
```

---

### 10. **playbooks/backup.yml** (Commit to Git)

Contains:
- PostgreSQL backup (all databases)
- Redis data backup
- NGINX config backup
- Application config backup
- Encryption tasks
- Old backup cleanup

**Purpose:** Create encrypted backups

**Usage:**
```bash
ansible-playbook -i inventory.ini playbooks/backup.yml -e @config.yaml
```

---

### 11. **playbooks/restore.yml** (Commit to Git)

Contains:
- Backup decryption
- Service stopping
- Database restoration
- Redis restoration
- NGINX restoration
- Service restarting

**Purpose:** Restore from backup

**Usage:**
```bash
ansible-playbook -i inventory.ini playbooks/restore.yml -e @config.yaml
```

---

### 12. **templates/systemd-service.j2** (Commit to Git)

Contains:
- Systemd unit file template
- Works for both Go and Node services
- Works for both backend and frontend
- Restart policies
- Dependencies

**Purpose:** Generate systemd service files

**Generated files:**
- `/etc/systemd/system/auth-service.service`
- `/etc/systemd/system/auth-service-frontend.service`
- ... for all services

---

### 13. **templates/nginx-site.conf.j2** (Commit to Git)

Contains:
- NGINX server block template
- Reverse proxy configuration
- Frontend static file serving
- Security headers
- Gzip compression
- Logging configuration

**Purpose:** Generate NGINX configs

**Generated files:**
- `/etc/nginx/sites-available/auth-service-api`
- `/etc/nginx/sites-available/auth-service-app`
- ... for all services

---

## 🔄 Setup Workflow

### Step 1: Create Project Structure

```bash
# Create main directory
mkdir ignition
cd ignition

# Create subdirectories
mkdir playbooks templates

# Verify structure
tree -L 2
```

### Step 2: Create All Files

```bash
# Root files
touch config.example.yaml
touch config.yaml
touch inventory.ini
touch ansible.cfg
touch .gitignore
touch README.md

# Playbooks
touch playbooks/provision.yml
touch playbooks/deploy.yml
touch playbooks/nginx.yml
touch playbooks/backup.yml
touch playbooks/restore.yml

# Templates
touch templates/systemd-service.j2
touch templates/nginx-site.conf.j2
```

### Step 3: Copy Content from Artifact

Copy the content for each file from the artifact I provided into the corresponding files.

### Step 4: Configure Your Setup

```bash
# Create your config from template
cp config.example.yaml config.yaml

# Edit with your values
nano config.yaml
```

### Step 5: Initialize Git

```bash
git init
git add .
git status  # Should NOT show config.yaml

# First commit
git commit -m "Initial Ansible infrastructure setup"
```

---

## ✅ Pre-Deployment Checklist

Before running any playbooks:

- [ ] All 13 files created
- [ ] `config.yaml` exists with your values
- [ ] `config.yaml` is NOT in git (`git status` doesn't show it)
- [ ] GitHub deploy key added to organization
- [ ] DNS records created for all services
- [ ] Server is accessible via SSH
- [ ] All service ports are unique
- [ ] Database passwords are strong

---

## 🗂️ What Gets Committed vs What Doesn't

### ✅ Commit to Git (Safe)

```
ignition/
├── config.example.yaml      ✅ Template only
├── inventory.ini            ✅ No secrets
├── ansible.cfg              ✅ Configuration
├── .gitignore              ✅ Protection
├── README.md               ✅ Documentation
├── playbooks/
│   ├── provision.yml       ✅ Infrastructure code
│   ├── deploy.yml          ✅ Infrastructure code
│   ├── nginx.yml           ✅ Infrastructure code
│   ├── backup.yml          ✅ Infrastructure code
│   └── restore.yml         ✅ Infrastructure code
└── templates/
    ├── systemd-service.j2  ✅ Template
    └── nginx-site.conf.j2  ✅ Template
```

### ❌ NEVER Commit (Secrets)

```
ignition/
├── config.yaml              ❌ YOUR SECRETS
├── .vault_pass              ❌ If you use vault
├── *.pem                    ❌ SSH keys
├── *.key                    ❌ Private keys
└── Any file with passwords  ❌ Credentials
```

---

## 🎯 Quick Commands Reference

```bash
# Initial setup
cp config.example.yaml config.yaml
nano config.yaml

# Provision server (first time)
ansible-playbook -i inventory.ini playbooks/provision.yml -e @config.yaml

# Deploy all services
ansible-playbook -i inventory.ini playbooks/deploy.yml -e @config.yaml

# Setup NGINX + SSL
ansible-playbook -i inventory.ini playbooks/nginx.yml -e @config.yaml

# Create backup
ansible-playbook -i inventory.ini playbooks/backup.yml -e @config.yaml

# Restore from backup
ansible-playbook -i inventory.ini playbooks/restore.yml -e @config.yaml
```

---

## 📊 File Size Reference

Approximate file sizes:

| File | Lines | Size |
|------|-------|------|
| config.example.yaml | 300 | ~12 KB |
| provision.yml | 200 | ~8 KB |
| deploy.yml | 150 | ~6 KB |
| nginx.yml | 100 | ~4 KB |
| backup.yml | 80 | ~3 KB |
| restore.yml | 60 | ~2.5 KB |
| systemd-service.j2 | 30 | ~1 KB |
| nginx-site.conf.j2 | 70 | ~3 KB |
| README.md | 400 | ~16 KB |
| .gitignore | 150 | ~3 KB |

**Total:** ~60 KB of infrastructure code

---

## 🔐 Security Verification

After setup, verify security:

```bash
# 1. Check config.yaml is gitignored
git status | grep config.yaml
# Should return nothing

# 2. Check what would be committed
git status
# Should NOT show config.yaml

# 3. Verify .gitignore is working
cat .gitignore | grep config.yaml
# Should show: config.yaml

# 4. List tracked files
git ls-files | grep config
# Should only show: config.example.yaml
```

---

## 🚀 Deployment Order

Always follow this order:

1. **provision.yml** - Sets up the server (once)
2. **deploy.yml** - Deploys all services
3. **nginx.yml** - Configures reverse proxy + SSL
4. **backup.yml** - Create backups (optional, can schedule via cron)
5. **restore.yml** - Only when restoring from backup

---

This is your complete Ansible infrastructure project! All files are included in the artifact above. Simply copy each section into its respective file.