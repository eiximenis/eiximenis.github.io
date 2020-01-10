---
title: Asp.Net Core – solucionar error al publicar en docker (assembly not found)

author: eiximenis

date: 2018-01-24T08:34:23+00:00
geeks_url: /?p=1990
geeks_ms_views:
  - 1007
categories:
  - asp.net core
  - docker

---
Buenas! Imagina que tienes una aplicación hecha en Asp.Net Core y que referencia al [metapaquete Microsoft.AspNetCore.All][1]. También tienes un Dockerfile y un fichero compose para generar la imagen [usando una ][2]_[multi-stage build][2]._
  
La imagen se genera sin problemas pero al ejecutarla recibes un error y no arranca.
  
<!--more-->


  
El _Dockerfile_ puede ser similar al que sigue (nota: es el _Dockerfile_ generado por defecto por VS2017 15.5.4 al agregar soporte para Docker):

<pre class="EnlighterJSRAW" data-enlighter-language="null">FROM microsoft/aspnetcore:2.0 AS base
WORKDIR /app
EXPOSE 80
FROM microsoft/aspnetcore-build:2.0 AS build
WORKDIR /src
COPY *.sln ./
COPY CrashOnStartup/CrashOnStartup.csproj CrashOnStartup/
RUN dotnet restore
COPY . .
WORKDIR /src/CrashOnStartup
RUN dotnet build -c Release -o /app
FROM build AS publish
RUN dotnet publish -c Release -o /app
FROM base AS final
WORKDIR /app
COPY --from=publish /app .
ENTRYPOINT ["dotnet", "CrashOnStartup.dll"]</pre>

No tiene mucha complicación. Es un fichero _multi-stage_ y realiza lo siguiente:

  1. Parte de la imagen base [_microsoft/aspnetcore:2.0_][3].
  2. Usa la imagen [_microsoft/aspnetcore-build:2.0_][4] como imagen de build
  3. En esta imagen realiza el _dotnet restore, build y publish_
  4. Recoge el resultado del _publish_ y lo copia en la imagen base para generar el contenedor final.

El error **que puedes obtener **es el siguiente:

<pre class="EnlighterJSRAW" data-enlighter-language="generic">Error:
  An assembly specified in the application dependencies manifest (CrashOnStartup.deps.json) was not found:
    package: 'Microsoft.AspNetCore.Mvc.Abstractions', version: '2.0.2'
    path: 'lib/netstandard2.0/Microsoft.AspNetCore.Mvc.Abstractions.dll'
  This assembly was expected to be in the local runtime store as the application was published using the following target manifest files:
    aspnetcore-store-2.0.5.xml</pre>

¿Qué está ocurriendo? Pues bien lo que ocurre es que cuando se ha publicado tu aplicación **no se están publicando las dependencias correspondientes de ASP.Net Core**. Recuerda que, por defecto, en ASP.Net Core 2 al usar _dotnet publish_ no se publican los ensamblados de ASP.Net Core 2, solo los de tu aplicación y ensamblados adicionales. La razón es **que se presupone que todos los ensamblados de ASP.Net Core 2 existen en cualquier máquina que tenga el _runtime_ instalado (en lo que llamamos el _runtime store_).**
  
La primera opción que te puede venir a la mente es que la imagen _microsoft/aspnetcore:2.0_ no tenga la _runtime store_, por lo que debamos publicar no solo nuestros ensamblados si no todos los del runtime al publicar nuestra aplicación. **Pero no, no es eso**. **Dicha imagen SÍ tiene la _runtime store_. **¿Entonces? **¿Qué está ocurriendo?**
  
Pues algo muy sencillo: **la versión que tienes de la imagen microsoft/aspnet-core NO se corresponde con la versión del metapaquete que estás usando.** Es decir si referencias al metapaquete Microsoft.AspNetCore.All en su versión 2.0.5 debes usar la imagen que tiene el runtime en su versión 2.0.5.
  
La razón por la que puedes tener la imagen desactualizada es que el tag _2.0_ que se usa es un tag &#8220;flotante&#8221;. Eso significa que es un tag que se mueve a medida que se van añadiendo nuevas versiones de dicha imagen. P. ej. **en el momento de redactar este post el tag 2.0 contiene la imagen con la versión 2.0.5** **(en ambas imágenes)**. Pero cuando salga la 2.0.6 el tag 2.0 se promocionará para que apunte a dicha imagen. La idea es que el tag 2.0 apunta &#8220;a la última imágen 2.0 en todo momento&#8221;.
  
Pero el tema está en que **docker no refresca automáticamente las imágenes**. Es decir imagina que te descargaste la imagen _microsoft/aspnetcore:2.0_ en tu sistema pero cuando el tag apuntaba a la imagen 2.0.4. Posteriormente sale la versión 2.0.5 y Microsoft actualiza el tag 2.0 para que apunte a la 2.0.5. Luego usas VS (que tienes debidamente actualizado) para crear un proyecto nuevo y este proyecto usa el metapaquete en su versión 2.0.5. Y con eso creas la imagen de Docker. **Y aquí obtendrás el error, porque TU imagen microsoft/aspnetcore:2.0 (que ya tenías en local) es la versión 2.0.4**. A pesar de que el tag se haya promocionado, Docker no refresca la imagen porque, por lo que el respecta, la versión 2.0 ya la tienes en tu máquina.
  
La solución es forzar a Docker que te refresque la imagen usando:

<pre class="EnlighterJSRAW" data-enlighter-language="shell">docker pull microsoft/aspnetcore:2.0</pre>

Luego reconstruyes la imagen **y ya te debería funcionar todo**.
  
Por lo tanto **la moraleja es: cuidado con los tags flotantes, ¡que debes &#8220;actualizarlos&#8221; a mano!**

 [1]: https://geeks.ms/etomas/2017/12/26/el-metapaquete-microsoft-aspnetcore-all/
 [2]: https://geeks.ms/etomas/2017/11/28/docker-multi-stage-builds-o-como-compilar-casi-cualquier-cosa-sin-tener-que-instalar-nada/
 [3]: https://hub.docker.com/r/microsoft/aspnetcore/
 [4]: https://hub.docker.com/r/microsoft/aspnetcore-build/