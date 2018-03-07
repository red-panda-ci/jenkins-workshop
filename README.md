# Jenkins Workshop

Guía de la charla acerca de Jenkins y entregabilidad https://youtu.be/rpyTevjM3X4

Debido a que el texto se compuso durante los días posteriores a la charla es posible que el contenido de esta guía difiera del contenido del vídeo. Por temas prácticos utilizaremos de forma deliberada repositorios distintos en la guía para las pruebas.

Pido disculpas por ello.

## Parte 1: Docker

Utilizaremos Docker para gestionar builds, test y en último término plataformado y operación de entornos.

### Gestión de elementos

Trabajaremos con 4 diferentes elementos de docker:

* Imágenes
* Containers
* Redes
* Volúmenes

Existen opciones comunes a todos los elementos:

```bash
$ docker ${item} ls          # Listar todos los elementos de su tipo. Opcional "-a" en containers e imágenes
$ docker ${item} rm          # Borrar un elemento. Opcional "-f" para forzar el borrado
$ docker ${item} prune       # Borrar todos los elementos de un tipo. Opcional "-f" no pide confirmación.
```

Como ${item} podemos usar:

* image
* container
* network
* volume

Tenemos sistaxis específicas para elementos concretos, por ejemplo son equivalentes:

```bash
$ docker ps -a              # Equivalente a "docker container ls -a"
$ docker images             # Equivalente a "docker image ls"
$ docker rmi ubuntu         # Equivalente a "docker image rm ubuntu"
```

Hay opciones que podemos usar en algunos elementos, en otros no:

```bash
$ docker container ps       # Equivalente a "docker container ls -a"
$ docker image ps           # Incorrecto, con esto tendremos un error
```

Podemos obtener ayuda de docker para cada uno de los comandos con "docker help"

```bash
$ docker help container     # Listado de opciones de "docker container"
$ docker help container ps  # Listado de opciones de "docker container ps"
```

#### Imágenes

Podemos considerarlo como plantillas que tomaremos de base para ejecutar containers. en los ejemplos trabajaremos con el registry "docker hub".

Un registry es un repositorio público donde guardaremos nuestras imágenes.

```bash
$ docker pull redpandaci/jenkins-dind  # Podemos usar "docker image pull redpandaci/jenkins-dind"
$ docker pull hello-world:latest       # Descargamos la imagen dicker del clásico "Hello, world!" en su última versión
```

Cada vez que hacemos "docker pull" de una imagen:

* En caso que imagen no exista en nuestro PC, se descargará del registry
* Si la imagen ya la tenemos en nuesto PC, comprobará si está actualizada, descargando la última versión en caso necesario

Podemos indicar una versión concreta a la hora de descargar una imagen:

```shell
$ docker pull redpandaci/jenkins-dind:2.89.4
```

En caso de no especificar versión, se bajará la última (latest)

```shell
$ docker pull redpandaci/jenkins-dind:latest    # Equivalente a "docker pull redpandaci/jenkins-dind"
```

#### Containers

Se pueden considerar "objetos" de la "clase" imagen.

