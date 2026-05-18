# Redis y cache

Redis acelera WordPress guardando datos frecuentes en RAM.

No reemplaza al servidor web ni a MariaDB.

---

## 1. Instalar Redis

```bash
sudo apt update
sudo apt install -y redis-server
sudo systemctl enable redis-server
sudo systemctl start redis-server
```

Edita:

```bash
sudo nano /etc/redis/redis.conf
```

Recomendado:

```conf
supervised systemd
maxmemory 256mb
maxmemory-policy allkeys-lru
appendonly yes
```

Reinicia:

```bash
sudo systemctl restart redis-server
redis-cli ping
```

Debe responder:

```text
PONG
```

---

## 2. Extension PHP Redis

Apache2 o Nginx:

```bash
sudo apt install -y php-redis
```

OpenLiteSpeed:

```bash
# Si usas LS PHP 8.4
sudo apt install -y lsphp84-redis

# Si usas LS PHP 8.3
sudo apt install -y lsphp83-redis

# Si usas LS PHP 8.2
sudo apt install -y lsphp82-redis
```

Reinicia:

```bash
# Apache2
sudo systemctl restart apache2

# Nginx
sudo systemctl restart php*-fpm nginx

# OpenLiteSpeed
sudo systemctl restart lsws
```

---

## 3. Configurar WordPress

En `wp-config.php`:

```php
define('WP_REDIS_HOST', '127.0.0.1');
define('WP_REDIS_PORT', 6379);
define('WP_REDIS_DATABASE', 0);
define('WP_CACHE', true);
```

Instala plugin:

```text
Redis Object Cache
```

Activa:

```text
Ajustes > Redis > Activar cache de objetos
```

---

## 4. Cache de pagina

| Servidor | Recomendado |
|---|---|
| Apache2 | W3 Total Cache o WP Super Cache |
| Nginx | W3 Total Cache o FastCGI cache |
| OpenLiteSpeed | LiteSpeed Cache |

Activa una cosa a la vez:

1. Cache de pagina.
2. Cache de navegador.
3. Cache de objetos con Redis.
4. Minificacion solo despues de probar.

---

## 5. Verificar

```bash
redis-cli ping
redis-cli info memory
redis-cli monitor
```

Abre el sitio y observa si Redis recibe actividad. Sal con `Ctrl+C`.

