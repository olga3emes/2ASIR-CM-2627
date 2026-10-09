# Ejercicios de Administración de Sistemas Debian 13

## Nivel Básico

### Ejercicio 1. Identificar el sistema operativo
- Mostrar la versión de Debian.
- Mostrar información del kernel.
  
  
## Ejercicio 1
```bash
cat /etc/os-release
uname -r
uname -a
hotsnamectl
```


### Ejercicio 2. Navegar por el sistema de archivos
- Mostrar el directorio actual.
- Ir a `/etc`.
- Volver al directorio personal.

## Ejercicio 2
```bash
pwd
cd /etc
cd ~
```

### Ejercicio 3. Listar archivos
- Mostrar archivos del directorio actual.
- Mostrar también los ocultos.
- Ver permisos y tamaños.


## Ejercicio 3
```bash
ls
ls -a
ls -lah
```
### Ejercicio 4. Crear directorios
Crear la estructura:

```text
empresa
├── usuarios
├── documentos
└── copias
```

## Ejercicio 4
```bash
mkdir -p empresa/{usuarios,documentos,copias}
```


### Ejercicio 5. Crear archivos
Crear tres archivos vacíos.
## Ejercicio 5
```bash
touch archivo1.txt archivo2.txt archivo3.txt
```

### Ejercicio 6. Copiar y mover archivos
- Copiar `archivo1.txt` a `/tmp`.
- Renombrarlo como `copia.txt`.

```bash
cp archivo1.txt /tmp/copia.txt
```

### Ejercicio 7. Buscar archivos
Buscar todos los archivos `.conf` en `/etc`.


```bash
find /etc -name "*.conf"
```

### Ejercicio 8. Ver contenido de archivos
Mostrar el contenido del archivo de usuarios.

```bash
cat /etc/passwd
less /etc/passwd
```

### Ejercicio 9. Comprobar espacio en disco
Ver uso de discos y particiones.
```bash
df -h
```

### Ejercicio 10. Mostrar memoria RAM.
```bash
free -h
```
## Nivel Intermedio

### Ejercicio 11. Gestión de usuarios
Crear un usuario llamado `alumno`.
### Gestión de usuarios
```bash
sudo useradd -m alumno
sudo passwd alumno`
sudo groupadd informatica
sudo usermod -aG informatica alumno
groups alumno
``` 
-m crea el directorio home del usuario.
-s asigna el shell por defecto.

### Ejercicio 12. Gestión de grupos
Crear el grupo `informatica` y añadir al usuario.

### Ejercicio 13. Consultar grupos
Ver a qué grupos pertenece un usuario.

### Ejercicio 14. Modificar permisos
Configurar permisos 640 para un fichero.

### Ejercicio 15. Cambiar propietario
Asignar propietario y grupo.
```bash
chmod 640 fichero.txt
sudo chown alumno:informatica fichero.txt
```
### Ejercicio 16. Monitorización de procesos.
```bash
ps aux
top
htop (hay que instalarlo previamente)
kill PID
kill -9 PID
pkill -9 nombre_proceso
*** y ojito que no pregunta y mata a todos los procesos con ese nombre.***
```

### Ejercicio 17. Finalizar procesos.

### Ejercicio 18. Gestión de servicios.

```bash
systemctl status ssh
sudo systemctl start ssh
sudo systemctl enable ssh
```
stop y disable para parar y deshabilitar servicios.

### Ejercicio 19. Ver puertos abiertos.
```bash
ip addr
ss -lntup
ip route
ping 8.8.8.8 -c numero_de_paquetes
dig google.com
```
```bash
journalctl
journalctl -p err
journalctl -u ssh
```
### Ejercicio 21. Ver configuración IP.
### Ejercicio 22. Consultar rutas.
### Ejercicio 23. Probar conectividad.
### Ejercicio 24. Comprobar DNS.
### Ejercicio 25. Consultar logs.



### Ejercicio 20. Instalar Apache.


## Nivel Avanzado

### Ejercicio 26. Crear una tarea programada.
### Ejercicio 27. Comprimir directorios.
### Ejercicio 28. Ver discos y particiones.



### Ejercicio 29. Montar sistemas de archivos.
### Ejercicio 30. Crear RAID software.
### Ejercicio 31. Configurar firewall UFW.
### Ejercicio 32. Analizar conexiones activas.
### Ejercicio 33. Buscar procesos que consumen más memoria.
### Ejercicio 34. Crear copias de seguridad automáticas.
### Ejercicio 35. Auditar usuarios conectados.
### Ejercicio 36. Analizar accesos SSH.
### Ejercicio 37. Crear claves SSH.

### Ejercicio 38. Gestión básica de Docker.

### Ejercicio 39. Crear un servicio systemd.
### Ejercicio 40. Diagnóstico completo del sistema.




## Reto Final
Realizar una auditoría básica completa de un servidor Debian verificando sistema, disco, RAM, usuarios, servicios, red, logs y conectividad.

### Gestión de usuarios
```bash
sudo useradd -m alumno
sudo passwd alumno
sudo groupadd informatica
sudo usermod -aG informatica alumno
groups alumno
```
Permiten crear usuarios, establecer contraseñas y gestionar grupos.

### Permisos
```bash
chmod 640 fichero.txt
sudo chown alumno:informatica fichero.txt
```
Controlan el acceso a archivos.

### Procesos
```bash
ps aux
top
htop
kill PID
kill -9 PID
```
Sirven para monitorizar y finalizar procesos.

### Servicios
```bash
systemctl status ssh
sudo systemctl start ssh
sudo systemctl enable ssh
```
Gestionan servicios del sistema.

### Red
```bash
ip addr
ip route
ping 8.8.8.8
dig google.com
ss -lntup
```
Permiten verificar direccionamiento, rutas, DNS y puertos.

### Logs
```bash
journalctl
journalctl -p err
journalctl -u ssh
```
Facilitan el diagnóstico de problemas.

### Tareas programadas
```bash
crontab -e
* * * * * date >> /tmp/fechas.log
```
Ejecutan tareas automáticas.

### Copias de seguridad
```bash
tar -czvf copia.tar.gz carpeta/
```
Comprime directorios para respaldos.

### Discos
```bash
lsblk
sudo fdisk -l
mount /dev/sdb1 /mnt/datos
```
Permiten administrar almacenamiento.

### Seguridad
```bash
sudo apt install ufw
sudo ufw allow ssh
sudo ufw enable
```
Configuran el cortafuegos.

### Docker
```bash
docker ps
docker exec -it nginx bash
docker attach nginx
```
Gestionan contenedores.

### Systemd
```bash
sudo systemctl daemon-reload
sudo systemctl enable prueba.service
```
Permiten crear servicios personalizados.

### Diagnóstico final
```bash
hostnamectl
free -h
df -h
who
systemctl --type=service --state=running
ip addr
ss -lntup
journalctl -xe
ps aux --sort=-%cpu | head
```
Ofrecen una visión general del estado del servidor.
