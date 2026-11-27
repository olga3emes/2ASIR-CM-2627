# Práctica: Instalación y configuración de un servidor Samba en Debian 13

## Objetivo

Instalar y configurar un servidor Samba en Debian 13 para compartir archivos en la red local y acceder a ellos desde equipos Windows, Linux o macOS. Basado en la guía Debian fileserver: cómo configurarlo paso a paso con Debian 13 

---

### 1. Comprobar la conectividad de red

Antes de comenzar, verifica que la máquina Debian tiene conexión a la red.

Mostrar la dirección IP:

```bash
ip a
```
Comprobar conectividad:

```bash
ping 8.8.8.8 -c 4
```
Mostrar el nombre del equipo:
```bash
hostname
```

### 2. Actualizar el sistema

Actualizar la lista de paquetes:

```bash
sudo apt update
```

Actualizar los paquetes instalados:

```bash
sudo apt upgrade -y
```

### 3. Instalar Samba

Instalar Samba y las herramientas necesarias:

```bash
sudo apt install samba smbclient samba-common-bin -y
```

Comprobar que Samba se ha instalado correctamente:

```bash
smbd --version
```
Deberá mostrarse la versión instalada.

Samba implementa el protocolo SMB/CIFS y permite compartir archivos entre Linux, Windows y macOS. 



### 4. Crear la carpeta compartida

Crear el directorio que se compartirá en la red:

```bash
sudo mkdir -p /srv/fileshare
```

Comprobar que existe:

```bash
ls -ld /srv/fileshare
```


### 5. Crear un usuario para acceder al recurso compartido

Crear un usuario local:

```bash
sudo useradd -m fileserveruser
```

Asignarle una contraseña:

```bash
sudo passwd fileserveruser
```


### 6. Asignar permisos a la carpeta

Hacer propietario al nuevo usuario:

```bash
sudo chown -R fileserveruser:fileserveruser /srv/fileshare
```

Asignar permisos:

```bash
sudo chmod -R 770 /srv/fileshare
```

Los permisos 770 permiten acceso completo al propietario y al grupo, impidiendo el acceso al resto de usuarios del sistema. 


### 7. Realizar copia de seguridad de la configuración

Crear una copia del archivo original:

```bash
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bak
```


### 8. Configurar Samba

Editar el fichero de configuración:

```bash
sudo nano /etc/samba/smb.conf
```

Añadir al final del archivo:

```ini
[Fileshare]
path = /srv/fileshare
browseable = yes
writable = yes
valid users = fileserveruser
create mask = 0660
directory mask = 0770
```

Verificar también que en la sección global aparece:

```ini
[global]
workgroup = WORKGROUP
```

Guardar los cambios.

Esta configuración crea un recurso compartido llamado **Fileshare** y limita el acceso al usuario configurado.


### 9. Crear el usuario de Samba

Añadir el usuario a la base de datos de Samba:

```bash
sudo smbpasswd -a fileserveruser
```

Activar la cuenta:

```bash
sudo smbpasswd -e fileserveruser
```

Samba utiliza su propia base de datos de usuarios independiente de las cuentas Linux. 



### 10. Comprobar la configuración

Validar el archivo de configuración (a nivel de sintaxis):

```bash
testparm
```

Si aparecen errores, corregirlos antes de continuar.


### 11. Reiniciar y habilitar los servicios

Reiniciar Samba:

```bash
sudo systemctl restart smbd nmbd
```

Configurar el inicio automático:

```bash
sudo systemctl enable smbd nmbd
```

Comprobar el estado:

```bash
sudo systemctl status smbd
```

Debería mostrarse:

```text
active (running)
```


### 12. Abrir el cortafuegos (si existe)

Si utilizas UFW:

```bash
sudo ufw allow samba
```

O bien:

```bash
sudo ufw allow 445/tcp
sudo ufw allow 139/tcp
sudo ufw allow 137/udp
sudo ufw allow 138/udp
```

Samba utiliza principalmente el puerto TCP 445 para conexiones SMB modernas. 【1-a9e315】



### 13. Probar el recurso compartido localmente

Listar los recursos compartidos del servidor:

```bash
smbclient -L localhost -U fileserveruser
```

Acceder al recurso:

```bash
smbclient //localhost/Fileshare -U fileserveruser
```



### 14. Acceder desde Windows 11

Abrir el Explorador de archivos.

En la barra de direcciones introducir:

```text
\\IP_DEBIAN\Fileshare
```

Ejemplo:

```text
\\192.168.1.100\Fileshare
```

Introducir:

- Usuario: `fileserveruser`
- Contraseña: la definida con `smbpasswd`



### 15. Comprobación final

Crear un archivo desde Windows:

```text
prueba.txt
```

Comprobar en Debian:

```bash
ls -l /srv/fileshare
```

Si el archivo aparece, la configuración es correcta.



### Resultado esperado

Al finalizar la práctica tendrás:

- Servidor Debian 13 actualizado

- Samba instalado

- Carpeta compartida `/srv/fileshare`

- Usuario autenticado mediante Samba

- Acceso desde Windows 11 mediante SMB

- Servicio configurado para arrancar automáticamente tras reiniciar el sistema

