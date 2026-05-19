# HTTPS global para varios sitios

Esta guia explica como preparar HTTPS de forma reutilizable cuando tienes varios WordPress en el mismo servidor.

La meta es que, cuando agregues otro WordPress, solo crees:

- Carpeta del sitio.
- Base de datos.
- Virtual Host o server block.
- DNS.

Y que la parte HTTPS ya este resuelta con una configuracion global.

---

## 1. Dos formas de hacerlo

### Opcion A: Cloudflare Tunnel

Cloudflare entrega HTTPS al visitante y manda el trafico al servidor por:

```text
http://localhost:80
```

Ventajas:

- No abres puertos.
- No instalas certificados en cada sitio.
- Puedes usar wildcard para subdominios.
- Es ideal para servidores caseros o redes donde no quieres tocar el router.

Sigue:

```text
07.2 - Cloudflare Tunnel desde el panel.md
```

### Opcion B: HTTPS directo en el servidor

El servidor escucha en:

```text
443/tcp
```

Y usa certificados TLS con Apache2, Nginx u OpenLiteSpeed.

Esta guia se enfoca en esta opcion.

---

## 2. Certificado recomendado

Si vas a usar muchos subdominios de un mismo dominio:

```text
tudominio.com
blog.tudominio.com
cv.tudominio.com
tienda.tudominio.com
```

usa un certificado wildcard:

```text
tudominio.com
*.tudominio.com
```

Si vas a usar dominios completamente diferentes:

```text
tudominio.com
otrodominio.com
misitio.net
```

necesitas un certificado para cada dominio o un certificado que incluya todos esos nombres.

---

## 3. Crear certificado wildcard con Certbot y Cloudflare DNS

Instala Certbot:

```bash
sudo apt update
sudo apt install -y certbot python3-certbot-dns-cloudflare
```

Crea carpeta de credenciales:

```bash
sudo mkdir -p /root/.secrets/certbot
sudo nano /root/.secrets/certbot/cloudflare.ini
```

Contenido:

```ini
dns_cloudflare_api_token = TU_TOKEN_DE_CLOUDFLARE
```

Permisos:

```bash
sudo chmod 600 /root/.secrets/certbot/cloudflare.ini
```

Solicita certificado:

```bash
sudo certbot certonly \
  --dns-cloudflare \
  --dns-cloudflare-credentials /root/.secrets/certbot/cloudflare.ini \
  -d tudominio.com \
  -d "*.tudominio.com"
```

Rutas resultantes:

```text
/etc/letsencrypt/live/tudominio.com/fullchain.pem
/etc/letsencrypt/live/tudominio.com/privkey.pem
```

---

## 4. Apache2

En Apache2 la configuracion global se hace con snippets reutilizables.

### 4.1 Activar modulos

```bash
sudo a2enmod ssl rewrite headers expires
sudo systemctl reload apache2
```

### 4.2 Crear snippet SSL global

```bash
sudo mkdir -p /etc/apache2/snippets
sudo nano /etc/apache2/snippets/ssl-global-tudominio.com.conf
```

```apache
SSLEngine on
SSLCertificateFile /etc/letsencrypt/live/tudominio.com/fullchain.pem
SSLCertificateKeyFile /etc/letsencrypt/live/tudominio.com/privkey.pem

SSLProtocol all -SSLv2 -SSLv3 -TLSv1 -TLSv1.1
SSLHonorCipherOrder off
```

### 4.3 Crear snippet de seguridad

```bash
sudo nano /etc/apache2/snippets/security-headers.conf
```

```apache
Header always set X-Content-Type-Options "nosniff"
Header always set X-Frame-Options "SAMEORIGIN"
Header always set Referrer-Policy "strict-origin-when-cross-origin"
Header always set Permissions-Policy "camera=(), microphone=(), geolocation=()"
```

### 4.4 Plantilla de sitio Apache2 con HTTPS reutilizable

Cada nuevo WordPress puede usar esta estructura:

