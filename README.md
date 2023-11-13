## Explica métodos para 'abrir' una consola/shell a un contenedor que se está ejecutando
En Visual Studio Code, podemos hacer click derecho sobre el contenedor y pulsar en la opción "Attach shell"
![VSCode attach shell](/img/1a.png)
Otra manera es a través de una terminal, lanzando el comando `docker exec -it NOMBRE_CONTENEDOR bash'

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
## Añade una zona tiendadeelectronica.int en tu docker DNS que tenga
### www a la IP 172.16.0.1
### owncloud sea un CNAME de www
### un registro de texto con el contenido "1234ASDF"
### Comprueba que todo funciona con el comando "dig"
### Muestra en los logs que el servicio arranca correctamente

## Realiza el apartado 9 en la máquina virtual con DNS
