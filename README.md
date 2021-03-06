###### Invitame un cafecito para seguir con mas ejemplos:

[![ko-fi](https://www.ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/kdetony)

# Docker Intermedio

Este espacio es para explicar de forma practica como entender los conceptos de imagen, contenedor, red y volumen, de la mano con Dockerfile y docker-compose, que nos ayudaran a desplegar la infraestructura necesaria relacionada con contenedores.

Tengamos en cuenta lo siguiente, **docker run** lo que hace es iniciar un contenedor, y si la imagen no la tenemos descargada, realizara **docker pull** internamente para realizar esta accion, pero, es muy tedioso aun para un sysadmin experimentado estar digitando los comandos en consola, por ello, recurrimos por usar **Dockerfile** quien nos ayudara¡ a *personalizar una imagen* a nuestra necesidad, y encima podemos tener versiones aka TAG por cada cambio que realicemos.

**docker-compose** esta relacionado con los contenedores contratamente, ya que en el archivo de extension YAML, podemos asociar imagenes ,puertos, volumenes de forma mas intuitiva y personalizada. Con esta introduccion vamos a como iniciar esto: 

## OBS
Para trabajar con este laboratorio, vamos a clonar todo el repositorio: 

> git clone https://github.com/kdetony/docker-cap2.git

## EJEMPLO1: Despliegue contenedor web y database

En la carpeta **docker-cap2**, tenemos el fichero *docker-compose.yml* quien orquestará toda la creacion de contenedores que necesitemos, para lo cual vamos a ejecutar: 

>**docker-compose up -d**

Lo que realizara *docker-compose.yml* primero interpretar la version de la misma ( aqui un [Link](https://docs.docker.com/compose/compose-file/compose-versioning/) donde explica esta parte  ) posterior a ello, y es aqui el tremendo potencial que tiene, y es la sintaxis **build**, pues mediante docker-compose podemos indicarle que lea un Dockerfile, caso contrario podemos usar **image** para que pueda usar la imagen descargada en nuestro Host. La sintaxis **container_name** asigna un nombre a nuestro contenedor, **enviroment** sirve para setear las variables de entorno, para nuestro caso, passwd, user para mariadb, la sintaxis **volume** asigno el mismo al contendor.

### OBS.

* Para visualizar la web, vamos a ingresar via un browser, colocando la IP Publica.

* Si al acceder a la Web, nos encontramos con un Forbiden ( 403 ), debemos entrar al contenedor y crear un **index.html**

> docker exec -it ID_CONTAINER bash
> touch index.html /var/Www/html/


## EJEMPLO2: Volumenes

Vamos ahora a colocar contenido en nuestro **VOLUME**, para este primer ejercicio, será cambiar el *index.html* relacionado a nuestro contenedor WEB y como segundo ejercicio, haremos un restore para la base de datos, aqui podemos hacerlo de forma manual y/o via Dockerfile. Vamos a ejecutarlo manualmente: 

### EJERCICIO 1

Para el contenedor web, *solo vamos a modificar el index.html*, para ello:

> cd /var/lib/docker/volumes 

> ls 

Debemos identificar la carpeta de nombre: **docker-cap2_sg1**, es esta carpeta que se asocia al volume: **sg1** que creamos en el docker-compose.
Dentro de esta carpeta, vamos a ver otra carpeta: *_data*, y dentro de ella se encuetra el archivo: **index.html**
Lo que haremos es modificar el archivo, para ver reflejados los cambios en el contenedor.

### EJERCICIO 2

Para la base de datos, lo que haremos es un restore manual, para ello: 

Nos ubicamos en la carpeta **mariadb** 

> cat bd.sql | docker exec -i cnt_db  mysql -uroot -ppassword centos_db

## EJEMPLO 3

Antes de realizar este ejemplo, vamos a modificar nuestro docker-compose, en esta linea:

**sg1:/var/www/html:ro**

Y volvemos a ejecutar:

> docker-compose up -d

Vamos a copiar ahora nuestra web ( data ) a nuestro contenedor, para ello vamos a usar el comando:

> **docker cp web/.  web_apache:/var/www/html**

A que se debe este mensaje?

**Error response from daemon: mounted volume is marked read-only**

Recordemos que el volumen **sg1** en **docker-compose** fue declarado como RO (Red-Only), por ende no tenemos permisos de escritura en el contenedor y hacia el, una forma de solucionarlo, es copiar la data desde el HOST hacia el volumen creado.

La segunda opcion es cambiar en *docker-compose* de RO a RW y /var/lib/docker/volumeses lo que vamos a realizar ( regresamos a como esta inicialmente )

Volvemos a lanzar: **docker-compose up -d**  y ejecutamos:

> docker cp web/. web_apache:/var/www/html

## EJEMPLO 4

Planteo lo sgt, como hariamos para instalar Wordpress en un contenedor? ... me adelanto, NO NO NO y NO no podemos instalar apache, mysql, php, wordpress en un contendor, ya que este esquema es para una Maquina Fisica o Virtual, en contenedores debemos separar todas las aplicaciones y/o servicios en unicos contenedores... ok? , entonces, ahora, vamos a realizar esta instalacion de Wordpress de esta manera: 1 contenedor para wordpress ( y todas sus dependencias ) - 1 contenedor para la base de datos.

Obs.
En la carpeta wordpress, encontrarás el archivo **docker-compose.yml** el cual abarca la instalacion de Docker como hemos comentado, y es la forma "actual" que se usa para "linkear" contenedores o "enlazarlos", ya que el comando linked esta deprecado pero aun se puede user, *teniendo en cuenta la version de 2.x/3 de docker-compose.*

Para ejecutarlo, debemos ubicarnos en la carpeta: *wordpress*

> docker-compose up -d

Entonces, lo que vamos hacer es usando este comando "deprecado" ojo! aun se puede seguir usandolo sin problema, y sera asi:

#### Instalando Mysql version 5.7

> docker volume create red_wordpress

> docker run -dit -v /home/kdetony/mysql:/var/lib/mysql --net red_wordpress --name **dbw-mysql** -e MYSQL_DATABASE=wordpress -e MYSQL_ROOT_PASSWORD=password  mysql:5.7

#### Realizando el Link entre contenedores 

>docker run -dit --name wordpress --link **dbw-mysql**:mysql -p 8380:80 wordpress

Abrimos un browser colocando la ip del HOST en el puerto 8380 y procedemos a realizar la instalacion.

**OBS.**
Debemos colocar el nombre del contenedor de mysql, para este ejm. *dbw-mysql*


## EJEMPLO 5

Demos un poco mas de complejidad a nuestra arquitectura actual, para ello, vamos a crear un proxy reverse con Nginx (solo para el puerto 80, para el puerto 443, es otro precio :p, no es broma! lo iré colocando mas adelante, como un *paso6*.
En el archivo **docker-compose.yml** tenemos la creación del proxy reverso, asi como la creación de un contenedor web ( apache )y un conenedor web nginx, el proxy accederá a los aplicativos por el puerto 8080, 8081.

Antes de crear los contenedores, vamos a crear la sgt red: **docker network create nginx_redproxy**

Para inicializar los contenedores, nos ubicamos en la carpeta *proxyapp*

> docker compose up -d 

Validamos ingresando la ip de http://HOST:8080 y/o http://HOST:8081


## LA YAPA 

Por último, vamos a crear usando un contenedor Portainer, que es un aplicativo visual para administrar contendores: 

>docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock --name admindocker portainer/portainer

Cusi-Cusa !!!

[PARTE3](https://github.com/kdetony/docker-swarm "Docker Swarm")
