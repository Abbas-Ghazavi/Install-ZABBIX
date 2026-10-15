# Install Zabbix 7.0 on AlmaLinux 10

**Environment:** AlmaLinux 10 · Zabbix 7.0 · MySQL 8.4 · Apache · PHP-FPM

## 1. Install MySQL 8.4

```bash
dnf install -y https://dev.mysql.com/get/mysql84-community-release-el10-2.noarch.rpm
dnf install -y mysql-community-server
systemctl enable --now mysqld
mysql_secure_installation
```

## 2. Install Zabbix

```bash
rpm -Uvh https://repo.zabbix.com/zabbix/7.0/alma/10/x86_64/zabbix-release-latest-7.0.el10.noarch.rpm
dnf clean all

dnf install -y zabbix-server-mysql zabbix-web-mysql zabbix-apache-conf zabbix-sql-scripts zabbix-selinux-policy zabbix-agent
```

## 3. Create Database

```sql
CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'YOUR_PASSWORD';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';
EXIT;
```

## 4. Import Database

```bash
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
```

## 5. Configure Zabbix Server

```bash
nano /etc/zabbix/zabbix_server.conf
```

Set the database parameters:

```ini
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=YOUR_PASSWORD
```

## 6. Start Services

```bash
systemctl enable --now zabbix-server zabbix-agent httpd php-fpm
```

Check status:

```bash
systemctl is-active mysqld zabbix-server zabbix-agent httpd php-fpm
```

## 7. Access Web Interface

```text
http://SERVER-IP/zabbix
```

Complete the installation wizard.

## Troubleshooting

```bash
journalctl -u zabbix-server -n 50 --no-pager
tail -n 50 /var/log/zabbix/zabbix_server.log
```

**Note:** Replace `YOUR_PASSWORD` with a strong password. Configure firewall and SELinux rules as required.

**Documentation:** [Zabbix 7.0](https://www.zabbix.com/documentation/7.0/en/manual)
