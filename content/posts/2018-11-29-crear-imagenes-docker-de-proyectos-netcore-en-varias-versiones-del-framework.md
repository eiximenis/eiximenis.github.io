---
title: Crear imágenes Docker de proyectos netcore en varias versiones del framework

author: eiximenis

date: 2018-11-29T14:33:33+00:00
geeks_url: /?p=2226
geeks_ms_views:
  - 1073
categories:
  - docker
  - netcore

---
Imagina que estás probando alguna versión _release_ de netcore (pongamos la 2.2-preview3) y quieres generar imágenes Docker de tu proyecto para esa imagen. Pero a la vez quieres también crear las imágenes usando la última versión estable (pongamos la 2.1).
  
**Asumiendo que el código fuente es compatible**, ¿como puedes gestionar eso sin morir en el intento?
  
<!--more-->


  
Pues la verdad es que la combinación de _Directory.Build.props_, docker-compose y argumentos de build de Docker lo hacen ridículamente sencillo. ¡Vamos a verlo!
  
**Directory.Build.props**
  
El principal problema para mantener más de una versión a la vez es que las versiones nuget de los paquetes son distintas. Así cuando compile la versión de 2.1 voy a querer usar los paquetes de 2.1 (p. ej. _Microsoft.AspNetCore.App_ en 2.1.6 o bien _Microsoft.EntityFrameworkCore_ en 2.1.4) pero si estoy compilando para 2.2-preview3 voy a querer ambos paquetes en la 2.2.0-preview3-35497.
  
Tener dos csprojs no es una opción pero por suerte Directory.Build.props acude a nuestro rescate.
  
Este fichero es una maravilla de la compilación de msbuild y te permite **definir variables y acciones de msbuild que automáticamente se integran en el proceso de construcción**. La mera existencia de este fichero basta para que sea usado y afecta a todos los proyectos situados en el mismo directorio o cualquier directorio hijo. Esto te permite controlar n csprojs con un solo Directory.Build.props. Si quieres ver un ejemplo de uso real, mira como lo usamos en [Beatpulse][1].
  
Os pongo un ejemplo real: quiero compilar el  mismo proyecto tanto para 2.1, como para 2.2-preview3 y también para 2.2 GA (usando las imágenes nightly). Lo primero que hago es crearme un fichero Directory.build.props tal y como sigue:

<pre class="EnlighterJSRAW" data-enlighter-language="xml">&lt;Project&gt;
  &lt;PropertyGroup&gt;
    &lt;NetCoreToTarget&gt;&lt;/NetCoreToTarget&gt;
  &lt;/PropertyGroup&gt;
  &lt;Choose&gt;
    &lt;When Condition=" '$(NetCoreVersion)' == 'netcore22' "&gt;
      &lt;PropertyGroup&gt;
        &lt;NetCoreToTarget&gt;net22&lt;/NetCoreToTarget&gt;
      &lt;/PropertyGroup&gt;
    &lt;/When&gt;
    &lt;When Condition=" '$(NetCoreVersion)' == 'netcore22-preview3' "&gt;
      &lt;PropertyGroup&gt;
        &lt;NetCoreToTarget&gt;net22-preview3&lt;/NetCoreToTarget&gt;
      &lt;/PropertyGroup&gt;
    &lt;/When&gt;
    &lt;When Condition=" '$(NetCoreVersion)' == 'netcore21' Or '$(NetCoreVersion)' == '' "&gt;
      &lt;PropertyGroup&gt;
        &lt;NetCoreToTarget&gt;net21&lt;/NetCoreToTarget&gt;
      &lt;/PropertyGroup&gt;
     &lt;/When&gt;
  &lt;/Choose&gt;
  &lt;Import Project="build/dependencies.props" /&gt;
&lt;/Project&gt;</pre>

Se trata de un fichero muy sencillo que hace lo siguiente:

  1. En base al valor de un argumento msbuild llamado _NetCoreVersion_ crea una variable msbuild llamada _NetCoreTarget_ y lo mapea de forma que: 
      1. Si _NetCoreVersion_ vale netcore22, entonces _NetCoreTarget_ vale _net22_
      2. Si _NetCoreVersion_ vale netcore22-preview3 entonces _NetCoreTarget_ vale _net22-preview3_
      3. Si _NetCoreVersion_ vale netcore21 o no tiene valor, entonces _NetCoreTarget_ vale _net21_
  2. Finalmente importa el fichero &#8220;build/dependencies.props&#8221;

El fichero importado es el que me define la versiones de los paquetes a usar, en base al valor de _NetCoreTarget_ que hemos establecido antes:

