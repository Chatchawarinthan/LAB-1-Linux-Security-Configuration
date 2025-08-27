# 🛡️ การตั้งค่าความปลอดภัยบน Linux (Lab Assignment)

## 📌 ภาพรวม
Lab Assignment นี้เป็นการบ้านเชิงปฏิบัติการเกี่ยวกับ **การรักษาความปลอดภัยบน Linux** ครอบคลุม:

- การจัดการผู้ใช้และกลุ่ม  
- การกำหนดสิทธิ์ sudo  
- การรักษาความปลอดภัย SSH  
- การตั้งค่า Firewall  
- การตรวจสอบระบบและการจัดการ log  

---

## 🔑 งานที่ 1: การจัดการ User & Group

### 1.1 สร้างกลุ่ม
sudo groupadd developers
sudo groupadd testers
sudo groupadd dbadmin

### 1.2 สร้างผู้ใช้
sudo useradd -m -G developers chonl
sudo useradd -m -G developers nunt
sudo useradd -m -G testers tuser
sudo useradd -m -G dbadmin dbuser

### 1.3 ตั้งค่านโยบายรหัสผ่าน

อายุรหัสผ่านสูงสุด: 90 วัน

อายุขั้นต่ำก่อนเปลี่ยนใหม่: 7 วัน

แจ้งเตือนก่อนหมดอายุ: 14 วัน

ความยาวขั้นต่ำ: 12 ตัวอักษร (ต้องมีตัวใหญ่ ตัวเล็ก ตัวเลข และสัญลักษณ์)

sudo apt install libpam-pwquality
sudo nano /etc/security/pwquality.conf

ตั้งค่า
minlen = 12
minclass = 4

sudo chage -M 90 -m 7 -W 14 chonl
sudo chage -M 90 -m 7 -W 14 nunt
sudo chage -M 90 -m 7 -W 14 tuser
sudo chage -M 90 -m 7 -W 14 dbuser

## ⚡ งานที่ 2: การกำหนดสิทธิ์ Sudo
### 2.1 สร้างกลุ่ม sudo
sudo groupadd sudo-developers
sudo groupadd sudo-limited

### 2.2 กำหนดผู้ใช้ให้กลุ่ม
sudo usermod -aG sudo-developers chonl
sudo usermod -aG sudo-developers nunt
sudo usermod -aG sudo-limited tuser
sudo usermod -aG dbadmin dbuser

### 2.3 ตั้งค่า sudoers
sudo visudo


เพิ่มบรรทัด:

%sudo-developers ALL=(ALL) ALL
%sudo-limited ALL=(ALL) NOPASSWD: /bin/systemctl status, /usr/bin/tail, /bin/ps
%dbadmin ALL=(ALL) NOPASSWD: /usr/bin/mysql
Defaults timestamp_timeout=15
Defaults logfile="/var/log/sudo.log"

## 🔒 งานที่ 3: ความปลอดภัย SSH
### 3.1 ปรับแต่ง /etc/ssh/sshd_config
Port 2222
PermitRootLogin no
AllowUsers chonl nunt tuser dbuser
PasswordAuthentication no
MaxAuthTries 3
LoginGraceTime 1m
Banner /etc/ssh/ssh_banner

### 3.2 ตั้งค่า SSH Banner
sudo nano /etc/ssh/ssh_banner
# ใส่ข้อความแจ้งเตือนก่อน login
Authorized access only. Violators will be prosecuted.

### 3.3 Restart SSH
sudo systemctl restart ssh

## 🌐 งานที่ 4: การตั้งค่า Firewall (UFW)
### 4.1 ตั้งค่าเริ่มต้น
sudo ufw default deny incoming
sudo ufw default allow outgoing

### 4.2 อนุญาตบริการ
sudo ufw allow 2222/tcp       # SSH
sudo ufw allow 80/tcp         # HTTP
sudo ufw allow 443/tcp        # HTTPS
sudo ufw allow from 192.168.1.0/24 to any port 3306 # MySQL เฉพาะ LAN
sudo ufw limit 2222/tcp       # rate limiting SSH
sudo ufw enable
sudo ufw status verbose

## 📊 งานที่ 5: การตรวจสอบระบบ
### 5.1 ติดตั้งเครื่องมือ
sudo apt install fail2ban logwatch sysstat htop iotop
# ELK stack optional

### 5.2 ตั้งค่า Fail2Ban
sudo systemctl enable fail2ban
sudo systemctl start fail2ban

### 5.3 สร้างสคริปต์ system_monitor.sh
sudo nano /usr/local/bin/system_monitor.sh

#!/bin/bash
LOGFILE=/var/log/system_monitor.log
echo "===== $(date) =====" >> $LOGFILE
echo "CPU:" >> $LOGFILE
top -bn1 | grep "Cpu(s)" >> $LOGFILE
echo "Memory:" >> $LOGFILE
free -h >> $LOGFILE
echo "Disk:" >> $LOGFILE
df -h >> $LOGFILE
echo "Active users:" >> $LOGFILE
who >> $LOGFILE
echo "Failed logins:" >> $LOGFILE
lastb | head -n 10 >> $LOGFILE
echo "" >> $LOGFILE

### 5.4 ตั้งค่า Cron Job
sudo crontab -e
เพิ่มบรรทัด
0 * * * * /usr/local/bin/system_monitor.sh

### 5.5 ตั้งค่า Logrotate
sudo nano /etc/logrotate.d/system_monitor

/var/log/system_monitor.log {
    daily
    missingok
    rotate 30
    compress
    delaycompress
    notifempty
    copytruncate
}

##  ✅ สรุป

Lab นี้ครอบคลุมการทำ Linux Security Hardening ครบทุกขั้นตอน ได้แก่:

การจัดการผู้ใช้และนโยบายรหัสผ่าน

การกำหนดสิทธิ์และจัดการ sudo

การรักษาความปลอดภัย SSH

การตั้งค่า Firewall

การตรวจสอบระบบและจัดการ log

📂 Link Linux Security Server Configuration Download
