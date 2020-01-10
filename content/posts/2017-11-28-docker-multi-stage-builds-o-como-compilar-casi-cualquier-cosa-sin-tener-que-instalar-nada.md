---
title: Docker multi-stage builds… o como compilar (casi) cualquier cosa sin tener que instalar nada.

author: eiximenis

date: 2017-11-28T12:58:09+00:00
geeks_url: /?p=1926
geeks_ms_views:
  - 1771
categories:
  - docker
  - Sin categoría

---
Poco a poco los contenedores, y Docker en especial, se han ido abriendo paso en nuestro _workflow_ diario. Y si al principio era tan solo un mecanismo para encapsular aplicaciones, luego también vimos en ellos una magnífica opción para evitar tener que instalar mil dependencias en las máquinas de desarrollo y finalmente para compilar cualquier proyecto... sin tener que instalar ningún SDK en local.
  
<!--more-->


  
Hace muchos años, en mi primer trabajo, manteníamos un proyecto _legacy_. Tan legacy era que no iba en nuestros modernísimos Windows 2000 y teníamos que compilarlo en un NT 3.51. Así que teníamos un disco duro &#8220;externo&#8221; con un NT 3.51 instalado y las versiones **exactas** del compilador de C++ necesario. Lo de &#8220;externo&#8221; entre comillas es porque usábamos una caja que tenía dentro el disco duro y un conector IDE en el exterior. Para instalar el disco duro, &#8220;solo&#8221; hacía falta desmontar la caja, desconectar el disco duro y enchufar el conector IDE de la caja a la placa base. Cuando a algun nuevo desgraciado le tocaba colaborar en el proyecto se le daba la &#8220;caja mágica&#8221; con el disco duro, se le advertía de **no actualizar nada** y se le deseaba suerte.
  
Pero seguro que todos os habreis topado con una versión más o menos moderna de esa historia: un proyecto requiere algun SDK en concreto para compilarse y en tu máquina este SDK no está disponible. Con un poco de suerte puedes simplemente instalarlo y listos. Pero eso no siempre es fácil y/o posible... Por suerte en 2017 tenemos contenedores y ya no necesitamos andar con particiones, discos duros externos o máquinas virtuales.
  
**Docker al rescate**
  
La idea es muy sencilla: en lugar de usar un SDK en local para compilar mi código, podemos usar un contenedor para hacerlo en su lugar. Vamos a poner un ejemplo usando **netcore** pero lo mismo es aplicable a cualquier tecnología. Si tenemos un proyecto cualquiera en netcore, para compilarlo me basta con irme al directorio donde está el fichero _csproj_ y teclear &#8220;_dotnet publish -o app&#8221;_. Eso me dejará en un subdirectorio _app_ todo el contenido (binarios y otros posibles ficheros) que debo copiar en la máquina que debe ejecutar mi aplicación. En versiones anteriores a la 2.0 debo hacer antes &#8220;_dotnet restore_&#8220;, pero desde netcore2 eso ya no es necesario (el _publish_ hace el _restore_ por nosotros).
  
Eso puedo hacerlo yo en local si tengo el SDK de netcore instalado, pero el tema está en que no lo tengo ni quiero tenerlo. Así que vamos a ver como podemos hacerlo usando Docker.
  
Lo primero es crear &#8220;la imagen de compilación&#8221;. La imagen de compilación es una imagen que:

  * Contiene un SDK determinado (en este caso netcore2)
  * Contiene mis ficheros de código fuente
  * Usa el SDK para compilar mi proyecto

Si podemos partir de una imagen base que tenga el SDK instalado no necesitaremos ni un _Dockerfile_ propio: nos bastará con un fichero de compose. Por otro lado, si no tenemos una imagen base con el SDK o bien debemos personalizarla, entonces si que necesitaremos un _Dockerfile_ propio.
  
Para mi ejemplo tengo un proyecto (DemoWeb) que es una aplicación asp.net core 2.0. En &#8220;mi carpeta raíz&#8221; tengo el fichero DemoWeb.sln y una carpeta DemoWeb con el fichero csproj y todos los ficheros de código fuente.
  
En el caso de asp.net core, tenemos una imagen ([_microsoft/aspnetcore-build_][1]) que contiene el SDK de asp.net core (si solo queremos el de netcore, pues podemos usar [_microsoft/dotnet_][2] con los tags *-sdk). En nuestro caso, como la imagen base nos basta, no necesitamos ni _Dockerfile_. Nos basta con un fichero _docker-compose.build.yml_ tal y como sigue:

<pre class="EnlighterJSRAW" data-enlighter-language="json">version: '3'
services:
  ci-build:
    image: microsoft/aspnetcore-build:2.0.3
    volumes:
      - .:/src
    working_dir: /src
    command: /bin/bash -c "dotnet publish ./DemoWeb/DemoWeb.csproj -c Release -o obj/Docker/publish"</pre>

