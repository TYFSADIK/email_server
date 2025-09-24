📧 Self-Hosted SMTP Email Server – tyfsadik.org

A complete, production-ready self-hosted SMTP email server implementation for the domain tyfsadik.org. This setup ensures secure, reliable email delivery with enterprise-grade authentication protocols and security measures.

🚀 Quick Start
Prerequisites
Ubuntu 22.04 LTS server

Domain name (tyfsadik.org)

Static IP address

Root access to the server

Automated Installation
bash
# Clone the repository
git clone https://github.com/yourusername/smtp-server-tyfsadik.git
cd smtp-server-tyfsadik

# Run the setup script (review before executing)
chmod +x scripts/setup.sh
sudo ./scripts/setup.sh --domain tyfsadik.org --hostname mail
📋 Table of Contents
Features

Architecture

Installation

Configuration

DNS Setup

Security

Testing

Monitoring

Troubleshooting

Contributing

License

✨ Features
Feature	Implementation	Status
SMTP Service	Postfix with TLS/SSL	✅ Production
IMAP/POP3	Dovecot with Maildir	✅ Production
Email Authentication	SPF, DKIM, DMARC	✅ Enabled
Security	Fail2ban, UFW, SSL Certs	✅ Active
Spam Protection	Basic RBL checks	✅ Configured
Webmail	Roundcube (Optional)	🔄 Planned
Backup System	Automated scripts	🔄 Planned
🏗️ Architecture














📥 Installation
System Requirements
OS: Ubuntu 22.04 LTS / Debian 11+

RAM: 2GB minimum (4GB recommended)

Storage: 20GB+ (depending on mail volume)

CPU: 2 cores minimum

Step-by-Step Setup
1. System Preparation
bash
# Update system and install dependencies
sudo apt update && sudo apt upgrade -y
sudo apt install -y postfix dovecot-core dovecot-imapd dovecot-pop3d \
    openssl opendkim opendkim-tools ufw fail2ban mailutils
2. Postfix Configuration
Edit /etc/postfix/main.cf:

ini
# Basic Settings
myhostname = mail.tyfsadik.org
mydomain = tyfsadik.org
myorigin = $mydomain
inet_interfaces = all
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain

# Mailbox Settings
home_mailbox = Maildir/
mailbox_command = 

# TLS Settings
smtpd_use_tls = yes
smtpd_tls_cert_file = /etc/ssl/certs/mailserver.crt
smtpd_tls_key_file = /etc/ssl/private/mailserver.key
smtpd_tls_session_cache_database = btree:${data_directory}/smtpd_scache

# Security Restrictions
smtpd_helo_restrictions = permit_mynetworks, reject_invalid_helo_hostname
smtpd_sender_restrictions = permit_mynetworks, reject_unknown_sender_domain
3. SSL Certificate Generation
bash
# Create SSL directory
sudo mkdir -p /etc/ssl/private
sudo mkdir -p /etc/ssl/certs

# Generate self-signed certificate (replace with Let's Encrypt for production)
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/mailserver.key \
    -out /etc/ssl/certs/mailserver.crt \
    -subj "/C=US/ST=State/L=City/O=Organization/CN=mail.tyfsadik.org"
🌐 DNS Configuration
Essential Records
Type	Name	Value	TTL
A	mail.tyfsadik.org	YOUR_SERVER_IP	3600
MX	@	mail.tyfsadik.org (Priority 10)	3600
TXT	@	v=spf1 mx ~all	3600
TXT	_dmarc	v=DMARC1; p=none; rua=mailto:admin@tyfsadik.org	3600
DKIM Setup
bash
# Generate DKIM keys
sudo mkdir -p /etc/opendkim/keys/tyfsadik.org
sudo opendkim-genkey -D /etc/opendkim/keys/tyfsadik.org/ -d tyfsadik.org -s mail
sudo chown opendkim:opendkim /etc/opendkim/keys/tyfsadik.org/mail.private

# Add DKIM DNS record (extract from .txt file)
sudo cat /etc/opendkim/keys/tyfsadik.org/mail.txt
🔒 Security Hardening
Firewall Configuration
bash
# Configure UFW
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Open essential ports
sudo ufw allow ssh
sudo ufw allow 25/tcp    # SMTP
sudo ufw allow 587/tcp   # Submission
sudo ufw allow 465/tcp   # SMTPS
sudo ufw allow 993/tcp   # IMAPS
sudo ufw allow 995/tcp   # POP3S

