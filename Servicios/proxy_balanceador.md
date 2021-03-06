# Proxy y balanceador de carga

En esta práctica vamos a tener por un lado el proxy con dos máquinas en vagrant una de ella será el servidor y la otra un cliente en la red interna para hacer pruebas, por otro lado tendremos el balanceador de carga que tendremos 3 máquinas y la configuración sera la siguiente: [Escenario](https://fp.josedomingo.org/serviciosgs/u08/doc/haproxy/vagrant.zip).

## Proxy

### Tarea 1: Instala squid en la máquina squid y configúralo para que permita conexiones desde la red donde este tu ordenador.

Vamos a empezar instalando squid en nuestro servidor.
~~~
vagrant@proxy:~$ sudo apt install squid
~~~

Ahora entraremos en el fichero de configuración de `/etc/squid/squid.conf` y tendremos que poner las siguientes lineas para crear la acl.
~~~
acl externa src 192.168.0.0/24
http_access allow externa
~~~

Reiniciamos squid.
~~~
vagrant@proxy:~$ sudo systemctl restart squid
~~~

### Tarea 2: Prueba que tu ordenador está navegando a través del proxy (HTTP/HTTPS) configurando el proxy de dos maneras diferentes:

- Directamente indicándolo en el navegador.

Vamos a configurar el navegador firefox, para ello tendremos que entrar en preferencia --> configuracion de red y en la nueva ventana que nos saldrá tenemos que configurarlo de la siguiente forma.

![Inicio Proxy](imagenes/proxy.png)

Accedemos a marca por ejemplo y vemos el log de squid.
~~~
1613986861.519     77 192.168.0.167 TCP_MISS/200 981 POST http://r3.o.lencr.org/ - HIER_DIRECT/2.22.234.56 application/ocsp-response
1613986861.667    199 192.168.0.167 TCP_MISS/200 914 POST http://ocsp.digicert.com/ - HIER_DIRECT/72.21.91.29 application/ocsp-response
1613986861.794    199 192.168.0.167 TCP_TUNNEL/200 5658 CONNECT fundingchoicesmessages.google.com:443 - HIER_DIRECT/172.217.17.14 -
1613986861.796    216 192.168.0.167 TCP_TUNNEL/200 5655 CONNECT fundingchoicesmessages.google.com:443 - HIER_DIRECT/172.217.17.14 -
1613986862.021    199 192.168.0.167 TCP_MISS/200 817 POST http://ocsp.pki.goog/gts1o1core - HIER_DIRECT/216.58.209.67 application/ocsp-response
1613986862.064     74 192.168.0.167 TCP_MISS/200 1008 POST http://ocsp.usertrust.com/ - HIER_DIRECT/151.139.128.14 application/ocsp-response
1613986862.398    438 192.168.0.167 TCP_MISS/200 704 POST http://ocsp.digicert.com/ - HIER_DIRECT/72.21.91.29 application/ocsp-response
1613986862.433    129 192.168.0.167 TCP_TUNNEL/200 18441 CONNECT static.chartbeat.com:443 - HIER_DIRECT/13.32.89.178 -
1613986862.612    200 192.168.0.167 TCP_MISS/200 914 POST http://ocsp.digicert.com/ - HIER_DIRECT/72.21.91.29 application/ocsp-response
1613986862.623    141 192.168.0.167 TCP_MISS/200 2448 POST http://ocsp.starfieldtech.com/ - HIER_DIRECT/192.124.249.23 application/ocsp-response
1613986862.837    107 192.168.0.167 TCP_TUNNEL/200 13008 CONNECT api.unidadeditorial.es:443 - HIER_DIRECT/13.32.91.94 -
1613986863.350   3222 192.168.0.167 TCP_TUNNEL/200 3916 CONNECT pixelcounter.marca.com:443 - HIER_DIRECT/193.110.128.197 -
1613986863.490    228 192.168.0.167 TCP_MISS/200 915 POST http://status.thawte.com/ - HIER_DIRECT/72.21.91.29 application/ocsp-response
~~~

- Configurando el proxy del sistema en el entorno gráfico (tienes que indicar en el navegador que vas a hacer uso del proxy del sistema).

Para configurar el proxy del sistema en el entorno tendremos que especificarlo en el navegador y en nuestro sistema.
Primero vamos al navegador y en la misma pestaña de antes tendremos que seleccionar la siguiente opción.

![Inicio Proxy](imagenes/proxy2.png)

Ahora nos vamos a configuración del sistema nos vamos a red y dentro de red activamos proxy, seleccionamos manual y tendremos que introducir nuestro proxy.

![Inicio Proxy](imagenes/proxy3.png)

Ahora entramos en la página as y comprobamos el fichero de log.
~~~
1613987848.040     21 192.168.0.167 TCP_TUNNEL/200 39 CONNECT as01.epimg.net:443 - HIER_DIRECT/151.101.134.133 -
1613987848.049     21 192.168.0.167 TCP_TUNNEL/200 39 CONNECT as01.epimg.net:443 - HIER_DIRECT/151.101.134.133 -
1613987848.052     20 192.168.0.167 TCP_TUNNEL/200 39 CONNECT as01.epimg.net:443 - HIER_DIRECT/151.101.134.133 -
1613987848.056     20 192.168.0.167 TCP_TUNNEL/200 39 CONNECT as01.epimg.net:443 - HIER_DIRECT/151.101.134.133 -
1613987848.907     26 192.168.0.167 TCP_TUNNEL/200 39 CONNECT asmedia.epimg.net:443 - HIER_DIRECT/151.101.134.133 -
1613987849.647     20 192.168.0.167 TCP_TUNNEL/200 39 CONNECT as01.epimg.net:443 - HIER_DIRECT/151.101.134.133 -
~~~

### Tarea 3: Configura squid para que pueda ser utilizado desde el cliente interno. En el cliente interno configura el proxy desde la línea de comandos (con una variable de entorno). Fíjate que no hemos puesto ninguna regla SNAT y podemos navegar (protocolo HTTP), pero no podemos hacer ping o utilizar otro servicio.

Vamos a configurar nuestra red interna, para ello tendremos que entrar primero en el fichero de configuración de `/etc/squid/squid.conf` y tenemos que realizar lo siguiente.
~~~
acl redinterna src 10.0.0.0/24
http_access allow redinterna
~~~

Reiniciamos squid.
~~~
vagrant@proxy:~$ sudo systemctl restart squid
~~~

Vamos a instalar en nuestro cliente w3m para acceder a una página mediante terminal.
~~~
vagrant@buster:~$ sudo apt install w3m
~~~

Ahora vamos a configurar las variables de entorno.
~~~
root@buster:~# export https_proxy=http://10.0.0.10:3128/
root@buster:~# export http_proxy=http://10.0.0.10:3128/
~~~

Accedemos a la página elmundo y comprobamos el log.
~~~
root@buster:~# w3m elmundo.com
~~~

~~~
1613989082.751    372 10.0.0.11 TCP_MISS/301 524 GET http://elmundo.com/ - HIER_DIRECT/131.0.136.66 text/html
1613989083.519    760 10.0.0.11 TCP_TUNNEL/200 4256 CONNECT elmundo.com:443 - HIER_DIRECT/131.0.136.66 -
1613989088.859   5337 10.0.0.11 TCP_TUNNEL/200 199175 CONNECT www.elmundo.com:443 - HIER_DIRECT/131.0.136.66 -
~~~

### Tarea 4: Con squid podemos filtrar el acceso por url o dominios, realiza las configuraciones necesarias para implementar un filtro que funcione como lista negra (todo el acceso es permitido menos las url o dominios que indiquemos en un fichero.)

Bien, ahora vamos a crear un fichero en nuestra máquina servidor de proxy de listas negras, para ello vamos a crearlo en la ruta `/etc/squid/listanegra` y pondremos la siguiente página.
~~~
www.tutecnoinfor.wordpress.com/
~~~

Ahora tendremos que acceder a la configuracion de `/etc/squid/squid.conf` y añadimos la acl.
~~~
acl listanegra url_regex "/etc/squid/listanegra"
http_access deny listanegra
~~~

Reiniciamos squid.
~~~
vagrant@proxy:~$ sudo systemctl restart squid
~~~

Y comprobamos que no podemos acceder y miramos el log.

![Inicio Proxy](imagenes/proxy4.png)

~~~
1613989800.642     56 192.168.0.167 TCP_TUNNEL/200 39 CONNECT ssl.gstatic.com:443 - HIER_DIRECT/216.58.215.131 -
1613989800.766     49 192.168.0.167 TCP_TUNNEL/200 39 CONNECT apis.google.com:443 - HIER_DIRECT/142.250.184.174 -
1613989804.426      0 192.168.0.167 TCP_DENIED/403 3959 CONNECT www.marca.com:443 - HIER_NONE/- text/html
~~~

### Tarea 5: Realiza las configuraciones necesarias para implementar un filtro que funcione como lista blanca (todo el acceso es denegado menos las url o dominios que indiquemos en un fichero.)

Vamos a crear listas blancas, para ello vamos a crear un fichero de configuración que se llamará `/etc/squid/listasblancas` y pondremos que solo podemos acceder a www.marca.com

Tendremos que entrar en el fichero de configuración squid y tendremos que poner la acl.
~~~
acl listasblancas url_regex "/etc/squid/listasblancas
http_access allow listasblancas
http_access deny all
~~~

Reiniciamos squid.
~~~
vagrant@proxy:~$ sudo systemctl restart squid
~~~

Ahora intentamos acceder a marca y a elmundo y vemos que en marca podemos acceder y elmundo no. Miramos el log de squid también.

![Inicio Proxy](imagenes/proxy5.png)

![Inicio Proxy](imagenes/proxy6.png)

~~~
1613990779.365      0 192.168.0.167 TCP_TUNNEL/200 3974 CONNECT e00-marca.uecdn.es:443 - HIER_NONE/- text/html
1613990780.497      0 192.168.0.167 TCP_DENIED/403 3965 CONNECT www.elmundo.com:443 - HIER_NONE/- text/html
~~~

## Balanceador de carga

### Tarea 1: Entrega capturas de pantalla que el balanceador está funcionando.

Vamos a configurar el [escenario](https://fp.josedomingo.org/serviciosgs/u08/doc/haproxy/vagrant.zip) que tenemos en la página de Jose Domingo.

Una vez iniciado el vagrantfile entramos en la máquina balanceador y vamos a instalar haproxy.
~~~
vagrant@balanceador:~$ sudo apt install haproxy
~~~

Entramos en el fichero `/etc/haproxy/haproxy.cfg` y pondremos la siguiente configuración.
~~~
global
    daemon
    maxconn 256
    user    haproxy
    group   haproxy
    log     127.0.0.1       local0
    log     127.0.0.1       local1  notice
defaults
    mode    http
    log     global
    timeout connect 5000ms
    timeout client  50000ms
    timeout server  50000ms
listen granja_cda
    bind 192.168.0.180:80
    mode http
    stats enable
    stats auth  cda:cda
    balance roundrobin
    server uno 10.10.10.11:80 maxconn 128
    server dos 10.10.10.22:80 maxconn 128
~~~

Ahora vamos a entrar en el fichero `/etc/default/haproxy` y ponemos lo siguiente.
~~~
ENABLED=1
~~~

Reiniciamos el servicio.
~~~~
vagrant@balanceador:~$ sudo systemctl restart haproxy
~~~~

![Inicio Proxy](imagenes/balan.png)

![Inicio Proxy](imagenes/balan2.png)

### Tarea 2: Entrega una captura de pantalla donde se vea la página web de estadísticas de haproxy (abrir en un navegador web la URL http://172.22.x.x/haproxy?stats, pedirá un usuario y un password, ambos cda).

Ahora vamos a acceder por el navegador a las estadísticas, para acceder tendremos que poner el usuario cda y la constraseña cda.

![Inicio Proxy](imagenes/balan3.png)

![Inicio Proxy](imagenes/balan4.png)

### Tarea 3: Desde uno de los servidores (apache1 ó apache2), verificar los logs del servidor Apache. En todos los casos debería figurar como única dirección IP cliente la IP interna de la máquina balanceador 10.10.10.1. ¿Por qué?

Vemos los logs de apache1.
~~~
10.10.10.1 - - [22/Feb/2021:11:53:56 +0000] "GET / HTTP/1.1" 200 436 "-" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.152 Safari/537.36"
~~~

Vemos que en log aparece la ip interna del balanceador ya que nosotros accedemos a la ip externa del navegador y este es el que se encarga de preguntar a los servidores web que están en su red interna cual va a ser el que va a responder a la petición.

### Tarea 4:Verificar la estructura y valores de las cookies PHPSESSID intercambiadas. En la primera respuesta HTTP (inicio de sesión), se establece su valor con un parámetro HTTP SetCookie en la cabecera de la respuesta. Las sucesivas peticiones del cliente incluyen el valor de esa cookie (parámetro HTTP Cookie en la cabecera de las peticiones)

Vamos a verificar la estructuras de cookies para ello primero tendremos que configurar el fichero sesion.php que tenemos en las maquinas apache.
~~~
<?php
     header('Content-Type: text/plain');
     session_start();
     if(!isset($_SESSION['visit']))
     {
             echo "This is the first time you're visiting this server";
             $_SESSION['visit'] = 0;
     }
     else
             echo "Your number of visits: ".$_SESSION['visit'];             

     $_SESSION['visit']++;              

     echo "\nServer IP: ".$_SERVER['SERVER_ADDR'];
     echo "\nClient IP: ".$_SERVER['REMOTE_ADDR'];
     echo "\nX-Forwarded-for: ".$_SERVER['HTTP_X_FORWARDED_FOR']."\n";
     print_r($_COOKIE);
?>
~~~

Incluimos la siguientes lineas en el fichero `/etc/haproxy/haproxy.cfg`
~~~
cookie PHPSESSID prefix                               # <- aquí
server uno 10.10.10.11:80 cookie EL_UNO maxconn 128   # <- aquí
server dos 10.10.10.22:80 cookie EL_DOS maxconn 128   # <- aquí
~~~

Reiniciamos el servicio.
~~~
vagrant@balanceador:~$ sudo systemctl restart haproxy
~~~

Y accedemos mediante el navegador a sesion.php.

![Inicio Proxy](imagenes/balan5.png)

Aqui podemos ver la cookie que tenemos, si recargamos la página varias veces vemos que la cookie no cambia pero si el numero de veces que hemos recargado la página.

![Inicio Proxy](imagenes/balan6.png)

Si entramos en modo incognito vemos que la cookie si cambia.

![Inicio Proxy](imagenes/balan7.png)
