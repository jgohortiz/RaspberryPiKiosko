<!-- 
https://www.raspberrypi.com/tutorials/how-to-use-a-raspberry-pi-in-kiosk-mode/
https://core-electronics.com.au/guides/raspberry-pi-kiosk-mode-setup/
-->

# RaspberryPiKiosko
Raspberry Pi en modo quiosco

## CONFIGURAR SSH

### Edite el archivo de configuración
```
nano /etc/ssh/sshd_config
```

### Establezca los siguientes parámetros
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

### Reinicie el servicio
```
sudo systemctl restart ssh
```

## BANNER DE ENTRADA

## Edite el archivo de configuración
```
nano /etc/issue
```

## INSTALAR SO
Como sistema operativo, elige Raspberry Pi OS, instalalo usando Raspberry Pi Imager. Durante la etapa de personalización del sistema operativo, edite la configuración de la siguiente manera:
- Ingrese un nombre de host de su elección
- Ingrese un nombre de usuario y una contraseña
- Para conexión inalámbrica
	- Marque la casilla junto a Configurar LAN inalámbrica para que su Pi pueda conectarse automáticamente a Wi-Fi
	- Ingrese el SSID (nombre) y la contraseña de su red
- Marque la casilla junto a Habilitar SSH para que podamos conectarnos al Pi por Putty

## CONEXIÓN SSH
Para conectarse por SSH use Putty. Utilice el nombre de host o la dirección IP, el puerto por defecto es el 22.

## CONFIGURAR EL MODO QUIOSCO

### Actualice
```
sudo apt update
sudo apt full-upgrade
```

### Instale wtype + ydotool
```
sudo apt install wtype
sudo apt install ydotool
sudo apt install tightvncserver
```

### CONFIGURAR EL INICIO EN DESKTOP (MODO ESCRITORIO) CON AUTOLOGIN
```
sudo raspi-config
```
- System Options 
	- Boot
		- Desktop
- System Options 
	- Autologin
		- Yes
- Interface Option 
	- VNC
		- Yes		
- Advanced Options
	- Wayland
		- wayfire
- Finish
	- Reboot
	
### INICIAR RASPBERRY PI EN MODO QUIOSCO

> [!NOTE] 
> **run_kiosk.sh**: Este script ejecuta el navegador

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
chromium-browser https://time.is/London --kiosk --noerrdialogs --disable-infobars --no-first-run --ozone-platform=wayland --enable-features=OverlayScrollbar --start-maximized
```
- Otorge permisos
```
sudo chmod +x /home/pi/run_kiosk.sh
```
> [!NOTE] 
> **hide_cursor.sh**: Este script mueve el puntero

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
> [!NOTE] 
> **wayfire.ini**: Este archivo hace el llamado inicial  los script

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

> [!NOTE] 
> **tightvncserver**: Iniciar el servicio de VNC

- Ejecute
```
tightvncserver
```
- La primera vez deberá especificar la contraseña
```
Password >> passwordPiVNC*
```
Para conectarse use la IP y el puerto 5901
