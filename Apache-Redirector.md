# Guía para Generar y Configurar un Certificado TLS en Apache

## Pasos

```bash
# 1. Generar la Clave Privada
openssl genrsa -out private.key 2048

# 2. Crear una Solicitud de Firma de Certificado (CSR)
openssl req -new -key private.key -out request.csr

# Durante este paso, se te pedirá que ingreses información para el CSR. Esto puede incluir:
# - Nombre del país
# - Estado
# - Localidad
# - Organización
# - Unidad organizacional
# - Nombre común (dominio)
# - Dirección de correo electrónico

# 3. Crear un Archivo de Configuración para las Extensiones del Certificado
# Crea un archivo llamado `ca.ext` con el siguiente contenido:
# ----------------------------------------
# authorityKeyIdentifier=keyid,issuer
# basicConstraints=CA:FALSE
# keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
# subjectAltName = @alt_names
#
# [alt_names]
# DNS.1 = test.xyz
# DNS.2 = www.test.xyz
# ----------------------------------------

# 4. Firmar el CSR con la CA para Obtener el Certificado
# Asumiendo que tienes acceso a los archivos de la CA (`ca.crt` y `ca.key`):
openssl x509 -req -in request.csr -CA ca/ca.crt -CAkey ca/ca.key -CAcreateserial -out public.crt -days 365 -sha256 -extfile ca/ca.ext

# 5. Verificar el Certificado
openssl x509 -noout -text -in public.crt

# 6. Transferir los Archivos al Redirector
scp private.key attacker@[redirector]:/home/attacker/
scp public.crt attacker@[redirector]:/home/attacker/

# 7. Configurar Apache en el Redirector
# En el redirector, mueve los archivos a los directorios apropiados:
sudo cp /home/attacker/private.key /etc/ssl/private/
sudo cp /home/attacker/public.crt /etc/ssl/certs/

# Edita el archivo de configuración SSL de Apache:
sudo nano /etc/apache2/sites-enabled/default-ssl.conf

# Asegúrate de que las rutas a los archivos de certificado sean correctas:
# ----------------------------------------
# SSLCertificateFile /etc/ssl/certs/public.crt
# SSLCertificateKeyFile /etc/ssl/private/private.key
#
# SSLProxyEngine on
#
# <Directory /var/www/html/>
#     Options Indexes FollowSymLinks MultiViews
#     AllowOverride All
#     Require all granted
# </Directory>
# ----------------------------------------

# 8. Reiniciar Apache
sudo systemctl restart apache2

# 9. Verificar la Configuración del Certificado
# Abre tu navegador y visita https://[redirector] para verificar que el certificado está funcionando correctamente.

# SSH Tunnel (HTTPS)
# En el cliente (attacker), copia el certificado:
scp localhost.crt attacker@[redirector]:/home/attacker/

# En el redirector, instala el certificado y actualiza:
sudo cp localhost.crt /usr/local/share/ca-certificates/
sudo update-ca-certificates

# Verifica el túnel:
curl -v https://localhost:8443/r1   # Debería funcionar sin error
curl -v -k https://[attacker]      # Debería fallar

# Configura el túnel SSH en el cliente:
ssh -N -R 8443:localhost:443 attacker@[redirector]

# AutoSSH en el cliente (attacker), edita la configuración SSH:
nano ~/.ssh/config
# ----------------------------------------
# Host                 redirector-1
# HostName             10.10.0.100
# User                 attacker
# Port                 22
# IdentityFile         /home/attacker/.ssh/id_rsa
# RemoteForward        8443 localhost:443
# ServerAliveInterval  30
# ServerAliveCountMax  3
# ----------------------------------------
autossh -M 0 -f -N redirector-1

# SSH Tunnel (DNS)
# Configura el túnel SSH para DNS:
ssh -N -R 5353:localhost:5353 attacker@[redirector]

# En el redirector, usa `socat` para redirigir las peticiones:
sudo socat tcp-listen:5353,reuseaddr,fork udp4-sendto:localhost:53
sudo socat udp4-listen:53,reuseaddr,fork tcp:localhost:5353,retry,forever

# Agrega comandos `socat` al cron:
sudo nano /etc/cron.d/redirect
# ----------------------------------------
# @reboot root sudo socat tcp-listen:5353,reuseaddr,fork udp4-sendto:localhost:53
# ----------------------------------------

# AutoSSH en el cliente (attacker), edita la configuración SSH:
nano ~/.ssh/config
# ----------------------------------------
# Host                 redirector-2
# HostName             10.10.0.200
# User                 attacker
# Port                 22
# IdentityFile         /home/attacker/.ssh/id_rsa
# RemoteForward        5353 localhost:5353
# ServerAliveInterval  30
# ServerAliveCountMax  3
# ----------------------------------------
autossh -M 0 -f -N redirector-2

# Startup Service
# Crea un servicio de inicio para cada redirector:
sudo nano /etc/systemd/system/redirector.service
# ----------------------------------------
# [Unit]
# Description=SSH Tunnel to Redirector
#
# [Service]
# Type=forking
# User=attacker
# Restart=always
# RestartSec=1
# ExecStart=/usr/bin/autossh -M 0 -f -N redirector
#
# [Install]
# WantedBy=multi-user.target
# ----------------------------------------

# Habilita y arranca el servicio:
sudo systemctl daemon-reload
sudo systemctl enable redirector.service
sudo systemctl start redirector.service
