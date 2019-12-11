# Instalación Mezzanine


## Instala el CMS en el entorno de desarrollo. Debes utilizar un entorno virtual. 

Vamos a instalar mezzanine en un entorno virtual para ello creamos el entorno y entramos en el entorno:

~~~
python3 -m venv mezzanine
~~~

~~~
source mezzanine/bin/activate
~~~

Creamos los fichero de mezzanine.

~~~
mezzanine-project myproject
~~~

Creamos la base de datos.

~~~
python manage.py createdb
~~~

Y a continuación iniciamos mezzanine.

~~~
python manage.py runserver
~~~

![Primera página](img/mezzanine.png)