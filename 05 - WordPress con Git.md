# WordPress con Git

Git puede usarse de dos formas:

1. Para versionar temas y plugins.
2. Para desplegar un proyecto WordPress completo.

La opcion mas segura es versionar temas/plugins y mantener WordPress core actualizado con WP-CLI.

---

## 1. Instalar Git

```bash
sudo apt update
sudo apt install -y git
git --version
```

---

## 2. Opcion recomendada: Git para tema o plugin

Ejemplo con un tema:

```bash
cd /var/www/tudominio.com/public/wp-content/themes
sudo git clone https://github.com/usuario/mi-tema.git
sudo chown -R www-data:www-data mi-tema
```

Para OpenLiteSpeed con usuario `nobody`:

```bash
sudo chown -R nobody:nogroup mi-tema
```

Actualizar:

```bash
cd /var/www/tudominio.com/public/wp-content/themes/mi-tema
sudo git pull
```

---

## 3. Opcion avanzada: proyecto WordPress completo

Clona en una carpeta temporal:

```bash
cd /tmp
git clone https://github.com/usuario/mi-wordpress.git
```

Copia al sitio:

```bash
sudo rsync -a --delete mi-wordpress/ /var/www/tudominio.com/public/
sudo chown -R www-data:www-data /var/www/tudominio.com
```

No subas al repositorio:

```text
wp-config.php
wp-content/uploads/
*.sql
*.zip
.env
```

Ejemplo `.gitignore`:

```gitignore
wp-config.php
wp-content/uploads/
wp-content/cache/
*.sql
*.tar.gz
*.zip
.env
```

---

## 4. Deploy simple con Git

Crea script:

```bash
sudo nano /usr/local/bin/deploy-wordpress
```

```bash
#!/bin/bash
set -e
SITE_PATH="/var/www/tudominio.com/public"
cd "$SITE_PATH"
git pull
chown -R www-data:www-data "$SITE_PATH"
find "$SITE_PATH" -type d -exec chmod 755 {} \;
find "$SITE_PATH" -type f -exec chmod 644 {} \;
```

```bash
sudo chmod +x /usr/local/bin/deploy-wordpress
```

Ejecutar:

```bash
sudo deploy-wordpress
```

---

## 5. Usar rama de produccion

```bash
cd /var/www/tudominio.com/public
sudo git checkout main
sudo git pull origin main
```

Para proyectos grandes, usa ramas:

```text
dev
staging
main
```

---

## 6. Seguridad

- No guardes contrasenas en Git.
- No subas `wp-config.php`.
- No subas backups SQL.
- Usa llaves SSH para repositorios privados.
- Haz backup antes de `git pull` en produccion.

