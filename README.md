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

### Instale wtype
```
sudo apt install wtype
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
- Advanced Options
	- Wayland
		- wayfire
- Finish
	- Reboot
	
### INICIAR RASPBERRY PI EN MODO QUIOSCO

#### Edite el archivo de configuración
```
sudo nano .config/wayfire.ini
```

#### Establezca los siguientes parámetros
```
sudo nano /home/pi/.config/wayfire.ini
```

#### ADICIONE EN [AUTOSTART]
```
[autostart]
chromium = chromium-browser https://raspberrypi.com https://time.is/London --kiosk --noerrdialogs --disable-infobars --no-first-run --ozone-platform=wayland --enable-features=OverlayScrollbar --start-maximized
switchtab = bash ~/switchtab.sh
screensaver = false
dpms = false
```