sudo ufw enable
Fail2ban Protection
Create /etc/fail2ban/jail.local:

ini
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
System Hardening Script
bash
#!/bin/bash
# scripts/harden-server.sh

# Disable root SSH login
sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config

# Set up automatic security updates
apt install -y unattended-upgrades
dpkg-reconfigure -plow unattended-upgrades

# Configure system limits for mail processes
echo "postfix soft nofile 4096" >> /etc/security/limits.conf
echo "postfix hard nofile 8192" >> /etc/security/limits.conf
🧪 Testing & Verification
SMTP Server Test
bash
# Test SMTP connectivity
telnet mail.tyfsadik.org 25
EHLO tyfsadik.org
QUIT

# Send test email
echo "Test message from SMTP server" | mail -s "Server Test" your-email@gmail.com
DNS Record Verification
bash
# Verify MX records
dig MX tyfsadik.org +short

# Verify SPF record
dig TXT tyfsadik.org +short

# Verify DKIM record
dig TXT mail._domainkey.tyfsadik.org +short
Email Authentication Test
Use online tools to validate your setup:

MXToolbox SuperTool

DKIM Validator

Mail-Tester.com

📊 Monitoring & Maintenance
Log Monitoring
bash
# Monitor mail logs in real-time
tail -f /var/log/mail.log

# Check for failed authentication attempts
grep "authentication failure" /var/log/mail.log

# Monitor queue size
mailq
Performance Monitoring Script
bash
#!/bin/bash
# scripts/monitor-mail.sh

echo "=== Mail Queue Size ==="
mailq | grep -c "^[A-F0-9]"

echo "=== Active Connections ==="
netstat -an | grep :25 | wc -l

echo "=== Disk Usage ==="
du -sh /var/mail/

echo "=== Recent Errors ==="
tail -20 /var/log/mail.log | grep -i error
Backup Configuration
bash
#!/bin/bash
# scripts/backup-mail.sh

BACKUP_DIR="/backup/mail-$(date +%Y%m%d)"
mkdir -p $BACKUP_DIR

# Backup configurations
cp -r /etc/postfix $BACKUP_DIR/
cp -r /etc/dovecot $BACKUP_DIR/
cp -r /etc/opendkim $BACKUP_DIR/

# Backup mail directories (if any)
tar -czf $BACKUP_DIR/maildir.tar.gz /var/mail/

echo "Backup completed: $BACKUP_DIR"
🐛 Troubleshooting
Common Issues & Solutions
Problem	Symptoms	Solution
Emails not sending	Connection timeout on port 25	Check firewall and ISP port blocking
Authentication failures	"Login incorrect" errors	Verify user accounts and password policies
DNS issues	Emails marked as spam	Validate SPF/DKIM/DMARC records
Certificate errors	TLS handshake failures	Renew or properly configure SSL certificates
Diagnostic Commands
bash
# Check service status
sudo systemctl status postfix dovecot opendkim

# Test mail delivery locally
sendmail user@tyfsadik.org < /dev/null

# Verify configuration syntax
postfix check
dovecot -n

# Check for port listening
netstat -tlnp | grep -E ':25|:587|:993|:995'
🤝 Contributing
We welcome contributions to improve this SMTP server setup! Please see our Contributing Guidelines for details.

Development Setup
bash
# Fork and clone the repository
git clone https://github.com/yourusername/smtp-server-tyfsadik.git

# Create a feature branch
git checkout -b feature/improvement-name

# Test changes in a virtual environment
vagrant up  # Vagrantfile provided for testing
📄 License
This project is licensed under the MIT License - see the LICENSE file for details.

📞 Contact & Support
Maintainer: Taki Uddin Mohammad

Email: taki@tyfsadik.org

Domain: tyfsadik.org

Issues: GitHub Issues

🙏 Acknowledgments
Postfix and Dovecot development teams

OpenDKIM maintainers

Ubuntu community for excellent documentation

⭐ If this project helped you, please consider giving it a star on GitHub!
