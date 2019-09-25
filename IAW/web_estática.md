1.- Comenta la instalación del generador de página estática. Recuerda que el generador tienes que instalarlo en tu entorno de desarrollo. Indica el lenguaje en el que está desarrollado y el sistema de plantillas que utiliza. (1 punto)

Lo primero que vamos a hacer es crear un entorno virtual, empezamos instalando los paquetes que necesitamos que serán virtualenv y python3:

~~~
	sudo apt-get install python3-pip
~~~

~~~
	pip install virtualenv
~~~

~~~
	sudo pip3 install virtualenv
~~~

Una vez instalados los paquetes pasamos a crear el entorno virtual:

~~~
	virtualenv nombre_de_tu_entorno -p python3
~~~

Con este comando crearemos una carpeta y aqui es donde instalaremos todos los paquetes que nos
hagan falta para usar nuestro entorno.

Para activar el entorno o desactivarlo tendremos que poner los siguientes comandos:

Activar entorno: 
~~~
	source nombre_entorno_virtual/bin/activate
~~~

Desactivar entorno: 
~~~
	deactivate
~~~