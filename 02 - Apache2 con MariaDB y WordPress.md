# Apache2 con MariaDB y WordPress

Esta ruta instala un WordPress normal usando Apache2, PHP y MariaDB.

Para Multisite, termina esta guia y luego sigue `08 - WordPress Multisite.md`.

---

## 1. Instalar Apache2, PHP y MariaDB

```bash
sudo apt update
sudo apt install -y apache2 mariadb-server mariadb-client php php-mysql php-curl php-gd php-mbstring php-xml php-soap php-intl php-zip php-opcache libapache2-mod-php
sudo a2enmod rewrite headers expires ssl
sudo systemctl enable apache2 mariadb
sudo systemctl start apache2 mariadb
```

---

## 2. Crear base de datos

```bash
sudo mysql
```

```sql
CREATE DATABASE wordpress_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'wordpress_user'@'localhost' IDENTIFIED BY 'Cambia_Esta_Contrasena_Larga';
GRANT ALL PRIVILEGES ON wordpress_db.* TO 'wordpress_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

---

## 3. Descargar WordPress

```bash
sudo mkdir -p /var/www/tudominio.com/public
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
sudo rsync -a wordpress/ /var/www/tudominio.com/public/
sudo chown -R www-data:www-data /var/www/tudominio.com
sudo find /var/www/tudominio.com -type d -exec chmod 755 {} \;
sudo find /var/www/tudominio.com -type f -exec chmod 644 {} \;
```

---

## 4. Configurar WordPress

```bash
sudo cp /var/www/tudominio.com/public/wp-config-sample.php /var/www/tudominio.com/public/wp-config.php
sudo nano /var/www/tudominio.com/public/wp-config.php
```

Cambia:

```php
define( 'DB_NAME', 'wordpress_db' );
define( 'DB_USER', 'wordpress_user' );
define( 'DB_PASSWORD', 'Cambia_Esta_Contrasena_Larga' );
define( 'DB_HOST', 'localhost' );
```

Genera claves nuevas desde:

```text
https://api.wordpress.org/secret-key/1.1/salt/
```

---

## 5. Crear sitio Apache2

```bash
sudo nano /etc/apache2/sites-available/tudominio.com.conf
```

```apache
<VirtualHost *:80>
    ServerName tudominio.com
    ServerAlias www.tudominio.com
    DocumentRoot /var/www/tudominio.com/public

    <Directory /var/www/tudominio.com/public>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/tudominio.com-error.log
    CustomLog ${APACHE_LOG_DIR}/tudominio.com-access.log combined
</VirtualHost>
```

```bash
sudo a2ensite tudominio.com.conf
sudo a2dissite 000-default.conf
sudo apache2ctl configtest
sudo systemctl reload apache2
```

---

## 6. HTTPS global

Para HTTPS reutilizable con Apache2, sigue:

```text
06 - HTTPS global para varios sitios.md
```

La idea es crear snippets globales como:

```text
/etc/apache2/snippets/ssl-global.conf
/etc/apache2/snippets/security-headers.conf
```

Luego cada nuevo sitio solo incluye esos snippets.

---

## 7. Instalar desde navegador

Abre:

```text
http://tudominio.com
```

Si ya configuraste HTTPS:

```text
https://tudominio.com
```

Completa el asistente de WordPress.

---

## 8. Verificar

```bash
sudo apache2ctl configtest
systemctl is-active apache2
systemctl is-active mariadb
curl -I http://localhost -H "Host: tudominio.com"
```

