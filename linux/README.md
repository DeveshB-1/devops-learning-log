# Linux & RHEL Administration Notes

## RPM Package Management

### YUM/DNF Commands

```bash
# Install / Remove
dnf install httpd -y
dnf remove httpd
dnf groupinstall "Development Tools"

# Query
rpm -qa | grep kernel          # list installed packages
rpm -qi httpd                  # package info
rpm -ql httpd                  # list files in package
rpm -qf /etc/httpd/conf/httpd.conf  # which package owns this file

# Update
dnf update
dnf update httpd               # update specific package
dnf history                    # transaction history
dnf history undo 42            # rollback transaction

# Repos
dnf repolist
dnf config-manager --add-repo https://example.com/repo.repo
dnf config-manager --disable epel
```

### Building RPM Packages

```bash
# Spec file structure:
# Name, Version, Release, Summary, License, Source, BuildRequires
# %prep, %build, %install, %files, %changelog

# Build
rpmbuild -ba package.spec             # build src + binary RPM
rpmbuild --bb package.spec            # binary only

# Sign
rpm --addsign *.rpm

# Verify
rpm -K package.rpm                    # check signature
rpm --verify package-name
```

## systemd Service Management

```bash
# Control
systemctl start/stop/restart/reload nginx
systemctl enable/disable nginx         # start on boot
systemctl status nginx
systemctl list-units --type=service --state=running

# Logs
journalctl -u nginx                    # service logs
journalctl -u nginx -f                 # follow
journalctl -u nginx --since "1 hour ago"
journalctl -p err -b                   # errors since last boot

# Create a service
cat > /etc/systemd/system/myapp.service << EOF
[Unit]
Description=My Application
After=network.target

[Service]
Type=simple
User=appuser
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/python3 app.py
Restart=on-failure
RestartSec=5
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable --now myapp
```

## Kickstart Automation

```kickstart
# Basic RHEL Kickstart file
install
text
lang en_US.UTF-8
keyboard us
timezone Asia/Kolkata --utc
rootpw --iscrypted $6$...hashhere...
network --bootproto=dhcp --onboot=yes
bootloader --location=mbr
clearpart --all --initlabel
part /boot --fstype=ext4 --size=512
part / --fstype=ext4 --grow

%packages
@base
@core
vim
git
python3
curl
wget
%end

%post --log=/root/ks-post.log
# Post-install script
dnf update -y
systemctl enable sshd
useradd -m devops -G wheel
echo "devops ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/devops
%end
```

## System Health Monitoring

```bash
#!/bin/bash
# Quick health check script

echo "=== System Health $(date) ==="

# CPU usage
CPU=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
echo "CPU: ${CPU}%"
[[ $(echo "$CPU > 85" | bc) -eq 1 ]] && echo "ALERT: High CPU!"

# Memory
MEM=$(free | awk '/Mem:/ {printf "%.1f", $3/$2*100}')
echo "Memory: ${MEM}%"

# Disk
DISK=$(df / | tail -1 | awk '{print $5}' | tr -d '%')
echo "Disk /: ${DISK}%"
[[ $DISK -gt 85 ]] && echo "ALERT: Disk almost full!"

# Key services
for svc in sshd crond; do
    systemctl is-active --quiet $svc && echo "$svc: OK" || echo "ALERT: $svc DOWN!"
done
```

## Networking Essentials

```bash
# Interfaces
ip a                           # show interfaces + IPs
ip route show                  # routing table
ss -tulpn                      # listening ports (modern netstat)
firewall-cmd --list-all        # firewalld rules

# Firewalld
firewall-cmd --add-service=http --permanent
firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --reload

# SELinux
getenforce                     # Enforcing / Permissive / Disabled
setenforce 0                   # temporary permissive mode
sestatus
ausearch -m avc -ts recent     # SELinux denials
```
