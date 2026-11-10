# KATA 3: Cifrado de partición con LUKS

## 1. Preparar la partición (Desmontar y limpiar)
Desmontar la partición que vamos a cifrar:
`sudo umount /srv/historiales`

Eliminar la configuración antigua para que no interfiera en el arranque:
`sudo nano /etc/fstab`
*(Borra o comenta con un `#` al principio la línea de `/srv/historiales` que apuntaba al UUID antiguo de sdb1).*

## 2. Cifrar la partición con LUKS
Crear el contenedor cifrado (¡borra los datos previos!):
`sudo cryptsetup luksFormat /dev/sdb1`
*(El sistema te pedirá que escribas `YES` en mayúsculas para confirmar que asumes la pérdida de datos, y luego la frase de paso dos veces).*

## 3. Abrir el volumen y dar formato
Abrir la caja fuerte y crear el dispositivo mapeado virtual:
`sudo cryptsetup luksOpen /dev/sdb1 historiales_crypt`
*(Te pedirá la frase de paso que acabas de crear).*

Dar formato ext4 al interior de la caja fuerte virtual:
`sudo mkfs.ext4 /dev/mapper/historiales_crypt`

## 4. Configurar crypttab (Apertura al arranque)
Inyectar el UUID del disco físico cifrado directamente en el archivo crypttab para no teclearlo a mano:
`sudo blkid /dev/sdb1 | sudo tee -a /etc/crypttab`

Editar el archivo para darle el formato correcto:
`sudo nano /etc/crypttab`
*(Busca la línea que se ha pegado abajo del todo, borra el `/dev/sdb1:` y el `TYPE="crypto_LUKS"`, y déjala exactamente así):*
`historiales_crypt UUID=el-uuid-largo-que-se-ha-pegado none luks,discard`

## 5. Configurar fstab (Montaje seguro)
Editar el archivo de montajes para añadir el volumen descifrado virtual:
`sudo nano /etc/fstab`
*(Añade esta línea al final del archivo, fíjate que ahora usamos el mapper y no un UUID para el fstab):*
`/dev/mapper/historiales_crypt /srv/historiales ext4 defaults,nodev,nosuid 0 2`

## 6. Verificación final antes de reiniciar
Aplica los montajes para comprobar que no hay errores tipográficos:
`sudo mount -a`
*(Si no sale nada por pantalla, está perfecto. Ya puedes reiniciar con `sudo reboot`).*

---

## 📝 Notas requeridas para la Kata:
* **Órdenes usadas en orden:**
  1. `sudo umount /srv/historiales`
  2. `sudo cryptsetup luksFormat /dev/sdb1`
  3. `sudo cryptsetup luksOpen /dev/sdb1 historiales_crypt`
  4. `sudo mkfs.ext4 /dev/mapper/historiales_crypt`
  5. `sudo blkid /dev/sdb1` (Para obtener UUID e inyectarlo)
* **Líneas de configuración:**
  * **`/etc/crypttab`:** `historiales_crypt UUID="xxx-xxx-xxx" none luks,discard`
  * **`/etc/fstab`:** `/dev/mapper/historiales_crypt /srv/historiales ext4 defaults,nodev,nosuid 0 2`
* **Contra qué protege (2 frases):**
  * **Servidor apagado:** Protege totalmente la confidencialidad de los datos si alguien roba el disco físico o el servidor, ya que sin la clave maestra la partición es indescifrable.
  * **Servidor encendido:** No protege contra ataques a través de la red (malware, exploits web o intrusiones SSH); si el servidor está encendido y el volumen montado, el sistema operativo lee los datos en claro.

## 🤔 Respuestas "Para pensar":
1. **Hay un corte de luz de madrugada en la clínica. ¿Qué pasa con el servidor por la mañana? ¿Quién escribe la frase de paso?**
   El servidor arrancará, pero el proceso de inicio se detendrá en seco mostrando una pantalla negra pidiendo la contraseña para desbloquear `sdb1`. El sistema no terminará de arrancar ni cargará los servicios hasta que alguien del personal de la clínica llegue físicamente al cuarto del servidor con un teclado conectado y escriba la clave.
2. **Si la frase de paso se guardara en un fichero del propio servidor para que arranque solo, ¿de qué protegería el cifrado?**
   Solamente protegería contra el robo aislado de ese disco secundario (`/dev/sdb`). Si un ladrón roba el servidor completo (o se lleva el disco primario y el secundario juntos), podría leer el archivo con la clave y descifrar los historiales sin esfuerzo, perdiendo el propósito principal del cifrado de disco completo (FDE).