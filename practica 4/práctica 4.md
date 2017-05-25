#Documento Práctica 4 SWAP
##Granada a 20/04/2017 Alberto Villanueva Copado

Cuando se realizó este documento no se podía acceder a las maquinas virtuales debido a un error producido por una actualización del S.O es por eso que este documento carece de imagenes.

### 1º-Instalación Certificado SSl autofirmado
Lo primero es activar el módulo SSl de Apache, generar los certificados y especificarles la ruta donde se almacenarán:
`a2enmod ssl`
`service apache2 restart`
`mkdir /etc/apache2/ssl`
`openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout`
`/etc/apache2/ssl/apache.key -out /etc/apache2/ssl/apache.crt`
Cuando introduzcamos estos comandos, deberemos rellenar los campos de información sobre el certificado. Una vez rellenado accedemos al documento de configuración de default-ssl y añadimos 

`SSLCertificateFile /etc/apache2/ssl/apache.crt`
`SSLCertificateKeyFile /etc/apache2/ssl/apache.key`

### 2º-Configuración del cortafuegos e iptables
Debemos generar un script que se ejecute con el inicio de sistema de forma que el servidor solo acepte conexiones de tipo http o https. Para ellos estableceremos reglas usando ip tables.
Este es el script que hemos generado:

#(1) Eliminar todas las reglas (configuración limpia)
iptables -F

iptables -X

iptables -Z

iptables -t nat -F

 (2) Política por defecto: denegar todo el tráfico
 
iptables -P INPUT DROP

iptables -P OUTPUT DROP

iptables -P FORWARD DROP

 (3) Permitir cualquier acceso desde localhost (interface lo)

iptables -A INPUT -i lo -j ACCEPT

iptables -A OUTPUT -o lo -j ACCEPT

iptables -A INPUT -p tcp --dport 22 -j ACCEPT

iptables -A OUTPUT -p tcp --sport 22 -j ACCEPT

 (5) Abrir los puertos HTTP (80) de servidor web

iptables -A INPUT -p tcp --dport 80 -j ACCEPT

iptables -A OUTPUT -p tcp --sport 80 -j ACCEPT
 
Este script debe ser introducido en el ficher rc.local, el cual lanzará todos los comandos incluidos en él al iniciar el sistema. El problema es que al usar un sistema Linux con sistemd, este método no funciona por lo que debemos buscar otra solución.

En este caso hemos decidido generar un demonio el cual ejecuta el script al inicio del sistema. Para ello necesitaremos generar un .service además del script anteriormente mencionado.
Este es el .service que hemos generado:

[Unit]

Description=okboard reset to avoid fails

After=syslog.target

[Service]

Type=forking

ExecStart=/home/nemo/ejemplo.sh

[Install]

WantedBy=multi-user.target

Donde ejemplo.sh es el script que hemos generado anteriormente con las reglas de ip table. Es importante recordar, que el script anterior debe comenzar por  #!/bin/sh para que el sistema entienda que es un script ejecutable de bash en shell.
Una vez hemos hecho todo esto recargamos systemd usando  
`systemctl daemon-reload`
y lo lanzamos usando
`systemctl start ejemplo.sh`
si el mensaje que obtenemos es que el proceso ha sido lanzado sin problemas, solo tenemos que activarlo en el servicio de autoarranque usando:
`systemctl enable ejemplo.service`



