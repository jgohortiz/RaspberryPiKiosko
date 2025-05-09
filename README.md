
# RaspberryPiKiosko
El objetivo de esta guía es explicar, paso a paso, cómo configurar (fundamentos básicos para la configuración) una **Raspberry Pi 5** para mostrar una página web en modo quiosco utilizando **Chromium** sobre **Raspberry Pi OS de 64 bits**.


## 1. INSTALAR EL SISTEMA OPERATIVO **Raspberry Pi OS de 64 bits**
Como sistema operativo, elige Raspberry Pi OS, instalalo usando Raspberry Pi Imager. Durante la etapa de personalización del sistema operativo, edite la configuración de la siguiente manera:
- Ingrese un nombre de host de su elección. 
- Ingrese un nombre de usuario y una contraseña.
- Para conexión inalámbrica.
	- Marque la casilla junto a Configurar LAN inalámbrica para que su Pi pueda conectarse automáticamente a Wi-Fi
	- Ingrese el SSID (nombre) y la contraseña de su red
- Marque la casilla junto a Habilitar SSH para que podamos conectarnos al **Raspberry Pi 5** por Putty.


> [!TIP]
> En el siguiente enlace puedes ver como instalar el sistema operativo en detalle > [Install an operating system](https://www.raspberrypi.com/documentation/computers/getting-started.html#installing-the-operating-system).

## 2. CONFIGURAR SSH (Básica)
Para mejorar la seguridad de la conexión SSH, configuraremos algunos parámetros iniciales.

> [!IMPORTANT]
> **CONEXIÓN SSH**: Para conectarse por SSH use Putty. Utilice el nombre de host o la dirección IP, el puerto por defecto es el 22.

- Edite el archivo de configuración SSH
  ```
  sudo nano /etc/ssh/sshd_config
  ```

- Establezca los siguientes parámetros
  ```
  LoginGraceTime 1m
  PermitRootLogin prohibit-password
  StrictModes yes
  MaxAuthTries 3
  MaxSessions 2
  PasswordAuthentication yes
  PermitEmptyPasswords no
  KbdInteractiveAuthentication no
  UsePAM yes
  X11Forwarding yes
  PrintMotd no
  AcceptEnv LANG LC_*
  Banner /etc/issue
  ```
> El parámetro *Banner* es opcional.

> [!NOTE]
> Si se activa el banner de entrada, es posible personalizar el texto que se mostrará. Para ello, edite el archivo de configuración correspondiente e ingrese el contenido deseado, para hacerlo editeelarchivo.

- Definir el banner de entrada.
  ```
  sudo nano /etc/issue
  ```

- Reinicie el servicio
  ```
  sudo systemctl restart ssh
  ```


## 3. CONFIGURAR EL MODO QUIOSCO
Para lograr el objetivo, instalaremos algunos recursos, configuraremos parámetros en el sistema operativo, crearemos algunos scripts… ¡y el resultado estará listo como por arte de magia!

- Actualice el SO
  ```
  sudo apt update
  sudo apt full-upgrade
  ```

- Instale las utilidaes `wtype` y `ydotool`, es opcional `tightvncserver`
  ```
  sudo apt install wtype
  sudo apt install ydotool
  sudo apt install tightvncserver
  ```

- Configuración necesaria en *Raspberry Pi OS de 64 bits*
  Defina los siguientes parámetros del SO, para hacerlo Ingrese en la consola
  ```
  sudo raspi-config
  ```
- Establezca las propiedades
  Ahora, navegue entre las opciones listadas a continuación y establezca el valor señalado.
  - **System Options**
  	- Boot
  		- :Desktop
  	- Autologin
  		- :Yes
  - *Advanced Options*
  	- Wayland
  		- :wayfire
  
Por último, finalice y reinicie la Raspberry.

## 4. SCRIPTS
En esta sección se presentan scripts diseñados para optimizar la experiencia de usuario en el kiosco. Se incluye un script para la apertura automatizada de Chromium en modo kiosco, garantizando una navegación sin distracciones, y otro para ocultar el puntero del mouse moviéndolo fuera del área visible, mejorando la estética y funcionalidad de la interfaz.

### SCRIPT PARA LA APERTURA DE CHROMIUM
Este script está diseñado para iniciar Chromium en modo kiosco, proporcionando una experiencia de navegación limpia y sin interrupciones, el navegador se abre a pantalla completa, sin barras de herramientas ni mensajes de error.

- Cree el archivo con el script de inicio
  ```
  sudo nano /home/pi/wrk_pikiosk_call_chromium.sh
  ```

- Adicione el siguiente contenido
  ```
  #!/bin/bash
  
  # Establezca este valor como predeterminado, use la variable de entorno URL_KIOSK.
  URL_KIOSK="https://www.youtube.com/watch?v=yul4gq_LrOI"
  
  # Cerrar las instancias de Chromium
  killall -I chromium
  sleep 5
  
  # Limpiar el cache de Chromium
  rm -rf ~/.cache/chromium
  sleep 5
  
  # iniciar Chromium
  chromium-browser $URL_KIOSK --kiosk --noerrdialogs --disable-infobar --no-first-run --ozone-platform=wayland --enable-features=OverlayScrollbar --start-maximized
  ```

- Otorgue permisos
  ```
  sudo chmod +x /home/pi/wrk_pikiosk_call_chromium.sh
  ```

### SCRIPT PARA OCULTAR EL PUNTERO DEL MOUSE
Este script utiliza la herramienta ydotool para mover el puntero del mouse fuera del área visible de la pantalla, simulando su ocultamiento, el cursor se desplaza automáticamente, después de un breve retraso, a una posición lejana, minimizando distracciones visuales en entornos donde no se requiere interacción directa con el mouse. 

- Cree el archivo con el script de inicio
  ```
  sudo nano /home/pi/wrk_pikiosk_hide_cursor.sh
  ```

- Adicione el siguiente contenido
  ```
  #!/bin/bash
  
  # Mueve el cursor fuera del rango visual
  sleep 15
  sudo ydotool mousemove --delay 1000 10000 10000
  ```

- Otorgue permisos
  ```
  sudo chmod +x /home/pi/wrk_pikiosk_hide_cursor.sh
  ```


## 5. CONFIGURACIÓN PARA EL INICIO AUTOMÁTICO
Para el inicio automático usaremos `wayland`, para eso evitaremos el archivo `wayfire.ini`, el cual hace el llamado inicial los scripts.

- Edite el archivo de configuración
  ```
  sudo nano /home/pi/.config/wayfire.ini
  ```

- Adicione en [autostart]
  ```
  [autostart]
  kiosk = /home/pi/wrk_pikiosk_call_chromium.sh
  cursor = /home/pi/wrk_pikiosk_hide_cursor.sh
  screensaver = false
  dpms = false
  ```

## 6. SEGURIDAD

En entornos donde la seguridad es prioritaria, es fundamental limitar los medios de conexión física e inalámbrica que puedan representar riesgos. Este apartado aborda la implementación de medidas básicas de protección, como la deshabilitación de puertos USB, conexiones Ethernet o Wi-Fi, y la desactivación del Bluetooth. Estas acciones ayudan a prevenir accesos no autorizados, fugas de información y otros vectores comunes de ataque en sistemas expuestos o de uso público.

### Deshabilitar puertos USB y Ethernet o WIFI
- Ejecute
  ```
  sudo -i
  crontab -e
  ```
  
- Adicione al final
  ```
  @reboot echo 'usb1' | tee /sys/bus/usb/drivers/usb/unbind; echo 'usb1'
  @reboot echo 'usb2' | tee /sys/bus/usb/drivers/usb/unbind; echo 'usb2'
  @reboot echo 'usb3' | tee /sys/bus/usb/drivers/usb/unbind; echo 'usb3'
  @reboot echo 'usb4' | tee /sys/bus/usb/drivers/usb/unbind; echo 'usb4'
  #@reboot sudo ifconfig eth0 down
  @reboot sudo ifconfig wlan0 down
  ```
> [!NOTE]
> Comente con `#` la linea de los puertos que son necesarios dejar habilitados.

### Desactivar Bluetooth
- Ejecute
  ```
  sudo -i
  sudo nano /boot/firmware/config.txt
  ```
  
- Adicione al final del archivo
  ```
  # Desactivar bluetooth
  dtoverlay=disable-bt
  ```
> [!NOTE]
> Para habilitarlo solo debera comentar la linea y reiniciar.

### Modo de solo lectura de la tarjeta SD
- Ejecute
  ```
  sudo raspi-config
  ```
- Active `Overlay File System`  
  Ahora, navegue entre las opciones listadas a continuación y establezca el valor señalado.
  - **Performance options**
  	- Overlay File System
  		- :yes
> [!NOTE]
> Para habilitar la escritura debera rever el `Overlay File System` .

Por último, finalice y reinicie la Raspberry.


## OPCIONALES
A continuación, se explica como configurar algunos opcionales.

### ACCESO REMOTO CON TIGHTVNC SERVER
- Ejecute
  ```
  tightvncserver
  ```

La primera vez deberá especificar la contraseña, por ejemplo, `passwordPiVNC*`. Para conectarse use la `IP` y el puerto `5901`.
> [!NOTE] 
> **tightvncserver**: Iniciar el servicio de VNC

### AUTOMATIZACIÓN DE TAREAS CON CRONTAB
Con `crontab`, tienes control total sobre cuándo y cómo se ejecutan los trabajos. Puedes usarlo para reiniciar la Raspberry de forma automática cada cierto tiempo. `crontab` tiene bajos requisitos de recursos ya que no reserva memoria del sistema cuando no se está ejecutando.

Definir el trabajo 
- Ejecute
  ```
  crontab -e
  ```
- Adicione al final
  ```  
  * */6 * * * reboot now >/dev/null 2>&1
  @rebot date>>/home/pi/date.log
  ```

Habilitar el servicio
- Ejecute 
  ```
  sudo systemctl status cron.service
  sudo systemctl enable cron.service
  ```
> [!NOTE] 
> **crontab**: Reiniciar la Raspberry cada seis horas


## Links de interés
> Créditos a cada autor :+1:
> 
> - https://www.raspberrypi.com/tutorials/how-to-use-a-raspberry-pi-in-kiosk-mode/
> - https://core-electronics.com.au/guides/raspberry-pi-kiosk-mode-setup/
> - https://crontab-generator.org/
> - https://phoenixnap.com/kb/crontab-reboot
> - https://github.com/geerlingguy/pi-kiosk/tree/master
> 
> Excelente material, no olvides pasar a verlos.
