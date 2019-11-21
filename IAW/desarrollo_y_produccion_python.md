# Entorno de desarrollo y producción con aplicaciones web python.

## Tarea 1

Vamos a clonar el repositorio indicado en la tarea:

~~~
git clone https://github.com/jd-iesgn/iaw_gestionGN
~~~

Ahora pasamos a crear el virtualenv y entramos en el entorno:

~~~
python3 -m venv desa_prod_python
~~~

~~~
source desa_prod_python/bin/activate
~~~

Instalamos el fichero requirement.txt del repositorio:

~~~
pip3 install -r requirements.txt
~~~

Creamos la base de datos:

~~~
python3 manage.py migrate
~~~

Añadimos los datos de prueba del fichero datos.json:

~~~
./manage.py loaddata datos.json
~~~

Activamos la pagina y comprobamos que estan todos los datos del fichero datos.json:

~~~
python3 manage.py runserver
~~~

![Primera página](img/python1.png)

Entramos en la zona usuario:

![Primera página](img/python2.png)

## TAREA 2

