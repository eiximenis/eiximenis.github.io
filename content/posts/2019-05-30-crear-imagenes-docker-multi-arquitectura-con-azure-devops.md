---
title: Crear imágenes Docker multi-arquitectura con Azure Devops
description: Crear imágenes Docker multi-arquitectura con Azure Devops
author: eiximenis

date: 2019-05-30T10:12:23+00:00
geeks_url: /?p=2357
geeks_ms_views:
  - 819
categories:
  - devops
  - docker

---
Ahora que los contenedores windows empiezan a funcionar decentemente, nos **puede interesar crear imágenes Docker multi-arquitectura** para que se puedan desplegar en contenedores Windows o Linux dependiendo de las necesidades. En este post te cuento como hacerlo usando Azure Devops.
  
Vamos a ver primero qué es una imagen multi-arquitectura, como crearla con la CLI de Docker y finalmente como hacerlo desde Azure Devops 🙂
  
<!--more-->


  
**¿Qué es una imagen multi-arquitectura?**
  
**Docker no es un sistema de virtualización**, eso significa que un _host_ Docker solo puede ejecutar contenedores cuyos binarios sean de su misma arquitectura. En este caso por &#8220;arquitectura&#8221; no me refiero solo a la arquitectura hardware (si es x86, ARM o x64 p. ej.) si no también el sistema operativo. Eso significa que **un _host_ Windows no puede ejecutar contenedores Linux y viceversa**.
  
Si intentas descargarte una imagen que no es de la arquitectura correcta, vas a recibir un error. Hasta hace relativamente poco para diferenciar entre arquitecturas distitntas se usaban distintos _tags_ de la imagen Docker. Así, p. ej. podía tener una imagen con los tags _1.0-linux_ y _1.0-windows_. Y cada cual debía usar el _tag_ que necesitaba en función del sistema operativo.
  
Eso genera ciertas fricciones. P. ej. las imágenes de ASP.NET Core se proporcionan para ambos sistemas operativos. Si yo creo una imagen a partir de ASP.NET Core, dependiendo del tag que use en mi Dockerfile, mi imagen generada será para Windows o para Linux. Esto es contraproducente, ya que p. ej. desarrolladores Linux y Windows no pueden compartir las imágenes a pesar de ASP.NET Core multiplataforma. Hay técnicas para lidiar con ello (como usar argumentos de build) pero no es la situación ideal. **Para solucionar este problema se creó el concepto de imagen multi-arquitectura**. La idea es que una misma imagen Docker contiene dos o más arquitecturas. De este modo ya no debes tener _tags_ para cada arquitectura: si un usuario de Windows hace un _docker pull_ de tu imagen se descargará la versión Windows y si es uno de Linux, se descargará la de Linux (de hecho, recuerda que eso incluye la arquitectura hardware también: si un usuario de Linux bajo ARM hace el _docker pull_ de descargará la versión Linux ARM). Si la versión correspondiente no existe, el usuario recibirá un error.
  
Docker Hub te informa de si una imágen es multi-arquitectura:
  
