# RaspberryPiKiosko
Raspberry Pi en modo quiosco

##CONFIGURAR SSH
### Edite el archivo de configuración

nano /etc/ssh/sshd_config


### Establezca los siguientes parámetros

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

### Reinicie el servicio
sudo systemctl restart ssh

##BANNER DE ENTRADA
## Edite el archivo de configuración

nano /etc/issue


