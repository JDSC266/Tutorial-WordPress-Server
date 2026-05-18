# Cloudflare Tunnel y DNS

Cloudflare Tunnel permite publicar WordPress sin abrir puertos del router.

Tambien simplifica HTTPS: Cloudflare entrega HTTPS al visitante y tu servidor puede escuchar en HTTP local.

---

## 1. Instalar cloudflared

```bash
sudo apt update
sudo apt install -y curl lsb-release ca-certificates gnupg
curl -fsSL https://pkg.cloudflare.com/cloudflare-main.gpg | sudo tee /usr/share/keyrings/cloudflare-archive-keyring.gpg >/dev/null
echo "deb [signed-by=/usr/share/keyrings/cloudflare-archive-keyring.gpg] https://pkg.cloudflare.com/cloudflared $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/cloudflared.list
sudo apt update
sudo apt install -y cloudflared
```

Si el repositorio falla en Debian, instala el `.deb` desde la pagina oficial de Cloudflare.

---

## 2. Login y tunel

```bash
cloudflared tunnel login
cloudflared tunnel create wordpress-tunnel
cloudflared tunnel list
```

Guarda el ID del tunel.

---

## 3. Configuracion simple

```bash
mkdir -p ~/.cloudflared
nano ~/.cloudflared/config.yml
```

```yaml
tunnel: ID_DEL_TUNEL
credentials-file: /home/tu_usuario/.cloudflared/ID_DEL_TUNEL.json

ingress:
  - hostname: tudominio.com
    service: http://localhost:80
  - hostname: www.tudominio.com
    service: http://localhost:80
  - hostname: cv.tudominio.com
    service: http://localhost:80
  - service: http_status:404
```

---

## 4. Configuracion wildcard para subdominios

Si quieres que todos los subdominios vayan al mismo servidor:

```yaml
tunnel: ID_DEL_TUNEL
credentials-file: /home/tu_usuario/.cloudflared/ID_DEL_TUNEL.json

ingress:
  - hostname: tudominio.com
    service: http://localhost:80
  - hostname: "*.tudominio.com"
    service: http://localhost:80
  - service: http_status:404
```

Esto es ideal para WordPress Multisite con subdominios.

---

## 5. Crear rutas DNS

```bash
cloudflared tunnel route dns wordpress-tunnel tudominio.com
cloudflared tunnel route dns wordpress-tunnel www.tudominio.com
cloudflared tunnel route dns wordpress-tunnel cv.tudominio.com
```

Para wildcard:

```bash
cloudflared tunnel route dns wordpress-tunnel "*.tudominio.com"
```

Si el comando wildcard no funciona en tu caso, crea el registro desde el panel de Cloudflare o agrega cada subdominio manualmente.

---

## 6. Instalar como servicio

```bash
sudo cloudflared service install
sudo systemctl enable cloudflared
sudo systemctl start cloudflared
sudo systemctl status cloudflared
```

---

## 7. WordPress detras de Cloudflare

Si WordPress cree que esta en HTTP, agrega en `wp-config.php`:

```php
if ( isset( $_SERVER['HTTP_CF_VISITOR'] ) && strpos( $_SERVER['HTTP_CF_VISITOR'], 'https' ) !== false ) {
    $_SERVER['HTTPS'] = 'on';
}
```

---

## 8. Verificar

```bash
systemctl is-active cloudflared
cloudflared tunnel list
curl -I http://localhost -H "Host: tudominio.com"
curl -I http://localhost -H "Host: cv.tudominio.com"
```

Navegador:

```text
https://tudominio.com
https://cv.tudominio.com
```

