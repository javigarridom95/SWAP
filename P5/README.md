**Práctica 5**
==============

1°. Crear una BD con al menos una tabla y algunos datos.

<img src="https://github.com/javigarridom95/SWAP/P5/blob/master/imagenes/img1.png">

Para crear una base de datos introduciremos los siguientes comandos:

mysql -u root -p
mysql> create database amigos;
mysql> use amigps;
mysql> show tables;
mysql> create table info(nombre varchar(20),tlf int);
mysql> show tables;
Ahora introduciremos los datos en la tabla:

mysql> insert into datos(nombre,tlf) values ("Carlos",685943689);
mysql> select * from info;

<img src="https://github.com/javigarridom95/SWAP/P5/blob/master/imagenes/img2.png">

<img src="https://github.com/javigarridom95/SWAP/P5/blob/master/imagenes/img3.png">


2°. Realizar la copia de seguridad de la BD completa usando mysqldump.

Antes de hacer la copia de seguridad en el archivo .SQL debemos evitar que se acceda a la BD para cambiar nada.

mysql -u root -p
mysql> FLUSH TABLES WITH READ LOCK; 
mysql> quit
Ahora ya sí podemos hacer el mysqldump para guardar los datos.

mysqldump amigos -u root -p > /tmp/amigos.sql
Ahora desbloqueamos las tablas:

mysql -u root -p
mysql> UNLOCK TABLES; 
mysql> quit


3°. Restaurar dicha copia en la segunda máquina (clonado manual de la BD).

Ya podemos ir a la máquina esclavo (maquina2, secundaria) para copiar el archivo .SQL con todos los datos salvados desde la máquina principal (maquina1):

scp root@192.168.56.105:/root/amigos.sql /tmp/

<img src="https://github.com/javigarridom95/SWAP/P5/blob/master/imagenes/img4.png">

Con el archivo de copia de seguridad en el esclavo ya podemos importar la BD completa en el MySQL. Para ello, en un primer paso creamos la BD:

mysql -u root -p
mysql> CREATE DATABASE amigos; 
mysql> quit
Y en un segundo paso restauramos los datos contenidos en la BD:

mysql -u root -p amigos < /tmp/amigos.sql

<img src="https://github.com/javigarridom95/SWAP/P5/blob/master/imagenes/img5.png">


4°. Realizar la configuración maestro-esclavo de los servidores MySQL para que la replicación de datos se realice automáticamente.

Lo primero que debemos hacer es la configuración de mysql del maestro. Para ello editamos, como root, el /etc/mysql/my.cnf. Una vez aqui, comentamos:

#bind-address 127.0.0.1
Y descomentamos:

log_error = /var/log/mysql/error.log
server-id = 1
log_bin = /var/log/mysql/bin.log
Una vez realizados dichos cambios, guardamos el documento y reiniciamos el servicio con la siguiente instrucción:

/etc/init.d/mysql restart
A continuación introducimos los siguientes comandos en mysql para configurar la máquina "MAESTRO":

mysql> CREATE USER esclavo IDENTIFIED BY 'esclavo';
mysql> GRANT REPLICATION SLAVE ON *.* TO 'esclavo'@'%' IDENTIFIED BY 'esclavo';
mysql> FLUSH PRIVILEGES;
mysql> FLUSH TABLES;
mysql> FLUSH TABLES WITH READ LOCK;
Por último introducimos los siguientes comandos en mysql para configurar la máquina "ESCLAVO":

mysql> CHANGE MASTER TO MASTER_HOST='192.168.98.128', MASTER_USER='esclavo', MASTER_PASSWORD='esclavo', MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=980, MASTER_PORT=3306;
mysql> START SLAVE;

<img src="https://github.com/javigarridom95/SWAP/P5/blob/master/imagenes/img7.png">

<img src="https://github.com/javigarridom95/SWAP/P5/blob/master/imagenes/img8.png">

<img src="https://github.com/javigarridom95/SWAP/P5/blob/master/imagenes/img9.png">


