# Introducción aplicaciones python

## TAREA 1

Vamos a instalar django en un entorno virtual para ello creamos el entorno y entramos en el entorno:

~~~
python3 -m venv aplicacion_python
~~~

~~~
source aplicacion_python/bin/activate
~~~

A continuacion copiamos el repositorio de jose domingo e instalamos el fichero requeriment.txt:

~~~
git clone https://github.com/josedom24/django_tutorial
~~~

~~~
pip3 install -r requeriment.txt
~~~

Tendremos que hacer un migrate del manage.py y crear un usuario como administrador.
Una vez echo esto entramos en localhost:8000/admin:

![Primera página](img/django1.png)

Y por último crearemos preguntas con respuestas:

![Primera página](img/django2.png)