# Install-ZABBIX
## Zabbix 7.0 Installation on AlmaLinux 10

نصب Zabbix Server با MySQL 8.4 روی AlmaLinux 10

## Environment

* OS: AlmaLinux 10
* Zabbix: 7.0
* Database: MySQL 8.4
* Web Server: Apache
* PHP: PHP-FPM

## 1. Check System

```bash
cat /etc/os-release
ip -br a
hostname -I
```

## 2. Install MySQL 8.4

```bash
dnf install -y https://dev.mysql.com/get/mysql84-community-release-el10-2.noarch.rpm

dnf clean all

dnf install -y mysql-community-server mysql-community-client

systemctl enable --now mysqld

systemctl status mysqld
```

## 3. Secure MySQL

```bash
grep 'temporary password' /var/log/mysqld.log

mysql_secure_installation
```

Login to MySQL:

```bash
mysql -uroot -p
```

## 4. Add Zabbix Repository

```bash
rpm -Uvh https://repo.zabbix.com/zabbix/7.0/alma/10/x86_64/zabbix-release-latest-7.0.el10.noarch.rpm

dnf clean all
```

## 5. Install Zabbix Packages

```bash
dnf install -y zabbix-server-mysql zabbix-web-mysql zabbix-apache-conf zabbix-sql-scripts zabbix-selinux-policy zabbix-agent
```

## 6. Create Database and User

Login to MySQL:

```bash
mysql -uroot -p
```

Execute SQL commands:

```sql
CREATE DATABASE zabbix
CHARACTER SET utf8mb4
COLLATE utf8mb4_bin;

CREATE USER 'zabbix'@'localhost'
IDENTIFIED BY 'CHANGE_ME_TO_A_STRONG_PASSWORD';

GRANT ALL PRIVILEGES ON zabbix.*
TO 'zabbix'@'localhost';

EXIT;
```

## 7. Import Database Schema

```bash
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
```

## 8. Configure Zabbix Server

Edit configuration:

```bash
nano /etc/zabbix/zabbix_server.conf
```

Set database parameters:

```ini
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=CHANGE_ME_TO_A_STRONG_PASSWORD
```

Save and exit.

## 9. Enable and Start Services

```bash
systemctl enable --now zabbix-server zabbix-agent httpd php-fpm
```

## 10. Check Services

```bash
systemctl is-active mysqld
systemctl is-active zabbix-server
systemctl is-active zabbix-agent
systemctl is-active httpd
systemctl is-active php-fpm
```

## 11. Troubleshooting

Check Zabbix Server logs:

```bash
journalctl -u zabbix-server -n 50 --no-pager
```

```bash
tail -n 50 /var/log/zabbix/zabbix_server.log
```

Check packages and dependencies:

```bash
rpm -qa | grep -i zabbix
dnf check
```

Check database tables:

```bash
mysql -uzabbix -p -e "SHOW TABLES;" zabbix
```

## 12. Access Web Interface

```text
http://SERVER-IP/zabbix
```

Example:

```text
http://192.168.40.131/zabbix
```

Complete the web installation wizard.

## Important Notes

* Replace `CHANGE_ME_TO_A_STRONG_PASSWORD` with your own strong password.
* Do not use `rpm --nodeps` during a normal installation.
* Do not remove MariaDB or MySQL packages without checking dependencies.
* Do not import the database schema repeatedly into an existing database.
* Verify repository compatibility before installing packages.
* Configure SELinux and firewall rules according to your environment.

## Official Documentation

* [Zabbix 7.0 Documentation](https://www.zabbix.com/documentation/7.0/en/manual)
* [Zabbix Installation from Packages](https://www.zabbix.com/documentation/7.0/en/manual/installation/install_from_packages)
* [MySQL 8.4 Installation Guide](https://dev.mysql.com/doc/refman/8.4/en/linux-installation-yum-repo.html)
