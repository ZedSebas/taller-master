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

## Instalación y configuración de equipos virtuales

Se instalan 3 servidores virtuales:

 - Taller Bastion **Servidor de Controlador**
 - Taller Rocky **Servidor Web y de Aplicación**
 - Taller Ubuntu **Servidor de Base de Datos**

[![Captura-Bastion005.png](https://i.postimg.cc/d13Rkh1J/Captura-Bastion005.png)](https://postimg.cc/Vry02Lnh)

[![Captura-Ubuntu005.png](https://i.postimg.cc/DzHQG2bs/Captura-Ubuntu005.png)](https://postimg.cc/mc3FfWj2)

## Particiones

[![Captura-Rocky011.png](https://i.postimg.cc/m2fdqSkM/Captura-Rocky011.png)](https://postimg.cc/JtxcHZb4)

[![Captura-Ubuntu015.png](https://i.postimg.cc/g0X8mfB0/Captura-Ubuntu015.png)](https://postimg.cc/bGPsRFPX)

## Redes

[![Captura-Servers002.png](https://i.postimg.cc/mr5jSYKP/Captura-Servers002.png)](https://postimg.cc/nsqqFDFx)

[![Captura-Servers003.png](https://i.postimg.cc/Tw59NyXC/Captura-Servers003.png)](https://postimg.cc/gXd8n2jZ)

[![Captura-Servers004.png](https://i.postimg.cc/KjX56ZGF/Captura-Servers004.png)](https://postimg.cc/8JHWWgfn)

Utilizando nmtui en Rocky y netplan en Ubuntu se verifican y/o establecen ips en las distintas tarjetas de red, en Ubuntu también se habilita el firewall.

> #sudo netplan generate

> #sudo apt install nano

> #sudo nano /etc/netplan/00-installer-config.yaml

network: <br />
 version: 2 <br />
 renderer: networkd <br />
 ethernets: <br />
  enp0s3: <br />
   dhcp4: yes <br />
  enp0s8: <br />
   dhcp4: no <br />
   dhcp6: no <br />
   addresses: [192.168.10.12/24, ] <br />
   gateway4: 192.168.10.1 <br />
   nameservers: <br />
           addresses: [8.8.8.8, 8.8.8.4] <br />

> #sudo netplan apply

> #ip a

---------

> #sudo apt-get update <br />
> #sudo apt-get upgrade <br />
> #sudo apt ufw <br />
> #sudo ufw status <br />
> #sudo ufw default allow outgoing <br />
> #sudo ufw default deny incoming <br />
> #sudo ufw allow ssh <br />
> #sudo ufw enable <br />
> #sudo ufw status <br />

## Usuario para ejecución de tareas

Se crearon usuarios ansible de clave 'ansible01' en los tres servidores 
Se les otorgaron permisos de SUDO

Rocky <br />
> #usermod -aG wheel ansible

Ubuntu <br />
> #sudo usermod -aG sudo ansible

Verificación 
> #su - ansible <br />
> #sudo ls -la /root <br />

## Instalación de Git, Ansible y establecimiento de clave pública/privada

> #dnf install git -y <br />
> #dnf install epel-release -y <br />
> #dnf install ansible -y <br />

> #ssh-keygen <br />
> #ssh-copy-id 192.168.10.4 <br />
> #ssh-copy-id 192.168.10.12 <br />

[![Captura-SSH001.png](https://i.postimg.cc/280Zvzs6/Captura-SSH001.png)](https://postimg.cc/gn6jbPkF)

> #eval $(ssh-agent) <br />
> #ssh-add <br />

Verificación
> #ansible -i 192.168.10.4, all -m ping <br />
> #ansible -i 192.168.10.12, all -m ping <br />

en /home/ansible
> #ansible-config init --disabled > ansible.cfg <br />

modificar ansible.cfg con 'vi' cambiar ;inventory=/etc/ansible/hosts por inventory=./inventario-taller 
para administrar separadamente un inventario en el que se pueden generar grupos
> #vi inventario-taller <br />

 [ubuntu] <br />

 tallerubuntu ansible_host=192.168.10.12 <br />
 
 [rocky] <br />

 tallerrocky ansible_host=192.168.10.4 <br />
 
 [linux:children] <br />
	
 ubuntu <br />
	
 rocky <br />

## Distribución de claves entre los servidores del grupo linux usando ansible

> #ansible linux -m authorized_key -a "user=ansible state=present key={{ lookup('file', '/home/ansible/.ssh/id_rsa.pub') }}"

[![Captura-SSH010.png](https://i.postimg.cc/qBLhVJML/Captura-SSH010.png)](https://postimg.cc/hhXPVqmz)

## Configuración de Github

> #git init <br />
> #git config --global user.name "Sebastián Díaz" <br />
> #git config --global user.email "ZedSebas@gmail.com" <br />
> #git config --global core.editor vi <br />
> #git remote add origin git@github.com:ZedSebas/taller-master.git <br />

## Se crea SSH Key en repositorio de Github para permitir comunicación segura.

[![Captura-SSH014.png](https://i.postimg.cc/gJJ7y27c/Captura-SSH014.png)](https://postimg.cc/9461CCVs)

Verificación
> #git status -s <br />
> #git push origin master <br />

## Creación de repositorios para roles de tareas requeridas

> #ansible-galaxy init application <br />
> #ansible-galaxy init webserver <br />
> #ansible-galaxy init database <br />
> #ansible-galaxy init firewall <br />
