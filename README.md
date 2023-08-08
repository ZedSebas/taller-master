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
> #usermod -aG wheel ansible <br />
> #visudo <br />
  ansible ALL=(ALL) NOPASSWD: ALL <br />

Ubuntu <br />
> #sudo usermod -aG sudo ansible <br />
> #visudo <br />
  ansible ALL=(ALL:ALL) NOPASSWD: ALL <br />

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

## Playbooks

> #mkdir /home/ansible/playbooks <br />
> #vi tallerlab.yml <br />

[![Captura-SSH015.png](https://i.postimg.cc/NGJBPYKZ/Captura-SSH015.png)](https://postimg.cc/5jCh6hqm)

Playbook iniciador el cual lista los roles que serán ejecutados en los miembros del grupo linux

## Roles

# Application

en lo que se aprende automatización de podman, generación de imagen y creación de instancia de la misma se realiza en bastión configuración manual donde disponibilizar la aplicación para el caso de no llegar a tiempo

```sh
> #loadkeys es
> #dnf update -y

////if dnf error with packages bucle
//# dnf clean all
//# dnf check
//# dnf check-update
//# dnf update -y
------------ 

//Preparar interfaces de red 
> #nmcli

//establecer metodo, ipv4 y deshabilitar ipv6
> #nmcli connection modify enp0s8 ipv6.method disabled

> #systemctl restart NetworkManager
------------

> #dnf install samba samba-common samba-client -y

> #mv /etc/samba/smb.conf /etc/samba/smb.conf.old

> #mkdir -p /srv/samba/repo

> #chmod -R 775 /srv/samba/repo
> #chown -R nobody:nobody /srv/samba/repo
> #chcon -t samba_share_t /srv/samba/repo

------------
> #vi /etc/samba/smb.conf
	[Global]
	server string = Samba Server $v
	netbios name = rocky linux 9.2
	security = user
	map to guest = bad user
	dns proxy = no
	
	[Public]
	path = /srv/samba/repo
	browsable = yes
	writable = yes
	guest ok = yes
	read only = no
------------
// verificar
> #testparm

------------
//colocar las interfaces en la zona correcta
> #firewall-cmd --zone=public --add-service=samba --permanent
> #firewall-cmd --zone=internal --add-interface=enp0s3 --permanent
> #firewall-cmd --reload

> #systemctl enable smb --now
> #systemctl status smb

> #systemctl enable nmb --now
> #systemctl status nmb

------------
//Se descargan a compartido el repositorio de ejemplo y todo.war proporcionado
	https://github.com/emverdes/docker-tomcat-tutorial

//Acceso mediante tarjeta de red temporal para compartir
	\\192.168.0.253\Public

//Destino de compartido 
	/srv/samba/repo/

//Instalación de paquete unzip para archivos comprimidos
> #dnf install unzip -y

//Se descomprime
> #unzip /srv/samba/repo/docker-tomcat-tutorial-master.zip -d /

//Se modifica nombre de la carpeta extraida
> #mv docker-tomcat-tutorial-master docker-container

//Se intercambia sample.war por todo.war
> #mv /srv/samba/repo/todo.war /docker-container/todo.war
> #mv /docker-container/sample.war /srv/samba/repo/sample.war

//Se edita Dockerfile basado en el contenido de https://dlcdn.apache.org/tomcat/ 
> #vi /docker-container/Dockerfile
	//se modifican referencias a v8.5.88 por existente v8.5.91
	//sample.war por todo.war
	//puerto de exposición 8080 por 8008

------------
//Información de versión disponible 
> #dnf info podman

> #dnf install podman

//Consulta de intancias de contenedores activos
> #podman ps -a

> #cd docker-container

//Se construye imagen de nombre "todo" para Docker de acuerdo a las instrucciones de Dockerfile
> #podman build -f Dockerfile -t todo:1

//Se crea una intancia del contenedor todo de nommbre web1 la cual será accesible mediante el puerto 8008
> #podman run -it --rm -d -p 8008:8080 --name web1 todo:1

//Se puede crear una segunda intancia del contenedor todo de nommbre web2 accesible mediante el puerto 8009
> #podman run -it --rm -d -p 8009:8080 --name web2 todo:1

//Detener 
> #podman stop web2

//Limpiar resultado de ps -a de procesos finalizados 
> #podman rm $(podman ps --filter "status=exited" -q)

//Se prueba la web en http://192.168.10.40:8008/todo/

------------
//Se crea carpeta config y archivo de configuración 

> #cd /opt
> #mkdir config
> #vi app.properties
	tipoDB=mysql
	jdbcURL=jdbc:mysql://192.168.10.12:3306/todo
	jdbcUsername=todo
	jdbcPassword=ZSe4RFvP84
```

[![Captura-Ansible-004.png](https://i.postimg.cc/bNnQMXzh/Captura-Ansible-004.png)](https://postimg.cc/ykVggbgQ)

[![Captura-Ansible-005.png](https://i.postimg.cc/Gm1knM2s/Captura-Ansible-005.png)](https://postimg.cc/XpQZwk9V)

[![Captura-Ansible-006.png](https://i.postimg.cc/bwH1yQDV/Captura-Ansible-006.png)](https://postimg.cc/VJvJ40FX)

# WebServer

dentro del rol webserver se crea usando 'vi' en /files el archivo tomcat.service

**/webserver/files/tomcat.service**
```sh
[Unit]
Description=Tomcat 8.5.91 servlet container para Rocky Linux 9.2
After=network.target

[Service]
User=tomcat
Group=tomcat
WorkingDirectory=/opt/tomcat
Environment=JRE_HOME=/opt/jdk
Environment=JAVA_HOME=/opt/jdk
Environment=CATALINA_BASE=/opt/tomcat
Environment=CATALINA_HOME=/opt/tomcat
ExecStart=/opt/tomcat/bin/catalina.sh run
ExecStop=/opt/tomcat/bin/shutdown.sh

[Install]
WantedBy=multi-user.target
```

usando 'vi' en /tasks se edita el archivo main.yml prestando atención a la disponibilidad de la versión de java y apache-tomcat 

**/webserver/tasks/main.yml**
```sh
---
- name: Actualizacion de Paquetes Ubuntu
  apt:
   upgrade: yes
   update_cache: yes
  when: ansible_distribution == "Ubuntu"

- name: Actualizacion de Paquetes Rocky
  dnf:
   name: "*"
   state: latest
  when: ansible_distribution == "Rocky"
  
- name: Instalar tar para descomprimir
  ansible.builtin.yum:
    name: tar
    state: latest
  when: ansible_distribution == "Rocky"

- name: Crear directorio JDK 1.8
file:
path: /opt/jdk
recurse: yes

- name: Descargar y descomprimir java 1.8
  unarchive:
    src: https://builds.openlogic.com/downloadJDK/openlogic-openjdk/8u332-b09/openlogic-openjdk-8u332-b09-linux-x64.tar.gz
    dest: /opt/jdk
    remote_src: yes
    extra_opts: [--strip-components=1]
  
- name: Crear usuario Tomcat
  user:
   name: tomcat

- name: Crear grupo Tomcat
  group:
   name: tomcat
  
- name: Crear Tomcat Directorio
  file:
   path: /opt/tomcat
   owner: tomcat
   group: tomcat
   mode: 755
   recurse: yes
  
- name: Descargar Tomcat
  unarchive:
  src: https://dlcdn.apache.org/tomcat/tomcat-8/v8.5.91/bin/apache-tomcat-8.5.91.tar.gz
  dest: /opt/tomcat
  remote_src: yes
  extra_opts: [--strip-components=1]
  
- name: Cambiar permisos de directorio Tomcat
  file:
   path: /opt/tomcat
   owner: tomcat
   group: tomcat
   mode: "u+rwx,g+rx,o=rx"
   recurse: yes
   state: directory
  
- name: Copiar archivo de servicio Tomcat
  copy:
   src: /home/ansible/webserver/files/tomcat.service
   dest: /etc/systemd/system/
   mode: 0755

- name: Iniciar y habilitar el servicio Tomcat
  systemd:
    name: tomcat
    state: started
    daemon_reload: true
```

# DataBase

dentro del rol database se crean usando 'vi' en /files los archivos app.properties y tablas.sql

**/database/files/app.properties**
```sh
tipoDB=mariadb
jdbcURL=jdbc:mariadb://localhost:3306/todo
jdbcUsername=todo
jdbcPassword=ZSe4RFvP84
```

**/database/files/tablas.sql**
```sh
CREATE TABLE `users` (
  `id` int(3) NOT NULL AUTO_INCREMENT,
  `first_name` varchar(20) DEFAULT NULL,
  `last_name` varchar(20) DEFAULT NULL,
  `username` varchar(250) DEFAULT NULL,
  `password` varchar(20) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

CREATE TABLE `todos` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `description` varchar(255) DEFAULT NULL,
  `is_done` bit(1) NOT NULL,
  `target_date` datetime(6) DEFAULT NULL,
  `username` varchar(255) DEFAULT NULL,
  `title` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=8 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```

usando 'vi' en /vars se edita el archivo main.yml

**/database/vars/main.yml**
```sh
---
mysql_root_password: "Pass.123"
mariadb_socket: /run/mysqld/mysqld.sock
database: "todo"
priviliges: "ALL"
db_user_name: "todo"
db_user_password: "ZSe4RFvP84"
create_database: True
create_db_user: True
import_sql_file: True
deny_remote_connections: True
```

usando 'vi' en /tasks se edita el archivo main.yml

**/database/tasks/main.yml**
```sh
---
- name: Instalar Mariadb en Rocky
  package:
    name: mariadb-server, python3-PyMySQL
    state: latest
  when: ansible_distribution == "Rocky"

- name: Instalar Mariadb en Ubuntu
  apt:
    name: mariadb-server, python3-pymysql
    state: latest
  when: ansible_distribution == "Ubuntu"

- name: Iniciar y ejecutar Mariadb
  service:
    name: mariadb
    enabled: true
    state: started

- name: Establecer contraseña root para Rocky
  mysql_user:
    login_user: root
    login_password: "{{ mysql_root_password }}"
    user: root
    check_implicit_admin: true
    password: "{{ mysql_root_password }}"
    host: localhost
  #notify: Flush Priviliges
  when: ansible_distribution == "Rocky"

- name: Establecer contrasena root Para Ubuntu
  mysql_user:
    name: root
    password: "{{ mysql_root_password }}"
    host: "{{ item }}"
    login_user: root
    login_unix_socket: "{{ mariadb_socket }}"
    state: present
  with_items:
    - 127.0.0.1
    - localhost
  #notify: Flush Priviliges
  when: ansible_distribution == "Ubuntu"

- name: Copiar archivo de conexion
  copy:
    src: /home/ansible/database/files/app.properties
    dest: /opt/config/
    mode: 0755

- name: Creacion de base de datos
  mysql_db:
    name: "{{ database }}"
    login_user: root
    login_password: "{{ mysql_root_password }}"
    state: present
  when:
    - create_database
    - database is defined

- name: Crear usuario para base de datos
  mysql_user:
    name: "{{ db_user_name }}"
    password: "{{ db_user_password }}"
    priv: "{{ database }}.*:{{ priviliges }}"
    login_user: root
    login_password: "{{ mysql_root_password }}"
    state: present
  when:
    - create_database
    - database is defined
    - create_db_user
    - db_user_name is defined

- name: Copiar archivo SQL
  copy:
    src: /home/ansible/database/files/tablas.sql
    dest: /tmp/
  when: import_sql_file

- name: Ejecutar scipt SQL
  mysql_db:
    name: "{{ database }}"
    state: import
    target: /tmp/tablas.sql
    login_user: root
    login_password: "{{ mysql_root_password }}"
  when:
    - database is defined and create_database
    - import_sql_file

- name: No permitir conexiones remotas con root
  mysql_user:
    check_implicit_admin: true
    login_user: root
    login_password: "{{ mysql_root_password }}"
    user: root
    host: localhost
    state: absent
```

# Firewall

dentro del rol firewall se edita usando 'vi' en /tasks el archivo main.yml

**/firewall/tasks/main.yml**
```sh
---
- name: Firewalld reglas tomcat http port
  firewalld:
    port: 8080/tcp
    permanent: true
    state: enabled
    immediate: yes
  when: ansible_distribution == "Rocky"
   
- name: Firewalld reglas tomcat https port
  firewalld:
    port: 8443/tcp
    permanent: true
    state: enabled
    immediate: yes
  when: ansible_distribution == "Rocky"

- name: Firewalld reglas app tcp todo port
  firewalld:
    port: 3306/tcp
    permanent: true
    state: enabled
    immediate: yes
  when: ansible_distribution == "Rocky"
    
- name: Firewalld reglas app udp todo port
  firewalld:
    port: 3306/udp
    permanent: true
    state: enabled
    immediate: yes
  when: ansible_distribution == "Rocky"
```

## Ejecución de playbook tallerlab

> #ansible-playbook tallerlab.yml -i /home/ansible/inventario-taller 

## Aplicación 

Verificación

[![Captura-Ansible-001.png](https://i.postimg.cc/cJChxwrS/Captura-Ansible-001.png)](https://postimg.cc/CZykJZGP)

[![Captura-Ansible-002.png](https://i.postimg.cc/7ZmXrmP9/Captura-Ansible-002.png)](https://postimg.cc/yD30FhZ3)

[![Captura-Ansible-003.png](https://i.postimg.cc/zfQjVvgR/Captura-Ansible-003.png)](https://postimg.cc/nsv7WHTH)


