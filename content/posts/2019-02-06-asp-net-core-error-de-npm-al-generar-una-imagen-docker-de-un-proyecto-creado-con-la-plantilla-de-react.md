---
title: 'ASP.NET Core: Error de npm al generar una imagen Docker de un proyecto creado con la plantilla de React'
author: eiximenis

date: 2019-02-06T18:09:40+00:00
geeks_url: /?p=2277
geeks_ms_views:
  - 652
categories:
  - asp.net core
  - docker
  - reactjs
  - visual studio

---
He visto este problema con un proyecto generado a partir de la plantilla de SPA de React, pero quizá puede aplicar a otras plantillas de SPA (como Angular).
  
<!--more-->


  
El error se puede reproducir muy fácilmente. Desde un directorio vacío puedes crear una SPA de react:

<pre class="EnlighterJSRAW" data-enlighter-language="null">dotnet new react --name testspa
dotnet new sln --name testspa
dotnet sln add testspa\testspa.csproj</pre>

Con eso tenemos el proyecto &#8220;testspa.csproj&#8221; y una solución de Visual Studio (testspa.sln) para abrirla con Visual Studio y que este nos genere el Dockerfile. Para ello abre la solución con Visual Studio y usa la opción &#8220;Add -> Docker Support&#8221; para que Visual Studio nos cree el Dockerfile:
  
