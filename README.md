<!-- 
https://www.raspberrypi.com/tutorials/how-to-use-a-raspberry-pi-in-kiosk-mode/
https://core-electronics.com.au/guides/raspberry-pi-kiosk-mode-setup/
-->

# RaspberryPiKiosko
El objetivo de esta guía es explicar, paso a paso, cómo configurar una **Raspberry Pi 5** para mostrar una página web en modo quiosco utilizando **Chromium** sobre **Raspberry Pi OS de 64 bits**.


## 1. INSTALAR EL SISTEMA OPERATIVO **Raspberry Pi OS de 64 bits**
Como sistema operativo, elige Raspberry Pi OS, instalalo usando Raspberry Pi Imager. Durante la etapa de personalización del sistema operativo, edite la configuración de la siguiente manera:
- Ingrese un nombre de host de su elección. 
- Ingrese un nombre de usuario y una contraseña.
- Para conexión inalámbrica.
	- Marque la casilla junto a Configurar LAN inalámbrica para que su Pi pueda conectarse automáticamente a Wi-Fi
	- Ingrese el SSID (nombre) y la contraseña de su red
- Marque la casilla junto a Habilitar SSH para que podamos conectarnos al **Raspberry Pi 5** por Putty.


## 2. CONFIGURAR SSH (Básica)
Para mejorar la seguridad de la conexión SSH, configuraremos algunos parámetros iniciales; más adelante, reforzaremos aún más estas medidas.

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
A continuación crearemos los scrips necesarios	.

### run_kiosk.sh
Este script ejecuta el navegador

- Cree el archivo con el script de inicio
  ```
  sudo nano /home/pi/run_kiosk.sh
  ```

- Adicione el siguiente contenido
  ```
  sudo killall -I chromium
  sleep 5
  rm -rf ~/.cache/chromium
  sleep 5
  chromium-browser https://time.is/London --kiosk --noerrdialogs --disable-infobar --no-first-run --ozone-platform=wayland --enable-features=OverlayScrollbar --start-maximized
  ```

- Otorge permisos
  ```
  sudo chmod +x /home/pi/run_kiosk.sh
  ```

### hide_cursor.sh
Este script mueve el puntero

- Cree el archivo con el script de inicio
  ```
  sudo nano /home/pi/hide_cursor.sh
  ```

- Adicione el siguiente contenido
  ```
  sleep 15
  sudo ydotool mousemove --delay 1000 10000 10000
  ```

- Otorge permisos
  ```
  sudo chmod +x /home/pi/hide_cursor.sh
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
  kiosk = /home/pi/run_kiosk.sh
  cursor = /home/pi/hide_cursor.sh
  screensaver = false
  dpms = false
  ```

## 6. OPCIONALES
A continuación, se explica como configurar algunos opcionales.

> [!NOTE] 
> **tightvncserver**: Iniciar el servicio de VNC

- Ejecute
  ```
  tightvncserver
  ```

La primera vez deberá especificar la contraseña, por ejemplo, `passwordPiVNC*`. Para conectarse use la `IP` y el puerto `5901`.
