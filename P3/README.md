**Práctica 3**
==============


En primer lugar creamos una nueva máquina virtual, la cual configuraremos mas tarde como un balanceador de carga nginx.

Una vez instalado nginx cambiamos el archivo de configuración para unsar un algoritmo round-robin con la misma prioridad para todos los servidores.

upstream apaches {
  server 192.168.98.128; 
  server 192.168.98.129;
}

server{
  listen 80;
  server_name balanceador;
  access_log /var/log/nginx/balanceador.access.log; error_log /var/log/nginx/balanceador.error.log; 
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
}

service nginx restart
Por último comprobamos que todo funciona correctamente mediante la orden curl:

<img src="https://github.com/javigarridom95/SWAP/blob/master/P3/imagenes/img1.png">

Si sabemos que alguna de las máquinas finales es más potente, podemos modificar la definición del upstream para pasarle más tráfico que al resto de las del grupo. Para ello, tenemos un modificador llamado weight, al que le damos un valor numérico que indica la carga que le asignamos.

Por defecto tiene el valor 1, por lo que si no modificamos ninguna máquina del grupo, todas recibirán la misma cantidad de carga:


upstream apaches 
{
    server 192.168.56.105 weight=1; 
    server 192.168.56.110 weight=2;
}

<img src="https://github.com/javigarridom95/SWAP/blob/master/P3/imagenes/img2.png">


Tras instalar el sistema básico, instalamos haproxy.

sudo apt-get install haproxy

Un balanceador sencillo debe escuchar tráfico en el puerto 80 y redirigirlo a alguna de las máquinas servidoras finales. Usemos como configuración inicial la siguiente:

global
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
    server m1 192.168.56.105:80 maxconn 32 
    server m2 192.168.56.110:80 maxconn 32










