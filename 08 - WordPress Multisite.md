# WordPress Multisite

WordPress Multisite permite manejar varios sitios desde una sola instalacion.

Ejemplo:

```text
tudominio.com
cv.tudominio.com
blog.tudominio.com
```

---

## 1. Antes de activar Multisite

Debes tener:

- WordPress instalado.
- Dominio funcionando.
- Enlaces permanentes funcionando.
- Plugins desactivados temporalmente.

---

## 2. Activar opcion Multisite

Edita:

```bash
sudo nano /var/www/tudominio.com/public/wp-config.php
```

Agrega antes de `/* That's all, stop editing! */`:

```php
define('WP_ALLOW_MULTISITE', true);
```

---

## 3. Instalar red desde WordPress

En el panel:

```text
Herramientas > Configuracion de la red
```

Elige:

```text
Subdominios
```

WordPress te dara dos bloques:

- Bloque para `wp-config.php`.
- Bloque para `.htaccess` si usas Apache2/OpenLiteSpeed.

---

## 4. wp-config.php

Ejemplo:

```php
define( 'MULTISITE', true );
define( 'SUBDOMAIN_INSTALL', true );
define( 'DOMAIN_CURRENT_SITE', 'tudominio.com' );
define( 'PATH_CURRENT_SITE', '/' );
define( 'SITE_ID_CURRENT_SITE', 1 );
define( 'BLOG_ID_CURRENT_SITE', 1 );
```

Copia el bloque exacto que WordPress te muestre.

---

## 5. Apache2 y OpenLiteSpeed

Edita:

```bash
sudo nano /var/www/tudominio.com/public/.htaccess
```

Ejemplo para subdominios:

```apache
RewriteEngine On
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
RewriteBase /
RewriteRule ^index\.php$ - [L]

RewriteRule ^wp-admin$ wp-admin/ [R=301,L]

RewriteCond %{REQUEST_FILENAME} -f [OR]
RewriteCond %{REQUEST_FILENAME} -d
RewriteRule ^ - [L]
RewriteRule ^(wp-(content|admin|includes).*) $1 [L]
RewriteRule ^(.*\.php)$ $1 [L]
RewriteRule . index.php [L]
```

Recarga:

```bash
# Apache2
sudo systemctl reload apache2

# OpenLiteSpeed
sudo systemctl restart lsws
```

---

## 6. Nginx

Nginx no usa `.htaccess`.

El bloque principal debe tener:

```nginx
location / {
    try_files $uri $uri/ /index.php?$args;
}
```

Y el `server_name` debe aceptar subdominios:

```nginx
server_name tudominio.com www.tudominio.com *.tudominio.com;
```

---

## 7. Crear sitio nuevo

En WordPress:

```text
Mis sitios > Administrador de la red > Sitios > Agregar nuevo
```

Direccion:

```text
cv
```

Resultado:

```text
cv.tudominio.com
```

---

## 8. DNS

Necesitas que los subdominios lleguen al servidor.

Con Cloudflare Tunnel, usa:

```text
*.tudominio.com -> tunnel
```

Con conexion directa, usa:

```text
*.tudominio.com -> IP_DEL_SERVIDOR
```

