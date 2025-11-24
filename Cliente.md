# Configuración del Cliente

Antes de instalar los paquetes de LDAP, debemos preparar la red del cliente para que se comunique correctamente con el servidor.

### 1. Configuración de Red Estática
Editamos el fichero de interfaces para asignar una IP fija al cliente.

**Fichero:** `/etc/network/interfaces`
```bash
# Configuración de la interfaz de red (comprueba si tu interfaz es 'enp0s3' o 'eth0')
auto enp0s3
iface enp0s3 inet static
    address 192.168.1.30      # <--- CAMBIAR por la IP que asignarás al CLIENTE
    netmask 255.255.255.0
    gateway 192.168.1.1       # <--- CAMBIAR por la puerta de enlace de tu red
```
Nota: Guarda los cambios y reinicia el servicio de red con:
```bash
systemctl restart networking.service 
```

### 2. Configuración del Hostname
Asignamos el nombre completo (FQDN) a la máquina cliente.

Fichero: /etc/hostname
```bash
# Formato: nombre_maquina.dominio
cliente209.megainfo209.com
```
### 3. Configuración de DNS
Editamos el archivo de resolución de nombres para asegurar que tenemos salida a internet y resolución de dominios.

Fichero: /etc/resolv.conf
```bash
nameserver 8.8.8.8
nameserver 8.8.4.4
```
### 4. Resolución de Nombres Local (Hosts)
Es crítico mapear la IP del servidor LDAP en el archivo local para que el cliente pueda encontrarlo por su nombre de dominio.

Fichero: /etc/hosts
```bash
127.0.0.1       localhost
127.0.1.1       cliente209.megainfo209.com  cliente209

# AÑADIR la línea del servidor LDAP:
192.168.1.29    servidor209.megainfo209.com  servidor209   # <--- IMPORTANTE: IP y Nombre de tu servidor LDAP
```
Verificación: Haz un ping al servidor para confirmar conectividad:
```bash
ping -c 2 servidor209.megainfo209.com 
```
- También comprobamos con nmap (instalamos si hace falta apt install nmap).
```bash
nmap -p 389 servidor209.megainfo209.com
```
### 5. Instalación de Paquetes de Cliente
Una vez configurada la red, instalamos las librerías necesarias para la autenticación remota.
```bash
sudo apt update
sudo apt install libnss-ldap libpam-ldap nscd
```
- [ ] Durante la instalación aparecerá un asistente (pantallas azules). Rellena con los datos de tu servidor:

- [ ] URI del servidor LDAP: ldap://servidor209.megainfo209.com (Usa el nombre que definiste en /etc/hosts).

- [ ] Base de búsqueda (DN): dc=megainfo209,dc=com (Tu base DN).

- [ ] Versión de LDAP: 3

- [ ] Hacer que root local sea administrador de la base de datos: Sí.

- [ ] ¿La base de datos requiere inicio de sesión?: No.

- Configuramos los valores por defecto: /etc/ldap/ldap.conf
```bash
BASE  dc=megainfo209,dc=com
URI  ldap://servidor209.megainfo209.com
```

- Paramos el servicio:
```bash
/etc/init.d/nscd stop
```
### 6. Configuración NSS (Name Service Switch)
Modificamos este fichero para indicar al sistema que busque usuarios en LDAP si no los encuentra en local.

Fichero: /etc/nsswitch.conf Busca las líneas correspondientes y añade ldap (o systemd ldap) al final:
```bash
passwd:         files systemd ldap
group:          files systemd ldap
shadow:         files ldap
gshadow         files ldap
hosts:          files dns
networks:       files ldap
```
- Reiniciamos el servicio:
```bash
/etc/init.d/nscd restart
```
### 7. Configuración PAM (Creación Automática de Home)
Configuramos PAM para que cree el directorio /home/usuario automáticamente la primera vez que un usuario de LDAP inicie sesión.

Fichero: /etc/pam.d/common-session Añade la siguiente línea al final del archivo:
```bash
session required    pam_mkhomedir.so 
```
### 8. Reiniciar Servicios
Para aplicar los cambios en el caché de nombres y autenticación:
```bash
sudo /etc/init.d/nscd restart
```
#### ✅ Verificación Final
Desde el cliente, realiza las siguientes pruebas:

Comprobar visibilidad del usuario LDAP:
```bash
getent passwd juan
```
Debe devolver una línea con los datos de 'juan' (uid 2000, etc).

Iniciar sesión como usuario LDAP:
```bash
su - juan
```
Debe pedir contraseña. Al entrar, deberías ver que se crea el directorio: Creating directory '/home/juan'.

<p align="center">
  <a href="Servidor.md">⬅️ Anterior</a> |
  <a href="README.md">🏠 Índice</a> |
  <a href="Comprobaciones_Servidor.md">➡️ Siguiente</a>
</p>