```bash
$ docker run hello-world                   # Ejecución del clásico "Hello, World!

[...]

$ docker container ps -a                        # Listado de containers, tanto que están en ejecución como los que finalizaron
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
64897191cb95        hello-world         "/hello"            1 second ago        Exited (0) 2 seconds ago                       unruffled_ardinghelli
$ docker container rm unruffled_ardinghelli     # Borrado de container
unruffled_ardinghelli
$ docker ps -a                                  # Atajo para "docker container ps -a"
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

#### Redes

Siguiendo la filosofia como gestión de elementos, podemos usar "docker network [comando]"

```shell
$ docker network ls                                                 # Listamos todas las redes
NETWORK ID          NAME                DRIVER              SCOPE
618223a0b923        bridge              bridge              local
916f5b82fd27        host                host                local
ddab82ea952e        none                null                local
$ docker network create test                                        # Creamos una nueva red llamada "test"
6d2646754a9ebfb024c8f0e50df5dc31f4a8d78342c0173599c2fd33c9c9408a
$ docker network ls                                                 # Verificamos que la red se ha creado
NETWORK ID          NAME                DRIVER              SCOPE
618223a0b923        bridge              bridge              local
916f5b82fd27        host                host                local
ddab82ea952e        none                null                local
6d2646754a9e        test                bridge              local
$ docker network inspect test                                       # Vemos las propiedades de la red "test"
[
    {
        "Name": "test",
        "Id": "6d2646754a9ebfb024c8f0e50df5dc31f4a8d78342c0173599c2fd33c9c9408a",
        "Created": "2018-02-22T16:23:52.08840954Z",

[...]

        "Labels": {}
    }
]
$ docker network rm test                                            # Borramos la red "test"
test
```

Con la visión "cattle" de gestión de elementos (los elementos son "ganado"), siguiendo el ejemplo del vídeo vamos a crear y destruir con "prune" 10 redes

```shell
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
618223a0b923        bridge              bridge              local
916f5b82fd27        host                host                local
ddab82ea952e        none                null                local
$ for a in 1 2 3 4 5 6 7 8 9 10; do docker network create test_$a; done
50d89397a430f77ff6f506842dced6406f4e4097d1fbd0428ab2277059556d6c
a56cf873f879cc656b7f10bdb20450eff0866c019999a13229b15822d81a9092
e1c2a069eb34b5c509019e96a065099c0e6516fa2a2faeb71c4b6be5aa8c2545
75b6d460e36ea222d6373c35bdf762cc3fc3f1b3da21a29b1d2f2dbad57974bc
e0069de70155a9edad494e79025cd7d93880262fbafbf369578e581b3c9e4eec
6441aa7795c09f68d8e14c1923232ee0212dc1e3f78c3c9f4534cb14c45627b6
89db9380b8fe312fb16dc34c563c0c3ebdabfd423d9fe5ad94ea6c0b21d2c29b
6836e20be411799585661cdea6bbcfa2b29f021eaa6132ad44f39bdf049b0908
cdd4d068b596a82ffa725945e86aa3dcff29cf9fabfe175e83d2efdf16df17e9
a693514bf8d6599f4ee879f779e655b8bb95cf78acb5d298ed5b212e3da3e60a
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
618223a0b923        bridge              bridge              local
916f5b82fd27        host                host                local
ddab82ea952e        none                null                local
50d89397a430        test_1              bridge              local
a693514bf8d6        test_10             bridge              local
a56cf873f879        test_2              bridge              local
e1c2a069eb34        test_3              bridge              local
75b6d460e36e        test_4              bridge              local
e0069de70155        test_5              bridge              local
6441aa7795c0        test_6              bridge              local
89db9380b8fe        test_7              bridge              local
6836e20be411        test_8              bridge              local
cdd4d068b596        test_9              bridge              local
$ docker network prune
WARNING! This will remove all networks not used by at least one container.
Are you sure you want to continue? [y/N] y
Deleted Networks:
test_3
test_5
test_8
test_9
test_10
test_2
test_6
test_7
test_1
test_4

$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
618223a0b923        bridge              bridge              local
916f5b82fd27        host                host                local
ddab82ea952e        none                null                local
```

Podemos crear una red y ejecutar un container que la utilice
```shell
$ docker network create test
d3fc0ff8c1a473b83b78eadc9313b1aacaf2790b37e8cf40eb7f8c80bc1a1406
$ docker run --network test hello-world

Hello from Docker!

[...]

