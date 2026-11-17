# Práctica: Instalar un servidor FTP en Debian 13 y probarlo desde Windows 11

## Objetivo

Instalar y configurar un servidor FTP en una máquina virtual con Debian 13 y comprobar su funcionamiento desde un equipo con Windows 11.

---

## 1. Instalar el servidor FTP (vsftpd)

Actualizar el sistema:

```bash
sudo apt update
sudo apt upgrade -y
```

Instalar vsftpd:

```bash
sudo apt install vsftpd -y
```

Comprobar que el servicio está activo:

```bash
sudo systemctl status vsftpd
```

Si no está iniciado:

```bash
sudo systemctl enable --now vsftpd
```

---

## 2. Crear un usuario para FTP

Crear un usuario específico:

```bash
sudo adduser alumnoftp
```

Crear una carpeta compartida:

```bash
sudo mkdir -p /home/alumnoftp/ftp
```

Crear un archivo de prueba:

```bash
echo "Servidor FTP funcionando" > /home/alumnoftp/ftp/prueba.txt
```

Asignar permisos:

```bash
sudo chown -R alumnoftp:alumnoftp /home/alumnoftp/ftp
```

---

## 3. Configurar vsftpd

Realizar una copia de seguridad:

```bash
sudo cp /etc/vsftpd.conf /etc/vsftpd.conf.bak
```

Editar la configuración:

```bash
sudo nano /etc/vsftpd.conf
```

Contenido recomendado:

```ini
anonymous_enable=NO
local_enable=YES
write_enable=YES

local_umask=022

dirmessage_enable=YES
use_localtime=YES

xferlog_enable=YES

connect_from_port_20=YES

chroot_local_user=YES
allow_writeable_chroot=YES

listen=NO
listen_ipv6=YES
```

Reiniciar el servicio:

```bash
sudo systemctl restart vsftpd
```

---

## 4. Abrir el cortafuegos

Si se utiliza UFW:

```bash
sudo ufw allow 21/tcp
sudo ufw reload
```

Comprobar reglas:

```bash
sudo ufw status
```

---

## 5. Obtener la IP de Debian

```bash
hostname -I
```

Ejemplo:

```text
192.168.1.150
```

---

## 6. Comprobar conectividad desde Windows 11

Abrir CMD:

```cmd
ping 192.168.1.150
```

Debe responder correctamente.

---

## 7. Acceder desde Windows 11

### Opción A: Explorador de archivos

Introducir en la barra de direcciones:

```text
ftp://192.168.1.150
```

Credenciales:

- Usuario: alumnoftp
- Contraseña: la configurada durante la creación del usuario.

Debería mostrarse el archivo:

```text
prueba.txt
```

### Opción B: Cliente FTP por consola

```cmd
ftp 192.168.1.150
```

Comandos útiles:

```text
dir
get prueba.txt
put archivo.txt
bye
```

---

## 8. Comprobar subida de archivos

Crear un archivo en Windows llamado:

```text
saludo.txt
```

Subirlo:

```text
put saludo.txt
```

Verificar en Debian:

```bash
ls -l /home/alumnoftp/ftp
```

Resultado esperado:

```text
prueba.txt
saludo.txt
```

---

## 9. Verificaciones finales

Comprobar estado del servicio:

```bash
sudo systemctl status vsftpd
```

Comprobar que escucha en el puerto 21:

```bash
sudo ss -tlnp | grep :21
```

Salida esperada:

```text
LISTEN 0 32 *:21 *:*
```

---

## Resultado esperado

- El servidor FTP está en funcionamiento.

- Windows 11 puede conectarse mediante FTP.

- Es posible descargar archivos del servidor.

- Es posible subir archivos al servidor.

- Los archivos transferidos aparecen en `/home/alumnoftp/ftp`.