[<img class="alignnone size-full wp-image-2358" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/05/docker-tags-multiarch.png" alt="Docker hub muestra, para cada tag, sus arquitecturas" width="292" height="384" />][1]
  
Observa como los _tags_ master, dev y latest, tienen los iconos de Linux y Windows, indicando que están disponibles para esos dos sistemas operativos.
  
Vale, **a pesar de que las llamamos &#8220;imágenes multiarquitectura&#8221;, en el fondo eso no existe**. Una imagen de Docker sigue siendo para una sola arquitectura. Realmente lo que se hace es **crear un manifiesto que asocia un _tag virtual _a _uno o más imágenes_.  **Por ejemplo, el tag &#8220;dev&#8221; de la imagen anterior está asociado a imágenes reales (de los tags &#8220;linux-dev&#8221; y &#8220;win-dev&#8221;). De este modo, cuando el usuario hace un _docker pull_ usando el tag _dev_, la imagen real que se descargará, será la de _linux-dev_ o _win-dev_, en función de su arquitectura. Básicamente lo que ocurre es que cuando Docker se va a descargar el _tag dev_, descubre que no es una imagen real, si no un manifiesto que le indica que imagen debe bajarse para cada plataforma, y entonces se descarga dicha imagen.
  
De hecho, **y eso es importante, el manifiesto no contiene referencia a los _tags_ subyacentes si no a los identificadores (SHA256) de las imágenes**. Eso puede parecer que no importa demasiado, pero significa que el manifiesto **debe recrearse cada vez que se sube una de las imágenes subyacentes**, ya que en caso contrario quedaría desfasado.
  
**Crear un manifiesto (imagen multi-arquitectura)**
  
Para crear una imagen multi-arquitectura se debe usar el comando _docker manifest_. Este comando es experimental, así que debes habilitar los comandos experimentales en Docker. Pero ojo, no en el daemon si no en la CLI. Tener seleccionada la casilla &#8220;Experimental features&#8221; en la pestaña &#8220;Daemon&#8221; de las opciones de Docker no te va a servir. En su lugar debes editar el fichero ~/.docker/config.json (si no existe lo creas) y agregar:

<pre class="EnlighterJSRAW" data-enlighter-language="json">"experimental": "enabled"</pre>

Ahora el comando &#8220;docker manifest&#8221; ya debe funcionarte.
  
La sintaxis es muy sencilla:

<pre class="EnlighterJSRAW" data-enlighter-language="null">docker manifest create &lt;imagen&gt;:&lt;tag&gt; &lt;imagen&gt;:&lt;tag-linux&gt; &lt;imagen&gt;:&lt;tag-windows&gt; &lt;imagen&gt;:&lt;otro-tag-para-otra-arquitectura&gt;</pre>

Simplemente entras primero el nombre de la imagen y el tag &#8220;virtual&#8221; que quieras que sea multi-arquitectura y luego entras todas y cada una de las imagenes reales. **No es necesario que tengas las imágenes en disco, solo que estén en Docker Hub:**
  
[<img class="alignnone size-full wp-image-2359" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/05/docker-manifest-create.png" alt="Uso de docker manifest create (sin tener las imágenes en local)" width="869" height="295" />][2]
  
Observa como puedo crear el manifiesto _eshop/webmvc:dev_ a partir de las imágenes _eshop/webmvc:linux-dev_ y _eshop/webmvc:win-dev_ sin necesidad de tener dichas imágenes descargadas.
  
Esto ha creado el manifiesto, que lo tengo en local. Los manifiestos se guardan en ~/.docker/manifests:
  
[<img class="alignnone size-full wp-image-2360" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/05/manifest-content.png" alt="Contenido de un manifiesto" width="631" height="122" />][3]
  
El comando _docker manifest inspect <imagen>:<tag>_ te da la información del manifiesto. **Si el manifiesto lo tienes en local, te da la información del manifiesto local, en caso contrario te da la información del manifiesto que está en Docker Hub**.
  
**Una vez tienes el manifiesto en local, puedes subirlo a Docker Hub, con un docker manifest push**, como si fuese una imagen normal y corriente.
  
**<span style="text-decoration: underline;">Importante</span>:** Dado que el manifiesto contiene los SHA256 de las imágenes debes recrear y volver subir (con **_docker manifest push_) el manifiesto cada vez que subas (actualices) una de las imágenes subyacentes**.
  
**Crear una imagen multi-arquitectura con Azure Devops**
  
Vamos a suponer una _build_ que usa Azure Devops para construir una imagen tanto en Linux como en Windows:

<pre class="EnlighterJSRAW" data-enlighter-language="json">variables:
    registryEndpoint: eshop-registry
jobs:
- job: BuildLinux
  pool:
    vmImage: 'ubuntu-16.04'
  steps:
  - task: DockerCompose@0
    displayName: Compose build webmvc
    inputs:
      dockerComposeCommand: 'build webmvc'
      containerregistrytype: Container Registry
      dockerRegistryEndpoint: $(registryEndpoint)
      dockerComposeFile: docker-compose.yml
      qualifyImageNames: true
      projectName: ""
      dockerComposeFileArgs: |
        TAG=$(Build.SourceBranchName)
  - task: DockerCompose@0
    displayName: Compose push webmvc
    inputs:
      dockerComposeCommand: 'push webmvc'
      containerregistrytype: Container Registry
      dockerRegistryEndpoint: $(registryEndpoint)
      dockerComposeFile: docker-compose.yml
      qualifyImageNames: true
      projectName: ""
      dockerComposeFileArgs: |
        TAG=$(Build.SourceBranchName)
- job: BuildWindows
  pool:
    vmImage: 'windows-2019'
  steps:
  - task: DockerCompose@0
    displayName: Compose build webmvc
    inputs:
      dockerComposeCommand: 'build webmvc'
      containerregistrytype: Container Registry
      dockerRegistryEndpoint: $(registryEndpoint)
      dockerComposeFile: docker-compose.yml
      qualifyImageNames: true
      projectName: ""
      dockerComposeFileArgs: |
        TAG=$(Build.SourceBranchName)
        PLATFORM=win
  - task: DockerCompose@0
    displayName: Compose push webmvc
    inputs:
      dockerComposeCommand: 'push webmvc'
      containerregistrytype: Container Registry
      dockerRegistryEndpoint: $(registryEndpoint)
      dockerComposeFile: docker-compose.yml
      qualifyImageNames: true
      projectName: ""
      dockerComposeFileArgs: |
        TAG=$(Build.SourceBranchName)
        PLATFORM=win
- template: ../multiarch.yaml
  parameters:
    image: webmvc
    branch: $(Build.SourceBranchName)
    registryEndpoint: $(registryEndpoint)</pre>

Este ejemplo esté sacado de eShopOnContainers (<https://github.com/dotnet-architecture/eShopOnContainers/tree/dev/build/azure-devops/webmvc>). Solo que aquí he eliminado lo que no es relevante para este post.
  
La build define dos jobs (_BuildLinux_ y _BuildWindows_) que usan tareas de _Docker Compose_ para construir las dos imágenes. Cada uno de los jobs se ejecuta en un agente distinto, ya que necesitamos un agente Linux para construir la imagen Linux y un agente Windows para construir la imagen Windows.
  
El fichero compose de eShopOnContainers espera un parámetro llamado &#8220;PLATFORM&#8221; que puede valer Linux o Windows (por defecto asume Linux). Dicho parámetro se usa básicamente para generar _tags_ distintos de las imágenes (el tag se genera con la forma PLATFORM-TAG). Así generamos tags &#8220;win-dev&#8221; o &#8220;linux-dev&#8221;.
  
Lo interesante viene en el paso final, **donde se invoca a un template** que recibe tres parámetros:

  * image: El nombre de la imagen
  * branch: La rama. Se usa para los nombres de los tags
  * registryEndpoint: Conexión de Azure Devops al registro de docker a usar

**Este template es el que crea el manifiesto y lo publica, definiendo un _job _adicional:**

<pre class="EnlighterJSRAW" data-enlighter-language="null">parameters:
  image: ''
  branch: ''
  registry: 'eshop'
  registryEndpoint: ''
jobs:
- job: manifest
  condition: and(succeeded(),ne('${{ variables['Build.Reason'] }}', 'PullRequest'))
  dependsOn:
    - BuildWindows
    - BuildLinux
  pool:
    vmImage: 'Ubuntu 16.04'
  steps:
  - task: Docker@1
    displayName: Docker Login
    inputs:
      command: login
      containerregistrytype: 'Container Registry'
      dockerRegistryEndpoint: ${{ parameters.registryEndpoint }}
  - bash: |
      mkdir -p ~/.docker
      sed '$ s/.$//' $DOCKER_CONFIG/config.json &gt; ~/.docker/config.json
      echo ',"experimental": "enabled" }' &gt;&gt; ~/.docker/config.json
      docker --config ~/.docker manifest create ${{ parameters.registry }}/${{ parameters.image }}:${{ parameters.branch }} ${{ parameters.registry }}/${{ parameters.image }}:linux-${{ parameters.branch }} ${{ parameters.registry }}/${{ parameters.image }}:win-${{ parameters.branch }}
      docker --config ~/.docker manifest create ${{ parameters.registry }}/${{ parameters.image }}:latest ${{ parameters.registry }}/${{ parameters.image }}:linux-latest ${{ parameters.registry }}/${{ parameters.image }}:win-latest
      docker --config ~/.docker manifest push ${{ parameters.registry }}/${{ parameters.image }}:${{ parameters.branch }}
      docker --config ~/.docker manifest push ${{ parameters.registry }}/${{ parameters.image }}:latest
    displayName: Create multiarch manifest
</pre>

Es bastante sencillo:

  1. Usa una tarea de Docker@2 para hacer el Docker Login. Por eso necesitamos pasar la conexión del registro. Si usáramos &#8220;docker login&#8221; desde una tarea bash, deberíamos pasarle el login y el password, pero usando la tarea podemos usar la conexión de Azure Devops. La tarea de Docker@2, **crea un fichero de config temporal de Docker donde guarda el token de autorización.** La ubicación de este fichero está en la variable de entorno DOCKER_CONFIG
  2. Luego la tarea _bash_ lo que hace es: 
      1. **Crear una copia del fichero de configuración temporal que nos ha creado la tarea Docker@2 , pero quitando el último carácter (usando _sed_), que es la llave de cierre de JSON.** Esta copia se ubica en ~/.docker
      2. Añadir a esta copia el valor &#8220;experimental&#8221; a &#8220;enabled&#8221;, para así **habilitar el comando &#8220;docker manifest&#8221;**
      3. Usar &#8220;docker manifest create&#8221; para crear el manifiesto (usando este nuevo fichero de configuración)
      4. Usar &#8220;docker manifest push&#8221; para subir el manifiesto (usando este nuevo fichero de configuración)
  3. Finalmente el &#8220;dependsOn&#8221; nos asegura que este _job_ solo se ejecutará una vez el BuildWindows y BuildLinux haya terminado. Si no, se podría ejecutar antes y entonces subiríamos el manifiesto antes de la imagen real (y estaría desactualizado).

Puede parecer un poco complejo el usar una tarea Docker@2 para el login y luego tener que hacer la guarrada esa del _sed_ y poder añadir la entrada &#8220;experimental&#8221; en el fichero de configuración, pero **creo que es un poco mejor que usar una tarea bash con un &#8220;docker login&#8221;**, ya que entonces deberíamos pasarle el usuario y el password (y no podríamos usar una conexión de Azure Devops que es más segura).
  
Observa la sintaxis _${{ ... }}_ que usamos dentro de los templates. La verdad es que esta sintaxis es muy potente, permitiendo no solo acceder a los parámetros si no también iterar sobre ellos para generar el yaml final. En nuestro caso solo lo usamos para acceder a los parámetros. Pero es [un sistema bastante potente][4].
  
¡Y listos! Ahora la build ejecuta los dos jobs &#8220;BuildWindows&#8221; y &#8220;BuildLinux&#8221; que construyen y suben las imagenes de Windows y Linux al registro de Docker y finalmente, se crea el manifiesto y se sube al registro. Oberva como el agente que ejecuta el último job, no tiene porque ser el mismo que ha ejecutado los anteriores (recuerda que para crear un manifiesto no tienes por qué tener las imágenes en local... cosa obvia, ya que no vas a poder tener imágenes de varias arquitecturas a la vez!).
  
Espero que os haya sido útil!

 [1]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/05/docker-tags-multiarch.png
 [2]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/05/docker-manifest-create.png
 [3]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/05/manifest-content.png
 [4]: https://docs.microsoft.com/en-us/azure/devops/pipelines/process/templates?view=azure-devops