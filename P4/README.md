**Práctica 4**
==============

Instalar un certificado SSL autofirmado para configurar el acceso por HTTPS

Para generar un certificado SSL autofirmado en Ubuntu Server debemos activar el módulo SSL de Apache, generar los certificados y especificarle la ruta a los certificados en la configuración.

Por lo que ejecutamos el comando a2enmod ssl y pasarán a activarse una serie de módulos.

a2enmod ssl
Ahora tenemos que reiniciar Apache2.

service apache2 restart

Creamos la carpeta ssl en el directorio de Apache. Aquí guardaremos nuestro certificado SSL.

mkdir /etc/apache2/ssl
Una vez hecho esto, generamos nuestro certificado SSL, dónde nos pedirá una serie de datos para configurar el dominio.

openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/a

<img src="https://github.com/javigarridom95/SWAP/blob/master/P4/imagenes/img1.png">

<img src="https://github.com/javigarridom95/SWAP/blob/master/P4/imagenes/img2.png">

Editamos el archivo de configuración del sitio default-ssl:

nano /etc/apache2/sites-available/default-ssl
Y agregamos estas lineas:

SSLCertificateFile /etc/apache2/ssl/apache.crt 
SSLCertificateKeyFile /etc/apache2/ssl/apache.key
Activamos el sitio default-ssl y reiniciamos apache:

a2ensite default-ssl 
service apache2 reload
Para hacer peticiones por HTTPS utilizando la herramienta curl, ejecutaremos:

curl –k https://192.168.56.105/hola.html


Configurar las reglas del cortafuegos con IPTABLES para asegurar el acceso alos servidores web.

<img src="https://github.com/javigarridom95/SWAP/blob/master/P4/imagenes/img3.png">

Para configurar el cortafuegos utilizamos el siguiente script:

iptables -F
iptables -X
iptables -Z
iptables -t nat -F

iptables -P INPUT DROP
iptables -P OUTPUT DROP

iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

iptables -A INPUT -i eth0 -p tcp -m multiport --dports 22,80,443 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -o eth0 -p tcp -m multiport --sports 22,80,443 -m state --state ESTABLISHED -j ACCEPT


Para ello, añadir en /etc/crontab la siguiente linea:

@reboot /home/javi/script.sh