```apache
<VirtualHost *:80>
    ServerName blog.tudominio.com
    ServerAlias www.blog.tudominio.com
    Redirect permanent / https://blog.tudominio.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName blog.tudominio.com
    ServerAlias www.blog.tudominio.com
    DocumentRoot /var/www/blog.tudominio.com/public

    Include /etc/apache2/snippets/ssl-global-tudominio.com.conf
    Include /etc/apache2/snippets/security-headers.conf

    <Directory /var/www/blog.tudominio.com/public>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/blog.tudominio.com-error.log
    CustomLog ${APACHE_LOG_DIR}/blog.tudominio.com-access.log combined
</VirtualHost>
```

Activa el sitio:

```bash
sudo a2ensite blog.tudominio.com.conf
sudo apache2ctl configtest
sudo systemctl reload apache2
```

### 4.5 Agregar otro WordPress en Apache2

Para un nuevo sitio solo cambias:

```text
blog.tudominio.com
/var/www/blog.tudominio.com/public
```

La parte HTTPS queda igual:

```apache
Include /etc/apache2/snippets/ssl-global-tudominio.com.conf
Include /etc/apache2/snippets/security-headers.conf
```

---

## 5. Nginx

En Nginx la configuracion global se hace con snippets dentro de `/etc/nginx/snippets/`.

### 5.1 Crear snippet SSL global

```bash
sudo mkdir -p /etc/nginx/snippets
sudo nano /etc/nginx/snippets/ssl-global-tudominio.com.conf
```

```nginx
ssl_certificate /etc/letsencrypt/live/tudominio.com/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/tudominio.com/privkey.pem;

ssl_protocols TLSv1.2 TLSv1.3;
ssl_prefer_server_ciphers off;
ssl_session_cache shared:SSL:10m;
ssl_session_timeout 1d;
ssl_session_tickets off;
```

### 5.2 Crear snippet de seguridad

```bash
sudo nano /etc/nginx/snippets/security-headers.conf
```

```nginx
add_header X-Content-Type-Options "nosniff" always;
add_header X-Frame-Options "SAMEORIGIN" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Permissions-Policy "camera=(), microphone=(), geolocation=()" always;
```

### 5.3 Crear snippet WordPress comun

Este snippet evita repetir reglas de WordPress en cada sitio:

```bash
sudo nano /etc/nginx/snippets/wordpress-common.conf
```

```nginx
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
```

Si usas Debian 12 o PHP 8.2, cambia:

```nginx
fastcgi_pass unix:/run/php/php8.2-fpm.sock;
```

Puedes ver el socket real con:

```bash
ls /run/php/
```

### 5.4 Plantilla de sitio Nginx con HTTPS reutilizable

```nginx
server {
    listen 80;
    server_name blog.tudominio.com www.blog.tudominio.com;
    return 301 https://blog.tudominio.com$request_uri;
}

server {
    listen 443 ssl http2;
    server_name blog.tudominio.com www.blog.tudominio.com;
    root /var/www/blog.tudominio.com/public;

    include snippets/ssl-global-tudominio.com.conf;
    include snippets/security-headers.conf;
    include snippets/wordpress-common.conf;
}
```

Activa:

```bash
sudo ln -s /etc/nginx/sites-available/blog.tudominio.com /etc/nginx/sites-enabled/blog.tudominio.com
sudo nginx -t
sudo systemctl reload nginx
```

### 5.5 Agregar otro WordPress en Nginx

Para un nuevo sitio solo cambias:

```text
blog.tudominio.com
/var/www/blog.tudominio.com/public
```

La parte HTTPS queda igual:

```nginx
include snippets/ssl-global-tudominio.com.conf;
include snippets/security-headers.conf;
include snippets/wordpress-common.conf;
```

---

## 6. OpenLiteSpeed

En OpenLiteSpeed la forma mas limpia es configurar HTTPS en un listener global del puerto `443` y luego mapear cada Virtual Host.

### 6.1 Crear listener HTTPS global

Entra al panel:

```text
https://IP_DEL_SERVIDOR:7080
```

Ve a:

```text
WebAdmin > Listeners > Add
```

Valores:

```text
Listener Name: HTTPS
IP Address: ANY
Port: 443
Secure: Yes
```

Guarda.

### 6.2 Configurar certificado en el listener

Dentro del listener `HTTPS`, ve a:

```text
SSL > Edit
```

Configura:

```text
Private Key File: /etc/letsencrypt/live/tudominio.com/privkey.pem
Certificate File: /etc/letsencrypt/live/tudominio.com/fullchain.pem
Chained Certificate: Yes
```

