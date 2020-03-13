---
title: "Como ejecutar (fácilmente) operator-sdk en Windows"
author: eiximenis
description: Este es un post preliminar de una serie sobre Operator SDK que estoy preparando. Ahora vamos a ver simplemente como poder ejecutarlo en Windows. Si usas Linux o MacOS no necesitas mirar este post :)
date: 2020-03-13T13:00:00
series:
  - Operadores de Kubernetes
categories:
  - k8s
---

Estoy haciendo una serie de _posts_ (que ya irán saliendo) sobre como crear operadores para Kubernetes. Crear esos operadores implica usar la herramienta [operator-sdk](https://github.com/operator-framework/operator-sdk/releases) que **no está disponible para Windows** (está solo para MacOS y Linux).

Por supuesto puedes usar una MV Linux y olvidarte de este problema o incluso usar WSL (instalar go, git, kubectl y la [CLI de Docker configurada contra el daemon de Docker for Windows](https://nickjanetakis.com/blog/setting-up-docker-for-windows-and-wsl-to-work-flawlessly)), pero si no quieres hacerlo, aquí te propongo una solución con relativa poca fricción: se trata de empaquetar en un contenedor esta aplicación y usar docker para ejecutarla.

Así pues he creado un pequeño Dockerfile y un fichero compose que empaquetan la aplicación y todas sus dependencias (Go, Docker CLI, git y kubectl). Luego hay un fichero compose que os permite lanzar el contenedor rápidamente con los volúmenes adecuados. El fichero compose define 4 volúmenes:

* `/var/run/docker.sock` vinculado a `/var/run/docker.sock` del _host_ para que así, la CLI de Docker funcione contra el daemon de Docker for Windows
* `${userprofile}\.kube:/root/.kube`: Para compartir la configuración de `kubectl`
* `package-cache:/go/pkg`: Para guardar los resultados de descargar los paquetes de Go.
* `${SOURCE:-.}:/src`: Para vincular el directorio de nuestro proyecto (que debemos establecer en la variable `SOURCE`) al directorio `/src` del contenedor. La herramienta se ejecuta siempre vinculada a ese directorio.

Además el fichero compose fija el nombre del contenedor a `operator-sdk`.

Puedes ejecutar el contenedor con `docker run` pero entonces debes declarar esos cuatro volúmenes y el nombre del contenedor. Simplemente el fichero compose te lo pone más fácil:

```
docker-compose up -d
```

Y ya lo tienes en marcha y configurado con los valores anteriores. El contenedor se queda "en espera" (sin hacer nada), ya que su uso es permitirte usar `operator-sdk` en un entorno que lo permita.

Ahora simplemente para ejecutar un comando de `operator-sdk` usa `docker exec operator-sdk` antes. Así, p. ej. en lugar de usar `operator-sdk build test-operator:v1` puedes usar `docker exec operator-sdk operator-sdk build test-operator:v1`.
O te puedes crear un alias con `doskey`:

```
DOSKEY operator-sdk=docker exec operator-sdk operator-sdk $*
```

Y ahora simplemente usas "operator-sdk" como si lo tuvieras instalado y dejas que Docker haga su magia :)

## Descargando los ficheros y creando la imagen

He [dejado en un gist](https://gist.github.com/eiximenis/dc79ebde69ad6015f78836ef1748a186) ambos ficheros (el Dockerfile y el fichero compose). Los puedes [descargar desde este enlace](https://gist.github.com/eiximenis/dc79ebde69ad6015f78836ef1748a186).

Para crear la imagen de operator-sdk simplemente teclea:

```
docker-compose build
```

Por defecto descargará la versión v0.15.2, si quieres descargar una versión distinta de operator-sdk, simplemente establece la variable de entorno `RELEASE_VERSION` con el valor deseado. La imagen generada se llamará `operator-sdk` y la etiqueta será la versión usada.

Para poner en marcha el contenedor establece la variable de entorno `SOURCE` con el valor del directorio (local) donde tienes el código de tu operador, y luego simplemente ejecuta `docker-compose up`. Si descargaste una versión específica, asegúrate que la variable de entorno `RELEASE_VERSION` contiene la versión deseada.

Si creas un operador desde cero, hay un poco de fricción ya que no tienes el directorio creado. Hay varias maneras de lidiar con ella. La más sencilla és:

1. Estableces a `SOURCE` el valor del directorio "padre" donde se creará el nuevo operador
2. Ejecutas el comando `operador-sdk new` que te creará el nuevo operador en el subdirectorio especificado
3. Paras el contenedor
4. Estableces a `SOURCE` el valor del directorio donde está el esqueleto del operador creado
5. Pones en marcha el contenedor otra vez

No descarto facilitar este paso, pero no le he dedicado más tiempo por ahora :)

## Usándolo con Minikube

Dado que el fichero de configuración de `kubectl` se comparte via un _bind mount_ es importante que te asegures de que los certificados de acceso están embebidos en él. En caso contrario recibirás un error parecido a este cuando se ejecute cualquier comando que requiera `kubectl`:

```raw
root@52b1af725abb:/src# kubectl get nodes
Error in configuration:
* unable to read client-cert /root/.kube/C:\Users\etoma\.minikube\client.crt for minikube due to open /root/.kube/C:\Users\etoma\.minikube\client.crt: no such file or directory
* unable to read client-key /root/.kube/C:\Users\etoma\.minikube\client.key for minikube due to open /root/.kube/C:\Users\etoma\.minikube\client.key: no such file or directory
* unable to read certificate-authority /root/.kube/C:\Users\etoma\.minikube\ca.crt for minikube due to open /root/.kube/C:\Users\etoma\.minikube\ca.crt: no such file or directory
```

Eso es porque el fichero de configuración no contiene el certificado si no la ruta de donde está el certificado, y obviamente, esa ruta no existe en el contenedor. Lo más sencillo es asegurarte de que el certificado se incluye embebido en el fichero de configuración. Para ello, en el caso de [Minikube](https://github.com/kubernetes/minikube), teclea:

```
minikube config set embed-certs true
```

Y luego relanza Minikube.

Con ello el fichero de `kubectl` tendrá los certificados y todo te funcionará. Si usas un Kubernetes en el _cloud_ (como AKS p. ej) lo habitual es que el fichero de `kubectl` ya tenga los certificados embebidos en su interior y no debas hacer nada.

¡Espero que te sirva... y que te apuntes a ver como crear operadores de Kubernetes, que es algo realmente interesante!

