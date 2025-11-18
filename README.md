# Trabajo Práctico Integrador – Computación Aplicada
Universidad de Palermo – 2025

## Integrantes del grupo
- Mateo Bonomi
- Giovanni Feruglio
- Santiago Valle
- Candela Sánchez
- Rozo Cruz
- Nahuel Matías Frutos
---

## 1. Configuración del entorno

- Sistema operativo: **Debian GNU/Linux 11 (Bullseye)**
- Hostname configurado: **TPServer**
- Contraseña del usuario root establecida según consigna: **palermo**
- Acceso root habilitado mediante autenticación SSH con **clave pública** provista por la cátedra.
- Claves utilizadas:
  - Clave privada: `clave_privada.txt`
  - Clave pública: `clave_publica.pub`

---

## 2. Servicios instalados y configurados

### SSH
- Servicio **OpenSSH Server** instalado y habilitado.
- Directorio `/root/.ssh` creado con permisos correctos (700).
- Clave pública provista por la cátedra copiada a:

/root/.ssh/authorized_keys

- Permisos de authorized_keys ajustados a **600**.
- Archivo `/etc/ssh/sshd_config` configurado para permitir:

PermitRootLogin yes
PubkeyAuthentication yes

- Conectividad verificada desde Windows mediante:

ssh -i clave_privada.txt root@192.168.56.10

(Coincide con la evidencia del PDF.)

---

### Servidor Web (Apache + PHP)

- Apache2 instalado desde repositorios oficiales de Debian.
- PHP 7.4 y módulos necesarios instalados correctamente (`php7.4-mysql`, `php7.4-xml`, `php7.4-mbstring`, etc.).
- Directorio `/www_dir` creado y configurado como raíz del sitio.
- Archivos requeridos por la cátedra copiados a `/www_dir`:
- `index.php`
- `logo.png`
- `db.sql`
- VirtualHost específico creado en:

/etc/apache2/sites-available/tp.conf

- `000-default.conf` fue deshabilitado y reemplazado por `tp.conf`:

a2dissite 000-default.conf
a2ensite tp.conf
systemctl reload apache2

- Verificación realizada desde la máquina física mediante:

http://192.168.56.10


---

### Base de Datos (MariaDB)

- MariaDB instalado mediante:

apt install mariadb-server -y

- Servicio verificado activo.
- Base de datos del TP importada directamente desde:


mysql -u root < /www_dir/db.sql


- No se crearon usuarios adicionales. La importación se realizó con el usuario root, tal como figura en el PDF.
- El sitio PHP conectó correctamente al motor MariaDB, verificándose visualización de datos.

---

## 3. Configuración de Red

- Adaptador NAT + Adaptador Host-Only configurados en VirtualBox.
- Interfaz utilizada para la IP estática: **enp0s3**
- Configuración persistente en `/etc/network/interfaces`:


auto enp0s3
iface enp0s3 inet static
address 192.168.56.10
netmask 255.255.255.0
gateway 192.168.56.1

- Verificación realizada con:


ip a
ping 192.168.56.1


---

## 4. Almacenamiento

- Se añadió un disco adicional de **10 GB**.
- Creación de dos particiones estándar tipo 83:
- `/dev/sdc1` – 3 GB → montada en **/www_dir**
- `/dev/sdc2` – 6 GB → montada en **/backup_dir**
- Formateo con ext4:


mkfs.ext4 /dev/sdc1
mkfs.ext4 /dev/sdc2

- Montaje permanente en `/etc/fstab`:


/dev/sdc1 /www_dir ext4 defaults 0 0
/dev/sdc2 /backup_dir ext4 defaults 0 0

- Archivo requerido por la consigna:
- Debido a que `/proc/partitions` es efímero, se creó el directorio persistente `/proc_dir`:
  ```
  mkdir /proc_dir
  cat /proc/partitions > /proc_dir/particion
  ```

---

## 5. Script de Backup

**Ubicación:** `/opt/scripts/backup_full.sh`

### Funcionalidad:
- Recibe **ORIGEN** y **DESTINO** como parámetros.
- Verifica que ambos existan.
- Comprueba que DESTINO sea un punto de montaje válido.
- Incluye opción de ayuda con `-help`.
- Genera archivos comprimidos `.tar.gz` con fecha en formato ANSI: `YYYYMMDD`.
- Ejemplo de uso:


/opt/scripts/backup_full.sh /var/log /backup_dir

- Ejemplo de archivo generado:


log_bkp_20251114.tar.gz


---

## 6. Tareas automáticas (cron)

Configuradas mediante `crontab -e`:



Backup diario de /var/log

0 0 * * * /opt/scripts/backup_full.sh /var/log /backup_dir

Lunes, miércoles y viernes – backup de /www_dir

0 23 * * 1,3,5 /opt/scripts/backup_full.sh /www_dir /backup_dir


- Verificación mediante `crontab -l` confirmada.

---

## 7. Archivos comprimidos para entrega

Todos los directorios solicitados fueron comprimidos individualmente en formato `.tar.gz`.

Archivos generados (según evidencia del PDF):

| Archivo | Contenido |
|---------|----------|
| `root.tar.gz` | /root |
| `etc.tar.gz` | /etc |
| `opt.tar.gz` | /opt |
| `proc_dir.tar.gz` | /proc_dir (incluye particion) |
| `www_dir.tar.gz` | /www_dir |
| `backup_dir.tar.gz` | /backup_dir |
| `var.tar.gz.part-aa`, `var.tar.gz.part-ab`, `var.tar.gz.part-ac` | /var dividido en partes |
| `clave_publica.pub` | Clave SSH provista |
| `clave_privada.txt` | Clave privada |

---

## 8. Diagrama topológico

Se incluye el archivo correspondiente en el repositorio:

![Diagrama Topológico](./Infografia_Diagrama_Topologica.jpg)

> **Figura 1.** Diagrama topológico del entorno implementado (Apache, PHP, MariaDB, SSH y backups automáticos).

---

## 9. Estado final

- Todos los servicios funcionan correctamente.
- El servidor es accesible vía SSH mediante clave privada.
- El sitio web funciona correctamente y se conecta a MariaDB.
- Las particiones adicionales están montadas y operativas.
- El sistema de backups funciona manual y automáticamente.
- Todos los archivos requeridos fueron comprimidos y preparados para GitHub.
- El proyecto está completamente documentado según la consigna.

---

Universidad de Palermo – Computación Aplicada (2025)  
Grupo: Bonomi, Feruglio, Valle, Sánchez, Rozo Cruz, Frutos