En mi caso he creado el fichero en mi &#8220;carpeta raíz&#8221; (donde tengo el DemoWeb.sln).
  
El fichero compose define un contenedor (_ci-build_) basado en la imagen _microsoft/aspnetcore-build:2.0.3_. La etiqueta &#8220;command&#8221; es importante porque indica cual va a ser el proceso que va a ejecutar el contenedor cuando lo levantemos. En nuestro caso el &#8220;_dotnet publish&#8221;_.
  
El otro punto clave es como pasamos los ficheros de código fuente al contenedor. Podríamos hacerlo usando una sentencia &#8220;COPY&#8221; en el _Dockerfile_ (si lo tuvieramos), pero esto implicaría recrear la imagen cada vez que nuestros ficheros de código fuente cambiaran. Es mucho mejor usar un _bind mount_, que básicamente consiste en compartir una carpeta de nuestra máquina con el contenedor. Eso es lo que hacemos en la etiqueta &#8220;volumes&#8221;. Ahí indicamos que vamos a compartir el &#8220;directorio actual de mi máquina&#8221; (es decir, el directorio en el cual está el fichero _docker-compose.build.yml_) con el contenedor. Y que dentro del contenedor usaremos el nombre &#8220;/src&#8221; para referirnos a dicho directorio. Como los _bind mounts_ se crean al poner en marcha un contenedor, si modificamos nuestros ficheros de código fuente, no tenemos que recrear ninguna imagen.
  
Observa también la etiqueta &#8220;_working_dir&#8221;_: esta etiqueta es equivalente a lanzar el comando &#8220;cd&#8221; dentro del contenedor. Es decir nos situamos dentro del directorio /src de nuestro contenedor... y recuerda que este directorio es realmente el directorio donde esté el fichero _docker-compose.build.yml_.
  
Cuando ponga este fichero compose (_docker-compose -f docker-compose.build.yml up_) veré un log parecido a:

<pre class="EnlighterJSRAW" data-enlighter-language="raw">Starting demoweb-build_1 ...
Starting demoweb-build_1 ... done
Attaching to demoweb-build_1
build_1  | Microsoft (R) Build Engine version 15.4.8.50001 for .NET Core
build_1  | Copyright (C) Microsoft Corporation. All rights reserved.
build_1  |
build_1  |   DemoWeb -&gt; /src/DemoWeb/bin/Release/netcoreapp2.0/DemoWeb.dll
build_1  |   DemoWeb -&gt; /src/DemoWeb/obj/Docker/publish/
demoweb-build_1 exited with code 0</pre>

El contenedor se ha creado, se ha puesto en marcha, ha ejecutado el _dotnet publish_ y ha muerto.
  
Ahora en **mi máquina ya tengo el resultado compilado.** Lo tengo en mi máquina porque &#8220;dotnet publish&#8221; ha dejado el resultado en el directorio /src/DemoWeb/obj/Docker/publish del contenedor... Pero recuerda que el directorio /app estaba mapeado a una carpeta de mi máquina local. Así, si voy a DemoWeb/obj/Docker/publish en mi máquina veo el resultado de la compilación. **He compilado el código de netcore2 sin necesidad de tener el SDK en local**.
  
Bien. Con esto usamos Docker para compilar nuestros proyectos. Ahora nos queda crear la imagen de Docker. Para ello necesitaría un _Dockerfile _(en mi carpeta /DemoWeb) parecido al siguiente:

<pre class="EnlighterJSRAW" data-enlighter-language="json">FROM microsoft/aspnetcore:2.0.3
WORKDIR /app
EXPOSE 80
COPY obj/Docker/publish .
ENTRYPOINT ["dotnet", "DemoWeb.dll"]
</pre>

No tiene mucha complicación: Partimos de la imagen _microsoft/aspnetcore:2.0.3_ y básicamente **copiamos el contenido del directorio obj/Docker/publish al contenedor**. Precisamente este es el directorio que hemos creado a través del contenedor Docker anterior. Ahora ya podemos construir y ejecutar contenedores con mi DemoWeb.
  
**Multi-stage builds**
  
La situación anterior nos evita tener el SDK instalado en local ya que el proceso de compilación se ejecuta en un contenedor Docker. Sin embargo el proceso tiene dos pequeños inconvenientes:

  1. Deja residuos en mi máquina: en este caso mi objetivo final es tener una imagen Docker con DemoWeb. El residuo es, por supuesto, el directorio local obj/Docker/publish.
  2. Se requieren Dockerfiles y/o ficheros compose separados para la creación del contenedor de compilación y la creación del contenedor final.

Ambos problemas pueden solucionarse usando una _multi-stage build_. Básicamente **una _multi-stage build_ es un proceso para construir un imagen final en el que pueden participar contenedores intermedios**. En nuestro caso el objetivo final es una imagen para crear contenedores que ejecuten DemoWeb. Y para crear esta imagen, vamos a crear un contenedor intermedio (con su propia imagen) que contenga el SDK. Dicho compilador compilará el código y transferiremos ficheros de un contenedor a otro **sin tener que usar ningún _bind mount_,** por lo que no habrá ningún residuo en mi máquina. Al finalizar el _multi-stage build_ lo único que tendré en mi máquina es una imagen para ejecutar DemoWeb.
  