<pre class="EnlighterJSRAW" data-enlighter-language="xml">&lt;Project&gt;
  &lt;PropertyGroup Label="SDK Versions"&gt;
    &lt;NetStandardTargetVersion&gt;netstandard2.0&lt;/NetStandardTargetVersion&gt;
  &lt;/PropertyGroup&gt;
  &lt;PropertyGroup Label="Global csproj settings"&gt;
    &lt;LangVersion&gt;latest&lt;/LangVersion&gt;
    &lt;DockerDefaultTargetOS&gt;Linux&lt;/DockerDefaultTargetOS&gt;
    &lt;AspNetCoreHostingModel&gt;inprocess&lt;/AspNetCoreHostingModel&gt;
  &lt;/PropertyGroup&gt;
  &lt;Choose&gt;
    &lt;When Condition=" '$(NetCoreToTarget)' == 'net22-preview3' "&gt;
      &lt;PropertyGroup Label="Netcore Versions"&gt;
          &lt;NetCoreTargetVersion&gt;netcoreapp2.2&lt;/NetCoreTargetVersion&gt;
          &lt;MicrosoftEntityFrameworkCore&gt;2.2.0-preview3-35497&lt;/MicrosoftEntityFrameworkCore&gt;
          &lt;MicrosoftEntityFrameworkCoreSqlServer&gt;2.2.0-preview3-35497&lt;/MicrosoftEntityFrameworkCoreSqlServer&gt;
      &lt;/PropertyGroup&gt;
    &lt;/When&gt;
    &lt;When Condition=" '$(NetCoreToTarget)' == 'net22' "&gt;
      &lt;PropertyGroup Label="Netcore Versions"&gt;
          &lt;NetCoreTargetVersion&gt;netcoreapp2.2&lt;/NetCoreTargetVersion&gt;
          &lt;MicrosoftEntityFrameworkCore&gt;2.2.0&lt;/MicrosoftEntityFrameworkCore&gt;
          &lt;MicrosoftEntityFrameworkCoreSqlServer&gt;2.2.0&lt;/MicrosoftEntityFrameworkCoreSqlServer&gt;
      &lt;/PropertyGroup&gt;
    &lt;/When&gt;
    &lt;When Condition=" '$(NetCoreToTarget)' == 'net21' "&gt;
      &lt;PropertyGroup Label="Netcore Versions"&gt;
          &lt;NetCoreTargetVersion&gt;netcoreapp2.1&lt;/NetCoreTargetVersion&gt;
          &lt;MicrosoftEntityFrameworkCore&gt;2.1.2&lt;/MicrosoftEntityFrameworkCore&gt;
          &lt;MicrosoftEntityFrameworkCoreSqlServer&gt;2.1.2&lt;/MicrosoftEntityFrameworkCoreSqlServer&gt;
      &lt;/PropertyGroup&gt;
     &lt;/When&gt;
  &lt;/Choose&gt;
&lt;/Project&gt;</pre>

Con eso defino variables de msbuild addicionales, llamadas:

  * NetCoreTargetVersion
  * MicrosoftEntityFrameworkCore
  * MicrosoftEntityFrameworkCoreSqlServer

La idea es que definiría una variable por cada paquete nuget que deseo usar y dicha variable tiene la versión del paquete u otro setting necesario.
  
Ahora con esto no nos basta claro. Si usamos esa técnica **no podemos tener las versiones de los paquetes nuget en los csprojs. **Pero ese es un cambio trivial. Abre el csproj, busca los <PackageReferences> y modifica la versión para que use la variable msbuild del paquete correspondiente:

<pre class="EnlighterJSRAW" data-enlighter-language="xml">&lt;ItemGroup&gt;
  &lt;PackageReference Include="Microsoft.AspNetCore.App" /&gt;
  &lt;PackageReference Include="Microsoft.AspNetCore.Mvc.Versioning" Version="$(MicrosoftAspNetCoreMvcVersioning)" /&gt;
  &lt;PackageReference Include="Microsoft.EntityFrameworkCore" Version="$(MicrosoftEntityFrameworkCore)" /&gt;
  &lt;PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="$(MicrosoftEntityFrameworkCoreSqlServer)" /&gt;
&lt;/ItemGroup&gt;</pre>

Vale. Observa que **de golpe y porrazo has centralizado todas las versiones de paquetes nuget en un solo lugar: el fichero dependencies.props**. Si quieres actualizar paquetes no necesitas modificar todos los csprojs: te basta con tocar en un solo lugar. También puedes ver que _Microsoft.AspNetCore.App_ no tiene versión. De este modo se usa la versión exacta del SDK que se esté usando en cada momento.
  
No tienes porque preocuparte por Visual Studio: VS entiende Directory.Build.props y vas a ver tus paquetes nuget resueltos a la versión correcta. Lo jodido es que VS no pasa parámetros de msbuild, así que verás &#8220;la versión por defecto&#8221; (en este caso netcore21).
  
