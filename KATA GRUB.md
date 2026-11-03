# KATA 1: Proteger GRUB manteniendo el arranque desatendido

## 1. Mostrar el menú de GRUB
Editar el fichero de configuración principal de GRUB:
`sudo nano /etc/default/grub`

Modificar estas dos líneas:
* `GRUB_TIMEOUT_STYLE=menu` (Cambiar "hidden" por "menu" para que se muestre)
* `GRUB_TIMEOUT=5` (Dar 5 segundos de espera antes de arrancar solo)

## 2. Generar la contraseña cifrada
Ejecutar el comando de GRUB para generar el hash:
`grub-mkpasswd-pbkdf2`
*(Introducir la contraseña dos veces y copiar la cadena generada que empieza por `grub.pbkdf2.sha512...`)*

## 3. Definir el superusuario y proteger la edición
Añadir la configuración en el fichero de scripts personalizados:
`sudo nano /etc/grub.d/40_custom`

Añadir al final del archivo:
`set superusers="admin"`
`password_pbkdf2 admin <PEGAR_EL_HASH_AQUÍ>`

## 4. Permitir el arranque normal sin contraseña (Unrestricted)
Editar el script que genera las entradas de Linux para inyectar el permiso:
`sudo nano /etc/grub.d/10_linux`

Buscar la línea que define la variable CLASS (suele estar sobre la línea 140) y añadir `--unrestricted` al final de las comillas:
**Línea original:** `CLASS="--class gnu-linux --class gnu --class os"`
**Línea modificada:** `CLASS="--class gnu-linux --class gnu --class os --unrestricted"`

## 5. Regenerar la configuración
Aplicar todos los cambios generando el nuevo grub.cfg:
`sudo update-grub`

---

## 📝 Notas requeridas para la Kata:
* **Ficheros modificados y contenido:**
  * `/etc/default/grub`: Se cambió `GRUB_TIMEOUT_STYLE` a `menu` y `GRUB_TIMEOUT` a `5`.
  * `/etc/grub.d/40_custom`: Se añadió `set superusers="admin"` y `password_pbkdf2 admin [hash]`.
  * `/etc/grub.d/10_linux`: Se añadió `--unrestricted` a la variable `CLASS`.
* **Órdenes ejecutadas:** 
  * Para generar contraseña cifrada: `grub-mkpasswd-pbkdf2`
  * Para regenerar configuración: `sudo update-grub`
* **En una frase (Qué protege y qué no):** 
  Protege contra la modificación temporal de los parámetros de arranque (como añadir `init=/bin/bash` para robar acceso root), pero no protege los datos ni el sistema si el atacante extrae físicamente el disco duro.

## 🤔 Respuestas "Para pensar":
1. **¿Por qué no basta con ocultar el menú?** Porque un atacante con conocimientos puede forzar la aparición del menú pulsando y manteniendo la tecla `Shift` o `ESC` durante la carga de la BIOS/UEFI.
2. **¿Qué pasaría en la clínica si la contraseña se pidiera también en el arranque normal?** Ante un corte de suministro eléctrico, cuando la luz volviera, el servidor se quedaría bloqueado en la pantalla negra pidiendo credenciales. La clínica se quedaría sin sistema hasta que el técnico informático llegase a introducirla físicamente.
3. **Alguien saca el disco del servidor y lo conecta a otro equipo. ¿Le frena esta medida?** No. GRUB solo controla cómo arranca el sistema operativo. Si sacan el disco, pueden leer los archivos montando las particiones desde otro equipo. Para frenar eso, se requiere cifrado de disco completo (como LUKS).



HASH 256: grub.pbkdf2.sha512.10000.FA8961E0E22930A092275E2A318395F853C4BAAECA8173F73EC7030D9FBE41CA5483A925338DDA91EBA376CC62D03F47C2307C4448F9D86B27EDD28C5ED9601D.68A2679F8330C524561E16A95A5AB1F47FBCBA877A21C6DA48BE1A9382E626AA25DED6D7AA8053E475540AF475DC7508D576A7916C5BF2314B2829D6CCD362D8