# Zabbix 7.0 Installation on Ubuntu Server 24.04.2 LTS

This guide walks you through installing **Zabbix 7.0** on **Ubuntu Server 24.04.2 LTS**.
It includes setting up Apache, PHP, MySQL, and Zabbix server and frontend.

## Prerequisites

* Ubuntu Server 24.04.2 LTS installed
* SSH access enabled

---

## 📦 Step-by-Step Installation

### 1. SSH Into Your Server

```bash
ssh your-user@your-server-ip
```

### 2. Install Apache and PHP

```bash
sudo apt update
sudo apt install apache2
sudo apt install php php-{cgi,common,mbstring,net-socket,gd,xml-util,mysql,bcmath,imap,snmp}
sudo apt install libapache2-mod-php
```

### 3. Add Zabbix Repository

```bash
sudo -s
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest+ubuntu24.04_all.deb
dpkg -i zabbix-release_latest+ubuntu24.04_all.deb
apt update
```

### 4. Install Zabbix Server, Frontend, and Agent

```bash
apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent
```

### 5. Install MySQL Server

```bash
apt-get install mysql-server
systemctl start mysql
```

### 6. Create Zabbix Database

```bash
mysql
```

Inside MySQL shell:

```sql
create database zabbix character set utf8mb4 collate utf8mb4_bin;
create user zabbix@localhost identified by 'zabbix';
grant all privileges on zabbix.* to zabbix@localhost;
set global log_bin_trust_function_creators = 1;
quit;
```

### 7. Import Zabbix Schema

```bash
sudo zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
```

> You'll be prompted to enter the password for the `zabbix` MySQL user.

### 8. Disable `log_bin_trust_function_creators`

```bash
sudo mysql
```

Inside MySQL shell:

```sql
set global log_bin_trust_function_creators = 0;
quit;
```

### 9. Configure Zabbix Server

Edit the config file:

```bash
sudo nano /etc/zabbix/zabbix_server.conf
```

Set the DB password line:

```ini
DBPassword=zabbix
```

### 10. Start Zabbix Services

```bash
systemctl restart zabbix-server zabbix-agent apache2
systemctl enable zabbix-server zabbix-agent apache2
```

---

## 🌐 Access Zabbix Web UI

Open your browser and visit:

```
http://your-server-ip/zabbix
```

Proceed with the web-based configuration.
**Default MySQL Port:** `3306`

---

## 🔗 References

* [Original Installation Guide](https://reasonableitservice.com/install-zabbix-6-4-on-ubuntu-server-22-04-1-in-under-10-minutes/)
* [Official Zabbix Download](https://www.zabbix.com/download?os_distribution=ubuntu)

---

## ✅ Notes

* This guide uses the Zabbix 7.0 repo (latest at time of writing).
* Always ensure system packages are up-to-date.

---
