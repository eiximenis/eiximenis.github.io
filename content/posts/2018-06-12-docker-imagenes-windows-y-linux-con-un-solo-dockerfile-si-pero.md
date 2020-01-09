---
title: 'Docker: Imágenes Windows y Linux con un solo Dockerfile. Sí, pero…'
author: eiximenis

date: 2018-06-12T15:28:11+00:00
geeks_url: /?p=2107
geeks_ms_views:
  - 838
  - 838
categories:
  - asp.net core
  - docker

---
Una de las características menos conocidas de Docker son las imágenes multi-arch. Es una característica que agradecerás si trabajas tanto en contenedores Linux como Windows.
  
Como ya debes saber, Docker no permite ejecutar contenedores cuyos binarios no sean los mismos de la plataforma que los hospeda: es decir, si estás en Windows solo puedes ejecutar contenedores cuyos binarios sean Windows y si estás en Linux pues lo mismo. Y sí, Docker CE For Windows &#8220;hace tramas&#8221; para permitirte ejecutar contenedores Linux: los ejecuta en una máquina virtual Hyper-V.
  
Si desarrollas un producto multi-plataforma y quieres ofrecer contenedores tanto Linux como Windows, el uso de imágenes multi-arch es algo que, como digo, vas a agradecer, **ya que te permite usar un solo Dockerfile** para generar ambas imágenes.
  
<!--more-->


  
A pesar de que hablamos de &#8220;imagen multi-arch&#8221; realmente son distintas imágenes que se agrupan bajo un mismo nombre y en función de la arquitectura sobre la cual corre Docker se usa una u otra. Para nosotros es como si solo hubiese una imagen, pero realmente son N de distintas (una por arquitectura soportada).
  
Tomemos por ejemplo que **desarrollas un producto con NetCore**, que es multiplataforma, y que deseas proporcionar tu producto tanto en contenedores Linux como Windows. **Antes de multi-arch necesitabas dos Dockerfiles**, uno para cada plataforma, ya que la imagen base era distinta. Generalmente se usaban _tags_ para diferenciarlas. Así en un Dockerfile de Linux tenías:

<pre class="EnlighterJSRAW" data-enlighter-language="null">FROM microsoft/dotnet:2.1.0-aspnetcore-runtime-alpine3.7</pre>

Y en el de Windows:

<pre class="EnlighterJSRAW" data-enlighter-language="null">FROM microsoft/dotnet:2.1.0-aspnetcore-runtime-nanoserver-1803</pre>

Por decir algo. Solo esa distinción de la imagen base te obligaba a tener _Dockerfiles_ distintos,[ **al menos hasta que allá por el Abril del 2017 se añadió la capacidad de usar argumentos de builds (ARG) en la sentencia FROM**][1].
  
De hecho, desde entonces, usando argumentos de build podemos tener un solo Dockerfile que use imágenes base distintas:

<pre class="EnlighterJSRAW" data-enlighter-language="null">ARG imageTag=2.1.0-aspnetcore-runtime-alpine3.7
FROM microsoft/dotnet:imageTag</pre>

El uso de ARG admite más escenarios pero es más &#8220;viscoso&#8221;: debes pasar el argumento de build, ya sea con _docker build_ o usando _compose_. Usando imágenes multi-arch por su parte todo es más sencillo:

<pre class="EnlighterJSRAW" data-enlighter-language="null">FROM microsoft/dotnet:2.1.0-aspnetcore-runtime</pre>

Si construyes ese Dockerfile bajo Windows se usará la imagen base en arquitectura Windows y si lo construyes bajo Linux, la imagen base en arquitectura Linux. Todo muy sencillo y fácil.
  
Ambos mecanismos (ARG y multi-arch) no son exclusivos y de hecho se combinan. Es habitual un Dockerfile tipo:

<pre class="EnlighterJSRAW" data-enlighter-language="null">ARG imageTag=2.1.0-aspnetcore-runtime
FROM microsoft/dotnet:imageTag</pre>

De este modo, por defecto, se usa la imagen multi-arch, pero permitimos pasar una etiqueta concreta por si se desea usar un tag concreto (p. ej. 2.1.0-aspnetcore-runtime-bionic). Eso nos permite usar una imagen base concreta, lo que en algunos escenarios es deseable.
  
