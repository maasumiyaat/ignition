# Automated Infrastructure Deployment with Ansible

Deploy 25+ Go/Node.js microservices on Ubuntu 22.04 LTS with PostgreSQL, Redis, NGINX, and automated SSL.

## 📁 Project Structure

```
ignition/
├── config.yaml              # YOUR SECRETS (gitignored, never commit)
├── config.example.yaml      # Template (commit this)
├── inventory.ini            # Server inventory
├── ansible.cfg              # Ansible configuration
├── .gitignore              # Protects secrets
├── README.md               # This file
├── playbooks/
│   ├── provision.yml       # Server provisioning
│   ├── deploy.yml          # Service deployment
│   ├── nginx.yml           # NGINX + SSL setup
│   ├── backup.yml          # Automated backups
│   └── restore.yml         # Restore from backup
└── templates/
    ├── systemd-service.j2  # Systemd unit template
    └── nginx-site.conf.j2  # NGINX config template
```

## 🚀 Quick Start

### 1. Install Ansible

```bash
# Ubuntu/Debian
sudo apt update && sudo apt install ansible

# macOS
brew install ansible

# Verify
ansible --version
```

### 2. Setup Configuration

```bash
# Copy template
cp config.example.yaml config.yaml

# Edit with your values
nano config.yaml
```

### 3. Configure Your Services

In `config.yaml`, each service needs:

```yaml
services:
  - name: auth-service           # Service identifier
    type: go                     # 'go' or 'node'
    github_repo: auth-service    # GitHub repo name
    backend_port: 8001           # Unique port
    frontend_enabled: true       # Has frontend?
    frontend_port: 3001          # Frontend port (if enabled)
    database_enabled: true       # Needs database?
    database_name: auth_db       # Database name
    run_migrations: true         # Run migrations?
```

**Important:** Each service reads its own `config.yaml` from its repository. This Ansible config only handles:
- ✅ Which repos to clone
- ✅ Port assignments
- ✅ Database creation
- ✅ Service orchestration

### 4. Setup GitHub Deploy Key

```bash
# Generate SSH key
ssh-keygen -t rsa -b 4096 -C "deploy@yourcompany.com" -f ~/.ssh/github_deploy_key

# Add public key to GitHub org
cat ~/.ssh/github_deploy_key.pub
# Go to: https://github.com/organizations/YOUR-ORG/settings/keys
# Add as deploy key with READ access only

# Add private key to config.yaml
cat ~/.ssh/github_deploy_key
# Paste entire output (including BEGIN/END lines) into config.yaml
```

### 5. Setup DNS

Create A records for each service:

```
api.auth-service.example.com     → YOUR_SERVER_IP
app.auth-service.example.com     → YOUR_SERVER_IP
api.user-service.example.com     → YOUR_SERVER_IP
app.user-service.example.com     → YOUR_SERVER_IP
# ... for all services
```

### 6. Deploy

```bash
# Step 1: Provision server (one time)
ansible-playbook -i inventory.ini playbooks/provision.yml -e @config.yaml

# Step 2: Deploy all services
ansible-playbook -i inventory.ini playbooks/deploy.yml -e @config.yaml

# Step 3: Setup NGINX + SSL
ansible-playbook -i inventory.ini playbooks/nginx.yml -e @config.yaml
```

## 📝 Configuration Reference

### Server Connection

```yaml
server:
  hostname: 123.45.67.89        # Server IP or domain
  ssh_user: root                # SSH user
  ssh_key_path: ~/.ssh/id_rsa   # SSH private key path
```

### GitHub Settings

```yaml
github:
  organization: your-github-org  # GitHub organization name
  ssh_deploy_key: |              # Private deploy key
    -----BEGIN OPENSSH PRIVATE KEY-----
    ...
    -----END OPENSSH PRIVATE KEY-----
```

### Database Credentials

```yaml
database:
  postgresql:
    version: 17
    admin_user: postgres
    admin_password: "YourStrongPassword123!"  # Change this
    port: 5432
  
  redis:
    password: "AnotherStrongPassword456!"     # Change this
    port: 6379
```

### Service Configuration

Each service in the `services` array:

```yaml
- name: payment-service          # Unique service name
  type: node                     # 'go' or 'node'
  github_repo: payment-service   # Repo name (not full URL)
  backend_port: 8003             # Unique backend port (8001-8999)
  frontend_enabled: false        # true if has frontend
  frontend_port: 3003            # Frontend port (3001-3999) if enabled
  database_enabled: true         # true if needs PostgreSQL
  database_name: payment_db      # Database name
  run_migrations: true           # true to run migrations on deploy
```

**Service Config Files:** Each service repo should have its own `config.yaml` with:
- Database connection strings
- API keys
- Third-party credentials
- Service-specific settings

## 🔧 Operations

### Deploy Single Service

```bash
# Re-deploy one service after code changes
ansible-playbook -i inventory.ini playbooks/deploy.yml -e @config.yaml \
  --extra-vars "services=[{'name':'auth-service','type':'go','github_repo':'auth-service','backend_port':8001,'database_enabled':true,'database_name':'auth_db'}]"
```

### Check Service Status

```bash
# SSH into server
ssh deployer@YOUR_SERVER_IP

# Check service status
sudo systemctl status auth-service
sudo systemctl status auth-service-frontend

# View logs
sudo journalctl -u auth-service -f
sudo journalctl -u auth-service-frontend -f

# Restart service
sudo systemctl restart auth-service
```

### Manual Backup

```bash
ansible-playbook -i inventory.ini playbooks/backup.yml -e @config.yaml
```

