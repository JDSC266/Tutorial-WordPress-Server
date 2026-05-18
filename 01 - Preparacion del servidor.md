# Preparacion del servidor

Usa esta guia antes de instalar Apache2, Nginx u OpenLiteSpeed.

Sistema recomendado:

- Ubuntu Server 22.04/24.04.
- Debian 12.
- 2 GB RAM minimo.
- 4 GB RAM o mas si usaras Redis.

---

## 1. Actualizar paquetes

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install -y curl wget unzip git ca-certificates gnupg lsb-release software-properties-common
```

Si estas en Debian minimo y no tienes `sudo`:

```bash
su -
apt update
apt install -y sudo
usermod -aG sudo tu_usuario
exit
```

Cierra sesion y vuelve a entrar.

---

## 2. Crear estructura base

```bash
sudo mkdir -p /var/www
sudo chown root:root /var/www
```

Cada sitio se recomienda asi:

```text
/var/www/tudominio.com/public
```

Ejemplo:

```bash
sudo mkdir -p /var/www/tudominio.com/public
```

---

## 3. Firewall basico

Si usas Cloudflare Tunnel, normalmente no necesitas abrir 80/443 al publico.

Si usaras conexion directa:

```bash
sudo apt install -y ufw
sudo ufw allow OpenSSH
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
sudo ufw status
```

Si usaras OpenLiteSpeed con panel web:

```bash
sudo ufw allow 7080/tcp
```

---

## 4. Nombre del servidor

Opcional:

```bash
sudo hostnamectl set-hostname wordpress-server
```

---

## 5. Verificacion

```bash
lsb_release -a
hostnamectl
df -h
free -h
```

Cuando esto este listo, escoge una ruta:

- Apache2.
- Nginx.
- OpenLiteSpeed.