El uso de imágenes multi-arch funciona no solo en el Dockerfile claro, si no también en docker pull, run o similares. Si la imagen tiene soporte para multi-arch, al hacer docker pull o run usarás la de tu arquitectura (si existe, claro).
  
Un temilla: A pesar de que he hablado solo de Linux y Windows hay varias arquitecturas dentro de esos dos grupos. P. ej. x32 o x64, o bien ARM32 o ARM64. E incluso dentro de Windows hay cuatro &#8220;subarquitecturas&#8221;: RS1, RS2, RS3 y RS4 (RS es _RedStone_) y se refieren a los cuatro (hasta hoy) major updates:

  * RS1: Windows 10 o Windows Server 2016
  * RS2: Versión 1607 (Anniversary Edition)
  * RS3: Versión 1709 (Fall Creators Update)
  * RS4: Versión 1803 (Spring 2018 Update)

Hay una [matriz de compatibilidad][2] entre esas subarquitecturas, que te dice qué versiones de imagen se soporta en función de la versión Windows del _host_. Eso significa que, si usas una imagen multi-arch, la imagen real que obtengas será distinta en función de la versión de Windows en la que estés.
  
**El problema: personalización de imágenes**
  
Con independencia de que uses multi-arch o ARG para tener un solo Dockerfile, en general todo funciona bien si te limitas a las instrucciones básicas del Dockerfile: COPY, ENV, ENTRYPOINT y alguna más. Pero en cuando debas hacer personalizaciones avanzadas **la abstracción se rompe y empieza a ser complicado (o imposible) usar un solo Dockerfile**. Voy a usar como ejemplo [eShopOnContainers][3] y lo que hemos hecho en la migración a .NET Core 2.1. El proyecto que usaré para contar eso es la web SPA, pero aplica también a otros proyectos.
  
Hasta Net Core 2.1 no teníamos ningún problema y un solo Dockerfile multi-stage build, nos permitía crear la imagen:

<pre class="EnlighterJSRAW" data-enlighter-language="null">FROM microsoft/aspnetcore:2.0.5 AS base
WORKDIR /app
EXPOSE 80
FROM microsoft/aspnetcore-build:2.0.5-2.1.4 AS build
WORKDIR /src
COPY . .
RUN dotnet restore -nowarn:msb3202,nu1503
WORKDIR /src/src/Web/WebSPA
RUN dotnet build --no-restore -c Release -o /app
FROM build AS publish
RUN dotnet publish --no-restore -c Release -o /app
FROM base AS final
WORKDIR /app
COPY --from=publish /app .
ENTRYPOINT ["dotnet", "WebSPA.dll"]</pre>

Este Dockerfile usa las imágenes multi-arch _microsoft/aspnetcore2.0.5_ y _microsoft/aspnetcore-build:2.0.5-2.1.4_ como imágenes base. Por lo tanto puedes construir ese Dockerfile tanto en Windows como en Linux.
  
El problema viene porque en Net Core 2.1 han cambiado las imágenes base y ahora son _microsoft/dotnet:2.1-aspnetcore-runtime_ para el runtime y _microsoft/dotnet:2.1-sdk_ para la imagen del SDK. Y, a diferencia de la imagen _microsoft/aspnetcore-build_ la de _dotnet:2.1-sdk _**no contiene NodeJs preinstalado**. Si requieres NodeJs para la construcción de tu imagen (p. ej. usas NPM) entonces vas a tener un problema.
  
**La solución es sencilla a priori: basta con agregar nodejs, pero aquí es donde se rompe la abstracción**. Como ejemplo te pongo la primera aproximación que se realizó:

