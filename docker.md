Paquete docker --> docker.io

Construir nueva imagen --> docker build -t ismael:v1 .

Para borrar imagenes <none> en docker --> docker rmi $(docker images -f "dangling=true" -q)

Para levantar apache con docker --> docker run -d --name ismael -p 8080:80 ismael:v1

Para ver las máquinas levantadas --> docker ps

Desactivar contenedor que está activo --> docker rm -f ismael

----------------------------------------

Creamos un nuevo contenedor y en el fichero `Dockerfile` la configuración para crear el contenedor:

~~~
FROM debian
RUN apt-get update -y && apt-get install -y \
    apache2 \
    && apt-get clean && rm -rf /var/lib/apt/lists/*
COPY ./mkdocs /var/www/html/
ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
~~~

Creamos la imagen con el usuario de docker hub --> `docker build -t ismaelse95/mkdocs:v1 .`

Accedemos a docker hub (máquina desarrollo) --> `docker login`

Para subir la imagen a docker desde la máquina de desarrollo --> `docker push ismaelse95/mkdocs:v1`

En la máquina de producción hacemos el pull --> `docker pull ismaelse95/mkdocs:v1`

En producción levantamos el contenedor --> `docker run -d --name ismael -p 8080:80 ismaelse95/mkdocs:v1`