## Explica métodos para 'abrir' una consola/shell a un contenedor que se está ejecutando
En Visual Studio Code, podemos hacer click derecho sobre el contenedor y pulsar en la opción "Attach shell"

![VSCode attach shell](/img/1a.png)

Otra manera es a través de una terminal, lanzando el comando `docker exec -it NOMBRE_CONTENEDOR bash`.

## En el contenedor anterior con que opciones tiene que haber sido arrancado para poder interactuar con las entradas y salidas del contenedor
Para poder interactuar con entradas y salidas, añadiremos como en el caso anterior el parámetro `-it`.

![Interactuando con el contenedor](/img/2a.png)
## ¿Cómo sería un fichero docker-compose para que dos contenedores se comuniquen entre si en una red solo de ellos?
```
services:
  examen1:
    image: ubuntu:latest
    container_name: examen1
    networks:
      network:
        ipv4_address: 172.28.0.30

  examen2:
    image: ubuntu:latest
    container_name: examen2
    networks:
      network:
        ipv4_address: 172.28.0.40

networks:
  NOMBRE_RED:
    external: true
```
Además, en Visual Code añadimos la red desplegando el apartado "Networks", dánole el mismo nombre que en el fichero de Docker Compose.
## ¿Qué hay que añadir al fichero anterior para que un contenedor tenga la IP fija?
En la parte de código relativa a cada contenedor añadiremos el apartado `networks`, especificando la IP que le queremos asignar, de la siguiente manera:
```
    networks:
      NOMBRE_RED:
        ipv4_address: 172.28.5.1
```
En este ejemplo, le asignamos la IP 172.28.5.1 al contenedor.
## ¿Que comando de consola puedo usar para saber las ips de los contenedores anteriores? Filtra todo lo que puedas la salida. El comando no es "ip a", tiene que ser desde fuera del contenedor.
Podemos utilizar desde fuera del contenedor el comando `docker inspect`. Pero primero necesitamos saber el ID del contenedor a comprobar. Para ellos usamos el comando `docker ps`:
```console
$ docker ps
CONTAINER ID   IMAGE           COMMAND       CREATED       STATUS          PORTS     NAMES
60a5b149db23   ubuntu:latest   "/bin/bash"   3 weeks ago   Up 28 minutes             asir_cliente
```
Ahora ya podemos utilizar 'docker inspect`. Para filtrar la salida, que es bastante larga, y sólo obtener la IP, podemos utilizar el siguiente comando con filtro:
```console
$ docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' 60a5b149db23
172.28.5.0
```
## ¿Cual es la funcionalidad del apartado "ports" en docker compose?
Con el apartado `ports` podemos redirigir puertos del equipo a puertos específicos del contenedor. Como por ejemplo:
`
ports:
      - "8001:5432"
`
En este caso, 8001 sería el puerto de la máquina y 5432 el del contenedor.
## ¿Para que sirve el registro CNAME? Pon un ejemplo
Un CNAME es un nombre canónico que, en el caso del DNS, sirve como alias de un dominio. 
Por ejemplo, si tenemos un dominio `asir.net` que redirige a la IP 192.0.2.1, y establecemos un registro CNAME llamado `blog.asir.net` con el valor `asir.net`, si nosotros visitamos `blog.asir.net` se nos redirigirá igualmente a la IP 192.0.2.1.
## ¿Como puedo hacer para que la configuración de un contenedor DNS no se borre si creo otro contenedor?
Crearemos las carpetas que alojan la configuración del DNS en el propio host, y mediante el apartado `volumes` de Docker Compose haremos que cualquier contenedor que creemos para utilizar como DNS, utilice dichas carpetas como las suyas propias del contenedor.
Por ejemplo:
```
    volumes:
      - ./conf:/etc/bind
      - ./zonas:/var/lib/bind
```
## Añade una zona tiendadeelectronica.int en tu docker DNS que tenga:
* www a la IP 172.16.0.1
* owncloud sea un CNAME de www
* un registro de texto con el contenido "1234ASDF"

Realizamos los siguientes pasos en las carpetas que tenemos enlazadas en el host con el contenedor DNS:

Modificamos el archivo `named.conf.local` para añadir el el archivo de la base de datos de la nueva zona:

![Modificando named.conf.local](/img/9a.png)