$ docker container inspect romantic_einstein
[
    {
        "Id": "b6343150a7301c9109fd501c1ba91191751c202596c7cd5eb4e386bc9f02d10e",
        "Created": "2018-02-22T16:36:46.07837747Z",

 [...]

            "Networks": {
                "test": {

[...]
```

#### Volúmenes

Teniendo en cuenta que los containers son _efímeros_ y desaparecen cuando finaliza su ejecución, podremos utilizar los volúmenes para que los datos de un directorio determinado de un container puedan persistir más allá del fin de ejecución del propio container.

Aplicamos los mismos comandos que con los otros elementos: "ls", "rm", "inspect" o "prune".
Vamos a crear, listar, inspeccionar y luego eliminar un volumen llamado "test"

```shell
$ docker volume ls
DRIVER              VOLUME NAME
$ docker volume create test
test
$ docker volume ls
DRIVER              VOLUME NAME
local               test
$ docker volume inspect test
[
    {
        "CreatedAt": "2018-02-27T17:34:01+01:00",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/test/_data",
        "Name": "test",
        "Options": {},
        "Scope": "local"
    }
]
$ docker volume rm test
test
$ docker volume ls
DRIVER              VOLUME NAME
```

Tratamiento "cattle" (ganado) de volúmenes, al igual que hicimos con las redes
```bash
$ docker volume ls
DRIVER              VOLUME NAME
$ for a in 1 2 3 4 5 6 7 8 9 10; do echo "Creando volumen test_$a"; docker volume create test_$a; done
Creando volumen test_1
test_1
Creando volumen test_2
test_2
Creando volumen test_3
test_3
Creando volumen test_4
test_4
Creando volumen test_5
test_5
Creando volumen test_6
test_6
Creando volumen test_7
test_7
Creando volumen test_8
test_8
Creando volumen test_9
test_9
Creando volumen test_10
test_10
$ docker volume ls
DRIVER              VOLUME NAME
local               test_1
local               test_10
local               test_2
local               test_3
local               test_4
local               test_5
local               test_6
local               test_7
local               test_8
local               test_9
$ docker volume prune -f
Deleted Volumes:
test_5
test_6
test_7
test_9
test_1
test_3
test_4
test_2
test_8
test_10

Total reclaimed space: 0B
$ docker volume ls
DRIVER              VOLUME NAME
```

Ejemplo de persistencia de datos usando un volumen con nombre

* Creamos nuevo container "ubuntu" con un volumen con nombre montado en la carpeta /opt (opción "-v test:/opt") con ejecución interactiva (opción "-ti" junto con el "/bin/bash" al final) y creamos un fichero con contenido en /opt
```shell
$ docker volume ls
DRIVER              VOLUME NAME
$ docker run -ti --rm -v test:/opt ubuntu /bin/bash
root@3754a029ecac:/# ls /opt  
root@3754a029ecac:/# echo "test content" > /opt/testfile
root@3754a029ecac:/# ls -l /opt
total 4
-rw-r--r-- 1 root root 13 Feb 27 16:44 testfile
root@3754a029ecac:/# cat /opt/testfile 
test content
```
* Finalizams ejecución ("exit") y comprobamos que el container no existe (debido a opción "--rm") y tenemos un volumen nuevo "test"
```shell
root@3754a029ecac:/# exit
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
$ docker volume ls
DRIVER              VOLUME NAME
local               test
$ docker volume inspect test
[
    {
        "CreatedAt": "2018-02-27T17:44:57+01:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/test/_data",
        "Name": "test",
        "Options": {},
        "Scope": "local"
    }
]
```
* Ejecutamos un nuevo container "ubuntu" con el volumen "test" montado en /opt, mismas condiciones que el anterior. Comprobamos que el contenido existe en el nuevo container ejecutado desde la imagen "ubuntu"
```shell
$ docker run -ti --rm -v test:/opt ubuntu /bin/bash
root@4c3d264d087b:/# ls -l /opt
total 4
-rw-r--r-- 1 root root 13 Feb 27 16:44 testfile
root@4c3d264d087b:/# cat /opt/testfile 
test content
```
* Finalizamos ejecución y eliminamos volumen
```shell
root@4c3d264d087b:/# exit
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
$ docker volume ls
DRIVER              VOLUME NAME
local               test
$ docker volume prune
WARNING! This will remove all volumes not used by at least one container.
Are you sure you want to continue? [y/N] y
Deleted Volumes:
test

Total reclaimed space: 13B
$ docker volume ls
DRIVER              VOLUME NAME
```

Podemos exponer una carpeta del sistema "host" en el container

* Creamos carpeta nueva y ponemos contenido en un fichero en esa carpeta
```shell
$ ls -l
total 0
$ mkdir test
$ ls -l
total 4
drwxrwxr-x 2 prodriguez prodriguez 4096 feb 27 17:59 test
$ echo "New test content" > test/testfile_1
$ cat test/testfile_1 
New test content
```
* Ejecutamos nuevo container con esa carpeta montada en "/opt" (comando "-v `pwd`/test:/opt"). Comprobamos que tiene el fichero creado en el paso anterior
```shell
$ docker run --rm -ti -v `pwd`/test:/opt ubuntu /bin/bash
root@e82278e6cd80:/# ls -l /opt/testfile_1 
-rw-rw-r-- 1 1000 1000 17 Feb 27 16:59 /opt/testfile_1
root@e82278e6cd80:/# cat /opt/testfile_1 
New test content
```
* Desde el container añadimos contenido al fichero
```shell
root@e82278e6cd80:/# echo "More content generated into the container" >> /opt/testfile_1 
root@e82278e6cd80:/# cat /opt/testfile_1 
New test content
More content generated into the container
```
* Finalizamos ejecución y comprobamos que tenemos en nuestro fichero en el "host" el contenido que hemos añadimos desde dentro del contanier
```shell
root@e82278e6cd80:/# exit
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
$ cat test/testfile_1 
New test content
More content generated into the container
```

Son ejemplos simples con los que tener un poco de visión de cómo orquestar los distintos componentes: imágenes, redes, containers, volúmenes

### Docker compose

Utilizando imágenes (plantillas) y la capacidad de crear y eliminar containers basados en esas imágenes junto con redes y volúmenes podemos preparar plataformas efímeras completas usando docker compose.

Esta documentación no pretende ser una guía de lo que se puede hacer o no utilizando compose, sino una referencia mínima para poder orquestar pipelines.

La plataforma efímera se define en el archivo "docker-compose.yml". Algunos comandos básicos:
```shell
$ docker-compose up     # Crea los servicios. Si añadimos la opción "-d" se levantarán los servicios en modo "daemon"
$ docker-compsose ps    # Muestra los servicios en ejecución
$ docker-compose down   # Elimina los servicios. Si añadimos la el parámetro "-v" se eliminan también los volúmenes
```

Vamos a utilizar un repositorio de Red Panda para ver el funcionamiento de docker-compose. Utilizaremos un script del propio repositorio "bin/test.sh"

```shell
$ git clone https://github.com/red-panda-ci/jenkins-pipeline-library                            # Clonamnos el repositorio "jenkins-pipeline-library" 
Cloning into 'jenkins-pipeline-library'...
remote: Counting objects: 2131, done.
remote: Compressing objects: 100% (52/52), done.
remote: Total 2131 (delta 43), reused 85 (delta 38), pack-reused 2032
Receiving objects: 100% (2131/2131), 343.29 KiB | 1.60 MiB/s, done.
Resolving deltas: 100% (1300/1300), done.
$ cd jenkins-pipeline-library/ 
$ git checkout run-agents                                                                       # Para el ejemplo trabajaremos con el tag "run-agents" 
Note: checking out 'run-agents'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b <new-branch-name>

HEAD is now at 7d27a6f... New: Agent attachment

$ docker-compose ps                                                                             # Verificamos que no hay servicios en la plataforma
Name   Command   State   Ports
------------------------------
$ docker volume ls                                                                              # Verificamos que no existen volúmeneqs
DRIVER              VOLUME NAME
$ docker network ls                                                                             # Verificamos que únicamente tenemos las redes estándar
NETWORK ID          NAME                DRIVER              SCOPE
83cc08e29b76        bridge              bridge              local
8d0d0aa3be38        host                host                local
be98665938d5        none                null                local
$ bin/test.sh local                                                                             # Levantamos plataforma con el script
# Start jenkins as a docker-compose daemon
Creating network "jenkinspipelinelibrary_default" with the default driver
Creating volume "jenkinspipelinelibrary_jpl-dind-cache" with default driver
Creating jenkinspipelinelibrary_jenkins-dind_1   ... done
Creating jenkinspipelinelibrary_jenkins-agent1_1 ... done
Creating jenkinspipelinelibrary_jenkins-agent2_1 ... done
# Started platform with id 7383a990deed595fcd413a5691308c1189bcc06e8e523c6627560e78dfa4ad75 and port 0.0.0.0:32772
# Copy jenkins configuration and prepare code for testing
fatal: Needed a single revision
Switched to a new branch 'develop'
0da8f202679aecefdee812cba3be148e3cf123ec
Switched to a new branch 'release/v9.9.9'
Switched to a new branch 'hotfix/v9.9.9-hotfix-1'
Switched to a new branch 'jpl-test-promoted'
Switched to a new branch 'jpl-test'
# Waiting for jenkins service to be initialized
# Download jenkins cli
# Prepare agents
# Reload Jenkins configuration
$ docker-compose ps                                                                             # Revisamos los servicios de la plataforma en ejecución
                 Name                                Command               State            Ports         
----------------------------------------------------------------------------------------------------------
jenkinspipelinelibrary_jenkins-agent1_1   bash -c tail -f /var/log/*.log   Up                             
jenkinspipelinelibrary_jenkins-agent2_1   bash -c tail -f /var/log/*.log   Up                             
jenkinspipelinelibrary_jenkins-dind_1     wrapdocker java -jar /usr/ ...   Up      0.0.0.0:32772->8080/tcp
$ docker ps                                                                                     # Revisamos los containers que se han creado
CONTAINER ID        IMAGE                     COMMAND                  CREATED             STATUS              PORTS                     NAMES
1833d2576f67        jenkins/jnlp-slave        "bash -c 'tail -f /v…"   7 seconds ago       Up 5 seconds                                  jenkinspipelinelibrary_jenkins-agent2_1
feb35fd347dc        redpandaci/jenkins-dind   "wrapdocker java -ja…"   7 seconds ago       Up 5 seconds        0.0.0.0:32772->8080/tcp   jenkinspipelinelibrary_jenkins-dind_1
1ad4efe33b53        jenkins/jnlp-slave        "bash -c 'tail -f /v…"   7 seconds ago       Up 4 seconds                                  jenkinspipelinelibrary_jenkins-agent1_1
$ docker volume ls                                                                              # Revisamos los volúmenes que se han creado
DRIVER              VOLUME NAME
local               58772b1b07e0674cbc69cc053cda44d689d3ab6776ca21fe4063dc73e560da85
local               5e8e481500b30b8c2ae9cd091fe04cdcc625034a2847750e6549ce2b8cbc8f11
local               81b99e45d732f801ce042eabc5ec56d3f7e4a395af2f82e8d76befa0e542811c
local               9fdc23f342a48b9afef65ead049b12cd655265d61e4b447efcdc8d5dfba7583e
local               jenkinspipelinelibrary_jpl-dind-cache
$ docker network ls                                                                             # Revisamos las redes que se han creado 
NETWORK ID          NAME                             DRIVER              SCOPE
83cc08e29b76        bridge                           bridge              local
8d0d0aa3be38        host                             host                local
2d55068be4a8        jenkinspipelinelibrary_default   bridge              local
be98665938d5        none                             null                local
$ docker-compose down -v                                                                        # Paramos y eliminamos la plataforma y sus volúmenes 
Stopping jenkinspipelinelibrary_jenkins-agent2_1 ... done
Stopping jenkinspipelinelibrary_jenkins-agent1_1 ... done
Stopping jenkinspipelinelibrary_jenkins-dind_1   ... done
Removing jenkinspipelinelibrary_jenkins-agent2_1 ... done
Removing jenkinspipelinelibrary_jenkins-agent1_1 ... done
Removing jenkinspipelinelibrary_jenkins-dind_1   ... done
Removing network jenkinspipelinelibrary_default
Removing volume jenkinspipelinelibrary_jpl-dind-cache
$ docker-compose ps                                                                             # Verificamos que no tenemos containers en ejecución
Name   Command   State   Ports
------------------------------
$ docker volume ls                                                                              # Verificamos que los volúmenes se han eliminado
DRIVER              VOLUME NAME
$ docker network ls                                                                             # Verificamos que la red se ha eliminado
NETWORK ID          NAME                DRIVER              SCOPE
83cc08e29b76        bridge              bridge              local
8d0d0aa3be38        host                host                local
be98665938d5        none                null                local
```

Al levantar la plataforma:
* Se han creado los containers 
  * jenkinspipelinelibrary_jenkins-agent1_1
  * jenkinspipelinelibrary_jenkins-agent2_1
  * jenkinspipelinelibrary_jenkins-dind_1
* Se ha creado la red "jenkinspipelinelibrary_default"
* Se han creado los volúmenes
  * 58772b1b07e0674cbc69cc053cda44d689d3ab6776ca21fe4063dc73e560da85
  * 5e8e481500b30b8c2ae9cd091fe04cdcc625034a2847750e6549ce2b8cbc8f11
  * 81b99e45d732f801ce042eabc5ec56d3f7e4a395af2f82e8d76befa0e542811c
  * 9fdc23f342a48b9afef65ead049b12cd655265d61e4b447efcdc8d5dfba7583e
  * jenkinspipelinelibrary_jpl-dind-cache

Para el caso del ejemplo se crea una plataforma de Jenkins dockerizada compuesta por un nodo "master" y dos nodos "slave" conectados. Podemos obvservar cómo se ha añadido el prefijo "jenkinspipelineliberary" a redes, containers y volúmenes. Este prefijo está relacionado con el nombre del directorio donde hemos levantrado la plataforma, en este caso "jenkins-pipeline-library"

Utilizaremos esta misma "plataforma efímera dockerizada" con docker-compose en el apartado posterior de configuración y uso de agetes.

## Parte 2: Jenkins

Trabajaremos con un Jenkins dockerizado configurando todo lo necesario para poner a punto pipelines de CI

### Uso de Docker para montaje de Jenkins en local y dockerizado

Partiendo del proyecto Jenkins DIND https://github.com/red-panda-ci/jenkins-dind se genera esta imagen de docker https://hub.docker.com/r/redpandaci/jenkins-dind/ 

Utilizaremos la imagen de docker con Jenkins DIND para montar pipelines de ejemplo a partir de una organización de github. El propio proyecto dispone de un pipeline de CI/CD que cubre build, test, creación y "push" de imagen de docker y gestión de release. Se puede tomar ese archivo.

* Levantamos Jenkins en nuestro PC con Docker
```shell
$ docker run --privileged --rm -d --name jenkins-dind -e JENKINS_USER=redpanda -e JENKINS_PASS=redpanda -p 8080 redpandaci/jenkins-dind:2.89.4
Unable to find image 'redpandaci/jenkins-dind:2.89.4' locally
2.89.4: Pulling from redpandaci/jenkins-dind
1be7f2b886e8: Pull complete 
6fbc4a21b806: Pull complete 
c71a6f8e1378: Pull complete 
4be3072e5a37: Pull complete 
06c6d2f59700: Pull complete 
a93614305743: Pull complete 
e4144bdc31e8: Pull complete 
3c8f4870fc59: Pull complete 
0fa1644eaada: Pull complete 
f96180a32711: Pull complete 
ec55146fb59e: Pull complete 
a15b026a3e3f: Pull complete 
a32b2dfb0e74: Pull complete 
ac06df183566: Pull complete 
9be8a4b72272: Pull complete 
00d2f25b9939: Pull complete 
548a9b635033: Pull complete 
5042fd6bc83e: Pull complete 
eac30657d62c: Pull complete 
f50a5064892e: Pull complete 
Digest: sha256:bde5c5de8c30c4bda31d90552b6e845ed62bc886c808b858b74b6e70c534eca7
Status: Downloaded newer image for redpandaci/jenkins-dind:2.89.4
71a1203f7f9c3fb214925c3732a921af8f3048b84a469660bff04b715e221a92
```
* Arrancamos el navegador y abrimos el puerto que corresponda (para nuestro caso http://0.0.0.0:32775)
```shell
$ docker ps -a
CONTAINER ID        IMAGE                            COMMAND                  CREATED             STATUS              PORTS                     NAMES
71a1203f7f9c        redpandaci/jenkins-dind:2.89.4   "wrapdocker java -ja…"   22 seconds ago      Up 21 seconds       0.0.0.0:32775->8080/tcp   jenkins-dind
``` 

Tan sólo tenemos que acceder con el usuario / contraseña que hemos indicado, en nuestro caso "redpanda/redpanda". Vamos a comentar los parámetros del "docker run":

* --privileged => Utilizamos estrategia "docker in docker", es decir, que nuestro Jenkins dockerizado pueda a su vez correr docker
* --rm => El container se borrará de forma automática cuando se pare. ¡Cuidado! Al detener el container todos nuestros datos y configuraciones van a desaparecer
* -d => Ejecución en modo "daemon": nos devuelve el control, se queda ejecutando en segundo plano
* --name jenkins-dind => Le damos un nombre a nuesgtro container
* -e JENKINS_USER=redpanda -e JENKINS_PASS=redpanda => Variables de entorno de usuario y contraseña
* -p 8080 => Cuál es el puerto que vamos a "exponer" en nuestra red. Será mapeado con NAT a un puerto disponible a parti4r del 32767
* redpandaci/jenkins-dind:2.89.4 => Cuál es la imagen de docker hub de la que vamos a partir para levantar el container. Utilizamos la jenkins-dind:2.89.4, que se corresponde con Jenkins LTS 2.89.4

Las configuraciones de prueba del workshop las vamos a realizar sobre este container recién levantado

### Configuración y uso de agentes (nodos)

Si enlazamos con el apartado de Docker Compose, podremos ver como tenemos un Jenkins funcionando con dos agentes:

* agent1
* agent2

El usuario / contreaseña de nuestra instalación es "redpanda/redpanda"

### Gestión de plugins y configuración

La image Docker jenkins-dind con la que estamos trabajando tiene instalados todos los plugins necesarios para ejecutar los pipelines de los ejemplos. Se puede revisar la lista de plugins instalados accediendo en el menú de la izquierda en "Manage Jenkins -> Manage Plugins".
La configuración está accesible en "Manage Jenkins -> Configure System". Sobre este apartado de configuración es necesario realizar un ajuste en configuración para que los pipelines del ejemplo funcionen correctamente: se tiene que añadir el label "docker". 

* Accedemos a "Manage Jenkins -> Configure System"
* Buscamos el campo "Labels", que encontraremos vacío, y ponemos "docker"
* Pulsamos en "Save"

### Creación de organización Github y "Engagement" a Jenkins

Tenemos dos opciones para configurar proyectos en nuestro Jenkins: hacerlo uno por uno, repositorio a repositorio, o configurar una organización de Github completa. Cada método tiene sus ventajas e inconvenientes; trabajando con repositorios individuales tenemos configuración más granular, mientras que si lo hacemos a nivel organización únicamente tenemos que configurar una única vez.

Trabajando a "nivel organización github" tendremos:

* Configuración única
* "Engagement" de todos los repositorios de la organización que tengan "Jenkinsfile". Se revisan todas las ramas de todos los repositorios, y se crea un "Job" para cada rama
* Añadiendo un webhook en Github tendremos también auto-engagement de repositorios nuevos que tengan un fichero "Jenkinsfile" versionado

**Primer paso**: Crear una organización en Github

* Accedemos a github con nuestro usuario. En caso de no disponer de usuario de github se puede dar de alta un nuevo usuario de forma gratuita.
* En la página principal de nuestra cuenta pulsamos sobre el "+" en la parte superior derecha. Seleccionamos "New organization"
* Rellenamos los datos de la nueva organización
  * Una nombre que nos parezca adecuado y no esté adecuado. Para nuestro ejemplo será "jenkins-workshop-gigigo"
  * Una dirección de mail en "Billing address". Esto no implica coste si seleccionamos la opción "Free"
* Pulsamos sobre "Create organization"

**Segundo paso**: Añadimos un repositorio a nuestra organización

Haremos "fork" del proyecto "testlenium"

* Abrimos la URL https://github.com/red-panda-ci/testlenium en el navegador
* Pulsamos sobre "Fork" y seleccionamos la organización que acabamos de crear (en el caso del ejemplo "jenkins-workshop-gigigo")

Si todo ha ido bien tendremos una copia (fork) del repositorio en nuestra organización https://github.com/jenkins-workshop-gigigo/testlenium

**Tercer paso**: Creamos un proyecto nuevo en nuestro Jenkins "enganchado" a la organización Github

Embarcamos la organización Github en nuestro Jenkins efímero dockerizado:

* Abrimos la URL de Jenkins http://0.0.0.0:32775 y accedemos con "redpanda/redpanda"
* Pulsamos sobre "New item" en el menú de la izquierda
* En la página de "New item"
  * Introducimos el nombre de la organización github en la caja de texto ("jenkins-workshop-gigigo" para nuestro ejemplo)
  * Seleccionamos "GitHub Organization" (penúltimo elemento)
  * Pulsamos sobre "OK". Se creará el item y veremos la página de propiedades de la organización
* En la página de propiedades de la organización:
  * Añadimos nuestro usuario / contrraseña de Github en el apartado "Projects":
    * Seleccionamos el botón "Add" en "Credentials" (vemos que hay un desplegable que pone "-none-") y escogemos la primera opción ("jenkins-workshop-gigigo" en nuestro ejemplo)
    * Rellenamos usuario / contraseña con nuestras credenciales de Github y ponemos un texto descriptivo en ID, por ejemplo "github"
    * Al pulsar sobre "Add" volvemos a la página de propiedades, pero esta vez al desplegar en "Credentials" veremos la que acabamos de añadir. Seleccionamos esa.
    * Elegimos qué ramas van a disparar un "build": en la parte de abajo de la página hay una opción "Automatic branch project triggering" con una caja de texto, que rellenamos con "develop". De esta forma tendremos vigilada la rama develop de los repositorios. Para nuestro ejemplo es suficiente, en casos reales tendremos que incluir más ramas, o todas las ramas

Vamos a dejar el resto de opciones tal cual, y pulsamos en "Save" en la parte de abajo de la página para guardar los cambios. En ese momento Jenkins revisa todos los repositorios de la organización, buscando aquellos que tengan un archivo Jenkinsfile en cada una de las ramas vigiladas (recordemos: únicamente la rama "develop"). Para cada repositorio que cumpla con los criterios creará un "job" nuevo en cada rama que tenga archivo Jenkinsfile y ejecutará un "build"; en nuestro caso del ejemplo Jenkins se encontrará con el proyecto "testlenium".

En la parte de abajo a la izquierda de la página veremos "Build Executor Status" con un build en ejecución. Si pinchamos sobre él veremos el build en mientras se ejecuta, pudiendo acceder a la consola (Console Output de la izquierda), o bien podremos pinchar sobre "Open Blue Ocean" para tener una visión con esa interfaz. Sólo tendremos que esperar a que el build finalice para tener como artefactos vídeos de ejecución de los test, tal y como está programado realizarse por el Jenkinsfile del proyecto.

**Cuarto paso**: Configuramos un "hook" a nivel organización

Verificamos que Jenkins ha añadido un Webhook a nuestra organización apuntando a la URL del jenkins dockerizado (en nuestro ejemplo http://0.0.0.0:32775/github-webhook/). Si somos capaces de llevar tráfico HTTP al servicio Jenkins dockerizado, cuando sucedan eventos en la organización, como un push, creación de un pull request o creación de un repositorio, Github "llamará" a nuestro Jenkins para que se ponga a trabajar. Esta información la vemos accediendo a la organización Github desde el navegador, y pulsando en "Settings" => "Webhook"

Debido a la naturaleza de este taller (Jenkins efímero, dockerizado) no vamos a entrar en detalles de cómo hacerlo. Tendremos que buscar nosotros los cambios de forma explícita desde nuestro Jenkins. Es decir: usar una y otra vez la opción "Scan".

### Creación de proyecto Bitbucket y "Engagement" a Jenkins

La configuración es similar a la organización de Github. Debemos crear un proyecto bitbucket. En el caso de bitbucket podemos crear un proyecto privado con repositorios privados de manera gratuita. El vídeo no cobre esta parte y se puede experimentar con ello. 

RETO: crear proyecto Bitbucket y conectarlo a nuestro Jenkins de pruebas.

## Parte 3: Pipelines (enlazado con los dos últimos puntos de la parte 2)

### Uso de Jenkins Pipeline

### Configuración de job Android

### Configuración de job Back / Front

### Configuración de job iOS (en Jenkins QA)
