# Registro de Cambios - 2026-06-05 15:52:27-04:00

Este archivo registra las modificaciones realizadas en la infraestructura y configuración del Caddyfile y SSH para la VPS en DigitalOcean.

---

## 1. Cambios de Conectividad y Autenticación SSH
* **Archivo modificado**: [hosts.yml](file:///Users/nino/Developer/infraestructura/inventory/hosts.yml)
  * Se cambió `ansible_host: 64.227.108.23` por el alias `ansible_host: do` configurado en el archivo local de SSH `~/.ssh/config`.
  * Se configuró al usuario `enzo` como usuario de conexión en lugar del usuario `root`, debido a políticas de restricción de acceso en la VPS.
* **Archivo modificado**: [digitalocean.yml](file:///Users/nino/Developer/infraestructura/inventory/host_vars/digitalocean.yml)
  * Se actualizó la variable `ansible_user` a `enzo` para que no sobrescriba al usuario general con `root`.
* **Archivo modificado**: [ansible.cfg](file:///Users/nino/Developer/infraestructura/ansible.cfg)
  * Se actualizó la llave `private_key_file` de `~/.ssh/id_ed25519` a `~/.ssh/id_do` (y se permite heredar las llaves correspondientes mediante el cliente SSH de la máquina).

## 2. Corrección de Deprecaciones en Ansible
* **Archivo modificado**: [caddy.yml](file:///Users/nino/Developer/infraestructura/playbooks/caddy.yml)
  * Se modificó la línea `dest: /etc/caddy/Caddyfile.bak.{{ ansible_date_time.date }}` por la sintaxis moderna `dest: /etc/caddy/Caddyfile.bak.{{ ansible_facts['date_time']['date'] }}`.
  * Esto resolvió la advertencia `INJECT_FACTS_AS_VARS` introducida a partir de ansible-core v2.24.

## 3. Optimizaciones en la Plantilla de Caddy
* **Archivo modificado**: [Caddyfile.j2](file:///Users/nino/Developer/infraestructura/templates/Caddyfile.j2)
  * **Cabeceras de Seguridad**: Se extrajeron las cabeceras repetitivas de HSTS, Clickjacking (X-Frame-Options), Sniffing (X-Content-Type-Options) y la cabecera Server a un fragmento reutilizable llamado `(seguridad)` en la cabecera del archivo.
  * **Imports**: Se agregaron directivas `import seguridad` dentro de cada sitio activo (`iteravirtus.cl`, `cicpol.synatrix.io`, `tak.cicpol.synatrix.io`, `verintia.com`, `errors.iteravirtus.cl`).
  * **Sintaxis**: Se corrigió el orden del `import seguridad` en `tak.cicpol.synatrix.io` para colocarlo adentro de las llaves del sitio.
  * **Nuevo Servicio (GlitchTip)**: Se configuró el dominio `errors.iteravirtus.cl` redirigiendo peticiones al puerto `localhost:8001` y se habilitó su propio registro de logs en `/var/log/caddy/glitchtip_access.log`.
  * **Limpieza**: Se dio de baja la redirección y proxy para `ento.cl` y la ruta `/wfs/*` de `iteravirtus.cl`.
