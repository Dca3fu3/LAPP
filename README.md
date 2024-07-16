![image](https://github.com/user-attachments/assets/425a3f53-ff6d-4fd7-a818-16c2a8fcaf56)![image](https://github.com/user-attachments/assets/d4b0c880-80f9-44c3-8d50-cf21d57c7ce6)在Ubuntu 24.04上安装LAPP（Linux, Apache, PostgreSQL, PHP）环境的步骤如下：

### 1. 更新系统

首先，确保系统包是最新的：

```sh
apt update && apt upgrade -y
```

### 2. 安装 Apache

安装 Apache 服务器：

```sh
apt install apache2 -y
```

启动 Apache 并设置为开机启动：

```sh
systemctl start apache2
systemctl enable apache2
```

### 3. 安装 PostgreSQL

安装 PostgreSQL 数据库：

```sh
apt install postgresql postgresql-contrib -y
```

启动 PostgreSQL 并设置为开机启动：

```sh
systemctl start postgresql
systemctl enable postgresql
```

### 4. 配置 PostgreSQL

切换到 PostgreSQL 用户：

```sh
su - postgres
```

进入 PostgreSQL shell：

```sh
psql
```

设置 PostgreSQL 用户密码（例如：设置 `postgres` 用户的密码）：

```sql
ALTER USER postgres PASSWORD 'newpassword';
```
退出 PostgreSQL shell：
```sql
\q
```

退出 PostgreSQL 用户：

```sh
exit
```

### 5. 安装 PHP

安装 PHP 及其相关模块：

```sh
apt install php libapache2-mod-php php-pgsql php-mbstring php-json php-xml php-zip php-curl php-intl php-apcu php-imagick php-gd -y
```

### 6. 配置 Apache 使用 PHP

编辑 Apache 默认站点配置文件：

```sh
nano /etc/apache2/sites-available/000-default.conf
```

在 `<VirtualHost *:80>` 内添加 `DirectoryIndex` 设置：

```xml
<Directory /var/www/html>
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>

DirectoryIndex index.php index.html
```

保存并退出编辑器，然后重新启动 Apache：

```sh
systemctl restart apache2
```

### 7. 配置HTTP/2
配置：
```sh
apt install php8.3-fpm
a2dismod php8.3
a2enconf php8.3-fpm
a2enmod proxy_fcgi
a2dismod mpm_prefork
a2enmod mpm_event
a2enmod ssl
a2enmod http2
a2enmod rewrite
a2enmod expires
```

重启 Apache2：
```sh
systemctl restart apache2 php8.3-fpm
```

查看是否正确开启：
```sh
 cd /etc/apache2
 grep -r Protocols .
```

### 8. 安装和配置 phpPgAdmin（可选）

安装 phpPgAdmin：

```sh
apt install phppgadmin -y
```

配置 Apache 使用 phpPgAdmin：

编辑 Apache 配置文件：

```sh
nano /etc/apache2/conf-available/phppgadmin.conf
```

确保配置允许从您的客户端 IP 访问：

```apache
<Directory /usr/share/phppgadmin>
    DirectoryIndex index.php
    AllowOverride None

    # Only allow connections from localhost:
    # Require local

    # Allow from all:
    Require all granted
</Directory>
```

保存并退出编辑器。

编辑 phpPgAdmin 的配置文件：

```sh
nano /etc/phppgadmin/config.inc.php
```

找到并修改以下行：

```php
$conf['extra_login_security'] = true;
```

将其改为：

```php
$conf['extra_login_security'] = false;
```

启用配置并重启 Apache：

```sh
a2enconf phppgadmin
```

```sh
systemctl reload apache2 php8.3-fpm
```

现在可以在浏览器中访问 `http://your_server_ip/phppgadmin` 来使用 phpPgAdmin 管理 PostgreSQL 数据库。
