# Setting up GitLab Server

## Overview
GitLab can be deployed in multiple ways depending on your needs: self-hosted, cloud-hosted, or using GitLab.com (SaaS).

## Installation Methods

### 1. GitLab.com (SaaS)
The easiest way to get started with GitLab.

**Steps:**
1. Visit [gitlab.com](https://gitlab.com)
2. Sign up for a free account
3. Choose between Free, Premium, or Ultimate tier
4. Start creating projects immediately

**Advantages:**
- No infrastructure management
- Automatic updates
- High availability
- Free tier available

### 2. Self-Hosted GitLab (Omnibus Package)

#### System Requirements
- **Minimum:** 4 CPU cores, 4GB RAM, 20GB storage
- **Recommended:** 8 CPU cores, 8GB RAM, 50GB+ storage
- Supported OS: Ubuntu, Debian, CentOS, RHEL

#### Installation on Ubuntu/Debian

```bash
# Install dependencies
sudo apt-get update
sudo apt-get install -y curl openssh-server ca-certificates perl

# Add GitLab package repository
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh | sudo bash

# Install GitLab
sudo EXTERNAL_URL="https://gitlab.example.com" apt-get install gitlab-ee
```

#### Installation on CentOS/RHEL

```bash
# Install dependencies
sudo yum install -y curl policycoreutils openssh-server perl

# Add GitLab package repository
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.rpm.sh | sudo bash

# Install GitLab
sudo EXTERNAL_URL="https://gitlab.example.com" yum install -y gitlab-ee
```

### 3. Docker Installation

```bash
# Create volumes for persistent data
export GITLAB_HOME=/srv/gitlab

# Run GitLab container
sudo docker run --detach \
  --hostname gitlab.example.com \
  --publish 443:443 --publish 80:80 --publish 22:22 \
  --name gitlab \
  --restart always \
  --volume $GITLAB_HOME/config:/etc/gitlab \
  --volume $GITLAB_HOME/logs:/var/log/gitlab \
  --volume $GITLAB_HOME/data:/var/opt/gitlab \
  --shm-size 256m \
  gitlab/gitlab-ee:latest
```

### 4. Kubernetes Installation (Helm Chart)

```bash
# Add GitLab Helm repository
helm repo add gitlab https://charts.gitlab.io/
helm repo update

# Install GitLab
helm install gitlab gitlab/gitlab \
  --set global.hosts.domain=example.com \
  --set global.hosts.externalIP=10.10.10.10 \
  --set certmanager-issuer.email=admin@example.com
```

## Initial Configuration

### 1. Access GitLab
After installation, access GitLab at the URL you specified (e.g., `https://gitlab.example.com`)

### 2. Set Root Password
On first access, you'll need to set the root password. The initial root password is stored in:
```bash
sudo cat /etc/gitlab/initial_root_password
```

### 3. Configure GitLab
Edit the configuration file:
```bash
sudo nano /etc/gitlab/gitlab.rb
```

**Common configurations:**

```ruby
# External URL
external_url 'https://gitlab.example.com'

# Email configuration
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.gmail.com"
gitlab_rails['smtp_port'] = 587
gitlab_rails['smtp_user_name'] = "your-email@gmail.com"
gitlab_rails['smtp_password'] = "your-password"
gitlab_rails['smtp_domain'] = "smtp.gmail.com"
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['gitlab_email_from'] = 'your-email@gmail.com'

# Backup configuration
gitlab_rails['backup_keep_time'] = 604800  # Keep backups for 7 days

# Container registry
registry_external_url 'https://registry.example.com'
```

### 4. Reconfigure GitLab
After making changes:
```bash
sudo gitlab-ctl reconfigure
```

## Essential GitLab Commands

```bash
# Check GitLab status
sudo gitlab-ctl status

# Start GitLab
sudo gitlab-ctl start

# Stop GitLab
sudo gitlab-ctl stop

# Restart GitLab
sudo gitlab-ctl restart

# View logs
sudo gitlab-ctl tail

# Backup GitLab
sudo gitlab-backup create

# Restore from backup
sudo gitlab-backup restore BACKUP=<backup_timestamp>
```

## SSL/TLS Configuration

### Using Let's Encrypt (Automatic)
```ruby
# In /etc/gitlab/gitlab.rb
letsencrypt['enable'] = true
letsencrypt['contact_emails'] = ['admin@example.com']
external_url 'https://gitlab.example.com'
```

### Using Custom Certificates
```bash
# Copy certificates
sudo mkdir -p /etc/gitlab/ssl
sudo cp gitlab.example.com.crt /etc/gitlab/ssl/
sudo cp gitlab.example.com.key /etc/gitlab/ssl/
sudo chmod 600 /etc/gitlab/ssl/gitlab.example.com.*
```

```ruby
# In /etc/gitlab/gitlab.rb
external_url 'https://gitlab.example.com'
nginx['ssl_certificate'] = "/etc/gitlab/ssl/gitlab.example.com.crt"
nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/gitlab.example.com.key"
```

## Performance Tuning

### Database Optimization
```ruby
# In /etc/gitlab/gitlab.rb
postgresql['shared_buffers'] = "256MB"
postgresql['max_connections'] = 200
```

### Redis Configuration
```ruby
redis['maxmemory'] = "512MB"
redis['maxmemory_policy'] = "allkeys-lru"
```

### Worker Configuration
```ruby
unicorn['worker_processes'] = 4
sidekiq['concurrency'] = 25
```

## Monitoring and Health Checks

### Health Check Endpoint
```bash
curl http://gitlab.example.com/-/health
```

### Readiness Check
```bash
curl http://gitlab.example.com/-/readiness
```

### Liveness Check
```bash
curl http://gitlab.example.com/-/liveness
```

## Troubleshooting

### Check logs
```bash
# All logs
sudo gitlab-ctl tail

# Specific service
sudo gitlab-ctl tail nginx
sudo gitlab-ctl tail postgresql
sudo gitlab-ctl tail redis
```

### Restart specific services
```bash
sudo gitlab-ctl restart nginx
sudo gitlab-ctl restart postgresql
sudo gitlab-ctl restart unicorn
```

### Common Issues

**Port 80/443 already in use:**
```bash
# Check what's using the port
sudo netstat -tulpn | grep :80
sudo lsof -i :80
```

**GitLab not starting:**
```bash
# Check system resources
free -h
df -h

# Check GitLab status
sudo gitlab-ctl status
sudo gitlab-rake gitlab:check
```

## Security Best Practices

1. **Keep GitLab Updated**
   ```bash
   sudo apt-get update && sudo apt-get install gitlab-ee
   ```

2. **Enable Two-Factor Authentication**
   - Admin Area → Settings → General → Sign-in restrictions
   - Enable "Require two-factor authentication"

3. **Configure Firewall**
   ```bash
   sudo ufw allow 80/tcp
   sudo ufw allow 443/tcp
   sudo ufw allow 22/tcp
   sudo ufw enable
   ```

4. **Regular Backups**
   ```bash
   # Set up automatic backups
   sudo crontab -e
   # Add: 0 2 * * * /opt/gitlab/bin/gitlab-backup create CRON=1
   ```

5. **Limit SSH Key Types**
   ```ruby
   # In /etc/gitlab/gitlab.rb
   gitlab_rails['allowed_key_types'] = ['rsa', 'ecdsa', 'ed25519']
   ```

## Upgrade GitLab

### Check upgrade path
Visit [GitLab Upgrade Path](https://gitlab-com.gitlab.io/support/toolbox/upgrade-path/) to verify safe upgrade path.

### Backup before upgrade
```bash
sudo gitlab-backup create
```

### Upgrade
```bash
# For Omnibus installations
sudo apt-get update
sudo apt-get install gitlab-ee

# Reconfigure
sudo gitlab-ctl reconfigure
```

## References
- [Official GitLab Installation Docs](https://about.gitlab.com/install/)
- [GitLab Configuration Guide](https://docs.gitlab.com/omnibus/settings/)
- [GitLab Architecture](https://docs.gitlab.com/ee/development/architecture.html)
