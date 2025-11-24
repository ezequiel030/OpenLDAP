# Comprobaciones y Verificación Local en el Servidor

Antes de pasar a configurar los equipos clientes, debemos asegurarnos de que el servidor funciona y, además, configurarlo para que sea capaz de ver sus propios usuarios LDAP.

### 1. Verificaciones del Servicio LDAP
Comprobamos que la base de datos y los esquemas se han cargado correctamente en el backend.

* **Verificar la configuración de la BBDD:**
    ```bash
    # Comprobamos que el dominio (suffix) sea correcto
    sudo cat /etc/ldap/slapd.d/cn=config/olcDatabase={1}mdb.ldif
    ```
    *Busca la línea `olcSuffix: <BASE_DN>` (ej: dc=megainfo209,dc=com).*

* **Verificar ficheros de datos:**
    ```bash
    ls -l /var/lib/ldap
    ```
    *Debe existir el archivo `data.mdb`.*

* **Verificar esquemas cargados:**
    ```bash
    ls /etc/ldap/schema
    ```
    *Deben aparecer `inetorgperson.schema`, `nis.schema`, etc.*

---

### 2. Habilitar Comprobación de Usuarios (Configurar Servidor como Cliente)
Por defecto, el servidor tiene los usuarios en su base de datos, pero el sistema operativo (Debian) no los "ve". Para poder hacer comprobaciones con comandos como `getent` o `su`, debemos instalar las librerías de cliente **en el propio servidor**.

#### A. Instalación de paquetes:
```bash
sudo apt install libnss-ldap libpam-ldap nscd
 ```
- [ ] URI: ldap://localhost (o tu IP local 127.0.0.1).

- [ ] Base DN: <BASE_DN> (Ej: dc=megainfo209,dc=com).

- [ ] Versión: 3.

- [ ] Root local: Sí.

- [ ] Login requerido: No.

#### B. Configuración de NSS (Name Service Switch): Editamos /etc/nsswitch.conf para que el servidor busque usuarios en su propio LDAP.
```bash
sudo nano /etc/nsswitch.conf
```
Añade ldap al final de estas líneas:
```bash
passwd:         files systemd ldap
group:          files systemd ldap
shadow:         files ldap
```
#### C. Reiniciar el servicio de nombres:
```bash
sudo /etc/init.d/nscd restart
```
### 3. Prueba Final de Visibilidad (Local)
Ahora que el servidor actúa también como cliente de sí mismo, verificamos que el usuario <USUARIO> (ej. juan) es visible para el sistema.

Comprobar usuario:
```bash
getent passwd <USUARIO>
```
✅ Éxito: Debe devolver algo como: <USUARIO>:x:2000:2000:Nombre...:/home/<USUARIO>:/bin/bash

Comprobar grupo:
```bash
getent group <NOMBRE_GRUPO>
```
✅ Éxito: Debe devolver: <NOMBRE_GRUPO>:*:2000:

Si estas pruebas funcionan, tu servidor está perfectamente configurado y listo para recibir conexiones de clientes externos.

<p align="center">
  <a href="Cliente.md">⬅️ Anterior</a> |
  <a href="README.md">🏠 Índice</a> 
</p>
