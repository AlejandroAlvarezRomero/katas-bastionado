# KATA 2: Particionado, Formato y Montaje Seguro (fstab)

## 1. Identificar y particionar el disco
Primero, comprobamos qué nombre tiene el disco nuevo de 4 GB:
`lsblk` *(Normalmente será /dev/sdb)*

Entramos en el particionador para ese disco:
`sudo fdisk /dev/sdb`

**Secuencia de teclas en fdisk:**
1. `n` (Nueva partición) -> `p` (Primaria) -> `1` (Número de partición) -> `Enter` (Primer sector por defecto) -> `+3G` (Tamaño de 3 Gigabytes).
2. `n` (Nueva partición) -> `p` (Primaria) -> `2` (Número) -> `Enter` -> `Enter` (Usa todo el espacio restante, ~1 GB).
3. `w` (Escribir los cambios en el disco y salir).

## 2. Dar formato a las particiones
Formateamos ambas particiones con el sistema de archivos ext4:
`sudo mkfs.ext4 /dev/sdb1` *(Parámetro: /dev/sdb1 es la partición de historiales)*
`sudo mkfs.ext4 /dev/sdb2` *(Parámetro: /dev/sdb2 es la partición de subidas)*

## 3. Crear puntos de montaje y obtener UUID
Creamos las carpetas donde se montarán las particiones:
`sudo mkdir -p /srv/historiales /srv/subidas` *(Parámetro -p: crea los directorios padres si no existen, sin dar error)*

Averiguamos el UUID de las particiones (cópialos para el siguiente paso):
`sudo blkid /dev/sdb1`
`sudo blkid /dev/sdb2`

## 4. Configurar el montaje persistente y seguro
Editamos el archivo de tabla de sistemas de ficheros:
`sudo nano /etc/fstab`

Añadimos estas dos líneas al final del archivo (sustituyendo los UUID por los que obtuviste en el paso anterior):
`UUID="poner-uuid-sdb1-aqui" /srv/historiales ext4 defaults,nodev,nosuid 0 2`
`UUID="poner-uuid-sdb2-aqui" /srv/subidas ext4 defaults,nodev,nosuid,noexec 0 2`

*(Explicación de la cola de la línea: '0' es para que dump no haga backup, '2' es la prioridad del chequeo de disco fsck al arrancar)*

## 5. Verificación (¡CRÍTICO ANTES DE REINICIAR!)
Comprobamos que no hay errores de sintaxis en el fstab:
`sudo findmnt --verify`

Forzamos el montaje de todo lo definido en el fstab para ver si funciona:
`sudo mount -a` *(Si no devuelve ningún error, está perfecto)*

---

## 📝 Notas requeridas para la Kata:
* **Órdenes ejecutadas:** 
  * Particionar: `sudo fdisk /dev/sdb`
  * Formatear: `sudo mkfs.ext4 /dev/sdb1` y `sudo mkfs.ext4 /dev/sdb2`
  * Montar: Se usó `/etc/fstab` y luego `sudo mount -a`.
* **Las dos líneas de /etc/fstab:**
  `UUID="xxx-xxx-xxx" /srv/historiales ext4 defaults,nodev,nosuid 0 2`
  `UUID="yyy-yyy-yyy" /srv/subidas ext4 defaults,nodev,nosuid,noexec 0 2`
* **Por qué lleva esas opciones (en una frase):**
  * **historiales (`nodev,nosuid`):** Evita la creación de dispositivos especiales y bloquea la escalada de privilegios SUID, manteniendo el rendimiento para archivos de datos.
  * **subidas (`nodev,nosuid,noexec`):** Bloquea completamente la ejecución de binarios subidos por clientes (malware) y evita escaladas de privilegios, aislando el contenido como puramente pasivo.

## 🤔 Respuestas "Para pensar":
1. **Si /var estuviera en su propia partición y un registro creciera sin control, ¿qué ganaría la clínica?**
   Ganaría disponibilidad y estabilidad. Si `/var` se llena al 100%, solo afectaría a los servicios que escriben logs o bases de datos allí, pero el sistema operativo principal (la partición `/` o root) seguiría teniendo espacio libre, evitando un bloqueo total (kernel panic) y permitiendo al administrador entrar a arreglarlo.
2. **noexec impide ejecutar ./script.sh. ¿Impide también bash script.sh? Pruébalo. ¿Qué te dice eso de lo que protege esta opción?**
   **No lo impide.** `noexec` solo bloquea la ejecución directa pidiendo al kernel que ejecute el archivo. Sin embargo, al lanzar `bash script.sh`, el que se ejecuta es `/bin/bash` (que está en la partición root, donde sí hay permisos de ejecución) y simplemente "lee" el script como si fuera un archivo de texto normal. Esto nos dice que `noexec` protege contra binarios compilados (como un `.elf` o un ejecutable de C) y ejecuciones directas de scripts, pero **no** es una barrera infalible contra scripts interpretados si el atacante puede invocar al intérprete.


/dev/sdb1: UUID="acaa4d48-c583-4cfe-8558-d6886b9e9117" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="2d7ddea1-01"
/dev/sdb2: UUID="61e334a6-b06a-439a-8048-0ec605fb636f" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="2d7ddea1-02"