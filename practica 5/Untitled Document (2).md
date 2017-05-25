#Documento Práctica 5 SWAP
##Granada a 20/04/2017 Alberto Villanueva Copado


###1º-Generación base de datos
El obejtivo de la práctica es la creación de una estructura maestro-esclavo entre las bases de datos de dos servidores web diferentes. Para ello lo primero que tenemos que hacer es generar la base de datos en el servidor maestro:
1.Accedemos a mysql en el servidor maestro
`mysql -u root -p`
2.Creamos una base de datos
`CREATE DATABASE 'contactos';`
3.Accedemos a ella
`USE contactos;`
4.Creamos una tabla en la base de datos
`CREATE TABLE datos (nombre varchar(100),tlf int);`
5.Introducimos datos en la tabla
`INSERT INTO datos (nombre, tlf) values ("paco", 65142878);`
6.Mostramos el resultado final de la table
`SELECT *. FROM DATOS;`
7.Mostramos las tablas de la base de datos
`SHOW TABLES;`

En la figura 1.1 y 1.2 pueden verse la salida de los comandos del paso 6 y 7 respectivamente.

![Figura 1.1](http://i.imgur.com/2ohP5tX.png "Figura 1.1")
![Figura 1.2](http://i.imgur.com/DkAObbM.png "Figura 1.2")

Una vez tenemos la base de datos creada en el servidor maestro vamos a bloquear las tablas de forma que no se produzca ningún acceso a la bd:
`FLUSH TABLES WITH READ LOCK;`
Salimos de mysql y pasaremos a volcarla en el servidor esclavo, para ello usaremos la orden mysqldump, recordando que mysqldump contiene los comandos para recrear el estado de una bd, pero no la creación de la bd en si misma, por lo que necesitamos tenerla creada previamente en el servidor esclavo. Para ello repetimos el paso 3 y volvemos a la máquina maestro para volcar la bd en un archivo sql, para ello usamos:
`mysqldump contactos -u root -p > /root/contactos.sql`
Una vez tenemos el fichero sql en la máquina maestra y la bd creada en la máquina esclavo vamos a copiar el fichero sql en la máquina esclavo:
`scp 192.168.30.128:/tmp/ejemplodb.sql /tmp/`
Sin olvidarnos de desbloquear las tablas de la bd desde el servidor maestro
`UNLOCK TABLES`

###2º-Configuración de mysql
Llegados a este punto es momento de modificar los ficheros de configuración de mysql para establecer la estructura maestro esclavo. Primero debemos acceder al fichero 
`/etc/mysql/mysql.conf.d/mysqld.cnf`
Allí realizaremos los siguientes cambios:
1.Comentar la linea de bind-address
2.Descomentar la linea de server-id darle valor = 1 
3.Descomentar log_bin
Guardamos el resultado del fichero y recargamos el servicio usando
`service mysql restart`
Debemos repetir este proceso en la máquina esclavo pero dandole valor = 2 a server-id. Una vez hecho esto y si no hemos obtenido ningún error proseguimos con la configuración dinámica desde mysql.
Desde la máquina maestro debemos introducir los siguientes comandos:
`CREATE USER esclavo IDENTIFIED BY 'esclavo';`
`GRANT REPLICATION SLAVE ON *.* TO 'esclavo'@'%',`
`IDENTIFIED BY 'esclavo';`
`FLUSH PRIVILEGES;`
`FLUSH TABLES;`
Si todo ha salido bien la salida del comando `SHOW MASTER STATUS` debería ser como la de la imagen 2.1

![Figura 2.1](http://i.imgur.com/ztYsVOi.png "Figura 2.1")

Una vez hecho esto podemos pasar a configurar la máquina esclavo. Para ello accedemos a mysql e introducimos los siguientes comandos:
`CHANGE MASTER TO MASTER_HOST='192.168.30.128',`
`MASTER_USER='esclavo', MASTER_PASSWORD='esclavo',`
`MASTER_LOG_FILE='mysql-bin.000006', MASTER_LOG_POS=154,`
`MASTER_PORT=3306;`
Una vez introducido esto, podemos iniciarlo y comprobar el correcto funcionamiento del esclavo usando los comandos: `START SLAVE` y `SHOW SLAVE STATUS \G` En la figura 2.2 podemos ver la salida del comando status

![Figura 2.2](http://i.imgur.com/WEII14y.png "Figura 2.2")

El campo Seconds_Behind_Master nos indicará si lo hemos realizado todo correctamente, en caso de no ser así se mostrará un mensaje de error en alguna de las variables indicandonos que debemos arreglar.
Con esto ya tenemos finalizada y en funcionamiento la configuración maestro-esclavo