<pre class="EnlighterJSRAW" data-enlighter-language="null">FROM microsoft/dotnet:2.1-aspnetcore-runtime AS base
WORKDIR /app
EXPOSE 80
FROM microsoft/dotnet:2.1-sdk AS sdk-with-node
ENV NODE_VERSION 8.11.1
ENV NODE_DOWNLOAD_SHA 0e20787e2eda4cc31336d8327556ebc7417e8ee0a6ba0de96a09b0ec2b841f60
RUN curl -SL "https://nodejs.org/dist/v${NODE_VERSION}/node-v${NODE_VERSION}-linux-x64.tar.gz" --output nodejs.tar.gz \
    && echo "$NODE_DOWNLOAD_SHA nodejs.tar.gz" | sha256sum -c - \
    && tar -xzf "nodejs.tar.gz" -C /usr/local --strip-components=1 \
    && rm nodejs.tar.gz \
    && ln -s /usr/local/bin/node /usr/local/bin/nodejs
FROM sdk-with-node as build
WORKDIR /src
COPY . .
WORKDIR /src/src/Web/WebSPA
RUN dotnet restore -nowarn:msb3202,nu1503
RUN dotnet build --no-restore -c Release -o /app
FROM build AS publish
RUN dotnet publish --no-restore -c Release -o /app
FROM base AS final
WORKDIR /app
COPY --from=publish /app .
ENTRYPOINT ["dotnet", "WebSPA.dll"]</pre>

Básicamente se agrega un _stage_ nuevo, que genera la imagen intermedia &#8220;_sdk-with-node_&#8220;: es donde se usa curl para descargar el instalador de nodejs y se instala en la imagen.
  
El problema? En Linux ninguno, pero si construyes esa imagen en Windows, recibirás el _error de que sha256sum no existe_ (bueno o _curl_, depende de la versión de imagen de microsoft/dotnet de imagen use). El problema real es, como ya debes deducir, que **este _stage_ es solo para Linux**, esa sentencia RUN no funcionará ejecutándose en una imagen Windows (el primer error que da es ese, pero luego vienen más).
  
La primera aproximación fue crear dos scripts, uno llamado &#8220;install-node.sh&#8221; para Linux y otro llamado &#8220;install-node.cmd&#8221; para Windows. Se copiaban ambos scripts en el _stage sdk-with-node_ y mediante un parámetro de build (ARG) se especificaba cual ejecutar:

<pre class="EnlighterJSRAW" data-enlighter-language="raw">ARG NODE_SCRIPT=/docker-scripts/linux/install-node.sh
FROM microsoft/dotnet:2.1-aspnetcore-runtime AS base
WORKDIR /app
EXPOSE 80
FROM microsoft/dotnet:2.1-sdk AS sdk-with-node
ARG NODE_SCRIPT
WORKDIR /docker-scripts
COPY docker-scripts/ .
RUN ${NODE_SCRIPT}</pre>

En este caso el Dockerfile asume por defecto que el fichero script para instalar node se llama _/docker-scripts/linux/install-node.sh_ pero le puedes pasar otro valor usando _docker build_ o compose:

<pre class="EnlighterJSRAW" data-enlighter-language="json">webspa:
  image: eshop/webspa-win:${TAG:-latest}
  build:
    context: .
    dockerfile: src/Web/WebSPA/Dockerfile
    args:
      NODE_SCRIPT: C:\docker-scripts\win\install-node.cmd</pre>

¿El problema de esa aproximación? **Pues que l[a sintaxis para referirte a las variables de _build_ debes hacerla usando la sintaxis de variables de entorno del _shell_][4] que use la imagen que estás construyendo**. Vamos, que en Linux debes usar ${NODE\_SCRIPT} y en windows %NODE\_SCRIPT%. Si p. ej. usamos la sintaxis ${NODE_SCRIPT} en Windows nos da un error:
  
[<img class="alignnone size-medium wp-image-2108" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/error-windows-300x160.png" alt="" width="300" height="160" />][5]
  
En mi opinión Docker debería proporcionar un mecanismo para sustutír el valor de ARG y mandarlo al shell subyacente, ya que (a diferencia de ENV) ese valor es conocido por el _daemon_ de Docker.
  
Una posible solución sería usar Powershell como SHELL de la imagen en Windows. En Powershell usamos la misma sintaxis que sh para acceder a variables (es decir $variable o ${variable}) y eso nos debería solucionar el problema. Usar powershell como SHELL de la imagen no es nada complicado:

<pre class="EnlighterJSRAW" data-enlighter-language="raw">SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]</pre>