### Restore from Backup

```bash
# List backups
ssh deployer@YOUR_SERVER_IP "ls -lh /var/backups/automated/"

# Restore (will prompt for timestamp)
ansible-playbook -i inventory.ini playbooks/restore.yml -e @config.yaml
```

## 🗄️ Database Management

```bash
# Connect to PostgreSQL
ssh deployer@YOUR_SERVER_IP
sudo -u postgres psql

# List databases
\l

# Connect to specific database
\c auth_db

# List tables
\dt

# Connect to Redis
redis-cli
AUTH your_redis_password
PING
KEYS *
```

## 🌐 Service URLs

After deployment, services are accessible at:

**Backend APIs:**
```
https://api.auth-service.example.com
https://api.user-service.example.com
https://api.payment-service.example.com
```

**Frontend Apps:**
```
https://app.auth-service.example.com
https://app.user-service.example.com
https://app.catalog-service.example.com
```

## 🔐 Security Features

- ✅ UFW firewall (ports 22, 80, 443 only)
- ✅ SSL/TLS certificates via Let's Encrypt
- ✅ Auto-renewal of SSL certificates
- ✅ GitHub deploy key (read-only)
- ✅ PostgreSQL password authentication
- ✅ Redis password protection
- ✅ Encrypted backups (AES-256)
- ✅ Service isolation (dedicated user)
- ✅ Security headers in NGINX

## 🛠️ Troubleshooting

### Service Won't Start

```bash
# Check status
sudo systemctl status service-name

# Check logs
sudo journalctl -u service-name -n 50

# Check if port is in use
sudo lsof -i :8001

# Verify binary exists (Go services)
ls -la /home/deployer/apps/service-name/service-name

# Check service config file exists
ls -la /home/deployer/apps/service-name/config.yaml
```

### Database Connection Issues

```bash
# Check PostgreSQL is running
sudo systemctl status postgresql

# Verify database exists
sudo -u postgres psql -l | grep service_db

# Test connection
sudo -u postgres psql -d service_db -c "SELECT 1;"
```

### SSL Certificate Issues

```bash
# Check certificates
sudo certbot certificates

# Renew manually
sudo certbot renew --dry-run

# Check NGINX config
sudo nginx -t
```

### DNS Not Resolving

```bash
# Check DNS
nslookup api.auth-service.example.com
dig api.auth-service.example.com

# Wait for DNS propagation (can take 5-30 minutes)
```

## 📊 Service Configuration in Repos

Each service repository should have its own `config.yaml`:

```yaml
# Example: auth-service/config.yaml
server:
  port: 8001
  host: 0.0.0.0

database:
  host: localhost
  port: 5432
  name: auth_db
  user: postgres
  password: ${DB_PASSWORD}  # From environment or config

redis:
  host: localhost
  port: 6379
  password: ${REDIS_PASSWORD}

jwt:
  secret: ${JWT_SECRET}
  expiry: 24h

logging:
  level: info
  format: json
```

This separation allows:
- ✅ Service-specific configuration in each repo
- ✅ Infrastructure-level config in Ansible
- ✅ Easy local development
- ✅ Clear separation of concerns

## 🔄 Adding New Services

1. Add to `config.yaml`:

```yaml
services:
  - name: new-service
    type: go
    github_repo: new-service
    backend_port: 8026           # Use next available port
    frontend_enabled: false
    database_enabled: true
    database_name: new_service_db
    run_migrations: true
```

2. Create DNS records:
```
api.new-service.example.com → YOUR_SERVER_IP
```

3. Deploy:
```bash
ansible-playbook -i inventory.ini playbooks/deploy.yml -e @config.yaml
ansible-playbook -i inventory.ini playbooks/nginx.yml -e @config.yaml
```

## 📈 Monitoring

Service logs are available via `journalctl`:

```bash
# Real-time logs
sudo journalctl -u auth-service -f

# Last 100 lines
sudo journalctl -u auth-service -n 100

# Logs since specific time
sudo journalctl -u auth-service --since "1 hour ago"

# All service logs
sudo journalctl -f | grep -E '(auth|user|payment)-service'
```

NGINX logs:

```bash
# Access logs
sudo tail -f /var/log/nginx/api.auth-service.example.com_access.log

# Error logs
sudo tail -f /var/log/nginx/api.auth-service.example.com_error.log
```

## 🔒 Security Checklist

- [ ] config.yaml is gitignored
- [ ] Strong database passwords (20+ chars)
- [ ] GitHub deploy key is read-only
- [ ] SSH key has passphrase
- [ ] UFW firewall enabled
- [ ] SSL certificates active
- [ ] Backups are encrypted
- [ ] Services run as non-root user
- [ ] Each service has its own config.yaml with secrets

## 📚 Additional Resources

- [Ansible Documentation](https://docs.ansible.com/)
- [PostgreSQL Docs](https://www.postgresql.org/docs/)
- [NGINX Configuration](https://nginx.org/en/docs/)
- [Let's Encrypt](https://letsencrypt.org/)
- [systemd Service Management](https://www.freedesktop.org/software/systemd/man/systemd.service.html)

## 🤝 Support

For issues:
1. Check service logs: `sudo journalctl -u service-name -f`
2. Verify configuration: `cat /home/deployer/apps/service-name/config.yaml`
3. Test connectivity: `curl http://localhost:8001/health`
4. Check NGINX: `sudo nginx -t`

---

**⚠️ Remember:** 
- Never commit `config.yaml` to Git
- Keep backup encryption keys safe
- Rotate database passwords regularly
- Monitor service logs for issues