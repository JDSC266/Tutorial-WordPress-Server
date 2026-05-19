# Tutorial WordPress Server

Guia modular para instalar WordPress en un servidor Ubuntu o Debian usando Apache2, Nginx u OpenLiteSpeed.

Tambien incluye rutas para WordPress Multisite, Git, Redis, Cloudflare Tunnel, HTTPS global y solucion de problemas.

La idea es que puedas empezar con una instalacion sencilla y luego crecer a varios WordPress dentro del mismo servidor sin repetir toda la configuracion HTTPS cada vez.

---

## Rutas principales

| Guia | Cuando usarla |
|---|---|
| [01 - Preparacion del servidor](<01 - Preparacion del servidor.md>) | Primeros pasos en Ubuntu/Debian, paquetes base, firewall y estructura de carpetas. |
| [02 - Apache2 con MariaDB y WordPress](<02 - Apache2 con MariaDB y WordPress.md>) | Instalar Apache2, base de datos y WordPress normal. |
| [03 - Nginx con MariaDB y WordPress](<03 - Nginx con MariaDB y WordPress.md>) | Instalar Nginx, PHP-FPM, base de datos y WordPress normal. |
| [04 - OpenLiteSpeed con MariaDB y WordPress](<04 - OpenLiteSpeed con MariaDB y WordPress.md>) | Instalar OpenLiteSpeed, LS PHP, base de datos y WordPress normal. |
| [05 - WordPress con Git](<05 - WordPress con Git.md>) | Usar Git para desplegar WordPress, temas o plugins. |
| [06 - HTTPS global para varios sitios](<06 - HTTPS global para varios sitios.md>) | Configuracion HTTPS reutilizable para agregar nuevos WordPress sin rehacer todo. |
| [07.1 - Cloudflare Tunnel por terminal](<07.1 - Cloudflare Tunnel por terminal.md>) | Metodo con `config.yml`, comandos `cloudflared` y rutas DNS desde terminal. |
| [07.2 - Cloudflare Tunnel desde el panel](<07.2 - Cloudflare Tunnel desde el panel.md>) | Metodo recomendado si vas a manejar hostnames y servicios desde Cloudflare Zero Trust. |
| [08 - WordPress Multisite](<08 - WordPress Multisite.md>) | Convertir WordPress en red Multisite con subdominios. |
| [09 - Redis y cache](<09 - Redis y cache.md>) | Instalar Redis y configurar cache de objetos/pagina. |
| [10 - Agregar otro WordPress al mismo servidor](<10 - Agregar otro WordPress al mismo servidor.md>) | Crear un segundo sitio reutilizando la configuracion global. |
| [11 - WP-CLI, backups y mantenimiento](<11 - WP-CLI backups y mantenimiento.md>) | Administrar, actualizar y respaldar WordPress por consola. |
| [12 - Solucion de problemas](<12 - Solucion de problemas.md>) | Errores comunes con Apache2, Nginx, OpenLiteSpeed, DNS, HTTPS, Redis y permisos. |

---

## Que servidor web elegir

| Servidor | Ideal para | Dificultad |
|---|---|---:|
| Apache2 | Empezar rapido, usar `.htaccess`, compatibilidad clasica de WordPress. | Baja |
| Nginx | Mejor rendimiento con PHP-FPM y configuraciones limpias. | Media |
| OpenLiteSpeed | WordPress optimizado con LiteSpeed Cache. | Media |

Si estas empezando, usa Apache2. Si quieres eficiencia, usa Nginx. Si quieres usar LiteSpeed Cache como pieza central, usa OpenLiteSpeed.

## Ventajas y Desventajas

| Aspecto | **Varios sitios con WordPress separados** | **Un solo WordPress Multisite** |
|---|---|---|
| **Ventajas** | - Cada sitio es independiente. <br> - Si un sitio falla, los demás pueden seguir funcionando. - <br> Puedes usar temas, plugins y configuraciones distintas en cada instalación. <br> - Es más fácil migrar, vender o separar un sitio individual. | - Administración centralizada desde un solo panel. <br> - Actualizas WordPress, temas y plugins una sola vez. <br> - Puede ahorrar espacio y mantenimiento cuando los sitios son parecidos. <br> - Útil para redes de sitios relacionados, como sedes, marcas o departamentos. |
| **Desventajas** | - Requiere más mantenimiento porque cada instalación se actualiza por separado. <br> - Consume más espacio y recursos si hay muchos sitios. <br> - Hay que gestionar copias de seguridad, seguridad y usuarios en cada sitio. <br> - Puede ser más difícil mantener consistencia entre sitios. | - Si la instalación principal falla, puede afectar a todos los sitios. <br> - No todos los plugins funcionan bien en Multisite. <br> - Separar o migrar un sitio individual puede ser más complicado. <br> - Requiere más cuidado con permisos, usuarios, dominios y compatibilidad. |

---

## Dos formas de HTTPS global

### Opcion A: Cloudflare Tunnel

Cloudflare da HTTPS al visitante y el tunel conecta con tu servidor por `localhost:80`.

Ventajas:

- No abres puertos.
- No necesitas Certbot en el servidor.
- Puedes usar una regla wildcard para subdominios.
- Es comodo para servidores caseros o VMs locales.

### Opcion B: HTTPS directo en el servidor

El servidor escucha en `443` y usa certificados TLS.

Ventajas:

- No dependes de Cloudflare Tunnel.
- Puedes usar Certbot o certificados de origen de Cloudflare.
- Reutilizas snippets globales para todos los sitios.

La guia [06 - HTTPS global para varios sitios](<06 - HTTPS global para varios sitios.md>) explica las dos.

## Ventajas y Desventajas

| Aspecto | **Cloudflare Tunnel** | **Conexión directa** |
|---|---|---|
| **Ventajas** | - No necesitas abrir puertos en el router o firewall. <br> - Oculta la IP pública del servidor. <br> - Añade protección de Cloudflare, como filtros, reglas y mitigación DDoS. <br> - Facilita publicar servicios internos de forma más segura. | - Configuración más simple en algunos casos. <br> - No dependes de un intermediario externo para acceder al servicio. <br> - Puede tener menor latencia si la ruta de red es directa. <br> - Tienes control total sobre el tráfico y la infraestructura. |
| **Desventajas** | - Dependencia de Cloudflare: si falla o cambian condiciones, puede afectar el acceso. <br> - Puede añadir algo de latencia. <br> - Requiere configurar y mantener el túnel. <br> - Algunas funciones avanzadas pueden depender del plan contratado. | - Necesitas abrir puertos y exponer el servidor a internet. <br> - La IP pública puede quedar visible. <br> - Requiere más cuidado con firewall, certificados, ataques y actualizaciones. <br> - Puede ser más difícil de proteger frente a escaneos o ataques directos. |

---

## Estructura recomendada del servidor

```text
/var/www/
├── sitio1.com/
│   └── public/
├── sitio2.com/
│   └── public/
└── otrodominio.com/
    └── public/

/etc/
├── apache2/
├── nginx/
└── redis/
```

Cada WordPress debe tener su propia carpeta y su propia base de datos, excepto cuando uses WordPress Multisite.

---

## Convenciones de esta guia

Cambia estos valores por los tuyos:

```text
tudominio.com
www.tudominio.com
cv.tudominio.com
wordpress_db
wordpress_user
Cambia_Esta_Contrasena_Larga
```

Cuando un comando cambie por sistema, se marcara asi:

```bash
# Para Ubuntu
comando

# Para Debian
comando
```
