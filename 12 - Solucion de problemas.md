# Solucion de problemas

Guia rapida para diagnosticar errores comunes.

---

## 1. El sitio no carga

Verifica servicios:

```bash
systemctl is-active mariadb
systemctl is-active apache2 2>/dev/null || true
systemctl is-active nginx 2>/dev/null || true
systemctl is-active lsws 2>/dev/null || true
systemctl is-active cloudflared 2>/dev/null || true
```

Prueba local:

```bash
curl -I http://localhost -H "Host: tudominio.com"
```

---

## 2. Apache2 falla

```bash
sudo apache2ctl configtest
sudo journalctl -u apache2 -n 80 --no-pager
sudo tail -f /var/log/apache2/error.log
```

Revisa:

- `DocumentRoot`.
- `AllowOverride All`.
- Modulo `rewrite`.
- Permisos `www-data`.

---

## 3. Nginx falla

```bash
sudo nginx -t
sudo journalctl -u nginx -n 80 --no-pager
systemctl status php*-fpm
ls /run/php/
```

Revisa:

- Socket correcto en `fastcgi_pass`.
- `root`.
- `server_name`.
- PHP-FPM activo.

---

## 4. OpenLiteSpeed falla

```bash
systemctl status lsws
sudo tail -f /usr/local/lsws/logs/error.log
```

Revisa:

- Listener puerto 80/443.
- Virtual Host Mapping.
- Document Root.
- Permisos `nobody:nogroup` o usuario configurado.

---

## 5. Error de permisos en WordPress

Apache2 o Nginx:

```bash
sudo chown -R www-data:www-data /var/www/tudominio.com
sudo find /var/www/tudominio.com -type d -exec chmod 755 {} \;
sudo find /var/www/tudominio.com -type f -exec chmod 644 {} \;
```

OpenLiteSpeed:

```bash
sudo chown -R nobody:nogroup /var/www/tudominio.com
sudo find /var/www/tudominio.com -type d -exec chmod 755 {} \;
sudo find /var/www/tudominio.com -type f -exec chmod 644 {} \;
```

---

## 6. HTTPS redirige mal

Si usas Cloudflare Tunnel, agrega en `wp-config.php`:

```php
if ( isset( $_SERVER['HTTP_CF_VISITOR'] ) && strpos( $_SERVER['HTTP_CF_VISITOR'], 'https' ) !== false ) {
    $_SERVER['HTTPS'] = 'on';
}
```

Revisa tambien en WordPress:

```text
Ajustes > Generales > Direccion de WordPress
Ajustes > Generales > Direccion del sitio
```

Deben usar `https://`.

---

## 7. Redis no funciona

```bash
redis-cli ping
systemctl status redis-server
php -m | grep redis
```

OpenLiteSpeed:

```bash
/usr/local/lsws/lsphp84/bin/php -m | grep redis
/usr/local/lsws/lsphp83/bin/php -m | grep redis
/usr/local/lsws/lsphp82/bin/php -m | grep redis
```

---

## 8. Cloudflare Tunnel no funciona

```bash
systemctl status cloudflared
sudo journalctl -u cloudflared -f
cloudflared tunnel list
```

Revisa:

- `config.yml`.
- ID del tunel.
- Ruta de credenciales.
- Hostnames publicos.
- DNS creado en Cloudflare.

---

## 9. Multisite no detecta subdominio

Revisa:

- El subsite existe en WordPress.
- DNS wildcard o hostname del subdominio existe.
- Apache2 tiene `ServerAlias *.tudominio.com`.
- Nginx tiene `server_name *.tudominio.com`.
- OpenLiteSpeed tiene `*.tudominio.com` en el mapping.