Para eso tenemos que modificar el _Dockerfile_ de DemoWeb, que nos podría quedar parecido a:

<pre class="EnlighterJSRAW" data-enlighter-language="null">FROM microsoft/aspnetcore:2.0.3 AS base
WORKDIR /app
EXPOSE 80
FROM microsoft/aspnetcore-build:2.0.3 AS build
WORKDIR /src
COPY  . .
RUN dotnet build -c Release -o /app
FROM build AS publish
RUN dotnet publish -c Release -o /app
FROM base as final
WORKDIR /app
COPY --from=publish /app .
ENTRYPOINT ["dotnet", "SmartHotel.Services.Configuration.dll"]
</pre>

Vayamos por partes, instrucción por instrucción para que entiendas como funciona:

  * **FROM microsoft/aspnetcore:2.0.3 AS base:** Creamos un contenedor intermedio llamado base a partir de la imagen microsoft/aspnetcore:2.0.3
  * **WORKDIR /app**: Nos situamos en el directorio /app de dicho contenedor (si no existe, que no, se crea).
  * **EXPOSE 80:** Indicamos que este contenedor expondrá el puerto 80. Esto es solo indicativo para herramientas como compose.
  * **FROM microsoft/aspnetcore-build:2.0.3 AS build**: Creamos un contenedor intermedio (build) a partir de la imagen microsoft/aspnetcore-build:2.0.3. Este contenedor es el que usaremos para compilar el proyecto
  * **WORKDIR /src:** Nos situamos en el directorio /src de dicho contenedor**.**
  * **COPY . .**: Copiamos todo el directorio actual de la máquina local al contenedor de build. El significado de &#8220;directorio actual de la máquina local&#8221; depende del valor del contexto de build, pero vamos a asumir que es el directorio donde está el _Dockerfile_ (es decir el directorio DemoWeb). Por supuesto el directorio de destino en el contenedor es el directorio /src (por el WORKDIR anterior)
  * **RUN dotnet build -c Release -o /app**:  Ejecutamos dotnet build y dejamos el resultado en el directorio /app
  * **FROM build AS publish**: Creamos un contenedor intermedio (publish) a partir del contenedor intermedio publish. Esto crea un contenedor nuevo cuyo estado es idéntico al del contenedor build.
  * **RUN dotnet publish -c Release -o /app**:  Ejecutamos (en el contenedor publish) el dotnet publish y dejamos el resultado en el directorio /app.
  * **FROM base as final**: Creamos un contenedor (final) a partir del contenedor base (que habíamos creado al principio). Este contenedor será el resultado final del _Dockerfile_ (simplemente porque ya no crearemos ningún otro).
  * **WORKDIR /app**: Nos situamos en el directorio /app.
  * **COPY &#8211;from=publish /app .**: La clave que nos permite transferir ficheros de un contenedor a otro. En este caso, copiamos todo el contenido del directorio /app del contenedor &#8220;publish&#8221; al directorio actual (nuestro /app por el WORKDIR anterior).
  * **ENTRYPOINT [&#8220;dotnet&#8221;, &#8220;DemoWeb.dll&#8221;]: **Indicamos que el punto de entrada del contenedor es DemoWeb.dll

Listos. No necesitamos ya para nada el fichero _docker-compose.build.yml_. Si ahora hacemos simplemente un &#8220;docker build .&#8221; lo que ocurrirá es que se usarán los contenedores intermedios build y publish para compilar nuestro proyecto y el resultado final será el contenedor llamado &#8220;final&#8221; (simplemente porque es el de la última sentencia FROM).
  
¿El resultado? Hemos generado un contenedor que contiene unos binarios de netcore2 a partir de código fuente de nuestra máquina, sin necesidad alguna de tener instalado el SDK de netcore2, sin que quede ningun tipo de residuo en nuestra máquina y en una sola sentencia de Docker... ¿Quien da más? Por supuesto si tienes que construir más de un contenedor basta con crearse un fichero de compose y ya está.
  
En un proyecto en el que he estado teníamos código fuente en nodejs, netcore2 y java todo junto y un cliente Xamarin que usaba los servicios expuestos en estos contenedores. Tener _multi-stage builds_ permitía que cualquier desarrollador se generase todos los contenedores sin **tener que preocuparse simplemente de nada**. Solo necesitaba hacer &#8220;docker-compose build&#8221; y listos: Ya podía probar los contenedores en local...
  
... qué lejos han quedado los tiempos de ir con el disco duro dentro de una caja 😉

 [1]: https://hub.docker.com/r/microsoft/aspnetcore-build/
 [2]: https://hub.docker.com/r/microsoft/dotnet/