# 📧 Self-Hosted SMTP Email Server – tyfsadik.org


A complete, production-ready **self-hosted SMTP email server** implementation for the domain **tyfsadik.org**. This setup ensures secure, reliable email delivery with enterprise-grade authentication protocols and security measures.

## 🚀 Quick Start

### Prerequisites
- Ubuntu 22.04 LTS server
- Domain name (tyfsadik.org)
- Static IP address
- Root access to the server

## ✨ Features

- **SMTP Service** - Postfix with TLS/SSL encryption
- **IMAP/POP3 Service** - Dovecot with Maildir storage
- **Email Authentication** - SPF, DKIM, DMARC configured
- **Security** - Fail2ban, UFW firewall, SSL certificates
- **Spam Protection** - Basic RBL checks and filtering
- **Reliable Delivery** - Proper DNS configuration and reverse DNS


## 🏗️ Architecture

<img width="3415" height="1848" alt="image" src="https://github.com/user-attachments/assets/0b57468c-6c12-4e4b-8f53-59370db5cf7e" />


## ⚙️ Installation Steps

### 1. System Preparation
```bash
# Update system and install dependencies
sudo apt update && sudo apt upgrade -y
sudo apt install -y postfix dovecot-core dovecot-imapd dovecot-pop3d \
    openssl opendkim opendkim-tools ufw fail2ban mailutils
```

### 2. Basic Configuration

**Postfix Main Configuration** (`/etc/postfix/main.cf`):
```bash
sudo nano /etc/postfix/main.cf
```

Add/update these settings:
```ini
myhostname = mail.tyfsadik.org
mydomain = tyfsadik.org
myorigin = $mydomain
inet_interfaces = all
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
home_mailbox = Maildir/
smtpd_use_tls = yes
smtpd_tls_cert_file = /etc/ssl/certs/mailserver.crt
smtpd_tls_key_file = /etc/ssl/private/mailserver.key
```

### 3. SSL Certificate Generation
```bash
# Create SSL directories
sudo mkdir -p /etc/ssl/private
sudo mkdir -p /etc/ssl/certs

# Generate self-signed certificate
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/mailserver.key \
    -out /etc/ssl/certs/mailserver.crt \
    -subj "/C=US/ST=State/L=City/O=Organization/CN=mail.tyfsadik.org"
```

### 4. Dovecot Configuration
```bash
sudo nano /etc/dovecot/dovecot.conf
```

Basic configuration:
```ini
mail_location = maildir:~/Maildir
protocols = imap pop3
ssl = yes
ssl_cert = </etc/ssl/certs/mailserver.crt
ssl_key = </etc/ssl/private/mailserver.key
```

## 🌐 DNS Configuration

### Essential DNS Records:

| Type | Name | Value | Priority |
|------|------|-------|----------|
| **A** | `mail.tyfsadik.org` | `YOUR_SERVER_IP` | - |
| **MX** | `@` | `mail.tyfsadik.org` | 10 |
| **TXT** | `@` | `v=spf1 mx ~all` | - |
| **TXT** | `_dmarc` | `v=DMARC1; p=none; rua=mailto:admin@tyfsadik.org` | - |

### DKIM Setup
```bash
# Install and configure OpenDKIM
sudo apt install -y opendkim opendkim-tools

# Generate DKIM keys
sudo mkdir -p /etc/opendkim/keys/tyfsadik.org
sudo opendkim-genkey -D /etc/opendkim/keys/tyfsadik.org/ -d tyfsadik.org -s mail
sudo chown opendkim:opendkim /etc/opendkim/keys/tyfsadik.org/mail.private

# View DKIM record for DNS
sudo cat /etc/opendkim/keys/tyfsadik.org/mail.txt
```

## 🔒 Security Configuration

### Firewall Setup
```bash
# Configure UFW firewall
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 25/tcp    # SMTP
sudo ufw allow 587/tcp   # Submission
sudo ufw allow 465/tcp   # SMTPS
sudo ufw allow 993/tcp   # IMAPS
sudo ufw allow 995/tcp   # POP3S
sudo ufw enable
```

### Fail2ban Protection
```bash
sudo nano /etc/fail2ban/jail.local
```

Add these configurations:
```ini
[postfix]
enabled = true
port = smtp,465,submission
filter = postfix
logpath = /var/log/mail.log
maxretry = 3
bantime = 3600

[dovecot]
enabled = true
port = pop3,pop3s,imap,imaps
filter = dovecot
logpath = /var/log/mail.log
maxretry = 3
bantime = 3600
```

## 🧪 Testing Your Setup

### Test SMTP Service
```bash
# Test SMTP connectivity
telnet mail.tyfsadik.org 25

# Send test email
echo "Test message from SMTP server" | mail -s "Server Test" your-email@gmail.com
```

### Verify DNS Records
```bash
# Check MX record
dig MX tyfsadik.org +short

# Check SPF record
dig TXT tyfsadik.org +short

# Check if ports are open
nmap -p 25,587,465,993,995 mail.tyfsadik.org
```

### Service Management
```bash
# Restart all services
sudo systemctl restart postfix dovecot opendkim fail2ban

# Enable on boot
sudo systemctl enable postfix dovecot opendkim fail2ban

# Check status
sudo systemctl status postfix dovecot
```

## 📊 Monitoring & Maintenance

### Check Mail Logs
```bash
# Real-time log monitoring
sudo tail -f /var/log/mail.log

# Check for errors
sudo grep -i error /var/log/mail.log

# Monitor mail queue
mailq
```

### Backup Script
```bash
#!/bin/bash
# scripts/backup-mail.sh

BACKUP_DIR="/backup/mail-$(date +%Y%m%d)"
mkdir -p $BACKUP_DIR

# Backup configurations
cp -r /etc/postfix $BACKUP_DIR/
cp -r /etc/dovecot $BACKUP_DIR/
cp -r /etc/opendkim $BACKUP_DIR/

# Backup mail directories
tar -czf $BACKUP_DIR/maildir.tar.gz /var/mail/

echo "Backup completed: $BACKUP_DIR"
```

## 🐛 Troubleshooting Common Issues

### Emails Not Sending
```bash
# Check Postfix configuration
sudo postfix check
sudo postconf -n

# Check if Postfix is running
sudo systemctl status postfix

# Check mail queue
sudo mailq
```

### Authentication Problems
```bash
# Check Dovecot status
sudo systemctl status dovecot

# Test authentication
telnet localhost 993

# Check authentication logs
sudo grep "authentication" /var/log/mail.log
```

### DNS Issues
```bash
# Verify all DNS records
dig MX tyfsadik.org
dig A mail.tyfsadik.org
dig TXT tyfsadik.org
dig TXT _dmarc.tyfsadik.org
```

## 📞 Support

- **Email**: [taki@tyfsadik.org](mailto:taki@tyfsadik.org)
- **Domain**: [tyfsadik.org](https://tyfsadik.org)

## 📄 License

MIT License - feel free to use this configuration for your own projects.

---

*Last updated: March 2024 | Version: 2.0 | Status: Production Ready*

**⭐ If this project helped you, please consider giving it a star on GitHub!**