**Nota:** Si te estás preguntando porque mapeo el parámetro NetCoreVersion a NetCoreToTarget y no uso NetCoreVersion directamente en dependencies.props, la respuesta es que en este caso concreto lo que hago no es necesario. Es decir podría limitarme a importar el fichero build/dependencies.props y cuando compile pasar el parámetro msbuild _NetCoreToTarget_ y todo funcionaría igual. Pero yo siempre prefiero hacer ese mapeo porque me da flexibilidad: si en un futuro quiero hacer **más cosas** cuando haya un cambio de versión puedo agregarlas en el Directory.Build.props y me queda todo más limpio. Pero vamos, es pura preferencia personal.
  
Debes editar también algunas líneas más en la sección de PropertyGroup, el valor de <TargetFramework>:

<pre class="EnlighterJSRAW" data-enlighter-language="xml">&lt;PropertyGroup&gt;
  &lt;TargetFramework&gt;$(NetCoreTargetVersion)&lt;/TargetFramework&gt;
  &lt;DockerDefaultTargetOS&gt;$(DockerDefaultTargetOS)&lt;/DockerDefaultTargetOS&gt;
&lt;/PropertyGroup&gt;</pre>

¡Bien! Ya tenemos un conjunto de csprojs que se compilan a netcore21 o netcore2.2-preview3 o netcore2.2 según un parámetro de msbuild.
  
**Docker compose y argumentos de build**
  
Bien, ahora toca la segunda parte que es generar imágenes de docker distintas para cada entorno. Aquí nos ayudará tanto docker-compose como usar argumentos de build de Docker.
  
En el Dockerfile le vamos a pasar cuatro argumentos de build:

  1. La imagen
  2. El tag de runtime
  3. El tag del sdk
  4. El valor del parámetro msbuild _NetCoreVersion_

Aquí tienes una posible versión del Dockerfile (por supuesto deberás adaptarla):

<pre class="EnlighterJSRAW" data-enlighter-language="null">ARG sdkTag=2.1-sdk
ARG runtimeTag=2.1-aspnetcore-runtime
ARG coreversion=netcore21
ARG image=microsoft/dotnet
FROM ${image}:${runtimeTag} AS base
WORKDIR /app
FROM ${image}:${sdkTag} AS build
ARG coreversion
WORKDIR /src
COPY ./Directory.Build.props .
COPY ./build ./build
WORKDIR /src/project
COPY ./MyProjectFolder .
RUN dotnet build "MyProject.csproj"  -p:NetCoreVersion=${coreversion} -c Release -o /app
FROM build AS publish
ARG coreversion
RUN dotnet publish "MyProject.csproj"  -p:NetCoreVersion=${coreversion} -c Release -o /app
FROM base AS final
WORKDIR /app
COPY --from=publish /app .
ENTRYPOINT ["dotnet", "MyProject.dll"]</pre>

**Nota:** Observa que copio, obviamente, el Directory.build.props en la imagen de compilación. **Eso me obliga a que el contexto de build sea, como mínimo, el que contenga dicho fichero**.
  
Definimos los 4 argumentos de build (sdkTag, runtimeTag, coreversion y image) y los usamos donde los necesitamos: tanto en los FROM (para poder usar distintas imágenes base y tags) y también cuando llamamos a dotnet para pasarle el parámetro msbuild.
  
Porque eso es lo que nos faltaba: ver como pasar parámetros msbuild usando dotnet.exe. Pues es muy fácil basta con añadir &#8220;-p:parametro=valor&#8221;. De este modo pasamos el valor contenido en el argumento de build _coreversion_ al parámetro msbuild _NetCoreVersion_.
  
Ya solo nos queda una cosa más. Tener un fichero compose por cada versión (uno para 2.1, otro para 2.2-preview3 y otro para 2.2) que establezca esos valores de build (bueno, el de 2.1 te lo puedes ahorrar ya que los argumentos de build tienen los valores por defecto de 2.1 en este ejemplo).
  
Por ejemplo, el de 2.2 podría ser un docker-compose.net22.yml tal como ese:

<pre class="EnlighterJSRAW" data-enlighter-language="json">version: '3.4'
services:
  myproject:
    build:
      args:
        image: microsoft/dotnet-nightly
        sdkTag: 2.2-sdk
        runtimeTag: 2.2-aspnetcore-runtime
        coreversion: netcore22
</pre>

Si ahora quieres generar las imágenes docker para 2.2 te basta con un &#8220;_docker-compose -f docker-compose.yml -f docker-compose.override.yml -f **docker-compose.net22.yml**  build_&#8220;. Y solo modificando la parte en negrita (pasando otro fichero compose) generarías las imágenes para las otras versiones.
  
Por supuesto, lo habitual es que tu quieras generar _tags_ distintos para cada versión, así que en tu fichero compose principal tendrías algo como:

<pre class="EnlighterJSRAW" data-enlighter-language="json">myproject:
  image: myimage:${TAG:-latest}</pre>

Sencillo, eficaz y establecer el valor de la variable de entorno TAG al valor que quieras antes de ejecutar el docker-compose build.
  
Espero que te haya sido útil!

 [1]: https://github.com/xabaril/beatpulse