#TOMCAT Y JAVA

Instalacion de tomcat:

~~~
apt install tomcat9
apt install tomcat9-admin
~~~

##ENTORNO GRAFICO 

nano /etc/tomcat9/tomcat-users.xml --> crear usuario

~~~
	<role rolename="manager-gui"/>
	<user username="tomcat" password="s3cret" roles="manager-gui"/>
~~~

Accedemos al hostname:8080/manage


##TERMINAL

cd /var/lib/tomcat9/webapps --> para ver los war que tenemos instalados en nuestro tomcat.
Para introducir un war en el misma ruta (/var/lib/tomcat9/webapps) cogemos el enlace war y lo enviamos a la ruta con el comando wget.
Mas opciones de tomcat en el siguiente [link](https://fp.josedomingo.org/iawgs/u05/tomcat8.html).