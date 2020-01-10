---
title: Generar imágenes Docker de proyectos SPA de netcore

author: eiximenis

date: 2018-10-25T14:04:52+00:00
geeks_url: /?p=2194
geeks_ms_views:
  - 1155
categories:
  - asp.net core
  - docker

---
¡Buenas!
  
Cuando creas un proyecto SPA de netcore, ya sea mediante VS o bien usando _dotnet new_ y alguna plantilla SPA como react (_dotnet new react_), se genera una estructura parecida a la siguiente:
  
[<img class="alignnone size-full wp-image-2195" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/10/spa-estructura.png" alt="Estructura ficheros proyecto SPA" width="249" height="293" />][1]
  
La carpeta &#8220;ClientApp&#8221; contiene todo el código de cliente (javascript, CSS y demás) mientras que el resto es el código netcore que se limita a &#8220;lanzar&#8221; la SPA.
  
<!--more-->


  
En Startup.cs se usa el middleware _UseSpa_ que es el encargado de servir el fichero &#8220;index.html&#8221; y adicionalmente permite usar servidores de desarrollo. P. ej. en el caso de una aplicación react el código es:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">app.UseSpa(spa =&gt;
{
    spa.Options.SourcePath = "ClientApp";
    if (env.IsDevelopment())
    {
        spa.UseReactDevelopmentServer(npmScript: "start");
    }
});</pre>

Básicamente le indicamos que busque el fichero &#8220;index.html&#8221; del directorio &#8220;ClientApp&#8221; y, solo si estamos en desarrollo, use el servidor de desarrollo de React.
  
Este servidor de desarrollo requiere node, **por lo que, en desarrollo necesitamos tener NodeJs instalado**. No es ningún problema, ya que si desarrollas una SPA con React, tendrás NodeJs instalado con toda probabilidad.
  
Ahora bien, **en producción no hay necesidad alguna de usar Node**: en este caso es imperativo que **los _bundles_ se hayan generado y estén en ClientApp/build**. Si estamos ejecutando usando _dotnet myspa.dll_ debemos haber generado nosotros los bundles manualmente: no lo hace VS por nosotros.
  
**Pero si publicamos el proyecto** (usando _dotnet publish_) **entonces sí que se nos generan los _bundles_. **Eso es debido a las siguientes líneas del csproj:

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

Eso define una tarea propia, llamada &#8220;PublishRunWebpack&#8221;· que se ejecutará al publicar la aplicación (no al compilar, al publicar). Esta tarea hace tres cosas:

  1. Ejecuta npm install
  2. Ejecuta npm run build
  3. Añade los ficheros generados en ClientApp\build\ al resultado de publicación para que se copien al paquete de publicación (si no estos ficheros serían ignorados).

**Esto funciona perfectamente pero requiere que la máquina que lanza el dotnet publish tenga node instalado**. En determinados entornos no hay problemas (p. ej. en el caso de generar una build con VSTS el agente tiene el sdk de netcore y node por lo que todo funcionará), **pero en Docker tendremos fricciones**.
  
La razón es que la imagen del SDK de netcore, que usamos para compilar, no tiene nodejs preinstalado (antes de netcore 2.x lo tenía). Es una buena decisión: queremos imágenes pequeñas y nodejs no es necesario para la gran mayoría de proyectos de netcore. Una opción que puedes intentar es **instalar nodejs en la imagen del sdk de netcore**. De hecho [tengo un gist que muestra como hacerlo][2], pero la realidad **es que no es necesario**. Es mucho mejor y más fácil usar una multi-stage build con dos pasos de build: en el primero usamos nodejs para crear los bundles, en el segundo netcore sdk para compilar y finalmente lo combinamos todo en un stage final de ejecución:

<pre class="EnlighterJSRAW" data-enlighter-language="null">FROM node:10-alpine as build-node
WORKDIR /ClientApp
COPY ClientApp/package.json .
COPY ClientApp/package-lock.json .
RUN npm install
COPY ClientApp/ .
RUN npm run build
FROM microsoft/dotnet:2.2.100-preview3-sdk-stretch as build-net
WORKDIR /app
COPY *.csproj .
RUN dotnet restore
COPY . .
RUN dotnet build
RUN dotnet publish -o /myweb
FROM microsoft/dotnet:2.2.0-preview3-aspnetcore-runtime
WORKDIR /web
COPY --from=build-net /ttweb .
COPY --from=build-node /ClientApp/build ./ClientApp/build
ENTRYPOINT [ "dotnet","myspa.dll" ]</pre>

Observa como en el stage &#8220;build-node&#8221; usamos npm install y npm run build para generar los bundles necesarios. Luego en el stage &#8220;build-net&#8221; usamos dotnet build y dotnet publish para publicar la web y en el stage &#8220;runtime&#8221; partimos de la imagen con el runtime de asp.net core y copiamos por un lado los bundles y por otro lado el código netcore compilado.
  
Y ¡voilá! Listos. Sencillo, ¿verdad? Pues sí, **salvo que eso tal cual como está NO FUNCIONA**.
  
¿La razón? Pues lo que he comentado antes: la máquina que hace el &#8220;dotnet publish&#8221; requiere de nodejs y la imagen microsoft/dotnet no lo incorpora. Por suerte la solución es super sencilla, basta con dos pasos:
  
Primero edita el csproj y haz que la tarea _PublishRunWebpack_ sea condicional al valor de una variable, p. ej. llamada _BuildingDocker_:

<pre class="EnlighterJSRAW" data-enlighter-language="null">&lt;Target Name="PublishRunWebpack" AfterTargets="ComputeFilesToPublish" Condition=" '$(BuildingDocker)' == '' &gt;
</pre>

Hemos añadido el atributo _Condition_. Así de esta manera sólo si _BuildindDocker _no está definida se ejecutará esta tarea. En caso contrario, esta tarea nos la saltaremos, que es justo lo que queremos en el caso de Docker.
  
Así, por supuesto el segundo paso es definir esa variable de entorno llamada _BuildingDocker_ en el stage &#8220;build-net&#8221;:

<pre class="EnlighterJSRAW" data-enlighter-language="null">FROM microsoft/dotnet:2.2.100-preview3-sdk-stretch as build-net
ENV BuildingDocker true
WORKDIR /app
COPY *.csproj .
RUN dotnet restore
COPY . .
RUN dotnet build
RUN dotnet publish -o /ttweb</pre>

**Observa el uso de ENV para definir lavariable de entorno** _**BuildingDocker** _(ojo que el valor no es relevante, solo que esté definida, así que incluso si la defines con false la tarea no se ejecutará).
  
¡Y listos! Ahora sí que ¡ya puedes construir tus imágenes Docker de las aplicaciones SPA, sin ningún problema!

 [1]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/10/spa-estructura.png
 [2]: https://gist.github.com/eiximenis/35536993081d00cd0fc9760c29637e49