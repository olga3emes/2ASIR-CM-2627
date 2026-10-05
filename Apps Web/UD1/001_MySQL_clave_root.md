# 🔐 Recuperar la contraseña `root` de MySQL 8.4 en Debian 13

Guía paso a paso para restablecer la contraseña del usuario `root` en MySQL 8.4 bajo Debian 13 (Trixie).



## 🧩 Paso 1: Parar el servicio MySQL
Detén el servidor de forma limpia:

```bash
sudo systemctl stop mysql
```



## 🧩 Paso 2: Arrancar MySQL sin control de permisos
Ejecuta este comando:

```bash
sudo mysqld_safe --skip-grant-tables --skip-networking &
```

> Esto arranca MySQL en modo seguro y sin verificar contraseñas, **solo accesible localmente**.



## 🧩 Paso 3: Conectarte como root sin contraseña
En una nueva terminal:

```bash
mysql -u root
```

Si entras correctamente, verás el prompt de MySQL (`mysql>`).


## 🧩 Paso 4: Refrescar privilegios y restablecer la contraseña
Ejecuta los siguientes comandos dentro del cliente MySQL:

```sql
FLUSH PRIVILEGES;

ALTER USER 'root'@'localhost' 
IDENTIFIED WITH caching_sha2_password 
BY 'NuevaPass123!';

FLUSH PRIVILEGES;
EXIT;
```

> 🔸 `caching_sha2_password` es el plugin por defecto en MySQL 8.4.  
> 🔸 Sustituye `NuevaPass123!` por tu nueva contraseña segura.



## 🧩 Paso 5: Detener el modo seguro y reiniciar el servicio normal
Detén los procesos de MySQL en modo seguro y arráncalo normalmente:

```bash
sudo killall mysqld mysqld_safe
sudo systemctl start mysql
```

Verifica que está activo:

```bash
sudo systemctl status mysql
```



## 🧩 Paso 6: Probar el acceso normal
Inicia sesión con la nueva contraseña:

```bash
mysql -u root -p
```

Introduce la nueva clave y deberías acceder correctamente.



## 💡 (Opcional) Cambiar autenticación a `auth_socket`
Si prefieres que el usuario `root` pueda acceder directamente desde `sudo mysql` sin contraseña:

```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH auth_socket;
FLUSH PRIVILEGES;
```

Entonces podrás acceder con:

```bash
sudo mysql
```