# Install Zabbix Server on Alpine Linux

راهنمای نصب Zabbix Server روی Alpine Linux با استفاده از Lighttpd، PHP و MariaDB.

## 1. Configure Alpine Mirrors

Edit repository list:

```bash
vi /etc/apk/repositories
```

```text
https://linux-mirror.liara.ir/repository/alpine/v3.21/main
https://linux-mirror.liara.ir/repository/alpine/v3.21/community
```

Update packages and install basic tools:

```bash
apk update
apk add nano bash iproute2 tmux wget
```

## 2. Install Web Server and PHP

```bash
apk add lighttpd php82 fcgi php82-cgi php82-mysqli php82-mbstring
```

Edit Lighttpd configuration:

```bash
vi /etc/lighttpd/lighttpd.conf
```

Uncomment:

```text
include "mod_fastcgi.conf"
```

Edit FastCGI configuration:

```bash
nano /etc/lighttpd/mod_fastcgi.conf
```

Set the PHP-CGI binary to:

```text
"/usr/bin/php-cgi82"
```

Start and enable Lighttpd:

```bash
rc-service lighttpd start
rc-update add lighttpd default
```

## 3. Install and Configure MariaDB

```bash
apk add mysql mysql-client
mysql_install_db --user=mysql --datadir=/var/lib/mysql
rc-service mariadb start
rc-update add mariadb default
mysql_secure_installation
```

Create the Zabbix database and user:

```sql
CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'YOUR_PASSWORD';
GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'localhost';
SET GLOBAL log_bin_trust_function_creators = 1;
FLUSH PRIVILEGES;
EXIT;
```

## 4. Install Zabbix

```bash
apk add zabbix zabbix-mysql zabbix-webif zabbix-setup zabbix-utils
```

Import the database schema and initial data:

```bash
cd /usr/share/zabbix/database/

cat schema.sql | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
cat images.sql | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
cat data.sql | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
```

## 5. Configure Zabbix Server

Edit the configuration:

```bash
nano /etc/zabbix/zabbix_server.conf
```

Uncomment and set the database password:

```ini
DBPassword=YOUR_PASSWORD
```

Enable Zabbix Server at startup:

```bash
rc-update add zabbix-server default
rc-service zabbix-server start
```

## 6. Configure the Web Interface

Set the web root:

```bash
rm -rf /var/www/localhost/htdocs
ln -s /usr/share/webapps/zabbix /var/www/localhost/htdocs
chown -R lighttpd /usr/share/webapps/zabbix/conf
```

Edit PHP settings:

```bash
vi /etc/php82/php.ini
```

Adjust the following parameters as needed:

```ini
post_max_size = 16M
max_execution_time = 300
```

Restart Lighttpd:

```bash
rc-service lighttpd restart
```

## 7. Install and Enable Zabbix Agent

```bash
apk add zabbix-agent
rc-update add zabbix-agentd default
rc-service zabbix-agentd start
```

Restart the web server:

```bash
rc-service lighttpd restart
```

## 8. Access Zabbix

Open the server IP address in your browser:

```text
http://SERVER_IP/
```

**Note:** Replace `YOUR_PASSWORD` with your actual database password. Verify package versions and configuration paths against your installed Alpine and Zabbix versions.
**Start install Zabbix in your ip - login [Admin][zabbix]
