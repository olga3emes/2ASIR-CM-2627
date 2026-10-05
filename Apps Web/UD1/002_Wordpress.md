# Instalación de WordPress sobre LAMP en Debian 13

## 🧩 1. Preparar el entorno
Actualiza el sistema y verifica los servicios:
```bash
sudo apt update
sudo apt upgrade
sudo systemctl status apache2
sudo systemctl status mysql
```

## 🗄️ 2. Crear base de datos y usuario para WordPress
Entra en MySQL como root:
```bash
sudo mysql -u root -p
```
Crea la base de datos y el usuario:
```sql
CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'contraseña_segura';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

## 🌐 3. Instalar dependencias necesarias
Instala extensiones PHP necesarias para WordPress:
```bash
sudo apt install php-mysql php-gd php-xml php-mbstring php-curl php-zip php-intl unzip
sudo systemctl restart apache2
```

## 📦 4. Descargar WordPress
Descarga y descomprime WordPress:
```bash
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
```

## 📁 5. Copiar WordPress al directorio web
Copia los archivos al directorio de Apache:
```bash
sudo mkdir -p /var/www/html/wordpress
sudo cp -r /tmp/wordpress/* /var/www/html/wordpress/
sudo chown -R www-data:www-data /var/www/html/wordpress
sudo find /var/www/html/wordpress -type d -exec chmod 755 {} \;
sudo find /var/www/html/wordpress -type f -exec chmod 644 {} \;
```

## ⚙️ 6. Configurar WordPress
Copia el archivo de configuración:
```bash
cd /var/www/html/wordpress
sudo cp wp-config-sample.php wp-config.php
sudo nano wp-config.php
```
Edita las líneas correspondientes:
```php
define( 'DB_NAME', 'wordpress' );
define( 'DB_USER', 'wpuser' );
define( 'DB_PASSWORD', 'contraseña_segura' );
define( 'DB_HOST', 'localhost' );
```
Genera claves de seguridad en:
[https://api.wordpress.org/secret-key/1.1/salt/](https://api.wordpress.org/secret-key/1.1/salt/)


## 🌍 7. Configurar Apache para WordPress
Crea un archivo de configuración:
```bash
sudo nano /etc/apache2/sites-available/wordpress.conf
```
Ejemplo de configuración:
```apache
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html/wordpress
    ServerName midominio.local

    <Directory /var/www/html/wordpress/>
        AllowOverride All
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/wordpress_error.log
    CustomLog ${APACHE_LOG_DIR}/wordpress_access.log combined
</VirtualHost>
```
Activa el sitio y el módulo `rewrite`:
```bash
sudo a2ensite wordpress.conf
sudo a2enmod rewrite
sudo systemctl reload apache2
```

Guía de uso para `modulo rewrite`: [web](https://www.hostinger.com/es/tutoriales/como-configurar-mod_rewrite)

Ver log de errores de Apache en tiempo real

```
sudo tail -f /var/log/apache2/error.log
```

## 🚀 8. Instalar WordPress desde el navegador
Abre tu navegador y visita:
```
http://localhost/wordpress
```
Sigue el asistente de instalación (idioma, nombre del sitio, usuario, contraseña, correo).

## 🔒 9. Activar HTTPS con Let’s Encrypt (opcional)
Instala y configura un certificado SSL gratuito:
```bash
sudo apt install certbot python3-certbot-apache
sudo certbot --apache
```

✅ ¡Listo! Tu entorno **LAMP + WordPress** estará funcionando correctamente en Debian 13.
