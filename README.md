# Jenkins Workshop

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
```

Cada vez que hacemos "docker pull" de una imagen:

* En caso que imagen no exista en nuestro PC, se descargará del registry
* Si la imagen ya la tenemos en nuesto PC, comprobará si la está actualizada, descargando la última versión en caso necesario

Podemos indicar una versión concreta a la hora de descargar una imagen:

```shell
$ docker pull redpandaci/jenkins-dind:2.89.4
```

En caso de no especificar versión, se bajará la última (latest)

```shell
$ docker pull redpandaci/jenkins-dind:latest    # Equivalente a "docker pull redpandaci/jenkins-dind"
```

#### Containers


#### Redes

#### Volúmenes

### Docker compose

## Parte 2: Jenkins

### Uso de Docker para montaje de Jenkins en local y dockerizado

### Configuración y uso de agentes (nodos)

### Gestión de plugins y configuración

### Creación de organización Github y "Engagement" a Jenkins

### Creación de projecto Bitbucket y "Engagement" a Jenkins

## Parte 3: Pipelines (enlazado con los dos últimos puntos de la parte 2)

### Uso de Jenkins Pipeline

### Configuración de job Android

### Configuración de job Back / Front

### Configuración de job iOS (en Jenkins QA)
