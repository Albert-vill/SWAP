# Documento práctica 2 SWAP
## Granada 21/03/2017## Alberto Villanueva Copado

### 1ºClonación de un archivo de una máquina a otra usando ssh
se ha copiado el fichero archivo.txt desde la máquina2 hasta la máquina1 usando shh, para ello hemos usado el comando scp cuya sintaxis es la siguiente:
`scp archivo.txt 192.168.56.101:/home/alberto`
En la figura 1.1 se puede apreciar la ejecución del comando y la comprobación en la otra máquina de que el fichero existe:
![Figura1.1](http://i.imgur.com/RWozKVS.png "Figura 1.1")
(la terminal de la izquierda corresponde a la máquina2 y la de la derecha a la máquina1)

### 2ºClonación de una carpeta con ssh
Se ha realizado la clonación de la carpeta /www contenida en /var desde una maquina virtual a otra, para ello se ha ejecutado el comando:
` rsync -avz -e ssh 192.168.56.101:/var/www/ /var/www/`
el resultado puede verse en la figura 2.1:
![Figura 2.1](http://i.imgur.com/h9HS8Xg.png "Figura 2.1")

### 3ºAcceso ssh sin usar contraseñana
Se ha configurado un acceso sin contraseña desde la máquina2 hasta la máquina1, para ello se ha usado el comando:
`ssh-keygen -b 4096 -t rsa`
En la figura 3.1 se puede ver la ejecución del comando:
![Figura 3.1](http://i.imgur.com/xkgpd9r.png "Figura 3.1")
Con esto hemos generado una clave rsa la cual hemos compartido con la máquina usando con el comando:
`ssh-copy-id 192.168.56.101` (siendo 192.168.56.101 la ip de la máquina1)
Posteriormente realizamos un acceso a la máquina1 con el comando:
`ssh 192.168.56.101`
El cual debería permitirnos conectarnos sin necesidad de introducir la contraseña. En la figura 3.2 se puede ver la ejecución del comando de copia y la conexión ssh:
![Figura 3.2](http://i.imgur.com/g702WXz.png "Figura 3.2")

### 4º Generación Script crontab
Se ha modificado el contenido del archivo /etc/crontab para añadirle una orden que mantenga actualizado el contenido del fichero /var/www de ambas máquinas. Incialmente se quería añadir directamente la orden al fichero /etc/crontab, pero no la reconocía. Por lo que se ha diseñado un script con la orden y se ha ordenado a crontab que ejecute el script. El script es el siguiente:
`#!/bin/bash`
`rsync -avz -e ssh 192.168.56.101:/var/www/ /var/www/`

En el fichero crontab hemos añadido la orden:
`* * * * * alberto sh /home/alberto/script.sh`

este script copia el contenido de la carpeta /var/www/ de la máquina1 en la máquina2. Para demostrarlo hemos creado un fichero "fichero.html" en la carpeta /var/www/html de la máquina 1 y hemos esperado a que la máquina2 lo copie con el script. En la figura 4.1 se puede ver el script y el estado final del fichero /etc/crontab. En la figura 4.2 puede verse la comprobación de la copia del fichero:
![Figura 4.1](http://i.imgur.com/DHlIy4P.png "Figura 4.1")
![Figura 4.2](http://i.imgur.com/zSHb0pX.png "Figura 4.2")


