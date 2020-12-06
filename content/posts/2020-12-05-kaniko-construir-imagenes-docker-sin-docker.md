---
title: "Kaniko - Construir imágenes Docker sin  Docker"
author: eiximenis
description: "Kaniko es un proyecto de Google que permite construir imágenes Docker sin necesidad de tener Docker instalado. Si te preguntas por qué narices podemod querer eso... ¡sigue leyendo!"
date: 2020-12-06
draft: false
categories:
  - kubernetes
tags: ["docker","k8s" ]
---

En el contexto de este post, cuando digo "imágenes Docker" me refiero a imágenes [OCI](https://opencontainers.org/) y que se crean usando un [Dockerfile](https://docs.docker.com/engine/reference/builder/). La idea es **poder seguir usando nuestros Dockerfiles para generar imágenes OCI**, que luego pueden ser ejecutadas cualquier motor compatible, como el propio Docker o containerd (aunque si hablamos de ejecutar contenedores [Docker y containerd son lo mismo](https://www.docker.com/blog/what-is-containerd-runtime/)).

## Creando imágenes con Kaniko

Vamos a ver primero como usar Kaniko, para crear una imagen. En este ejemplo usaremos Docker para ejecutar Kaniko, lo que no es un escenario real: si tienes Docker disponible, no tienes necesidad alguna de usar ninguna otra herramienta adicional para crear imágenes. Pero es un escenario sencillo y nos sirve como introducción:

```
docker pull gcr.io/kaniko-project/executor:latest
docker run \
    -v $contextPath:/workspace \
    gcr.io/kaniko-project/executor:latest \
    --dockerfile /workspace/Dockerfile \
    --destination "$IMAGE:$TAG" \
    --context dir:///workspace/
```

Primero montamos un _bind mount_ entre `$contextPath`, que contiene la ruta local del contexto de build, y el directorio `/workspace` del contenedor. El parámetro `--dockerfile` indica donde se encuentra el Dockerfile (dentro del contenedor), en este ejemplo se asume que se encuentra en la raíz del contexto de build. El parámetro `--destination` indica el nombre y etiqueta de la imagen a crear y `--context` indica donde está el contenxto de build (el prefijo `dir://` indica un directorio local del contenedor).

Veamos un ejemplo. He creado el "Hello World" de ASP.NET Core (`dotnet new web --name Helloworld`) y luego he añadido el siguiente `Dockerfile`, en el mismo directorio donde está el `.csproj`:

```
FROM mcr.microsoft.com/dotnet/aspnet:5.0-buster-slim AS base
WORKDIR /app
EXPOSE 80

FROM mcr.microsoft.com/dotnet/sdk:5.0-buster-slim AS build
WORKDIR /src
COPY ["Helloworld.csproj", ""]
RUN dotnet restore "./Helloworld.csproj"
COPY . .
WORKDIR "/src/."
RUN dotnet build "Helloworld.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "Helloworld.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "Helloworld.dll"]
```

En este caso si quisiera construir esa imagen con Docker, me situaría en el directorio donde está el `Dockerfile` y teclearía:

```
docker build -t hellokaniko:latest .
``` 

Ahora vamos a usar Kaniko. Un tema a tener presente es que Kaniko no construye imágenes locales, siempre debe subirlas a un registro de imágenes, así que lo primero será autenticarnos con ese registro. En este ejemplo usaremos [Dockerhub](https://hub.docker.com/), en la documentación de Kaniko está como configurar otros registros.

Para configurar Dockerhub debe crearse un fichero `config.json` con el siguiente contenido:

```json
{
	"auths": {
		"https://index.docker.io/v1/": {
			"auth": "base64($login:$password)"
		}
	}
}
```

Así, `auth` es simplemente mi login y mi contraseña de DockerHub, separadas por dos puntos y codificadas en BASE64. Este fichero debe ubicarse en el directorio `/kaniko/.docker` del contenedor. Así si tengo el proyecto en `/mnt/d/test-kaniko/Helloworld` el comando seria el siguiente:

```
docker run \
    -v /mnt/d/test-kaniko/config.json:/kaniko/.docker/config.json \
    -v /mnt/d/test-kaniko/Helloworld:/workspace \
    gcr.io/kaniko-project/executor:latest \
    --dockerfile /workspace/Dockerfile \
    --destination "eiximenis/hellokaniko:latest" \
    --context dir:///workspace/
```

Una vez haya terminado, tu imagen estará en Dockerhub, y la podrás ejecutar con Docker:

```
docker run eiximenis/hellokaniko
```

¡Listos! Ya has construído un contenedor usando Kaniko, sin necesidad de Docker. Ahora pasemos al escenario interesante :)

## Construyendo imágenes Docker en Kubernetes

Si te planteas utilizar Kubernetes para ejecutar tus pipelines de CI/CD, tarde o temprano tendrás la necesidad de crear imágenes de Docker en Kubernetes. Ahí es donde Kaniko entra en juego, pero antes deja que te cuento como puedes construir imágenes Docker en un Kubernetes. Ambas alternativas que te contaré ahora requieren que Docker esté instalado en el clúster, algo que hasta ahora se daba casi por supuesto, pero que [con las recientes noticias](https://kubernetes.io/blog/2020/12/02/dockershim-faq/), va a ser cada vez menos común.

### Docker out of Docker

Esa alternativa consiste en conectar el docker de dentro del _pod_ al docker del nodo worker de Kubernetes (enlazando ambas pipes, la que usa la CLI de docker del pod con la del daemon del nodo). Los dos problemas de esa aproximación son que, al usar el docker del nodo worker, estamos ejecutando cosas sobre las que Kubernetes no tiene control. Por otro lado, el otro tema es, como te debes imaginar, de seguridad: El contenedor debe ejecutarse como privilegiado y al usar el docker del nodo tiene acceso a todos los contenedores que se ejecuten en este nodo. Si tu cluster que ejecuta los pipelines es un cluster dedicado, eso quizá no importe tanto, si por otro lado se trata del cluster productivo, puede ser algo muy peligroso.

Tradicionalmente "Docker out of Docker" se ha configurado creando un _bind mount_ entre la pipe del daemon del nodo worker y la pipe del docker ejecutándose en un contenedor del _pod_:

```yaml
containers:
  - name: ci-runner
    volumeMounts:
    - mountPath: /var/run/docker.sock
      name: dockerpipe
    securityContext:
      privileged: true
volumes:
  - name: dockerpipe
    hostPath:
      path: /var/run/docker.sock
      type: File
```

En este caso, el contenedor del pod debe tener la CLI de Docker, pero no es necesario que tenga el daemon, ya que la CLI del contenedor está conectada con el daemon del nodo.

### Docker in Docker

Docker in Docker consiste en ejecutar un contenedor que tiene su propio daemon de Docker. El contenedor debe ejecutarse también como privilegiado, pero en este caso al tener su propio daemon no tiene acceso a nada fuera de él, con lo que las implicaciones de seguridad son menores.

De todos modos Docker in Docker, [no está exento de problemas](https://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/) y no es ni mucho menos una panacea. Yo personalmente no lo he usado nunca, así que no puedo añadir mucho más al enlace pasado.

### Ejecutando Kaniko en Kubernetes

Si no hay una buena opción para crear imágenes de Docker en Kubernetes usando Docker, hay que buscar alternativas. Y ahí es donde entra Kaniko. A fin de cuentas, Kaniko hace justamente eso: construir imágenes Docker, sin necesidad alguna de Docker. Además no requiere ningún tipo de privilegio (salvo que Kaniko se ejecuta como `root`, pero el pod no es un pod privilegiado), por lo que no hay problemas adicionales de seguridad. Vamos, pues a ver como ejecutar Kaniko en Kubernetes para construir imágenes Docker desde un pod.

Necesitamos cuatro elementos para ejecutar Kaniko en Kubernetes:

* El fichero `Dockerfile` que usaremos
* El contenido del contexto de build (y credenciales para recuperarlo si son necesarias)
* La configuración de credenciales del registro de imágenes
* La propia definición del pod

El fichero `Dockerfile` se lo suministraremos usando un `ConfigMap` montado en un volumen, lo mismo que el fichero de credenciales del registro (aunque en este caso usaremos un `Secret`). Lo que genera más dudas es como pasarle al pod el contenido del contexto de build (que puede ser arbitrariamente grande).

Si el contexto de build no es muy grande, lo podemos comprimir y guardar el comprimido en un `Secret` de Kubernetes (para datos binarios, mejor usar secretos que ConfigMaps). Luego montamos el secreto en un directorio y ejecutamos Kaniko. Kaniko acepta ficheros comprimidos como contextos de build. Esa opción funciona, pero que los contextos de build deban ser subidos al cluster como secretos, es algo que chirría un poco.

Afortunadamente Kaniko está preparado para eso y soporta otros orígenes para los contextos de build, así que es capaz de obtener un contexto de build desde orígenes variopintos tales como un bucket de S3, un blob storage de Azure, un bucket de GCS o un repositorio de Git.

Para este ejemplo voy a usar un Azure Blob Storage, para guardar el contenido del contexto de build. Empecemos por preparar los recursos:

```bash
STORAGE=<nombre-storage>
RG=<nombre-grupo-recursos>
az storage account create -n $STORAGE -g $RG --sku Standard_LRS       # Creamos el storage
CONSTR=$(az storage account show-connection-string -g $RG -n $STORAGE --query connectionString -otsv)  # Obtenemos la cadena de conexión
STORAGE_KEY=$(az storage account keys list -g $RG -n $STORAGE  --query [0].value -otsv)  # Obtenemos la clave del storage
az storage container create -n context-builds --connection-string $CONSTR
```

Kaniko espera que pasemos la clave del storage (ojo, la clave, NO la cadena de conexión) a través de una variable de entorno llamada `AZURE_STORAGE_ACCESS_KEY`, así que la meteremos en un secret que llamaremos `kanikostorage`. Empecemos por crear este secreto:

```bash
kubectl create secret generic kanikostorage --from-literal AZURE_STORAGE_ACCESS_KEY=$STORAGE_KEY
```

Ahora vamos a crear el `ConfigMap` que contiene el `Dockerfile`:

```bash
kubectl create cm kanikodockerfile --from-file Dockerfile
```

El siguiente paso es crear otro `Secret` llamado `kanikoauth`, para almacenar las credenciales del DockerHub donde vamos a meter las imágenes (el fichero `config.json` con las credenciales en BASE64):

```bash
LOGIN=<login-dockerhub>
PWD=<password-dockerhub>
kubectl create secret generic kanikoauth --from-literal config.json="{ \"auths\": { \"https://index.docker.io/v1/\": { \"auth\":\"$(echo -n $LOGIN:$PWD | base64)\"}}}"
```

El penúltimo paso consiste en crear el archivo comprimido con el contexto de build y subirlo al storage. En este caso me situo en el directorio anterior al de mi contexto de build (en mi caso es `/mnt/d/test-kaniko`) y ejecuto `tar` para comprimir el directorio `Helloworld` que es el contexto de build. Finalmente subo el fichero comprimido al blob storage:

```bash
tar -C Helloworld -zcvf context.tar.gz .
az storage blob upload -c context-builds -n context.tar.gz -f context.tar.gz --account-name $STORAGE --account-key $STORAGE_KEY   # Subimos el fichero
```

Ya lo tenemos todo listo! Solo nos falta crear el job en Kubernetes para ejecutar un pod con Kaniko:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: build-helloworld
spec:
  template:
    spec:
      containers:
      - name: kaniko
        image: gcr.io/kaniko-project/executor:latest
        args:
        - --dockerfile=/workspace/Dockerfile
        - --destination=eiximenis/hellokaniko:latest
        - --context=https://testkaniko.blob.core.windows.net/context-builds/context.tar.gz
        env:
        - name: AZURE_STORAGE_ACCESS_KEY
          valueFrom:
            secretKeyRef:
              key: AZURE_STORAGE_ACCESS_KEY
              name: kanikostorage
        volumeMounts:
        - name: dockerfile
          mountPath: /workspace
        - name: auth
          mountPath: /kaniko/.docker/
      volumes:
        - name: dockerfile
          configMap:
            name: kanikodockerfile
        - name: auth
          secret:
            secretName: kanikoauth
      restartPolicy: Never
  backoffLimit: 4
```

Una vez se cree este job, el job creará un pod de kaniko que debe construir la imagen, y publicarla en Dockerhub. Y todo ello, sin necesidad de tener Docker en el cluster! :)

> **Nota**: Kaniko tiene soporte directo [para AWS ECR](https://github.com/GoogleContainerTools/kaniko#pushing-to-amazon-ecr) y obviamente [para Google GCR](https://github.com/GoogleContainerTools/kaniko#pushing-to-google-gcr), pero no para Azure ACR. Por suerte conseguir que Kaniko funcione con ACR es muy sencillo y, al menos hasta que no haya soporte directo, [los pasos a seguir están en esta issue](https://github.com/GoogleContainerTools/kaniko/issues/1180).

## Conclusiones

Kaniko es un interesante proyecto que permite la construcción de imágenes Docker sin necesidad de tener Docker instalado y sin necesidad de usar contenedores privilegiados. Se trata de una alternativa muy interesante si planeas ejecutar tus pipelines de CI/CD desde el propio Kubernetes.

Kaniko **no es el único proyecto** que permite construir imágenes OCI sin necesidad de Docker, a continuación te nombro algunos otros por si quieres echarles un vistazo:

* BuildKit: Vale, ese sí requiere Docker, ya que se trata, en definitiva, de la evolución de `docker build`. Si usas una versión moderna de Docker es posible **que ya venga habilitado por defecto**. Si tu `~/.docker/daemon.json` contiene `"features": { "buildkit": true }}` ya lo estás usando. En principio BuildKit puede ser ejecutado en un pod también, pero es algo que no he investigado mucho y no sé que requerimientos tiene. A diferencia de Kaniko, BuildKit es _rootless_ (el contendor no debe ser ejecutado con el usuario `root`), pero por contra requiere pods con privilegios, cosa que Kaniko no. Es algo un poco confuso

* [img](https://github.com/genuinetools/img): Se trata de un proyecto que usa BuildKit, pero sin el daemon de Docker. Eso nos deja en un escenario parecido al de Kaniko. No lo he mirado muchgo, así que poco más puedo añadir. Una ventaja _a priori_ de img sobre Kaniko sería que, dado que el primero usa BuildKit, es 100% compatible con cualquier Dockerfile y su comportamiento debería ser idéntico al de `docker build`. Kaniko es también compatible, pero al ser una implementación distinta, pueden haber errores o pequeños detalles que funcionen distinto.

* [Buildah](https://github.com/containers/buildah): Se trata de un proyecto de Red Hat. Soporta Dockerfiles y funciona sin daemon. Poco más puedo añadir.

* [Orca](https://github.com/cyphar/orca-build): Otro generador de imágenes OCI que soporta Dockerfile. No sé nada más de él :)

Bueno, como puedes ver, es perfectamente factible construir imágenes Docker (o sea, imágenes OCI usando un Dockerfile) sin necesidad de tener Docker, lo que nos abre interesantes escenarios para ejecutar pipelines de CI/CD en Kubernetes.

¡Espero que te haya resultado interesante! :)