Creamos el archivo de base de datos de la zona, llamándolo `db.tiendadeelectronica.int`, y añadimos el siguiente contenido:
```
 $TTL 38400	; 10 hours 40 minutes
@		IN SOA	ns.tiendadeelectronica.int. some.email.address. (
				10000002   ; serial
				10800      ; refresh (3 hours)
				3600       ; retry (1 hour)
				604800     ; expire (1 week)
				38400      ; minimum (10 hours 40 minutes)
				)
@		    IN NS	ns.tiendadeelectronica.int.
ns		    IN A		172.16.0.254
www	        IN A		172.16.0.1
owncloud	IN CNAME	www
texto	    IN TXT		1234ASDF
```
### Comprueba que todo funciona con el comando "dig"
#### www a la IP 172.16.0.1
```console
root@8909d831d149:/# dig @172.28.5.1 www.tiendadeelectronica.int

; <<>> DiG 9.18.18-0ubuntu0.23.04.1-Ubuntu <<>> @172.28.5.1 www.tiendadeelectronica.int
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 6098
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 31904dee4b246ec201000000655247c134198f29c013e83e (good)
;; QUESTION SECTION:
;www.tiendadeelectronica.int.   IN      A

;; ANSWER SECTION:
www.tiendadeelectronica.int. 38400 IN   A       172.16.0.1

;; Query time: 0 msec
;; SERVER: 172.28.5.1#53(172.28.5.1) (UDP)
;; WHEN: Mon Nov 13 15:58:57 UTC 2023
;; MSG SIZE  rcvd: 100
```
#### owncloud sea un CNAME de www
```console
root@8909d831d149:/# dig @172.28.5.1 owncloud.tiendadeelectronica.int

; <<>> DiG 9.18.18-0ubuntu0.23.04.1-Ubuntu <<>> @172.28.5.1 owncloud.tiendadeelectronica.int
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 11222
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: 2e684df45bad385e01000000655248874110dcdc1f1b6f49 (good)
;; QUESTION SECTION:
;owncloud.tiendadeelectronica.int. IN   A

;; ANSWER SECTION:
owncloud.tiendadeelectronica.int. 38400 IN CNAME www.tiendadeelectronica.int.
www.tiendadeelectronica.int. 38400 IN   A       172.16.0.1

;; Query time: 0 msec
;; SERVER: 172.28.5.1#53(172.28.5.1) (UDP)
;; WHEN: Mon Nov 13 16:02:15 UTC 2023
;; MSG SIZE  rcvd: 123
```
#### Un registro de texto con el contenido "1234ASDF"
```console
root@8909d831d149:/# dig @172.28.5.1 -t txt texto.tiendadeelectronica.int

; <<>> DiG 9.18.18-0ubuntu0.23.04.1-Ubuntu <<>> @172.28.5.1 -t txt texto.tiendadeelectronica.int
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 47044
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
; COOKIE: e8795165230527c60100000065524974042c38dd1fd624bd (good)
;; QUESTION SECTION:
;texto.tiendadeelectronica.int. IN      TXT

;; ANSWER SECTION:
texto.tiendadeelectronica.int. 38400 IN TXT     "1234ASDF"

;; Query time: 0 msec
;; SERVER: 172.28.5.1#53(172.28.5.1) (UDP)
;; WHEN: Mon Nov 13 16:06:12 UTC 2023
;; MSG SIZE  rcvd: 107
```
### Muestra en los logs que el servicio arranca correctamente

En Visual Studio, hacemos click derecho sobre el contenedor y pulsamos en "View Logs".
En la terminal se nos mostrarán los registros de arranque del contenedor, y en ellos podemos comprobar que tanto `bind9` como las zonas DNS se han cargado correctamente:

```console
Starting named...
exec /usr/sbin/named -u "bind" "-g" ""
13-Nov-2023 16:21:11.957 starting BIND 9.18.18-0ubuntu0.23.04.1-Ubuntu (Extended Support Version) <id:>
13-Nov-2023 16:21:11.957 running on Linux x86_64 6.1.0-13-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.55-1 (2023-09-29)
[...]
13-Nov-2023 16:21:11.965 zone tiendadeelectronica.int/IN: loaded serial 10000002
13-Nov-2023 16:21:11.965 zone asircastelao.int/IN: loaded serial 10000002
13-Nov-2023 16:21:11.965 all zones loaded
13-Nov-2023 16:21:11.965 running
13-Nov-2023 16:21:11.985 managed-keys-zone: Key 20326 for zone . is now trusted (acceptance timer complete)
```

## Realiza el apartado 9 en la máquina virtual con DNS
Modificamos el archivo `named.conf.local` para añadir el el archivo de la base de datos de la nueva zona:

![Modificando named.conf.local](/img/10a.png)

Creamos el archivo de base de datos de la zona, llamándolo `db.tiendadeelectronica.int`, y añadimos el siguiente contenido:

![Creando db.tiendadeelectronica.int](/img/10b.png)

Comprobamos con el comando `dig` que todo funciona:

![Modificando named.conf.local](/img/10c.png)

![Modificando named.conf.local](/img/10d.png)

![Modificando named.conf.local](/img/10e.png)

Comprobamos en los logs que el servicio arranca correctamente:

![Modificando named.conf.local](/img/10f.png)