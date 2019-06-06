# Docker-Practico

Este espacio es para explicar de forma práctica como entender los conceptos de imagen, contenedor, red y volumen, de la mano con Dockerfile y docker-compose, que nos ayudarán a desplegar la infraestructura necesaria relacionada con contenedores.

Tengamos en cuenta lo siguiente, **docker run** lo que hace es iniciar un contenedor, y si la imagen no la tenemos descargada, realizará **docker pull** internamente para realizar esta acción, pero, es muy tedioso aún para un sysadmin experimentado estar digitando los comandos en consola, por ello, recurrimos por usar **Dockerfile** quien nos ayudará a *personalizar una imagen* a nuestra necesidad, y encima podemos tener versiones aka TAG por cada cambio que realicemos.

**docker-compose** esta relacionado con los contenedores contratamente, ya que en el archivo de extension YAML, podemos asociar imagenes ,puertos, volumenes de forma más intuitiva y personalizada. Con esta introducción vamos a como iniciar esto: 

## PASO 1 

>**docker-compose up -d**

Lo que realizará docker compose es primero interpretar la version de la misma ( aqui un [Link](https://docs.docker.com/compose/compose-file/compose-versioning/) donde explica esta parte  )
posterior a ello, y es aquí el tremendo potencial que tiene, y es la sintaxis **build**, pues mediante docker-compose podemos indicarle que lea un Dockerfile, caso contrario podemos usar **image** para que pueda usar la imagen descargada en nuestro Host. La sintaxis **container_name** asigna un nombre a nuestro contenedor, **enviroment** sirve para setear las variables de entorno, para nuestro caso, passwd, user para mariadb, la sintaxis **volume** asigno el mismo al contendor.

## PASO 2 

Vamos ahora a colocar contenido en nuestro VOLUME, para este caso, será un simple html5 responsive y un restore de una base de datos, aquí podemos hacerlo de forma manual y/o via dockerfile. Vamos a ejecutarlo manualmente: 

Para la web solo copiar los archivos al bind mount declarado, para la base de datos lo haremos de la sgt manera: 

>**cat dump.sql | docker exec -i mysql-container mysql -uroot -ppassword db_name**

## PASO 3

Vamos a copiar ahora nuestra web ( data ) a nuestro contenedor, para ello vamos a usar el comando:

>**docker cp web/.  web_apache:/var/www/html 

A que se debe este mensaje?

**Error response from daemon: mounted volume is marked read-only**

Recordemos que el volumen en docker-compose fue declarado como RO (Red-Only), por ende no tenemos permisos de escritura en el contenedor y hacia el, una forma de solucionarlo, es copiar la data desde el HOST hacia el volumen creado, en este caso: *docker_practica_sg1*

La segunda opción es cambiar en docker-compose de RO a RW.

volvemos a lanzar: **docker-compose up -d**  y ejecutamos:

>docker cp web/. web_apache:/var/www/html

## PASO 4

Planteo lo sgt, como hariamos para instalar Wordpress en un contenedor? ... me adelanto, NO NO NO y NO no podemos instalar apache, mysql, php, wordpress en un contendor, ya que este esquema es para una Maquina Fisica o Virtual, en contenedores debemos separar todas las aplicaciones y/o servicios en unicos contenedores... ok? , entonces, ahora, vamos a realizar esta instalación de Wordpress de esta manera: 1 contenedor para wordpress ( y todas sus dependencias ) - 1 contenedor para la base de datos.

Obs.
En la carpeta wordpress, encontrará el archivo **docker-compose.yml** el cual abarca la instalación de Docker como hemos comentado, y es la forma "actual" que se usa para "linkear" contenedores o "enlazarlos", ya que el comando linked esta deprecado.

Entonces, lo que vamos hacer es usando este comando "deprecado" ojo! aun se puede seguir usandolo sin problema, y sería así:

#### Instalando Mysql version 5.7

>docker volume create red_wordpress

>docker run -dit -v /home/kdetony/mysql:/var/lib/mysql --net red_wordpress --name **dbw-mysql** -e MYSQL_DATABASE=wordpress -e MYSQL_ROOT_PASSWORD=password  mysql:5.7

#### Realizando el Link entre contenedores 

>docker run -dit --name wordpress --link **dbw-mysql**:mysql -p 8380:80 wordpress

Abrimos un browser colocando la ip del HOST en el puerto 8380 y procedemos a realizar la instalación.

**OBS.**
Debemos colocar el nombre del contenedor de mysql, para este ejm. *dbw-mysql*


## PASO 5

Demos un poco mas de complejidad a nuestra arquitectura actual, para ello, vamos a crear un proxy reverse con Nginx (solo para el puerto 80, para el puerto 443, es otro precio :p, no es broma! lo iré colocando mas adelante, como un *paso6* 
 en el archivo **docker-compose.yml** tenemos ingresados la creaci�n del proxy reverso, asi como la creaci�n de un contenedor web ( apache ), el proxy accederáal aplicat.
