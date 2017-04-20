# Documento Práctica 3 SWAP
## Granada a 20/04/2017 Alberto Villanueva Copado

### 1º-Instalación Nginx y haproxy en Ubuntu
Para la instalación de Nginx en ubuntu necesitamos ejecutar el comando:
`sudo apt-get install Nginx`
Posteriormente instalaremos haproxy usando el comando:
`sudo apt-get install haproxy`

### 2º-Configuración de Nginx y haproxy
La configuración por defecto de ambos balanceadores no nos interesa, por lo que debemos modificarla de la siguiente manera:
Para  Nginx modificaremos el fichero  `/etc/nginx/conf.d/default.conf` añadiendo lo siguiente:

`upstream apaches {
 server 192.168.56.101;
 server 192.168.56.102;
}
server{
 listen 80;
 server_name balanceador;
 access_log /var/log/nginx/balanceador.access.log;
 error_log /var/log/nginx/balanceador.error.log;
 root /var/www/;
location /
 {
 proxy_pass http://apaches;
 proxy_set_header Host $host;
 proxy_set_header X-Real-IP $remote_addr;
 proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
 proxy_http_version 1.1;
 proxy_set_header Connection "";
 }
}`

En el caso de haproxy modificaremos el fichero en la ruta: `/etc/haproxy/ifconfig/haproxy.cfg` y añadimos lo siguiente:

`global
daemon
maxconn 256
defaults
mode http
contimeout 4000
clitimeout 42000
srvtimeout 43000
frontend http-in
bind *:80
default_backend servers
backend servers
server m1 192.168.56.101:80 maxconn 32
server m2 192.168.56.102:80 maxconn 32`

### 3ºActivar los balanceadores y realizar curl
Lo primero es comprobar si los balanceadores han sido configurados correctamente, para ello intentamos lanzarlos con el comando `sudo service nginx/haproxy start` si se inician sin problemas es que hemos realizado la configuración correctamente, como podemos ver en la figura 3.1 3.2:

![Figura 3.1](http://i.imgur.com/CJwKH3U.png "Figura 3.1")
![Figura 3.2](http://i.imgur.com/csON5k9.png) "Figura 3.2")

Una vez comprobado que ambos servicios funcionan, pasamos a realizar un curl desde la máquina anfitrion (windows 10 en este caso) a la máquina balanceadora, para ello accedemos a la terminal de windows y usamos el comando :
`curl ip_maquina_balanceadora`
Como podemos ver en la figura 3.3:
![Figura 3.3](http://i.imgur.com/k49Uod1.png "Figura 3.3")