[<img class="alignnone size-full wp-image-2278" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/add-docker.png" alt="Opción Add -> Docker Support" width="812" height="594" />][1]
  
Eso nos creará el &#8220;Dockerfile&#8221; estándard:

<pre class="EnlighterJSRAW" data-enlighter-language="null">FROM microsoft/dotnet:2.2-aspnetcore-runtime AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443
FROM microsoft/dotnet:2.2-sdk AS build
WORKDIR /src
COPY ["testspa/testspa.csproj", "testspa/"]
RUN dotnet restore "testspa/testspa.csproj"
COPY . .
WORKDIR "/src/testspa"
RUN dotnet build "testspa.csproj" -c Release -o /app
FROM build AS publish
RUN dotnet publish "testspa.csproj" -c Release -o /app
FROM base AS final
WORKDIR /app
COPY --from=publish /app .
ENTRYPOINT ["dotnet", "testspa.dll"]</pre>

A priori parece todo correcto, y es que este esquema multistage es &#8220;generalmente correcto&#8221; para la gran mayoría de proyectos netcore:

  1. Usa la imagen del SDK para ejecturar un &#8220;dotnet restore&#8221; y un &#8220;dotnet build&#8221;
  2. Luego realiza el &#8220;dotnet publish&#8221; en un nuevo stage
  3. Finalmente copia el resultado del publish en una imagen que parte del runtime

El problema es que... **no funciona**. Lo puedes comprobar tu mismo lanzando el siguiente comando:

<pre class="EnlighterJSRAW" data-enlighter-language="null">docker build -t testspa -f testspa\Dockerfile .</pre>

Este comando debes lanzarlo desde el directorio raíz (el directorio donde está el fichero sln, no el fichero csproj). Eso es porque los Dockerfile que genera VS toman como contexto de build el directorio donde está la solución, no cada uno de los proyectos. No es algo que personalmente me guste, pero VS lo hace porque es la única manera de poder generar imágenes de proyectos que tengan referencias a otros proyectos de la solución.
  
Bueno... al cabo de un ratillo la build reventará con ese error:

<pre class="EnlighterJSRAW" data-enlighter-language="null">Step 11/17 : RUN dotnet build "testspa.csproj" -c Release -o /app
 ---&gt; Running in eccd6da4ab29
Microsoft (R) Build Engine version 15.9.20+g88f5fadfbe for .NET Core
Copyright (C) Microsoft Corporation. All rights reserved.
  Restore completed in 795.87 ms for /src/testspa/testspa.csproj.
The command '/bin/sh -c dotnet build "testspa.csproj" -c Release -o /app' returned a non-zero code: 137</pre>

¿La razón del fallo? Pues que **la imagen del SDK de netcore no contiene nodejs**. ¿Y porque se necesita nodejs? Pues porque quien generó la plantilla de proyecto pensó que era buena idea que al publicar el proyecto (con &#8220;dotnet publish&#8221;) se ejecutasen las sentencias npm necesarias para generar los _bundles_ de javascript. Y por eso tenemos lo siguiente **en el fichero csproj**:

<pre class="EnlighterJSRAW" data-enlighter-language="xml">&lt;Target Name="PublishRunWebpack" AfterTargets="ComputeFilesToPublish"&gt;
  &lt;!-- As part of publishing, ensure the JS resources are freshly built in production mode --&gt;
  &lt;Exec WorkingDirectory="$(SpaRoot)" Command="npm install" /&gt;
  &lt;Exec WorkingDirectory="$(SpaRoot)" Command="npm run build" /&gt;
  &lt;!-- Include the newly-built files in the publish output --&gt;
  &lt;ItemGroup&gt;
    &lt;DistFiles Include="$(SpaRoot)build\**" /&gt;
    &lt;ResolvedFileToPublish Include="@(DistFiles-&gt;'%(FullPath)')" Exclude="@(ResolvedFileToPublish)"&gt;
      &lt;RelativePath&gt;%(DistFiles.Identity)&lt;/RelativePath&gt;
      &lt;CopyToPublishDirectory&gt;PreserveNewest&lt;/CopyToPublishDirectory&gt;
    &lt;/ResolvedFileToPublish&gt;
  &lt;/ItemGroup&gt;
&lt;/Target&gt;</pre>

Esta tarea se lanza en el proceso de publicación (no de build) **y observa como se ejecuta el &#8220;npm install&#8221; y el &#8220;npm run build&#8221;** que son las tareas necesarias para construir los _bundles_ de JavaScript.
  
Por lo tanto **en tu máquina un &#8220;dotnet publish&#8221; funcionará**, ya que tendrás node instalado, pero **cuando eso se ejecuta en la imagen Docker del SDK, eso no funciona** porque nodejs no existe.
  
¿Como podemos solucionarlo? Por suerte la solución es bastante fácil. Primero **editamos el fichero csproj para que esta tarea se ejecute solo si una determinada variable de entorno no existe_:_**

<pre class="EnlighterJSRAW" data-enlighter-language="xml">&lt;Target Name="PublishRunWebpack" AfterTargets="ComputeFilesToPublish" Condition=" '$(BuildingDocker)' == '' "&gt;</pre>

Observa el &#8220;_Condition_&#8220;, para que se ejecute solo si la variable _BuildingDocker_ tiene algún valor (da igual cual).  De este modo en nuestra máquina local todo seguirá funcionando igual. Vayamos ahora a por el Dockerfile. Ojo con la sintaxis de _Condition_ que msbuild es muy puñetero y deben respetarse los espacios en blanco.
  
El Dockerfile hay que modificarlo bastante. Por un lado debemos usar la imagen del SDK de netcore para compilar el proyecto, pero también necesitamos la imagen de node para las tareas de npm. Finalmente el resultado de ambos stages los combinaremos en una imagen con el runtime de netcore:

<pre class="EnlighterJSRAW" data-enlighter-language="null">FROM microsoft/dotnet:2.2-aspnetcore-runtime AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443
FROM node:10-alpine as build-node
WORKDIR /ClientApp
COPY testspa/ClientApp/package.json .
COPY testspa/ClientApp/package-lock.json .
RUN npm install
COPY testspa/ClientApp/ .
RUN npm run build
FROM microsoft/dotnet:2.2-sdk AS build
ENV BuildingDocker true
WORKDIR /src
COPY ["testspa/testspa.csproj", "testspa/"]
RUN dotnet restore "testspa/testspa.csproj"
COPY . .
WORKDIR "/src/testspa"
RUN dotnet build "testspa.csproj" -c Release -o /app
FROM build AS publish
RUN dotnet publish "testspa.csproj" -c Release -o /app
FROM base AS final
WORKDIR /app
COPY --from=publish /app .
COPY --from=build-node /ClientApp/build ./ClientApp/build
ENTRYPOINT ["dotnet", "testspa.dll"]</pre>

En el stage _build-node_ usamos la imagen de Node para ejecutar las tareas &#8220;npm install&#8221; y &#8220;npm run build&#8221; que construyen los _bundles _y los dejan en ClientApp/build.
  
Luego en el stage _build_, usamos el SDK de dotnet para hacer el &#8220;dotnet build&#8221; y el &#8220;dotnet publish&#8221; pero **observa como usamos ENV para definir la variable de entorno &#8220;BuildingDocker&#8221; con el valor de true**. Gracias a tener definida esa variable de entorno en el contenedor, la tarea _PublishRunWebpack_ del csproj **no se ejecuta**, por lo que no recibiremos error alguno.
  
Finalmente en el último _stage_ combinamos la parte generada por nodejs (ClientApp/build) y la parte generada por el _dotnet publish_ (todo lo demás) para tener toda la aplicación junta.
  
**Conclusiones**
  
Nada nuevo, pero recuerda **que los Dockerfiles que crea Visual Studio son poco más que una plantilla** que funciona en un gran número de casos pero hay varias casuísticas en las que los ficheros generados no funcionan correctamente. Otra cosa que debes recordar es que en general, dado que se intenta que las imágenes sean lo más pequeñas posibles, se dedican a hacer una sola cosa (tómatelo como un SRP aplicado a las imágenes de Docker): por eso la imagen del SDK de ASP.NET Core no tiene nodejs. En casos de qué combines varias herramientas para la construcción de un proyecto, en Docker lo que se hace es combinar N imágenes en N _stages_ y al final agregar los resultados.
  
¡Un saludo!

 [1]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/02/add-docker.png