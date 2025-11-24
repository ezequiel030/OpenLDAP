# Servidor LDAP

## 1. Configuración de Red del Servidor
Es fundamental fijar una IP estática y definir correctamente el nombre del host (FQDN).

**Fichero:** `/etc/network/interfaces`
```bash
# Configuración de la interfaz de red (adaptar nombre de interfaz 'enp0s3')
auto enp0s3
iface enp0s3 inet static
    address 192.168.1.29      # <--- CAMBIAR por la IP de tu servidor
    netmask 255.255.255.0
    gateway 192.168.1.1       # <--- CAMBIAR por tu puerta de enlace
```
- también el fichero /etc/resolv.conf:
```bash
nameserver 8.8.8.8
nameserver 8.8.4.4
```
- Reiniciamos el servicio de la red
```bash
systemctl restart networking.service
```
#### Fichero: /etc/hosts Asociamos la IP al nombre de dominio completo.
```bash
127.0.0.1       localhost
127.0.1.1       servidor209.megainfo209.com  servidor209
192.168.1.29    servidor209.megainfo209.com  servidor209  # <--- Agrega tu IP y tu FQDN aquí
```
- Antes de continuar:
```bash
apt update
apt upgrade
```
## 2. Instalación del Servicio
Utiliza sudo sino estás como root
```bash
sudo apt update && sudo apt upgrade
sudo apt install slapd ldap-utils
```
Durante la instalación se pedirá una contraseña para el administrador. Guárdala bien.

3. Configuración del Cliente Local
Editamos este archivo para que el servidor sepa dónde buscarse a sí mismo por defecto.

Fichero: /etc/ldap/ldap.conf
```bash
# Ajusta la base (BASE) a tu dominio: dc=tu_nombre,dc=com
BASE    dc=megainfo209,dc=com
# Ajusta la URI apuntando a tu servidor
URI     ldap://localhost ldap://servidor209.megainfo209.com
```
4. Estructura del Directorio (LDIFs)
Creamos los archivos .ldif necesarios para definir la estructura de la organización.

A. Estructura Base (base.ldif)
Define las ramas para usuarios (people) y grupos (group).
```bash
# Cambia 'dc=megainfo209,dc=com' por tu base DN en todo el archivo
dn: ou=people,dc=megainfo209,dc=com
ou: people
objectClass: top
objectClass: organizationalUnit

dn: ou=group,dc=megainfo209,dc=com
ou: group
objectClass: top
objectClass: organizationalUnit
```
Comando para insertar:
```bash
# Recuerda cambiar el DN 'cn=admin...' según tu dominio
ldapadd -x -W -D "cn=admin,dc=megainfo209,dc=com" -f base.ldif
```
B. Grupos (sistemas.ldif)
```bash
# Cambia el DN según tu dominio
dn: cn=sistemas,ou=group,dc=megainfo209,dc=com
objectClass: posixGroup
objectClass: top
cn: sistemas
gidNumber: 2000  # <--- Puedes cambiar el GID si te piden otro número
```
Comando para insertar:
```bash
ldapadd -x -W -D "cn=admin,dc=megainfo209,dc=com" -f sistemas.ldif
```
C. Usuarios (juan.ldif)
Para la contraseña, genérala antes con: slappasswd -h {MD5}.
Para la imagen de juan usamos el comando: base64 -w 0 imagen_juan(nombre de la imagen)
```bash
dn: uid=juan,ou=people,dc=megainfo209,dc=com
objectClass: top
objectClass: posixAccount
objectClass: inetOrgPerson
objectClass: shadowAccount
uid: juan
sn: fernandez
givenName: juan
cn: juan fernandez
uidNumber: 2000       # <--- Asigna un UID único
gidNumber: 2000       # <--- Debe coincidir con el GID del grupo creado antes
userPassword: {MD5}1X0U4hqI0QEQRN3Ie8MCTQ==  # <--- Pega aquí tu contraseña generada
homeDirectory: /home/juan
loginShell: /bin/bash
mail: juan@gmail.com
jpegPhoto:: # <----- Pega lo generado por el comando base64 
```
Comando para insertar:
```bash
ldapadd -x -W -D "cn=admin,dc=megainfo209,dc=com" -f juan.ldif
```
Creamos el directorio del usuario nuevo y sus permisos:
```bash
mkdir -p /home/juan
cp -r /etc/skel/.* /home/juan/
chown -R 2000:2000 /home/juan # <--- El 2000 hace referencia al uidNumber y el siguiente separado por : al gidNumber
```
- También vemos todo lo que se copió dentro de /home/juan
```bash
ls -al
# Deberiamos ver los archivos que empiezan por .
```
