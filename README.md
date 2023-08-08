# Obligatorio Taller de Servidores Linux 2023
**Integrantes**

- Germán Charbonnier N°285059
- Sebastián Díaz N° 155833

**GitHub** [https://github.com/ZedSebas/taller-master](https://github.com/ZedSebas/taller-master)

# Letra

** Requisitos A **

Instalar un servidor denominado bastión que contará con Ansible, y usuario extra con permisos SUDO, más clave pública para conexiones SSH.
Instalar dos servidores con 14GB de Disco distribuidos de acuerdo al siguiente,

Esquema de particiones: 

 - 1GB /boot 
 - 6GB / 
 - 4GB /var 
 - 2GB swap

Un servidor debe tener el SO. Rocky otro Ubuntu.

Deben contar con 2 interfaces de red, 1 conectada a NAT y la otra a una red Interna que permita interconección con el Ansible del equipo bastión.
El usuario agregado de nombre ansible, contará con permisos SUDO sin contraseña. 
Desde el equipo bastión, se copia clave pública para conectarse al servidor sin contraseña.

** Tareas B **

Tareas a realizar mediante playbook con Ansible:

   1) Instalación de la aplicación Tomcat usada en el obligatorio de Administración de Servidores, dentro de un container en el servidor Rocky.
   2) Configuración de un servidor web apache como proxy reverso en el servidor Rocky.
   3) Instalación de una base de datos mariadb en el servidor Ubuntu.
   4) Firewall activo habilitando los accesos necesarios en ambos servidores.
   5) El estado de los servicios necesarios deben de estar iniciados y habilitados.
 - El playbook debe ser válido tanto para Rocky como para Ubuntu.
 - El playbook y todos los archivos necesarios deben ser subidos a un repositorio git llamado “tallerjulio2023”.

# Cumplimiento
