# Instalación y configuración de BIND 9 en Debian GNU/Linux

> Guía completa, reorganizada y ampliada a partir del documento original. Incluye fundamentos, instalación, estructura de configuración, seguridad, registros, zonas DNS, comprobaciones y ejemplos paso por paso.

## Índice

- [Instalación y configuración de BIND 9 en Debian GNU/Linux](#instalación-y-configuración-de-bind-9-en-debian-gnulinux)
  - [Índice](#índice)
  - [1. Introducción](#1-introducción)
  - [2. Requisitos y escenario de ejemplo](#2-requisitos-y-escenario-de-ejemplo)
  - [3. Instalación de BIND 9](#3-instalación-de-bind-9)
    - [Paso 1. Actualizar el índice de paquetes](#paso-1-actualizar-el-índice-de-paquetes)
    - [Paso 2. Instalar el servidor y las herramientas DNS](#paso-2-instalar-el-servidor-y-las-herramientas-dns)
  - [4. Comprobación del servicio](#4-comprobación-del-servicio)
  - [5. Prueba del servidor DNS caché](#5-prueba-del-servidor-dns-caché)
  - [6. Configuración del cliente DNS](#6-configuración-del-cliente-dns)
  - [7. Estructura de archivos de BIND](#7-estructura-de-archivos-de-bind)
- [Hemos hecho hasta aquí en clase, el resto son apuntes de Olga, que ha dejado de guía para ella y las clases, pero que no vamos a ver.](#hemos-hecho-hasta-aquí-en-clase-el-resto-son-apuntes-de-olga-que-ha-dejado-de-guía-para-ella-y-las-clases-pero-que-no-vamos-a-ver)
  - [8. Contenido técnico del documento original](#8-contenido-técnico-del-documento-original)
  - [db.root](#dbroot)
  - [db.local](#dblocal)
  - [Consideraciones generales](#consideraciones-generales)
  - [Instrucción include](#instrucción-include)
  - [Cláusula acl](#cláusula-acl)
  - [Cláusula logging](#cláusula-logging)
  - [Cláusula options](#cláusula-options)
    - [Estamento directory](#estamento-directory)
    - [Estamento version](#estamento-version)
    - [Estamento listen-on](#estamento-listen-on)
    - [Estamento recursion](#estamento-recursion)
    - [Estamento allow-recursion](#estamento-allow-recursion)
    - [Estamento allow-transfer](#estamento-allow-transfer)
  - [Cláusula zone](#cláusula-zone)
    - [Estamento type](#estamento-type)
    - [Estamento file](#estamento-file)
    - [Estamento masterfile-format](#estamento-masterfile-format)
    - [Estamento masters](#estamento-masters)
  - [named-checkconf](#named-checkconf)
  - [named-checkzone](#named-checkzone)
  - [15. Ejemplo completo paso por paso](#15-ejemplo-completo-paso-por-paso)
    - [Paso 1. Definir una ACL para la red local](#paso-1-definir-una-acl-para-la-red-local)
    - [Paso 2. Declarar las zonas](#paso-2-declarar-las-zonas)
    - [Paso 3. Crear el archivo de zona directa](#paso-3-crear-el-archivo-de-zona-directa)
    - [Paso 4. Crear el archivo de zona inversa](#paso-4-crear-el-archivo-de-zona-inversa)
    - [Paso 5. Ajustar propietarios y permisos](#paso-5-ajustar-propietarios-y-permisos)
    - [Paso 6. Comprobar la sintaxis](#paso-6-comprobar-la-sintaxis)
    - [Paso 7. Recargar BIND](#paso-7-recargar-bind)
    - [Paso 8. Probar la zona directa](#paso-8-probar-la-zona-directa)
    - [Paso 9. Probar la zona inversa](#paso-9-probar-la-zona-inversa)
  - [16. Pruebas y diagnóstico](#16-pruebas-y-diagnóstico)
    - [Consultar un registro concreto](#consultar-un-registro-concreto)
    - [Mostrar la respuesta completa](#mostrar-la-respuesta-completa)
    - [Consultar el registro SOA](#consultar-el-registro-soa)
    - [Consultar los servidores de nombres](#consultar-los-servidores-de-nombres)
    - [Vaciar la caché de BIND](#vaciar-la-caché-de-bind)
    - [Verificar el estado mediante RNDC](#verificar-el-estado-mediante-rndc)
    - [Consultar los registros del servicio](#consultar-los-registros-del-servicio)
    - [Errores frecuentes](#errores-frecuentes)
  - [17. Buenas prácticas de seguridad](#17-buenas-prácticas-de-seguridad)
  - [18. Resumen de comandos](#18-resumen-de-comandos)
  - [Conclusión](#conclusión)

---

## 1. Introducción

**BIND 9** es una implementación de DNS mantenida por Internet Systems Consortium. El proceso servidor se denomina `named` y atiende, normalmente, en el puerto **53/UDP** para la mayoría de consultas y en **53/TCP** cuando la respuesta lo requiere o se realizan transferencias de zona.

Esta guía emplea Debian o una distribución derivada. Los comandos administrativos se muestran con `sudo`. Si se trabaja como `root`, puede omitirse.

## 2. Requisitos y escenario de ejemplo

Para los ejemplos prácticos se utilizará esta red de laboratorio:

| Elemento | Valor |
|---|---|
| Red | `192.168.1.0/24` |
| Servidor BIND | `192.168.1.10` |
| Puerta de enlace | `192.168.1.1` |
| Dominio de laboratorio | `asir.local` |
| Nombre del servidor DNS | `ns1.asir.local` |
| Equipo cliente | `192.168.1.20` |

> En un entorno real conviene usar un subdominio propio. El sufijo `.local` puede entrar en conflicto con mDNS, por lo que aquí se utiliza únicamente con finalidad didáctica.

## 3. Instalación de BIND 9

### Paso 1. Actualizar el índice de paquetes

```bash
sudo apt update
```

### Paso 2. Instalar el servidor y las herramientas DNS

```bash
sudo apt install bind9 bind9-utils dnsutils
```

- `bind9`: instala el servidor DNS.
- `bind9-utils`: incorpora herramientas administrativas como `named-checkconf` y `named-checkzone`.
- `dnsutils`: proporciona utilidades de consulta como `dig` y `nslookup`.

## 4. Comprobación del servicio

```bash
sudo systemctl status bind9
```

Según la versión de Debian, la unidad también puede aparecer como `named`:

```bash
sudo systemctl status named
```

Comandos habituales:

```bash
sudo systemctl start bind9
sudo systemctl stop bind9
sudo systemctl restart bind9
sudo systemctl reload bind9
sudo systemctl enable bind9
```

Para confirmar que el proceso escucha en el puerto 53:

```bash
sudo ss -lntup | grep ':53'
```

## 5. Prueba del servidor DNS caché

Realizamos una consulta indicando expresamente que debe responder el servidor local:

```bash
dig @127.0.0.1 www.example.com
```

Repetimos la consulta:

```bash
dig @127.0.0.1 www.example.com
```

En la segunda ejecución el campo `Query time` suele disminuir porque la respuesta ya se encuentra en caché. Los campos más relevantes de `dig` son:

- `status`: resultado de la consulta, por ejemplo `NOERROR` o `NXDOMAIN`.
- `ANSWER SECTION`: respuesta obtenida.
- `SERVER`: servidor que respondió.
- `Query time`: tiempo empleado.
- `flags`: características de la respuesta. `aa` indica respuesta autoritativa y `ra` que la recursión está disponible.

## 6. Configuración del cliente DNS

Para una prueba temporal puede consultarse directamente con `dig @IP`. No es recomendable sobrescribir `/etc/resolv.conf` sin comprobar qué servicio lo administra, ya que NetworkManager, `systemd-resolved` o DHCP pueden regenerarlo.

Consulta del contenido actual:

```bash
cat /etc/resolv.conf
```

Ejemplo conceptual de configuración:

```text
nameserver 127.0.0.1
```

Después puede comprobarse la resolución:

```bash
getent hosts www.example.com
ping -c 3 www.example.com
```

> `ping` prueba conectividad además de resolución. Para diagnosticar DNS es preferible `dig`, porque muestra con mayor precisión la respuesta del servidor.

## 7. Estructura de archivos de BIND

Los archivos principales suelen estar en `/etc/bind`:

```text
/etc/bind/
├── named.conf
├── named.conf.options
├── named.conf.local
├── named.conf.default-zones
├── db.local
├── db.127
├── db.0
├── db.255
└── zones.rfc1918
```

- `named.conf`: archivo principal. Incluye otros archivos.
- `named.conf.options`: opciones globales.
- `named.conf.local`: declaración de zonas propias.
- `named.conf.default-zones`: zonas predeterminadas.
- `db.*`: archivos de zona, al principio no están.

El archivo principal suele contener:

```conf
include "/etc/bind/named.conf.options";
include "/etc/bind/named.conf.local";
include "/etc/bind/named.conf.default-zones";
```

---

# Hemos hecho hasta aquí en clase, el resto son apuntes de Olga, que ha dejado de guía para ella y las clases, pero que no vamos a ver.



## 8. Contenido técnico del documento original

El servidor DNS más utilizado actualmente es [**BIND**](https://www.isc.org/downloads/bind/) (*Berkeley Internet Name Domain*) de [**ISC**](https://www.isc.org/) (*Internet Systems Consortium*), que fue totalmente reescrito para la última versión estable oficial, la ***versión 9***. La instalación en *Debian GNU/Linux* es análoga a la de cualquier otro paquete, en este caso el paquete ***bind9***:

\# aptitude install bind9

La orden instalará BIND con una configuración por defecto que hace que funcione como un servidor DNS caché. Además el servicio se arranca automáticamente, lo cual podemos comprobar con el siguiente comando:

\# service bind9 status  
● bind9.service - BIND Domain Name Server  
   Loaded: loaded (/lib/systemd/system/bind9.service; enabled)  
  Drop-In: /run/systemd/generator/bind9.service.d  
           └─50-insserv.conf-\$named.conf  
   Active: active (running) since lun 2016-05-29 18:34:08 CEST; 6min ago  
     Docs: man:named(8)  
 Main PID: 590 (named)  
   CGroup: /system.slice/bind9.service  
           └─590 /usr/sbin/named -f -u bind

El demonio del servicio DNS que acabamos de instalar es el programa ***/usr/sbin/named*** y el script que utilizaremos para controlarlo es ***/etc/init.d/bind9***, al cual también podremos llamar con el comando ***service*** como hemos hecho anteriormente:

```bash
\# /etc/init.d/bind9  
\[info\] Usage: /etc/init.d/bind9 .

Podemos comprobar, que efectivamente funciona, obligándole a hacer una traducción de un nombre de dominio, para lo cual usamos el comando [dig](http://fpg.66ghz.com/DebianRed/dig.html) de la siguiente manera:

\# dig @localhost www.facebook.com  
  
; \<\<\>\> DiG 9.9.5-9+deb8u6-Debian \<\<\>\> @localhost www.facebook.com  
; (2 servers found)  
;; global options: +cmd  
;; Got answer:  
;; -\>\>HEADER\<\<- opcode: QUERY, status: NOERROR, id: 5364  
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 2, ADDITIONAL: 5  
  
;; OPT PSEUDOSECTION:  
; EDNS: version: 0, flags:; udp: 4096  
;; QUESTION SECTION:  
;www.facebook.com.        IN    A  
  
;; ANSWER SECTION:  
www.facebook.com.    3600    IN    CNAME    star-mini.c10r.facebook.com.  
star-mini.c10r.facebook.com. 60    IN    A    173.252.89.132  
  
;; AUTHORITY SECTION:  
c10r.facebook.com.    3600    IN    NS    a.ns.c10r.facebook.com.  
c10r.facebook.com.    3600    IN    NS    b.ns.c10r.facebook.com.  
  
;; ADDITIONAL SECTION:  
a.ns.c10r.facebook.com.    3600    IN    A    69.171.239.11  
a.ns.c10r.facebook.com.    3600    IN    AAAA    2a03:2880:fffe:b:face:b00c:0:99  
b.ns.c10r.facebook.com.    3600    IN    A    69.171.255.11  
b.ns.c10r.facebook.com.    3600    IN    AAAA    2a03:2880:ffff:b:face:b00c:0:99  
  
;; Query time: 231 msec  
;; SERVER: ::1#53(::1)  
;; WHEN: Mon May 29 19:01:53 CEST 2016  
;; MSG SIZE  rcvd: 213  
```
```bash
\# dig @localhost www.facebook.com  
  
; \<\<\>\> DiG 9.9.5-9+deb8u6-Debian \<\<\>\> @localhost www.facebook.com  
; (2 servers found)  
;; global options: +cmd  
;; Got answer:  
;; -\>\>HEADER\<\<- opcode: QUERY, status: NOERROR, id: 34237  
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 2, ADDITIONAL: 5  
  
;; OPT PSEUDOSECTION:  
; EDNS: version: 0, flags:; udp: 4096  
;; QUESTION SECTION:  
;www.facebook.com.        IN    A  
  
;; ANSWER SECTION:  
www.facebook.com.    3591    IN    CNAME    star-mini.c10r.facebook.com.  
star-mini.c10r.facebook.com. 51    IN    A    173.252.89.132  
  
;; AUTHORITY SECTION:  
c10r.facebook.com.    3591    IN    NS    b.ns.c10r.facebook.com.  
c10r.facebook.com.    3591    IN    NS    a.ns.c10r.facebook.com.  
  
;; ADDITIONAL SECTION:  
a.ns.c10r.facebook.com.    3591    IN    A    69.171.239.11  
a.ns.c10r.facebook.com.    3591    IN    AAAA    2a03:2880:fffe:b:face:b00c:0:99  
b.ns.c10r.facebook.com.    3591    IN    A    69.171.255.11  
b.ns.c10r.facebook.com.    3591    IN    AAAA    2a03:2880:ffff:b:face:b00c:0:99  
  
;; Query time: 0 msec  
;; SERVER: ::1#53(::1)  
;; WHEN: Mon May 29 19:02:02 CEST 2016  
;; MSG SIZE  rcvd: 213
```
El comando [dig](http://fpg.66ghz.com/DebianRed/dig.html) permite especificar el servidor que queremos que haga la traducción, para ello se usa el símbolo @ junto con el nombre o dirección del servidor DNS; en el ejemplo se ha puesto ***@localhost***, donde *localhost* se ha traducido vía el fichero ***/etc/hosts*** en el que aparece con la dirección ***127.0.0.1*** (*::1 IPv6*), es decir, el equipo local, donde acabamos de instalar BIND. Por lo tanto, BIND ha hecho la traducción y funciona, pero además, hemos ejecutado dos veces seguidas el mismo comando, y vemos que la primera ejecución ha tardado 231 milisegundos, en cambio, la segunda ha sido instantánea, 0 milisegundos; claramente el motivo es el funcionamiento de BIND como caché. Para resolver la primera consulta BIND hizo una consulta recursiva, para resolver la segunda, utilizó la caché.

En este momento, ya podemos cambiar el fichero ***/etc/resolv.conf*** para que se use siempre nuestro servidor BIND:
```bash
\# cat /etc/resolv.conf  
nameserver 8.8.8.8  
nameserver 8.8.4.4  
\# echo nameserver 127.0.0.1 \> /etc/resolv.conf  
\# cat /etc/resolv.conf  
nameserver 127.0.0.1  
\# ping www.facebook.com  
PING star-mini.c10r.facebook.com (173.252.89.132) 56(84) bytes of data.  
64 bytes from edge-star-mini-shv-06-atn1.facebook.com (173.252.89.132): icmp_seq=1 ttl=79 time=138 ms  
64 bytes from edge-star-mini-shv-06-atn1.facebook.com (173.252.89.132): icmp_seq=2 ttl=79 time=138 ms  
64 bytes from edge-star-mini-shv-06-atn1.facebook.com (173.252.89.132): icmp_seq=3 ttl=79 time=138 ms  
^C  
--- star-mini.c10r.facebook.com ping statistics ---  
3 packets transmitted, 3 received, 0% packet loss, time 2003ms  
rtt min/avg/max/mdev = 138.048/138.231/138.325/0.330 ms
```
Todos los ficheros de la configuración por defecto de BIND se instalan en el directorio ***/etc/bind***:
```bash
\# tree /etc/bind  
/etc/bind  
├── bind.keys  
├── db.0  
├── db.127  
├── db.255  
├── db.empty  
├── db.local  
├── db.root  
├── named.conf  
├── named.conf.default-zones  
├── named.conf.local  
├── named.conf.options  
├── rndc.key  
└── zones.rfc1918  
  
0 directories, 13 files
```
La configuración de BIND se encuentra en ***named.conf***, la cual se distribuye, con la instrucción ***include***, entre los ficheros: ***named.conf.options***, ***named.conf.local*** y ***named.conf.default-zones***:

- ***named.conf.options***: En este fichero se encuentra la instrucción ***options*** que es donde se incluyen todos los parámetros a nivel global de BIND.

- ***named.conf.local***: Aquí es donde crearemos nuestras zonas con la instrucción ***zone***.

- ***named.conf.default-zones***: Este fichero contiene las instrucciones zone de algunas zonas que incluye BIND por defecto.
```bash
\# cat /etc/bind/named.conf  
// This is the primary configuration file for the BIND DNS server named.  
//  
// Please read /usr/share/doc/bind9/README.Debian.gz for information on the  
// structure of BIND configuration files in Debian, \*BEFORE\* you customize  
// this configuration file.  
//  
// If you are just adding zones, please do that in /etc/bind/named.conf.local  
  
include "/etc/bind/named.conf.options";  
include "/etc/bind/named.conf.local";  
include "/etc/bind/named.conf.default-zones";

\# cat /etc/bind/named.conf.default-zones  
// prime the server with knowledge of the root servers  
zone "." {  
    type hint;  
    file "/etc/bind/db.root";  
};  
  
// be authoritative for the localhost forward and reverse zones, and for  
// broadcast zones as per RFC 1912  
  
zone "localhost" {  
    type master;  
    file "/etc/bind/db.local";  
};  
  
zone "127.in-addr.arpa" {  
    type master;  
    file "/etc/bind/db.127";  
};  
  
zone "0.in-addr.arpa" {  
    type master;  
    file "/etc/bind/db.0";  
};  
  
zone "255.in-addr.arpa" {  
    type master;  
    file "/etc/bind/db.255";  
};
```
Los ficheros ***db.\**** son ficheros de zona, que BIND carga por defecto, pues se han creado en *named.conf.default-zones*. Cabe destacar ***db.root*** donde se encuentran los servidores raíz.
```bash
\# cat /etc/bind/db.root  
.                        3600000  IN  NS    A.ROOT-SERVERS.NET.  
A.ROOT-SERVERS.NET.      3600000      A     198.41.0.4  
A.ROOT-SERVERS.NET.      3600000      AAAA  2001:503:BA3E::2:30  
;  
; FORMERLY NS1.ISI.EDU  
;  
.                        3600000      NS    B.ROOT-SERVERS.NET.  
B.ROOT-SERVERS.NET.      3600000      A     192.228.79.201  
...  
...
```
El fichero ***db.local*** permitirá resolver el nombre *localhost* a *127.0.0.1*. Ahora mismo, dicho nombre se resuelve vía */etc/hosts*, pero si se elimina, se puede seguir utilizando, pues BIND lo traduce, sin que nosotros hagamos nada, gracias a *db.local*.

\# cat /etc/bind/db.local  
;  
; BIND data file for local loopback interface  
;  
\$TTL    604800  
@    IN    SOA    localhost. root.localhost. (  
                  2        ; Serial  
             604800        ; Refresh  
              86400        ; Retry  
            2419200        ; Expire  
             604800 )    ; Negative Cache TTL  
;  
@    IN    NS    localhost.  
@    IN    A    127.0.0.1  
@    IN    AAAA    ::1

El fichero ***db.empty*** lo utilizaremos para hacerle copias y a partir de ahí comenzar a crear nuestros propios ficheros de zona.

La instalación crea un usuario ***bind*** que pertenece al grupo principal ***bind*** que también se crea. Este usuario es el que ejecuta el demonio ***/usr/sbin/named***, al que se le pasan los argumentos especificados en el fichero ***/etc/default/bind9***, que nosotros podemos modificar (como administradores) si lo vemos necesario.

\# cat /etc/default/bind9  
\# run resolvconf?  
RESOLVCONF=no  
  
\# startup options for the server  
OPTIONS="-u bind"

Un directorio importante en la instalación de BIND en Debian es ***/var/cache/bind***, utilizado por BIND como referencia para todas las rutas relativas de ficheros. Esto es así pues la instrucción ***directory***, que se encuentra a nivel global dentro de *options*, así lo indica, aunque si queremos podemos utilizar cualquier otro directorio, siempre que el usuario ***bind*** pueda acceder a él.

\# cat /etc/bind/named.conf.options  
options {  
    directory "/var/cache/bind";  
  
    dnssec-validation auto;  
  
    auth-nxdomain no;    \# conform to RFC1035  
    listen-on-v6 ;  
};

\# ls -ld /var/cache/bind/  
drwxrwxr-x 2 root bind 4096 ago 30 08:44 /var/cache/bind/

En este directorio crearemos nuestros ficheros de zona (*db.\**) a partir de */etc/bind/db.empty*, y a la hora de referirnos a ellos con la instrucción ***file*** dentro de ***zone***, no pondremos ninguna ruta. Es importante observar que en el caso de las zonas por defecto (*named.conf.default-zones*) se han utilizado rutas absolutas para hacer referencia a los ficheros *db.\**, de esta manera BIND siempre las encontrará en su sitio, independientemente del valor que nosotros le demos a la instrucción ***directory***.

\# cat /etc/bind/named.conf.default-zones  
// prime the server with knowledge of the root servers  
zone "." {  
    type hint;  
    file "/etc/bind/db.root";  
};  
...

La configuración por defecto presenta un problema de seguridad, pues al trabajar BIND como un servidor DNS totalmente abierto, acepta peticiones DNS de cualquier máquina y un usuario malintencionado podría utilizarlo para hacer ataques de [DDoS](https://es.wikipedia.org/wiki/Ataque_de_denegaci%C3%B3n_de_servicio), [envenenamiento de la caché DNS](https://es.wikipedia.org/wiki/DNS_cache_poisoning), etc. Sin entran en estos momentos en consideraciones y configuraciones de seguridad, una acción fácil para asegurar nuestro servidor DNS que podemos hacer inmediatamente después de la instalación, es limitar los equipos de los que escucharemos consultas, y esto se hace con la instrucción ***listen-on*** que va a nivel global dentro de ***options***; por ejemplo, si hemos instalado BIND en un equipo para que disponga de una caché DNS (GNU/Linux no dispone de caché DNS por defecto), y nada más, podemos asegurarlo haciendo que BIND solo acepte consultas del mismo equipo, y no de otro que se lo haya puesto como servidor DNS, o peor aún, que un usuario malintencionado le envíe consultas con la IP de origen falsificada.

listen-on ;

En este caso ***localhost*** no es un nombre de dominio a traducir vía DNS, sino una palabra reservada de BIND que hace referencia a todas las IP del equipo, incluida la de loopback (127.0.0.1), lo que nos ahorra tener que escribirlas todas, que como mínimo serían dos, la IP de una única tarjeta de red y la dirección de loopback.

## db.root
El fichero ***db.root*** es un fichero de zona que contiene los RR A y AAAA de los servidores DNS raíz. Cuando inicialmente BIND se carga consulta la zona raíz para obtener la lista completa de los servidores raíz autoritarios. BIND cuando no puede resolver una consulta utilizando sus ficheros de zona y/o su caché, según sea el tipo de la consulta hace lo siguiente:

- Si la consulta es *iterativa*, devuelve como referencia la lista de los servidores raíz obtenida de *db.root*.

- Si la consulta es *recursiva*, sigue buscando la respuesta accediendo a uno de los servidores raíz dados en *db.root*.

La instrucción ***zone*** que enlaza con *db.root* se encuentra en *named.conf.default-zones*:

\# cat /etc/bind/named.conf.default-zones  
// prime the server with knowledge of the root servers  
zone "." {  
    type hint;  
    file "/etc/bind/db.root";  
};  
...

Como nombre del dominio se usa el punto, el tipo de zona tiene que ser ***hint***, y ***file*** señala al fichero de zona donde están los RR. A esta zona, también se le llama la *zona hint* o *zona root*.

\# cat /etc/bind/db.root  
.                        3600000  IN  NS    A.ROOT-SERVERS.NET.  
A.ROOT-SERVERS.NET.      3600000      A     198.41.0.4  
A.ROOT-SERVERS.NET.      3600000      AAAA  2001:503:BA3E::2:30  
;  
; FORMERLY NS1.ISI.EDU  
;  
.                        3600000      NS    B.ROOT-SERVERS.NET.  
B.ROOT-SERVERS.NET.      3600000      A     192.228.79.201  
...

Los RR NS están indicando quienes son los servidores de nombres para el dominio raíz, el punto.

Si esta zona no estuviera, BIND sigue funcionando, pues al ser algo tan importante, fue compilado con una lista de servidores raíz, por lo que como último recurso siempre dispondrá de una lista interna de servidores raíz, el único inconveniente es que la lista es fija, pero los servidores raíz no suelen cambiar.

Por último, observa que este es el único fichero de zona que no tiene RR SOA, en todos los demás es obligatorio.

## db.local
El fichero ***db.local*** es un fichero de zona que va a permitir resolver el nombre ***localhost*** a la dirección de *loopback 127.0.0.1*. Es importante esta zona porque el nombre *localhost* es ampliamente utilizado en muchos contextos.

La instrucción ***zone*** que enlaza con *db.local* se encuentra en *named.conf.default-zones*:

\# cat /etc/bind/named.conf.default-zones  
...  
zone "localhost" {  
    type master;  
    file "/etc/bind/db.local";  
};  
...

El nombre del dominio es ***localhost***, el tipo de zona es ***master***, por lo que el servidor es autoritario sobre la zona, y ***file*** señala al fichero de zona donde están los RR.

\# cat /etc/bind/db.local  
;  
; BIND data file for local loopback interface  
;  
\$TTL    604800  
@    IN    SOA    localhost. root.localhost. (  
                  2        ; Serial  
             604800        ; Refresh  
              86400        ; Retry  
            2419200        ; Expire  
             604800 )    ; Negative Cache TTL  
;  
@    IN    NS    localhost.  
@    IN    A    127.0.0.1  
@    IN    AAAA    ::1

Como se ve en el fichero de zona, solo se asocia un RR A al nombre del dominio (*@ IN A 127.0.0.1*), por lo que el nombre *localhost* se traduce a 127.0.0.1 (::1 en caso de IPv6). Estrictamente esta zona sirve para traducir nombres como *www.localhost*, pero únicamente está pensada para traducir el nombre *localhost*, por eso solo hay un RR A para el dominio, y no para otros nombres.

Es importante recordar que en Debian, el fichero */etc/nsswitch.conf* establecía el orden en el que se consultan las distintas fuentes para traducir nombres, y este indica que la consulta al fichero */etc/hosts* va antes que la consulta al servidor dns, por lo que si no cambiamos nada, la traducción de *localhost* se hace vía /etc/hosts; en realidad el efecto es el mismo, pero es importante tenerlo bien presente.

## Consideraciones generales
Como ya se ha mencionado, el fichero ***/etc/bind/named.conf*** es el único fichero de configuración de *BIND*, y esto fue a partir de la versión 8 (nosotros trabajamos con la versión 9), en la versión 4 se llamaba *named.boot*. Este fichero describe el comportamiento y la funcionalidad de BIND, y para ello utiliza una serie de ***instrucciones*** que podemos encontrar en muchos documentos divididas en dos tipos:

- ***cláusulas***: Es una instrucción de alto nivel que va en el fichero *named.conf* y agrupa a un conjunto de estamentos. Siempre comienza en una nueva línea y termina con un punto y coma. Los estamentos que agrupa van encerrados entre llaves.

- ***estamentos***: Son instrucciones que van siempre dentro de una cláusula. Siempre terminan en punto y coma, y pueden ir uno seguido de otro, pero por claridad, se suele escribir cada estamento en una línea.

Lo siguiente son dos formas, entre muchas otras, de escribir lo mismo, pero la primera es la que se suele utilizar:

zone "." {  
    type hint;  
    file "/etc/bind/db.root";  
};

zone "." ;

Los comentarios en *named.conf* pueden ser de distintos tipos:

- Comienzan con **//** y terminan con el final de la línea.

- Comienzan con **\#** y terminan con el final de la línea.

- Comienzan con **/\*** y terminan con **\*/**

// zona raíz  
\# o zona hint  
zone "." {  
    type hint; // tipo para la zona raíz  
    file "/etc/bind/db.root"; \# fichero de zona con ruta absoluta de la zona raíz  
}; /\* comentario de  
varias  
líneas \*/

El uso de comillas es obligatorio para los nombres que contienen espacios, en el resto de los casos, de forma general, es opcional, aunque hay como un cierto convenio en escribir entre comillas ciertos nombres que no llevan espacios, como es el caso del nombre de la zona o dominio que va con la cláusula *zone*, pero en realidad no sería obligatorio. También se podría haber escrito así:

zone . {  
    type hint;  
    file "/etc/bind/db.root";  
};

## Instrucción include
La instrucción ***include*** ni es una cláusula ni una sentencia. Puede aparecer en cualquier sitio de *named.conf,* tanto dentro como fuera de una cláusula. Su función es la de incluir un fichero en el mismo punto donde se coloca esta instrucción. Su sintaxis es la siguiente:

include "fichero"

Ejemplo de la configuración en *named.conf* distribuida entre varios ficheros temáticos:

\# cat /etc/bind/named.conf  
include "/etc/bind/named.conf.acl";  
include "/etc/bind/named.conf.options";  
include "/etc/bind/named.conf.logging";  
include "/etc/bind/named.conf.local";  
include "/etc/bind/named.conf.default-zones";

## Cláusula acl
La cláusula ***acl*** sirve para crear *listas de control de acceso*, que en BIND van a ser grupos de direcciones IP, con el objetivo de simplificar la administración. Las *listas acl* se utilizarán con determinadas instrucciones para que estas solo afecten a los equipos que tengan las direcciones IP especificadas en la lista., es decir, nos van a permitir un ajuste fino sobre qué equipos pueden ejecutar qué operaciones sobre el servidor. Su sintaxis es la siguiente:

acl acl-name {  
    lista-de-direcciones-IP  
};

Las listas acl (pueden crearse cuantas se quieran) tienen que estar definidas antes de que se haga referencia a ellas, por este motivo, suelen ponerse al principio del fichero *named.conf*, y siguiendo con la filosofía de distribuir la configuración, podríamos crear el fichero ***named.conf.acl***, donde definir todas las listas acl e insertarlo al principio de *named.conf* con *include*.

El campo ***acl-name*** define el nombre de la lista acl que posteriormente se utilizará para hacer referencia a ella.

Existen cuatro listas acl predefinidas, sus nombres son:

- ***none***: Coincide con ningún equipo.

- ***any***: Se refiere a todos los equipos.

- ***localhost***: Casa con todas la direcciones IP del servidor donde se ejecuta BIND, incluida la dirección de loopback (127.0.0.1 y solo esta, la 127.0.0.2 no).

- ***localnets***: Coincide con todo el rango de IP de las subredes a las que esté conectado el servidor, más la dirección de loopback.

En el campo ***lista-de-direcciones-IP*** es donde pondremos las IP que constituirán la lista. Formalmente se define así:

lista-de-direcciones-IP = elemento; \[ elemento; \]...

donde ***elemento*** tiene la siguiente sintaxis simplificada:

elemento = \[ ! \] ( ip \[ /prefijo \] \| acl_name )

El signo de admiración sirve para negar la operación a aquellas IP a las que afecte. A las direcciones IP si le ponemos un prefijo, nos estaremos refiriendo al rango de direcciones IP de la subred correspondiente. A una IP sin prefijo se le supone el prefijo /32. Se puede incluir también una lista acl previa.

Cuando una dirección IP se compara con una lista acl, se recorre la lista por orden de izquierda a derecha, hasta que coincide con uno de los elementos, momento en el que se para la búsqueda y se lleva a cabo la acción que corresponda. Por ejemplo, con

listen-on ;

si una consulta viene del host 192.168.1.35, primero se comprueba si coincide con 192.168.1.2, al no coincidir, se pasa al siguiente elemento, y esta vez sí coincide con 192.168.1.0/24, por lo tanto, se detiene la búsqueda y la consulta se escucha (*listen-on*). Si la consulta viene del host 192.168.1.2, al coincidir con el primer elemento, la búsqueda ya se detiene, pero como el elemento está negado, entonces no se escucha la consulta. Esto nos hace ver que el orden es importante, pues si la anterior lista acl estuviera escrita al revés:

listen-on ;

la consulta del 192.168.1.2 sí se escucharía.

La regla general puede expresarse así:

1.  La búsqueda de una coincidencia dentro de una lista acl se hace de izquierda a derecha.

2.  La búsqueda se detiene con la primera coincidencia.

3.  Si la coincidencia no está negada se permite la operación.

4.  Si la coincidencia está negada no se permite a operación.

5.  Si no hay coincidencias no se permite la operación.

Ejemplos de escritura de direcciones IP:

<img src="media/image1.png" style="width:5.48958in;height:2.9375in" alt="Direcciones IP y subredes" />

Ejemplos de listas acl:

\# Equipos válidos  
acl host-validos {  
     !192.169.100.5/28; // deniega a los primeros 16 host  
     192.168.100/24;    // permite al resto de la subred  
     localnets;        // permite a todas las subredes a las que esté conectado el servidor  
};  
  
// lista acl simple con 3 IP  
acl tres-ip {  
  10.0.0.5; 192.168.23.10; 192.168.23.30;  
};  
  
// lista acl con una IP y una subred  
acl con-subred {  
  10.0.0.5;  
  192.168.23.128/25; // 128 IP  
};  
  
// anidamiento de listas acl  
acl todas {  
  tres-ip;  
  con-subred;  
};  
  
// un poco más de complejidad  
acl lista-compleja {  
  tres-ip;  
  10.0.30.0/24; \# permite desde 10.0.30.0 hasta 10.0.30.255  
  !10.0.40.1/24; \# deniega desde 10.0.40.0 hasta 10.0.40.255  
};  
  
// lista acl que permite a todas las IP menos a tres-ip  
acl todo-menos-tres-ip {  
!tres-ip;  
any;  
};

## Cláusula logging
El servidor BIND envía mensajes log para indicar múltiples situaciones por las que pasa, que van desde simples informaciones hasta notificaciones de errores. Estos mensajes log, tanto del servidor BIND como de otros servicios, los maneja en Debian el demonio ***rsyslogd*** (*/usr/sbin/rsyslogd*) a través del protocolo ***SYSLOG***, y por defecto acaban en el fichero ***/var/log/syslog***, mezclado con todos los demás mensajes log. Esto último se puede cambiar si estamos interesados en separar los log del servidor BIND de los demás, si no, también podemos ejecutar el siguiente comando que filtra los mensajes que llevan la palabra *named*, que es el nombre del demonio de BIND y todos sus mensajes log lo llevan:

fgrep named /var/log/syslog

Los mensaje log pertenecen a una categoría y poseen una prioridad. Las categorías pueden ser: *auth*, *authpriv*, *cron*, *daemon*, *kern*, *lpr*, *mail*, *mark*, *news*, *syslog*, *user*, *uucp* y *local0* hasta *local7*. Si no se especifica otra cosa, los log del servidor BIND son de la categoría *daemon*. Por otro lado, a cada log se le asigna una prioridad, que puede ser una de las siguientes ordenadas de menor a mayor prioridad: *debug*, *info*, *notice*, *warning*, *err*, *crit*, *alert* y *emerg*.

En el fichero ***/etc/rsyslog.conf*** se configura el comportamiento de *rsyslogd*, indicando entre otras cosas, en qué ficheros se guardan los mensajes log según su categoría y prioridad. Si queremos que los mensajes log del servidor BIND se guarden en un fichero aparte, seguiremos los siguientes pasos:

1.  En primer lugar haremos que los log de BIND se emitan con la categoría ***local0*** en vez de la categoría *deamon* que usa por defecto, ya que esta es utilizada también por otros servicios. Esto lo haremos ayudándonos de la siguiente cláusula ***logging*** (solo se puede poner una), que añadiremos a *named.conf*:

logging {  
    category default ;  
    channel mi_canal_log {  
        syslog local0;  
        print-severity yes;  
        print-category yes;  
    };  
};

2.  En el fichero ***/etc/rsyslog.conf*** añadimos la línea ***"local0.info /var/log/bind.log"*** para que todos los log de la categoría *local0* y prioridad *info* o superior se escriban en el fichero */var/log/bind.log*.

3.  Creamos el fichero vacío */var/log/bind.log*, pues *rsyslogd* no lo crea.

4.  Reiniciamos los servicios con */etc/init.d/rsyslog* y */etc/init.d/isc-dhcp-server*.

Otra forma de hacer lo mismo es explotando más las posibilidades que da la cláusula *logging* de BIND, que nos permite hacerlo todo desde BIND sin tocar a *rsyslogd*. Veamos como funciona todo esto.

El sistema log de BIND se basa en dos conceptos, el de *canal* y el de *categoría.*

Un ***canal*** define a dónde se enviarán los mensajes log, y puede ser a un fichero, a *rsyslogd*, a la salida estándar de error o a ningún sitio (*null*). Además puede llevar asociado algunos detalles que veremos más adelante.

Una ***categoría*** agrupa a todos los mensajes log de BIND que tienen una causa similar. Estas categorías ya están creadas, y son: *default*, *network*, *database*, *security*, etc. De todas estas la que más nos va a interesar es la categoría ***default***, que representa a todas las categorías que no se hayan especificado explícitamente dentro de la cláusula *logging*.

La cláusula *logging* va a ser la siguiente:

logging {  
    category default ;  
    channel mi_canal_log {  
        file "/var/log/bind/bind.log" versions 3 size 100k;  
        print-time yes;  
        print-category yes;  
        print-severity yes;  
    };  
};

Este trozo de código lo tendremos que poner, como ya se ha dicho, dentro de *named.conf*, pero siguiendo la filosofía de distribuir la configuración, también lo podríamos poner dentro de un fichero que crearíamos con el nombre ***named.conf.logging***, y lo añadimos dentro de *named.conf* con la cláusula ***include*** .

Veamos cada una de las líneas del código anterior.

    category default ;

La sentencia ***category*** lo que hace es asociar una categoría de mensajes log de BIND a un canal, es decir, a donde se enviarán dichos mensajes. Esta sentencia se puede repetir para cada una de las categorías de mensajes log. Como se ha dicho, la categoría ***default*** incluye a todas las categorías que no están explícitamente en una sentencia *category*, y como en el ejemplo no se ha escrito ninguna, entonces *default* se refiere a todos los mensajes log de BIND; por lo tanto, todo va a ***mi_canal_log***.

Definimos el canal:

    channel mi_canal_log {

El destino será un fichero, que rotará tres veces (*bind.log*, *bind.log.0*, *bind.log.1*, *bind.log.2*) cada vez que llegue a los 100KB de tamaño:

        file "/var/log/bind/bind.log" versions 3 size 100k;

Hay que recordar que el demonio *named* de BIND lo ejecuta el usuario *bind*, por lo que los fichero *bind.log\** los crea y modifica dicho usuario. Para no tener problemas de permisos, pues *bind* no puede crear ficheros en */var/log*, lo mejor es crear el directorio */var/log/bind* asignándole *bind* como usuario y grupo.

Por último, queremos que los mensajes log que pasen por *mi_canal_log* vayan acompañados de la fecha y la hora en la que se produjo, la categoría BIND y la prioridad:

        print-time yes;  
        print-category yes;  
        print-severity yes;

## Cláusula options
La cláusula ***options*** agrupa estamentos que tendrán un ámbito global, por lo que se aplicarán a todas las zonas (*zone*) y vistas (*view*, se hablará más adelante de ellas), a menos que el estamento se sobrescriba en dicha zona o vista.

En el fichero *named.conf* solo puede haber una cláusula *options.* Su sintaxis es la siguiente:

options {  
    // estamentos  
};

### Estamento directory
El estamento ***directory*** se utiliza para indicar el directorio base que se utilizará como referencia con todos aquellos ficheros que se especifiquen con una ruta relativa. Solo puede ir dentro de la cláusula ***options***. Su sintaxis es la siguiente:

directory "ruta-absoluta-directorio";

Por ejemplo:

options {  
    directory "/var/cache/bind";  
};

### Estamento version
El estamento ***version*** especifica el texto con el que se contestará a una consulta de versión de BIND como la siguiente:

\# dig version.bind txt chaos  
  
; \<\<\>\> DiG 9.9.5-9+deb8u6-Debian \<\<\>\> version.bind txt chaos  
;; global options: +cmd  
;; Got answer:  
;; -\>\>HEADER\<\<- opcode: QUERY, status: NOERROR, id: 59374  
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1  
;; WARNING: recursion requested but not available  
  
;; OPT PSEUDOSECTION:  
; EDNS: version: 0, flags:; udp: 4096  
;; QUESTION SECTION:  
;version.bind.            CH    TXT  
  
;; ANSWER SECTION:  
version.bind.        0    CH    TXT    "9.9.5-9+deb8u6-Debian"  
  
;; AUTHORITY SECTION:  
version.bind.        0    CH    NS    version.bind.  
  
;; Query time: 0 msec  
;; SERVER: 127.0.0.1#53(127.0.0.1)  
;; WHEN: Wed May 17 09:28:51 CEST 2016  
;; MSG SIZE  rcvd: 89

Solo puede ir dentro de la cláusula ***options***. Su sintaxis es la siguiente:

version "texto" ;

Por ejemplo:

options {  
...  
    version "No disponible";  
...  
};

Si ejecutamos ahora el mismo comando anterior:

\# dig version.bind txt chaos  
  
; \<\<\>\> DiG 9.9.5-9+deb8u6-Debian \<\<\>\> version.bind txt chaos  
;; global options: +cmd  
;; Got answer:  
;; -\>\>HEADER\<\<- opcode: QUERY, status: NOERROR, id: 15975  
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 1  
;; WARNING: recursion requested but not available  
  
;; OPT PSEUDOSECTION:  
; EDNS: version: 0, flags:; udp: 4096  
;; QUESTION SECTION:  
;version.bind.            CH    TXT  
  
;; ANSWER SECTION:  
version.bind.        0    CH    TXT    "No disponible"  
  
;; AUTHORITY SECTION:  
version.bind.        0    CH    NS    version.bind.  
  
;; Query time: 2 msec  
;; SERVER: 127.0.0.1#53(127.0.0.1)  
;; WHEN: Wed May 17 09:52:55 CEST 2016  
;; MSG SIZE  rcvd: 81

Este estamento se suele utilizar para no dar pistas a un usuario malintencionado de la versión de BIND, así lo tendrá más difícil a la hora de buscar fallos de seguridad de BIND y sus correspondientes exploit.

### Estamento listen-on
El estamento ***listen-on*** se utiliza para definir el puerto y las IP de las tarjetas de red del servidor por las que BIND escuchará las consultas. El puerto por defecto es el ***53***. Se pueden poner más de una instrucción *listen-on*, pero solo pueden ir dentro de la cláusula ***options***. Su sintaxis es la siguiente:

listen-on \[ port numero-puerto \] ;

El campo *lista-de-direcciones-IP* es como se describió en la cláusula *acl*, aunque en este caso, cada elemento puede llevar opcionalmente un puerto de escucha, pudiéndose especificar un puerto distinto para cada tarjeta de red del servidor. Por defecto este campo toma el valor ***any***, por lo que el servidor escucha por todas sus tarjetas de red.

listen-on ;

listen-on 5353 ;

listen-on 5353 ;

### Estamento recursion
El estamento ***recursion*** se utiliza para activar o desactivar las consultas recursivas en el servidor DNS. Puede ir dentro de las cláusulas ***options*** y ***view***. Su sintaxis es la siguiente:

recursion ( yes \| no )

Si *recursion* lo ponemos a ***yes*** (valor por ***defecto***), el servidor utilizará el método *recursivo* para responder a las consultas que le lleguen, pero si su valor es ***no***, entonces el método que utilizará será el *iterativo,* por lo que responderá con una referencia a otro servidor DNS. Todo esto siempre que la respuesta no esté en la caché, ya que si lo está, independientemente del valor de *recursion*, se responde con dicho valor y de forma no autoritaria.

Este estamento está íntimamente relacionado con la funcionalidad de la caché en el servidor, pues el uso de la caché requiere que el servidor puede hacer consultas recursivas. Si ponemos *recursion* a *no*, desactivaríamos el servicio de caché DNS.

### Estamento allow-recursion
El estamento ***allow-recursion*** sirve para definir la lista de los equipos a los que les estará permitido hacer consultas recursivas al servidor. Puede ir dentro de las cláusulas ***options*** y ***view***. Su sintaxis es la siguiente:

allow-recursion ;

Únicamente se puede usar si el estamento *recursion* está en el estado *yes*. Por defecto, si *allow-recursion* no se especifica, solo se permiten consultas recursivas a los equipos que pertenezcan a las mismas redes a las que esté conectado el servidor y a él mismo, es como si estuviera presente de la siguiente forma:

allow-recursion ;

Esto es así porque las consultas recursivas, junto con el protocolo UDP, pueden ser utilizadas por un usuario malintencionado para hacer ataques DDoS, por lo que si queremos que nuestro servidor sea un servidor DNS abierto, tendremos que especificarlo nosotros:

allow-recursion ;

y correr con los riesgos. BIND por defecto, como vemos, limita un poco las consultas recursivas y nosotros deberíamos plantearnos siempre cómo vamos a utilizar *allow-recursion* en nuestro servidor.

### Estamento allow-transfer
El estamento ***allow-transfer*** se utiliza para prohibir las transferencias de zona a todos los equipos que la soliciten menos a los especificados en la lista que lo acompaña. Puede ir dentro de las cláusulas ***options***, ***view*** y ***zone***. Su sintaxis es la siguiente:

allow-transfer ;

Una transferencia de zona puede ser realizada tanto por servidores maestros como esclavos, ambos tipos escuchan dichas peticiones. Es importante tener presente este estamento pues un usuario malintencionado podría utilizar las transferencias de zona para llevar a cabo un ataque DoS. Una estrategia puede ser la siguiente: prohibimos las transferencias de zona a todo el mundo y las habilitamos en cada una de las zonas a los equipos que nos interesen.

options {  
   ...  
   allow-transfer ;  
...  
};  
...  
zone "asir.com" in {  
  ...  
  allow-transfer ;  
...  
};

## Cláusula zone
La cláusula ***zone*** controla las propiedades y la funcionalidad de cada zona o dominio. Crearemos una zona para cada dominio del que sea autoritario el servidor. Su sintaxis es la siguiente:

zone nombre-zona {  
   // estamentos-de-zona  
};

El campo ***nombre-zona*** contendrá el nombre de la zona o dominio. No es necesario encerrarlo entre comillas pero suele hacerse. Por ejemplo:

zone asir.com {  
   ...  
};

zone "asir.com" {  
   ...  
};

Hay muchos estamentos que pueden incluirse en un zona, algunos exclusivos de las zonas y otros que pueden aparecer también en la cláusula *options* o en las vistas. Conforme vayamos viendo tipos de configuraciones, iremos aprendiendo nuevos estamentos; en este punto, vamos a describir los estamentos más habituales.

### Estamento type
El estamento ***type*** define el tipo de la zona. Solo puede ir dentro de la cláusula ***zone***. Su sintaxis es la siguiente:

type tipo-zona;

Los valores más utilizados para el campo ***tipo-zona*** son:

- ***hint***:Se utiliza para definir la zona raíz donde estarán las direcciones de los servidores raíz. Al iniciarse BIND, este busca en el fichero de zona hint para obtener la lista actualizada de los servidores raíz. Si no existiera esta zona, utilizaría una lista de servidores raíz que con la que se compiló BIND.

- ***master***: En una zona *master* el fichero de zona con los RR se encuentra en el disco local del servidor y sus respuestas sobre el dominio serán autoritarias.

- ***slave***: Una zona slave es una réplica de una zona *master*, donde el fichero de zona se obtiene mediante una operación de transferencia de zona. Las respuestas del servidor sobre la zona serán autoritarias.

type master;

### Estamento file
El estamento ***file*** establece el fichero que se utilizará como fichero de zona. Si se utiliza una ruta relativa, se utilizará el estamento *directory* para hacerla absoluta. Solo puede ir dentro de la cláusula ***zone***. Su sintaxis es la siguiente:

file "fichero-de-zona";

### Estamento masterfile-format
El estamento ***masterfile-format*** controla el formato que se utilizará con el fichero de zona *master*. Puede ir dentro de las cláusulas ***options***, ***zone*** y ***view***. Su sintaxis es la siguiente:

masterfile-format text \| raw \| map;

El valor por defecto de una zona *master* es ***text*** (texto), y de una zona *slave* es ***raw*** (binario). El formato ***map***, es también binario y BIND trabaja más rápido con él pues es el formato binario interno con el que representa BIND en memoria las zonas.

masterfile-format text;

### Estamento masters
El estamento ***masters*** es válido solo con zonas *slave* y define una o más direcciones IP a las que se les solicitará transferencias de zonas. El valor de *refresh* del RR SOA de la zona es el que indica cuando se realizará dicha solicitud. Es posible también, cambiar el puerto que se utilizará para la transferencia. Solo puede ir dentro de la cláusula ***zone***. Su sintaxis simplificada es la siguiente:

masters \[ port numero-puerto \] ;

Como se ve, se puede especificar otro puerto distinto al ***53***, que es el puerto por defecto. Este cambio puede hacerse para todas las IP o para IP concretas.

masters port 1234 ;

## named-checkconf
El comando ***named-checkconf*** sirve para chequear la sintaxis de los ficheros de configuración de BIND. En el chequeo incluye aquellos ficheros de la instrucción *include*. Su sintaxis es la siguiente:

\# named-checkconf \[ fichero \]

Si no se especifica ningún fichero, chequeará el fichero *named.conf* junto con todos los ficheros *include* que tenga.

\# named-checkconf /etc/bind/named.conf.options

\# named-checkconf

## named-checkzone
El comando ***named-checkzone*** se utiliza para chequear la sintaxis de un fichero de zona. Su sintaxis es la siguiente:

\# named-checkzone nombre-zona fichero

Por ejemplo:

\# named-checkzone asir.com /var/cache/bind/db.master.asir.com

---

## 15. Ejemplo completo paso por paso

A continuación se configura una zona directa y una zona inversa para la red de laboratorio.

### Paso 1. Definir una ACL para la red local

Editar `/etc/bind/named.conf.options`:

```conf
acl "red_local" {
    127.0.0.1;
    192.168.1.0/24;
};

options {
    directory "/var/cache/bind";

    listen-on { 127.0.0.1; 192.168.1.10; };
    listen-on-v6 { ::1; };

    recursion yes;
    allow-query { red_local; };
    allow-recursion { red_local; };
    allow-transfer { none; };

    dnssec-validation auto;
    version "No disponible";
};
```

### Paso 2. Declarar las zonas

Editar `/etc/bind/named.conf.local`:

```conf
zone "asir.local" {
    type master;
    file "/etc/bind/db.asir.local";
    allow-transfer { none; };
};

zone "1.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.192.168.1";
    allow-transfer { none; };
};
```

La zona inversa se escribe invirtiendo los octetos de red: `192.168.1.0/24` se convierte en `1.168.192.in-addr.arpa`.

### Paso 3. Crear el archivo de zona directa

```bash
sudo cp /etc/bind/db.empty /etc/bind/db.asir.local
sudo nano /etc/bind/db.asir.local
```

Contenido:

```dns
$TTL 86400
@   IN  SOA ns1.asir.local. admin.asir.local. (
        2026092501 ; serial
        3600       ; refresh
        1800       ; retry
        604800     ; expire
        86400      ; negative cache TTL
)

@       IN  NS      ns1.asir.local.
ns1     IN  A       192.168.1.10
www     IN  A       192.168.1.30
ftp     IN  A       192.168.1.40
correo  IN  A       192.168.1.50
@       IN  MX 10   correo.asir.local.
intranet IN CNAME   www.asir.local.
```

Puntos importantes:

1. Los nombres completos terminan en punto, por ejemplo `ns1.asir.local.`.
2. El segundo campo del SOA representa el correo del responsable: `admin.asir.local.` equivale conceptualmente a `admin@asir.local`.
3. El número de serie debe aumentar después de cada modificación.
4. `A` asocia un nombre con una dirección IPv4.
5. `CNAME` crea un alias.
6. `MX` declara el servidor de correo y su prioridad.

### Paso 4. Crear el archivo de zona inversa

```bash
sudo cp /etc/bind/db.127 /etc/bind/db.192.168.1
sudo nano /etc/bind/db.192.168.1
```

Contenido:

```dns
$TTL 86400
@   IN  SOA ns1.asir.local. admin.asir.local. (
        2026092501 ; serial
        3600       ; refresh
        1800       ; retry
        604800     ; expire
        86400      ; negative cache TTL
)

@   IN  NS  ns1.asir.local.
10  IN  PTR ns1.asir.local.
30  IN  PTR www.asir.local.
40  IN  PTR ftp.asir.local.
50  IN  PTR correo.asir.local.
```

En una zona `/24` solo se escribe el último octeto a la izquierda del registro `PTR`.

### Paso 5. Ajustar propietarios y permisos

```bash
sudo chown root:bind /etc/bind/db.asir.local /etc/bind/db.192.168.1
sudo chmod 640 /etc/bind/db.asir.local /etc/bind/db.192.168.1
```

### Paso 6. Comprobar la sintaxis

```bash
sudo named-checkconf
sudo named-checkzone asir.local /etc/bind/db.asir.local
sudo named-checkzone 1.168.192.in-addr.arpa /etc/bind/db.192.168.1
```

Si todo es correcto, `named-checkzone` mostrará `OK`. `named-checkconf` normalmente no muestra nada cuando no encuentra errores.

### Paso 7. Recargar BIND

```bash
sudo systemctl reload bind9
```

Si la recarga falla:

```bash
sudo systemctl status bind9
sudo journalctl -u bind9 -n 50 --no-pager
```

### Paso 8. Probar la zona directa

```bash
dig @192.168.1.10 www.asir.local A

dig @192.168.1.10 intranet.asir.local CNAME

dig @192.168.1.10 asir.local MX
```

### Paso 9. Probar la zona inversa

```bash
dig @192.168.1.10 -x 192.168.1.30
```

La respuesta esperada debe contener un registro `PTR` hacia `www.asir.local.`.

## 16. Pruebas y diagnóstico

### Consultar un registro concreto

```bash
dig @192.168.1.10 www.asir.local A +short
```

### Mostrar la respuesta completa

```bash
dig @192.168.1.10 www.asir.local
```

### Consultar el registro SOA

```bash
dig @192.168.1.10 asir.local SOA
```

### Consultar los servidores de nombres

```bash
dig @192.168.1.10 asir.local NS
```

### Vaciar la caché de BIND

```bash
sudo rndc flush
```

### Verificar el estado mediante RNDC

```bash
sudo rndc status
```

### Consultar los registros del servicio

```bash
sudo journalctl -u bind9 --since today
```

### Errores frecuentes

- **`SERVFAIL`**: suele indicar un error de configuración, archivo inaccesible, serial incorrecto o problema de validación.
- **`NXDOMAIN`**: el nombre consultado no existe en la zona.
- **`REFUSED`**: una ACL o una directiva como `allow-query` rechaza la consulta.
- **`connection timed out`**: revisar servicio, dirección de escucha, cortafuegos y conectividad.
- **Respuesta sin `aa`**: probablemente el servidor no es autoritativo para esa zona.

## 17. Buenas prácticas de seguridad

1. No ofrecer recursión a Internet. Limitar `allow-recursion` a redes conocidas.
2. Limitar `allow-query` cuando el servidor sea de uso interno.
3. Prohibir transferencias globalmente con `allow-transfer { none; };` y habilitarlas solo para secundarios autorizados.
4. Escuchar solo en las interfaces necesarias mediante `listen-on` y `listen-on-v6`.
5. Mantener BIND y el sistema actualizados.
6. Comprobar siempre la configuración antes de recargar el servicio.
7. Revisar los registros de `journalctl` o el canal de logging configurado.
8. No editar archivos generados automáticamente sin conocer qué servicio los administra.
9. Incrementar el serial SOA en cada cambio de zona.
10. Proteger los archivos de zona con permisos mínimos necesarios.

## 18. Resumen de comandos

| Objetivo | Comando |
|---|---|
| Instalar BIND | `sudo apt install bind9 bind9-utils dnsutils` |
| Ver estado | `sudo systemctl status bind9` |
| Reiniciar | `sudo systemctl restart bind9` |
| Recargar configuración | `sudo systemctl reload bind9` |
| Ver puerto 53 | `sudo ss -lntup \| grep ':53'` |
| Validar configuración | `sudo named-checkconf` |
| Validar zona directa | `sudo named-checkzone asir.local /etc/bind/db.asir.local` |
| Hacer consulta | `dig @192.168.1.10 www.asir.local` |
| Consulta inversa | `dig @192.168.1.10 -x 192.168.1.30` |
| Vaciar caché | `sudo rndc flush` |
| Ver registros | `sudo journalctl -u bind9 -n 50` |

---

## Conclusión

BIND 9 puede actuar como servidor de caché y como servidor autoritativo para zonas directas e inversas. Una configuración correcta exige declarar las zonas, crear sus archivos, validar la sintaxis, restringir la recursión y las transferencias, recargar el servicio y verificar la resolución mediante `dig`.
