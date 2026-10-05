# Copias de seguridad en WordPress

## 1. ¿Qué es una copia de seguridad?

Una **copia de seguridad** (o *backup*) es una duplicación de los datos y archivos de un sitio web para poder restaurarlos en caso de pérdida, fallo o ataque. Su principal finalidad es **proteger el sitio frente a la pérdida de información**.

## 2. Tipos de copias de seguridad

- **Completa**: incluye todos los archivos del sitio (temas, plugins, imágenes, base de datos).
- **Incremental**: guarda solo los cambios desde la última copia, ahorrando espacio y tiempo.
- **Automática**: se realiza de forma programada sin intervención del usuario.
- **Manual**: la realiza el administrador exportando la base de datos y copiando los archivos del sitio.

## 3. Elementos que se deben incluir

- **Base de datos**: contiene entradas, usuarios, configuraciones, comentarios, etc.
- **Archivos del sitio**: especialmente los contenidos en la carpeta `/wp-content/`, donde se guardan temas, plugins y medios.
- **Archivo `wp-config.php`**: contiene la configuración de conexión a la base de datos y otras opciones esenciales.

## 4. Plugins populares para hacer copias

Los plugins facilitan la creación y restauración de copias. Algunos de los más usados son:

- **UpdraftPlus**: permite hacer copias manuales o automáticas, almacenarlas en la nube y restaurarlas desde el panel.
- **BackupBuddy**
- **Duplicator**
- **Jetpack Backup**

## 5. Almacenamiento de las copias

Se recomienda **guardar las copias en una ubicación externa al servidor**, como:

- Google Drive  
- Dropbox  
- Amazon S3  
- Servidores FTP externos

Guardar las copias en el mismo servidor del sitio **no es seguro**, ya que si el servidor falla o es hackeado, se perderán también las copias.

## 6. Frecuencia de las copias

La frecuencia depende de la actividad del sitio:

- Si el sitio se **actualiza a diario**, conviene hacer **copias diarias**.
- Si apenas cambia, se pueden programar copias semanales o mensuales.
También se recomienda **hacer una copia antes de cada actualización importante** (de WordPress, plugins o temas).

## 7. Restauración de copias

**Restaurar** una copia significa devolver el sitio a un estado anterior utilizando una copia guardada.  
Puede hacerse:

- Desde el **plugin de backup** (por ejemplo, UpdraftPlus).
- Manualmente, subiendo los archivos y restaurando la base de datos.

Después de restaurar, es importante **verificar que el sitio funcione correctamente**.

## 8. Buenas prácticas

- Verificar periódicamente que las copias se realizan correctamente y se pueden restaurar.  
- Automatizar las copias para evitar olvidos.  
- Mantener varias versiones antiguas de las copias.  
- Almacenar las copias en **lugares seguros y cifrados**.  
- Realizar una copia completa **antes de realizar cambios importantes**.

## 9. Copias ofrecidas por el hosting

Muchos proveedores de alojamiento web incluyen copias automáticas diarias o semanales del sitio y la base de datos. Aun así, es recomendable **mantener una copia adicional propia**.

## 10. Errores comunes

- Guardar las copias en el mismo servidor.  
- No comprobar que las copias se restauran correctamente.  
- No establecer una frecuencia adecuada.  
- Confiar únicamente en el hosting sin tener copias propias.

---

> **Resumen:**  
> Las copias de seguridad son esenciales para proteger un sitio WordPress. Incluyen la base de datos y los archivos del sitio, pueden hacerse manual o automáticamente, y deben almacenarse de forma segura en ubicaciones externas. Los plugins como *UpdraftPlus* facilitan todo el proceso, incluyendo la restauración.
