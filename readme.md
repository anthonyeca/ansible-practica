<a name="top"></a>

# Entorno para practicar Ansible

## Índice de Contenidos

- [Configuración NAT](#item1)
- [Configuración de firewall(ufw)](#item2)
- [Configuración de la clave SSH](#item3)
- [Configuración de Ansible](#item4)

<a name="item1"></a>

## Configuramos nuestros servidores, tendremos 1 servidor Master y 1 Slave | La configuración de red sera una NAT

**Configuración del nodo master**

![Master ifconfig](/img/master_inconfig.png 'Master ifconfig.')

**Configuración del nodo Slave**

![Slave ifconfig](/img/slave_inconfig.png 'Slave ifconfig.')
[Subir](#top)
<a name="item2"></a>

## Configuración de un firewall básico

Instalaremos ufw, el cual usaremos para que solo se permitan las conexiones a ciertos servicios.

```
sudo apt install ufw
```

![Master ufw install](/img/master_ufw_install.png 'Master ufw install.')

Podemos listar las aplicaciones con el comando:

```
sudo ufw app list
```

![Master ufw list](/img/master_ufw_list.png 'Master ufw list.')

Tenemos que estar seguro que el firewall permita conexiones ssh para que podamos logearnos la siguiente vez, para ellos usamos el comando:

```
ufw allow OpenSSH
ufw enable
```

y comprobamos el status:

```
ufw status
```

![Master ufw openssh](/img/master_ufw_openssh_status.png 'Master ufw openssh.')

El servidor actualmente está bloqueando todas las conexiones excepto ssh.
[Subir](#top)
<a name="item3"></a>

## Configuración de la clave SSH

### Paso 1 - Crear la clave ssh

El primer paso es crear la clave ssh

```
ssh-keygen
```

```
Output
Generating public/private rsa key pair.
Enter file in which to save the key (/your_home/.ssh/id_rsa):
```

la contraseña es opcional pero se recomienda introducirla para añadir una capa más de seguridad

![Master keygen](/img/master_keygen.png 'Master keygen.')

Ya tendremos la clave pública y privada para autenticarnos, el siguiente paso es colocar la clave publica en el servidor el cual usaremos SSH-key-based para autenticar el login.

### Paso 2 - Copiar la clave ssh al servidor en el cual usaremos el login

La forma mas sencilla de copiar la clave es con el comando ssh-copy, esa herramienta está incluida por defecto en muchos sistemas operativos, por lo tanto, quizás la tengas disponible en tu sistema.

```
ssh-copy-id username@remote_host
```

![Master ssh-copy-id](/img/master_ssh-copy-id.png 'Master ssh-copy-id.')

Probablemente, al ser la primera vez no reconocerá al equipo local y debes decir "yes" para poder copiarla, además de escribir la contraseña de la cuenta remota.

### Paso 3 - Autenticarnos al servidor

Probamos la conexión al servidor slave

´´´
ssh usuario@servidor
´´´

![Master sshToSlave](/img/master_sshToSlave.png 'Master SSHtoSlave')

### Paso 4 - Deshabilitar la autenticación vía password

Una vez hecho esto , y probado que nos conectamos correctamente, debemos deshabilitar la autenticación por password en el servidor, esto se hace en el fichero /etc/ssh/sshd_config

´´´
. . .
PasswordAuthentication no
. . .
´´´

Y debemos reiniciar el servicio:
´´´
sudo systemctl restart ssh
´´´

![Slave PasswordNo](/img/slave_passwordNo.png 'Slave PasswordNo.')
[Subir](#top)
<a name="item4"></a>

## Configuración de Ansible

### Paso 1 - Instalación de Ansible

##### Debemos instalar ansible en la maquina que sera nuestro Ansible Control Node.

_**Ansible Control Node**_
_Este será el servidor que vamos a usar para controlar el resto de hosts mediante SSH._

_**Ansible Host Node**_
_Este será el servidor que nuestro nodo controlador esta configurado para automatizar._

Para instalar ansible debemos ejecutar el siguiente comando el cual nos permite añadir el "personal package archive" en nuestro sistema:

```
sudo apt-add-repository ppa:ansible/ansible
```

Seguidamente debemos actualizar los paquetes e instalar ansible

```
sudo apt update
sudo apt install ansible
```

Ahora nuestro nodo de control tiene todo el software requerido para administrar nuestro host(slave).

## Paso 2 - Preparando el fichero "Inventario"

Este fichero contiene información sobre los hosts que vamos a administrar con Ansible.

```
sudo vi /etc/ansible/hosts
```

Vamos a definir el grupo [servers] el cual contendrá nuestro host con el alias de server1

![Master Ansible hosts](/img/master_ansible_hosts.png 'Master Ansible hosts.')

Para chequear el inventario ejecutamos el comando:

```
ansible-inventory --list -y
```

![Master Ansible inventory](/img/master_ansible_inventory.png 'Master Ansible inventory.')

## Paso 3 - Testear la conexión

Desde nuestro Nodo de control podemos ejecutar el comando:

```
ansible all -m ping -u "usuario"
```

Este comando usara el módulo de ping para realizar una conectividad a todos los nodos en nuestro fichero inventario con el usuario que especifiquemos, el módulo de ping va a testear:

    - Si el host es accesible;
    - Si nuestra credencial SSH es válida;
    - Si el host tiene la posibilidad de ejecutar modulos de Ansible usando Python

![Master Ansible ping](/img/master_ansible_ping.png 'Master Ansible ping.')

## Paso 4 - Ejecutar comando Ad-Hoc

Una vez que hemos confirmado que nuestro nodo de control puede comunicarse con nuestro host(s), podemos empezar a ejecutar comandos Ad-hoc y playbooks en nuestro servidor.

```
ansible all -a "df -h" -u root
```

![Master Ansible df](/img/master_ansible_df.png 'Master Ansible df.')

También podemos utilizar comandos para un target especifico (en el caso de que nuestro inventario este confirme por varios servidores)

```
ansible servers -a "uptime" -u root
```

# Conclusión

Esta guía nos ha servido para instalar Ansible, configurar un inventario y ejecutar comandos desde un nodo de control (Ansible Control Node). Una vez confirmes que eres capaz de conectar y controlar tu infraestructura desde un Ansible Control node, puedes ejecutar cualquier comando o playbook que decidas en esos hosts.

[Subir](#top)
