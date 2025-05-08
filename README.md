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