Pero claro... eso lo metes en el Dockerfile y es solo para windows, lo que de nuevo nos rompe lo de tener un solo Dockerfile. Eso sin contar de que Powershell debe existir en la imagen (y en las basadas en nanoserver como, precisamente microsoft/dotnet:2.1-sdk no existe). Por supuesto si la imagen base multi-arch usa Powershell como shell eso lo tendrías ya superado.
  
**La solución implementada**
  
**No he encontrado una solución genérica** a esa personalización. Así que para este caso en concreto la solución ha sido partir la generación de código en dos imágenes. Es decir, **en lugar de intentar instalar nodejs en la imagen del SDK de netcore, usar la imagen de node para las tareas de node y la del SDK de netcore para las tareas de dotnet** y luego combinar los resultados:

<pre class="EnlighterJSRAW" data-enlighter-language="raw">ARG NODE_IMAGE=node:8.11
FROM microsoft/dotnet:2.1-aspnetcore-runtime AS base
WORKDIR /app
EXPOSE 80
FROM microsoft/dotnet:2.1-sdk as dotnet-build
WORKDIR /src
FROM ${NODE_IMAGE} as node-build
WORKDIR /src/src/Web/WebSPA
COPY src/web/WebSPA .
RUN npm install
RUN npm run build:prod
FROM dotnet-build as publish
WORKDIR /src/src/Web/WebSPA/wwwroot
COPY --from=node-build /src/src/Web/WebSPA/wwwroot .
WORKDIR /src
COPY . .
WORKDIR /src/src/Web/WebSPA
RUN dotnet publish -c Release -o /app
FROM base AS final
WORKDIR /app
COPY --from=publish /app .
ENTRYPOINT ["dotnet", "WebSPA.dll"]
</pre>

Para empezar, dado que **la imagen oficial de node no soporta Windows**, uso un argumento de build para poderle pasar una [imagen específica para el caso de Windows][6]. Luego los _stages_ interesantes son los siguientes:

  1. _dotnet-build_: Realiza el _dotnet restore_. O más bien lo realizaría. En este caso no lo hacemos porque tenemos dependencias entre proyectos y se deben copiar todos ellos &#8220;a mano&#8221;. Eso es pesado y además debes acordarte de añadir cualquier _csproj__ _que referencies. Pero **sí, deberías realizar un dotnet restore** aquí, para aprovechar mejor la cache de Docker.
  2. _node-build_: Este es un _stage_ independiente que **usa la imagen de nodejs** y realiza las tareas propias de nodejs. En este caso son un &#8220;npm install&#8221; y un &#8220;npm run:prod&#8221; que lanza las tareas de webpack. Esas tareas crean todos los assets y los dejan en wwwroot.
  3. _publish_: Parte del stage _dotnet-build_ (por lo tanto tendríamos el _restore_ hecho). Ahí copiamos el wwwroot generado en el stage anterior, copiamos el resto del código y hacemos el _dotnet publish_

**Es un cambio de estrategia que en este caso ha funcionado**. Eso sí, en eShopOnContainers hay 3 imágenes que usan nodejs (la web spa, identity.api y la web MVC) y cada una de ellos lo usa de forma distinta **y por lo tanto la adaptación del Dockerfile debe ser distinta en cada caso** (mientras que si se hubiera podido crear una imagen combinada con netcore sdk + nodejs entonces esa aproximación sirve para todas).
  
**Nota final importante:** Por supuesto que una opción que se podría haber hecho es crear una imagen multi-arch que fuese exactamente eso: la combinación de netcore sdk + nodejs y que heredar de esa. El problema es que, entonces, corre de tu parte el ir actualizando esa imagen base, pero **es otra opción a tener MUY en cuenta. **
  
En fin... Supongo que nada es perfecto 🙂

 [1]: https://github.com/moby/moby/pull/31352
 [2]: https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/version-compatibility
 [3]: https://github.com/dotnet-architecture/eShopOnContainers/
 [4]: https://github.com/moby/moby/issues/37268
 [5]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/06/error-windows.png
 [6]: https://hub.docker.com/r/stefanscherer/node-windows/