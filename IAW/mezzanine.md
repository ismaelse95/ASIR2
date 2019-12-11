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

![Primera página](img/mezzanine1.png)

## Personaliza la página y añade contenido.

![Primera página](img/mezzanine2.png)

## Guarda los ficheros generados durante la instalación en un repositorio github. Guarda también en ese repositorio la copia de seguridad de la bese de datos.

Primero hacemos las copias de seguridad de nuestra base de datos.

~~~
python manage.py dumpdata > db.json
~~~

~~~
python manage.py dumpdata admin > admin.json
~~~

Ahora subimos al repositorio los archivos.


