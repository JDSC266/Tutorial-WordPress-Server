# WP-CLI, backups y mantenimiento

WP-CLI permite administrar WordPress desde consola.

---

## 1. Instalar WP-CLI

```bash
cd /tmp
curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
php wp-cli.phar --info
chmod +x wp-cli.phar
sudo mv wp-cli.phar /usr/local/bin/wp
wp --info
```

---

## 2. Comandos utiles

```bash
wp core version --path=/var/www/tudominio.com/public --allow-root
wp plugin list --path=/var/www/tudominio.com/public --allow-root
wp theme list --path=/var/www/tudominio.com/public --allow-root
wp user list --path=/var/www/tudominio.com/public --allow-root
```

Actualizar:

```bash
wp core update --path=/var/www/tudominio.com/public --allow-root
wp plugin update --all --path=/var/www/tudominio.com/public --allow-root
wp theme update --all --path=/var/www/tudominio.com/public --allow-root
```

---

## 3. Backups

Crear carpeta:

```bash
sudo mkdir -p /var/backups/wordpress
```

Base de datos:

```bash
wp db export /var/backups/wordpress/tudominio.com.sql --path=/var/www/tudominio.com/public --allow-root
```

Archivos:

```bash
sudo tar -czf /var/backups/wordpress/tudominio.com-files.tar.gz /var/www/tudominio.com
```

---

## 4. Restaurar backup

Base de datos:

```bash
wp db import /var/backups/wordpress/tudominio.com.sql --path=/var/www/tudominio.com/public --allow-root
```

Archivos:

```bash
sudo tar -xzf /var/backups/wordpress/tudominio.com-files.tar.gz -C /
```

---

## 5. Multisite

Listar sitios:

```bash
wp site list --path=/var/www/tudominio.com/public --allow-root
```

Comando sobre un subsite:

```bash
wp option get siteurl --url=https://cv.tudominio.com --path=/var/www/tudominio.com/public --allow-root
```

---

## 6. Tareas recomendadas

Semanal:

```bash
wp plugin update --all --path=/var/www/tudominio.com/public --allow-root
wp theme update --all --path=/var/www/tudominio.com/public --allow-root
wp cache flush --path=/var/www/tudominio.com/public --allow-root
```

Antes de cambios grandes:

```bash
wp db export /var/backups/wordpress/antes-del-cambio.sql --path=/var/www/tudominio.com/public --allow-root
```

