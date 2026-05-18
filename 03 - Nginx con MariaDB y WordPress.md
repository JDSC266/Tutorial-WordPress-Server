# Nginx con MariaDB y WordPress

Esta ruta instala WordPress usando Nginx, PHP-FPM y MariaDB.

Nginx no usa `.htaccess`. Las reglas van en el archivo del sitio.

---

## 1. Instalar Nginx, PHP-FPM y MariaDB

```bash
sudo apt update
sudo apt install -y nginx mariadb-server mariadb-client php-fpm php-mysql php-curl php-gd php-mbstring php-xml php-soap php-intl php-zip php-opcache
sudo systemctl enable nginx mariadb
sudo systemctl start nginx mariadb
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

```php
define( 'DB_NAME', 'wordpress_db' );
define( 'DB_USER', 'wordpress_user' );
define( 'DB_PASSWORD', 'Cambia_Esta_Contrasena_Larga' );
define( 'DB_HOST', 'localhost' );
```

---

## 5. Detectar socket PHP-FPM

```bash
ls /run/php/
```

Ejemplos:

```text
php8.3-fpm.sock
php8.2-fpm.sock
php8.1-fpm.sock
```

Usa el que exista en tu servidor.

---

## 6. Crear sitio Nginx

```bash
sudo nano /etc/nginx/sites-available/tudominio.com
```

Cambia `php8.3-fpm.sock` si tu servidor usa otra version.

```nginx
server {
    listen 80;
    server_name tudominio.com www.tudominio.com;
    root /var/www/tudominio.com/public;
    index index.php index.html;

    client_max_body_size 64M;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.3-fpm.sock;
    }

    location ~* \.(css|js|jpg|jpeg|png|gif|ico|svg|webp|woff|woff2)$ {
        expires 30d;
        access_log off;
    }

    location ~ /\. {
        deny all;
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/tudominio.com /etc/nginx/sites-enabled/tudominio.com
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl reload nginx
```

---

## 7. HTTPS global

Para HTTPS reutilizable con Nginx, sigue:

```text
06 - HTTPS global para varios sitios.md
```

La idea es crear snippets como:

```text
/etc/nginx/snippets/ssl-global.conf
/etc/nginx/snippets/security-headers.conf
```

Luego cada nuevo sitio solo los incluye.

---

## 8. Verificar

```bash
sudo nginx -t
systemctl is-active nginx
systemctl status php*-fpm
curl -I http://localhost -H "Host: tudominio.com"
```

