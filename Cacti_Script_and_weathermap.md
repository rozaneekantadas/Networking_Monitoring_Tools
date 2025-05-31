# 🌐 Cacti Network Monitoring Tool Installation on Ubuntu 24.04 / 22.04

Cacti is a complete network graphing solution designed to harness the power of RRDTool's data storage and graphing functionality. This guide will walk you through installing Cacti and its popular **WeatherMap plugin** on Ubuntu 24.04 / 22.04.

---

## ✅ Prerequisites

* Ubuntu 24.04 / 22.04
* A non-root user with `sudo` privileges
* LAMP Stack (Linux, Apache, MySQL/MariaDB, PHP)

---

## 🛠️ Installation Steps

### Step 1: System Update and Timezone Configuration

```bash
sudo apt update && sudo apt upgrade -y
timedatectl
sudo timedatectl set-timezone Asia/Dhaka
```

---

### Step 2: Install Required Packages

```bash
sudo apt install fping snmp snmpd rrdtool unzip -y
sudo apt install apache2 -y
sudo systemctl enable --now apache2
```

### Step 3: Install PHP and Dependencies

```bash
sudo apt install php php-{mysql,curl,net-socket,gd,intl,pear,imap,memcache,pspell,tidy,xmlrpc,snmp,mbstring,gmp,json,xml,common,ldap} libapache2-mod-php -y
```

#### Configure PHP

Edit both files:

```bash
sudo nano /etc/php/*/apache2/php.ini
sudo nano /etc/php/*/cli/php.ini
```

Set the following values:

```
date.timezone = Asia/Dhaka
memory_limit = 512M
max_execution_time = 60
```

---

### Step 4: Install and Configure MariaDB

```bash
sudo apt install mariadb-server -y
sudo systemctl enable --now mariadb
sudo mysql_secure_installation
```

#### Configure Cacti Database

```bash
sudo mysql -u root -p
```

Inside MariaDB shell:

```sql
CREATE DATABASE cacti DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
GRANT ALL PRIVILEGES ON cacti.* TO 'cacti'@'localhost' IDENTIFIED BY 'cacti';
GRANT SELECT ON mysql.time_zone_name TO 'cacti'@'localhost';
ALTER DATABASE cacti CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
FLUSH PRIVILEGES;
EXIT;
```

#### Update MariaDB Configuration

```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

Add or modify:

```ini
innodb_file_format=Barracuda
innodb_large_prefix=1
collation-server=utf8mb4_unicode_ci
character-set-server=utf8mb4
innodb_doublewrite=OFF
max_heap_table_size=128M
tmp_table_size=128M
join_buffer_size=128M
innodb_buffer_pool_size=1G
innodb_flush_log_at_timeout=3
innodb_read_io_threads=32
innodb_write_io_threads=16
innodb_io_capacity=5000
innodb_io_capacity_max=10000
innodb_buffer_pool_instances=9
```

#### Load Timezone Info

```bash
sudo mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -u root mysql
```

---

### Step 5: Download and Install Cacti

```bash
wget https://www.cacti.net/downloads/cacti-latest.tar.gz
tar -zxvf cacti-latest.tar.gz
sudo mv cacti-* /var/www/html/cacti
sudo mysql -u root cacti < /var/www/html/cacti/cacti.sql
```

---

### Step 6: Configure Cacti

```bash
cd /var/www/html/cacti/include
cp config.php.dist config.php
sudo nano config.php
```

Set database credentials:

```php
$database_type = "mysql";
$database_default = "cacti";
$database_hostname = "localhost";
$database_username = "cacti";
$database_password = "cacti";
$database_port = "3306";
$database_ssl = false;
```

Set permissions:

```bash
sudo chown -R www-data:www-data /var/www/html/cacti
```

---

### Step 7: Create Systemd Service for Cactid

```bash
sudo nano /etc/systemd/system/cactid.service
```

Paste:

```ini
[Unit]
Description=Cacti Daemon Main Poller Service
After=network.target

[Service]
Type=forking
User=www-data
Group=www-data
EnvironmentFile=/etc/default/cactid
ExecStart=/var/www/html/cacti/cactid.php
Restart=always
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

```bash
sudo touch /etc/default/cactid
sudo systemctl daemon-reload
sudo systemctl enable --now cactid
```

Restart services:

```bash
sudo systemctl restart apache2 mariadb
```

Visit: `http://<your-server-IP>/cacti/` to complete the web-based setup.

---

## 🌦️ Install WeatherMap Plugin

```bash
wget https://github.com/Cacti/plugin_weathermap/archive/refs/tags/v1.3.1.zip
sudo apt install unzip -y
unzip v1.3.1.zip
mv plugin_weathermap-1.3.1 weathermap
sudo mv weathermap /var/www/html/cacti/plugins/
sudo chown -R www-data:www-data /var/www/html/cacti/plugins/weathermap
sudo systemctl restart cactid
```

### Activate via Web UI:

1. Log in to Cacti Web UI
2. Go to **Console > Plugin Management**
3. Install and enable **weathermap**
4. Refresh the page to see **WeatherMap** in the navigation

---

## 🎉 You're Done!

You now have a fully functional Cacti server with the WeatherMap plugin on Ubuntu 24.04 / 22.04.

---

## 🔗 Useful Links

* [Cacti Website](https://www.cacti.net/)
* [WeatherMap Plugin](https://github.com/Cacti/plugin_weathermap)
