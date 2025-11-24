# OpenLDAP

![OpenLDAP](img/Gemini_Generated_Image_fntafwfntafwfnta.png)

# Despliegue y Configuración de OpenLDAP (Servidor y Cliente) en Debian

Este repositorio documenta el proceso paso a paso para la instalación, configuración y despliegue de un servidor de directorio **OpenLDAP** y la conexión de un cliente Linux al mismo para la autenticación centralizada de usuarios.

## 📋 Escenario de Red

Se han utilizado dos máquinas virtuales con **Debian 13**:
<p align = "center">
| Rol | Hostname | IP | Dominio |
| :--- | :--- | :--- | :--- |
| **Servidor** | `servidor209` | `192.168.1.29` | `megainfo209.com` |
| **Cliente** | `cliente209` | `192.168.1.30` | `megainfo209.com` |
</p>

ÍNDICE:

[1. Instalación y configuración en el servidor](Servidor.md)

[2. Configuración del cliente](Cliente.md)

[3. Comprobaciones que hacer en el servidor después de configurar el servidor y el cliente](Comprobaciones_Servidor.md)
