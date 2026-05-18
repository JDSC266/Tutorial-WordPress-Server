# OpenLiteSpeed con MariaDB y WordPress

Esta ruta instala WordPress usando OpenLiteSpeed, LS PHP y MariaDB.

OpenLiteSpeed es muy buena opcion si vas a usar el plugin LiteSpeed Cache.

---

## 1. Instalar repositorio OpenLiteSpeed

```bash
sudo apt update
sudo apt install -y curl wget gnupg ca-certificates
wget -O - https://repo.litespeed.sh | sudo bash
sudo apt update
```

---

## 2. Instalar OpenLiteSpeed, PHP y MariaDB

Recomendado si tu repositorio ofrece PHP 8.4:

```bash
sudo apt install -y openlitespeed lsphp84 lsphp84-common lsphp84-mysql lsphp84-curl lsphp84-imagick lsphp84-opcache lsphp84-mbstring lsphp84-xml lsphp84-zip mariadb-server mariadb-client unzip
```

Si `lsphp84` no existe, busca versiones disponibles:

```bash
apt-cache search lsphp | grep common
```

Ejemplos alternativos:

```bash
# Para PHP 8.3
sudo apt install -y openlitespeed lsphp83 lsphp83-common lsphp83-mysql lsphp83-curl lsphp83-imagick lsphp83-opcache lsphp83-mbstring lsphp83-xml lsphp83-zip

# Para PHP 8.2
sudo apt install -y openlitespeed lsphp82 lsphp82-common lsphp82-mysql lsphp82-curl lsphp82-imagick lsphp82-opcache lsphp82-mbstring lsphp82-xml lsphp82-zip
```

```bash
sudo systemctl enable lsws mariadb
sudo systemctl start lsws mariadb
```

---

## 3. Crear clave del panel

```bash
sudo /usr/local/lsws/admin/misc/admpass.sh
```

Panel:

```text
https://IP_DEL_SERVIDOR:7080
```

---

## 4. Crear base de datos

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

## 5. Descargar WordPress

```bash
sudo mkdir -p /var/www/tudominio.com/public
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
sudo rsync -a wordpress/ /var/www/tudominio.com/public/
sudo chown -R nobody:nogroup /var/www/tudominio.com
sudo find /var/www/tudominio.com -type d -exec chmod 755 {} \;
sudo find /var/www/tudominio.com -type f -exec chmod 644 {} \;
```

Si tu OpenLiteSpeed usa `www-data`, cambia:

```bash
sudo chown -R www-data:www-data /var/www/tudominio.com
```

---

## 6. Configurar WordPress

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

## 7. Crear Virtual Host

En el panel de OpenLiteSpeed:

1. `Virtual Hosts > Add`.
2. Virtual Host Name: `tudominio.com`.
3. Virtual Host Root: `/var/www/tudominio.com`.
4. Config File: `$SERVER_ROOT/conf/vhosts/tudominio.com/vhconf.conf`.
5. Document Root: `/var/www/tudominio.com/public`.
6. Domains: `tudominio.com, www.tudominio.com`.
7. Enable Rewrite: `Yes`.
8. Listener `Default > Virtual Host Mappings`.
9. Agrega el dominio al listener.

Reinicia:

```bash
sudo systemctl restart lsws
```

---

## 8. HTTPS global

Para HTTPS reutilizable en OpenLiteSpeed, sigue:

```text
06 - HTTPS global para varios sitios.md
```

En OpenLiteSpeed normalmente conviene configurar SSL en el listener `443` y mapear los Virtual Hosts.

---

## 9. Verificar

```bash
systemctl is-active lsws
systemctl is-active mariadb
curl -I http://localhost -H "Host: tudominio.com"
sudo tail -n 50 /usr/local/lsws/logs/error.log
```