Guarda y reinicia OpenLiteSpeed:

```bash
sudo systemctl restart lsws
```

### 6.3 Crear Virtual Host para cada WordPress

En el panel:

```text
Virtual Hosts > Add
```

Ejemplo:

```text
Virtual Host Name: blog.tudominio.com
Virtual Host Root: /var/www/blog.tudominio.com
Config File: $SERVER_ROOT/conf/vhosts/blog.tudominio.com/vhconf.conf
```

Luego configura:

```text
General > Document Root: /var/www/blog.tudominio.com/public
General > Enable GZIP Compression: Yes
Rewrite > Enable Rewrite: Yes
```

### 6.4 Mapear el Virtual Host al listener 80

Ve a:

```text
Listeners > Default > Virtual Host Mappings > Add
```

Valores:

```text
Virtual Host: blog.tudominio.com
Domains: blog.tudominio.com, www.blog.tudominio.com
```

### 6.5 Mapear el Virtual Host al listener 443

Ve a:

```text
Listeners > HTTPS > Virtual Host Mappings > Add
```

Valores:

```text
Virtual Host: blog.tudominio.com
Domains: blog.tudominio.com, www.blog.tudominio.com
```

Asi todos los sitios usan el certificado del listener HTTPS global.

### 6.6 Redireccion HTTP a HTTPS

En el Virtual Host, activa rewrite y agrega reglas:

```apache
RewriteEngine On
RewriteCond %{HTTPS} !=on
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]
```

Si WordPress ya administra `.htaccess`, puedes poner esta regla arriba de las reglas de WordPress.

### 6.7 Agregar otro WordPress en OpenLiteSpeed

Para un nuevo sitio:

1. Crea carpeta `/var/www/nuevo.tudominio.com/public`.
2. Crea base de datos.
3. Crea Virtual Host.
4. Mapea el Virtual Host al listener `Default` y al listener `HTTPS`.
5. No repitas el certificado si ya esta en el listener `HTTPS`.

---

## 7. Recarga automatica despues de renovar certificados

Certbot suele crear renovacion automatica. Verifica:

```bash
sudo certbot renew --dry-run
```

Puedes crear un hook para recargar servicios despues de renovar:

```bash
sudo mkdir -p /etc/letsencrypt/renewal-hooks/deploy
sudo nano /etc/letsencrypt/renewal-hooks/deploy/reload-webservers.sh
```

Contenido:

```bash
#!/bin/bash
systemctl reload apache2 2>/dev/null || true
systemctl reload nginx 2>/dev/null || true
systemctl restart lsws 2>/dev/null || true
```

Permisos:

```bash
sudo chmod +x /etc/letsencrypt/renewal-hooks/deploy/reload-webservers.sh
```

---

## 8. Checklist para agregar un nuevo WordPress

1. Crear carpeta:

```bash
sudo mkdir -p /var/www/blog.tudominio.com/public
```

2. Crear base de datos.

3. Descargar WordPress.

4. Crear configuracion del servidor web:

```text
Apache2: nuevo VirtualHost con Include de snippets.
Nginx: nuevo server block con include de snippets.
OpenLiteSpeed: nuevo Virtual Host mapeado al listener HTTPS.
```

5. Crear DNS:

```text
blog.tudominio.com -> IP del servidor
```

o, si usas Cloudflare Tunnel:

```text
blog.tudominio.com -> tunnel
```

6. Probar:

```bash
curl -I https://blog.tudominio.com
```

---

## 9. Nota importante sobre dominios diferentes

Un certificado wildcard de:

```text
*.tudominio.com
```

sirve para:

```text
blog.tudominio.com
cv.tudominio.com
tienda.tudominio.com
```

pero no sirve para:

```text
otrodominio.com
blog.otrodominio.com
```

Para otro dominio debes crear otro certificado y otro snippet, por ejemplo:

```text
/etc/apache2/snippets/ssl-global-otrodominio.com.conf
/etc/nginx/snippets/ssl-global-otrodominio.com.conf
```

En OpenLiteSpeed puedes crear otro listener seguro o configurar certificados por Virtual Host si mezclas muchos dominios distintos.
