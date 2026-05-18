# Agregar otro WordPress al mismo servidor

Esta guia sirve para crear otro WordPress independiente dentro del mismo servidor.

Ejemplo:

```text
sitio1.com
sitio2.com
```

Cada sitio tendra:

- Su propia carpeta.
- Su propia base de datos.
- Su propio Virtual Host o server block.
- La misma configuracion HTTPS global.

---

## 1. Variables del nuevo sitio

Ejemplo:

```text
Dominio: sitio2.com
Carpeta: /var/www/sitio2.com/public
Base de datos: sitio2_db
Usuario DB: sitio2_user
```

---

## 2. Crear carpeta

```bash
sudo mkdir -p /var/www/sitio2.com/public
```

---

## 3. Crear base de datos

```bash
sudo mysql
```

```sql
CREATE DATABASE sitio2_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'sitio2_user'@'localhost' IDENTIFIED BY 'Otra_Contrasena_Larga';
GRANT ALL PRIVILEGES ON sitio2_db.* TO 'sitio2_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

---

## 4. Descargar WordPress

```bash
cd /tmp
wget https://wordpress.org/latest.tar.gz -O wordpress-latest.tar.gz
tar -xzf wordpress-latest.tar.gz
sudo rsync -a wordpress/ /var/www/sitio2.com/public/
```

Permisos:

```bash
# Apache2 o Nginx
sudo chown -R www-data:www-data /var/www/sitio2.com

# OpenLiteSpeed, si usa nobody
sudo chown -R nobody:nogroup /var/www/sitio2.com
```

---

## 5. Configurar wp-config.php

```bash
sudo cp /var/www/sitio2.com/public/wp-config-sample.php /var/www/sitio2.com/public/wp-config.php
sudo nano /var/www/sitio2.com/public/wp-config.php
```

```php
define( 'DB_NAME', 'sitio2_db' );
define( 'DB_USER', 'sitio2_user' );
define( 'DB_PASSWORD', 'Otra_Contrasena_Larga' );
define( 'DB_HOST', 'localhost' );
```

---

## 6. Apache2: nuevo sitio usando HTTPS global

```bash
sudo nano /etc/apache2/sites-available/sitio2.com.conf
```

```apache
<VirtualHost *:80>
    ServerName sitio2.com
    ServerAlias www.sitio2.com
    Redirect permanent / https://sitio2.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName sitio2.com
    ServerAlias www.sitio2.com
    DocumentRoot /var/www/sitio2.com/public

    Include /etc/apache2/snippets/ssl-global-sitio2.com.conf
    Include /etc/apache2/snippets/security-headers.conf

    <Directory /var/www/sitio2.com/public>
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

Si usas un wildcard de `tudominio.com`, solo aplica a subdominios de `tudominio.com`. Para `sitio2.com` necesitas certificado de `sitio2.com`.

```bash
sudo a2ensite sitio2.com.conf
sudo apache2ctl configtest
sudo systemctl reload apache2
```

---

## 7. Nginx: nuevo sitio usando HTTPS global

```bash
sudo nano /etc/nginx/sites-available/sitio2.com
```

```nginx
server {
    listen 80;
    server_name sitio2.com www.sitio2.com;
    return 301 https://sitio2.com$request_uri;
}

server {
    listen 443 ssl http2;
    server_name sitio2.com www.sitio2.com;
    root /var/www/sitio2.com/public;
    index index.php index.html;

    include snippets/ssl-global-sitio2.com.conf;
    include snippets/security-headers.conf;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.3-fpm.sock;
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/sitio2.com /etc/nginx/sites-enabled/sitio2.com
sudo nginx -t
sudo systemctl reload nginx
```

---

## 8. OpenLiteSpeed: nuevo sitio

En WebAdmin:

1. Crea Virtual Host `sitio2.com`.
2. Document Root: `/var/www/sitio2.com/public`.
3. Agrega mapping al listener `80`.
4. Agrega mapping al listener `443`.
5. Si usas listener HTTPS global, no repitas el certificado.

Reinicia:

```bash
sudo systemctl restart lsws
```

---

## 9. Cloudflare Tunnel

Si usas Cloudflare Tunnel, agrega hostname:

```text
sitio2.com -> http://localhost:80
www.sitio2.com -> http://localhost:80
```

Si ya tienes wildcard para subdominios de un dominio, no cubre dominios totalmente diferentes.

---

## 10. Instalar desde navegador

```text
https://sitio2.com
```

Completa el asistente.

