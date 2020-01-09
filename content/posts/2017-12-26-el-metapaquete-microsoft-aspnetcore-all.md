---
title: El metapaquete Microsoft.AspNetCore.All
description: El metapaquete Microsoft.AspNetCore.All
author: eiximenis

date: 2017-12-26T19:47:54+00:00
geeks_url: /?p=1961
geeks_ms_views:
  - 1562
categories:
  - asp.net 5
  - asp.net core
  - netcore
  - Sin categoría

---
Todos estamos acostumbrados a usar los **paquetes de NuGet en nuestros desarrollos**. Pero a raíz de Net Core 2.0, apareció el concepto de _metapaquete_. Qué es exactamente un _metapaquete_ y por qué existen?
  
La respuesta rápida es que un _metapaquete_ de NuGet es simplemente un paquete que **no incluye ningún ensamblado, solo referencia a otros paquetes**. Es, en definitiva, un mecanismo para &#8220;agrupar&#8221; paquetes de NuGet bajo un mismo número de version.
  
<!--more-->


  
Pero de todos los  _metapaquetes_ existentes, el más curioso es sin duda el [_metapaquete de Asp.Net Core_][1], que contiene referencias **a todos los paquetes que conforman ASP.NET Core**. Todos es básicamente todos, incluyendo Razor, proveedores de autenticación externa y también Entity Framework. Eso significa que podemos empezar cualquier desarrollo con ASP.NET Core usando tan solo una sola referencia en nuestro csproj:

<pre class="EnlighterJSRAW" data-enlighter-language="xml">&lt;PackageReference Include="Microsoft.AspNetCore.All" Version="2.0.3" /&gt;</pre>

No es necesario que incluyamos la pléyade paquetes que conforman ASP.NET Core y Net Core. Este _metapaquete _los incluye todos por nosotros bajo un solo número de version. Con eso solucionamos un problema típico: **actualizar versiones de paquetes NuGet en ASP.NET Core era un infierno**, especialmente si uno lo hacía editando el csproj a mano. Saber exactamente cual era la última versión de todos los paquetes no era sencillo. Ahora, gracias al _metapaquete_ nos basta con saber cual es la última version de éste para tener todos los paquetes base de ASP.NET Core actualizados. Sencillo, rápido y eficaz.
  
**Pero... ¿eso no es un paso atrás?**
  
Una de las novedades de Net Core (y ASP.NET Core) era, precisamente, que estaba segmentado en múltiples paquetes NuGet. Ya no teníamos la dependencia al mega-ensamblado _System.Web_ y solo instalábamos lo que necesitábamos. A cambio hemos visto que eso nos generaba unos csproj relativamente grandes, con muchas entradas _PackageReference_ para incluir todos los paquetes NuGet necesarios.
  
De acuerdo, es cierto, **tener muchos _PackageReference_ hace más complejo el actualizar las versiones pero permite que al publicar mi aplicación se publiquen solo los paquetes NuGet que usemos**. Si no voy a usar Entity Framwork no quiero tener que publicar todos sus paquetes junto con mi aplicación. Esa es, precisamente, la ventaja de segmentar el framework en varios paquetes: publicar solo lo que se use. Pero... si resulta que la única referencia de mi proyecto es un _metapaquete_ que, cual anillo del poder, los incluye a todos... al publicar mi aplicación, ¿qué se publicará?
  
Está claro que **nos hace falta una pieza más** y esa pieza tiene nombre: **la _default runtime store_**. ¿Qué narices es la _default r__untime store_? Pues simplemente, que ahora al instalar el _runtime_ de netcore **se preinstalan todos los paquetes oficiales de Microsoft**. Esos paquetes están disponibles en una ubicación central (esa _default __runtime store_) **por lo que no es necesario que se publiquen junto con cualquier aplicación**. Eso hace que los binarios no sean más grandes (de hecho, serán más pequeños que el equivalente de NetCore 1.1).
  
La _default r__untime store_ se encuentra en %PROGRAMFILES%/dotnet/store:
  
[<img class="alignnone size-medium wp-image-1964" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/12/runtimestore-233x300.png" alt="runtime store" width="233" height="300" />][2]
  
Para comparar vamos a ver la diferencia entre publicar la plantilla por defecto de ASP.NET Core 1.1 y 2.0 (en ambos casos uso VS2017 con _File->New Project->ASP.NET Core Web Application ->Web Application (Model View Controller)_ con la autenticación de usuarios locales, en un caso seleccionando .NET Core 2.0 y en el otro .NET Core 1.1).
  
Esas son todas las referencias incluidas de serie en el proyecto de NetCore 1.1:

<pre class="EnlighterJSRAW" data-enlighter-language="xml">&lt;PackageReference Include="Microsoft.ApplicationInsights.AspNetCore" Version="2.0.0" /&gt;
&lt;PackageReference Include="Microsoft.AspNetCore" Version="1.1.2" /&gt;
&lt;PackageReference Include="Microsoft.AspNetCore.Authentication.Cookies" Version="1.1.2" /&gt;
&lt;PackageReference Include="Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore" Version="1.1.2" /&gt;
&lt;PackageReference Include="Microsoft.AspNetCore.Identity.EntityFrameworkCore" Version="1.1.2" /&gt;
&lt;PackageReference Include="Microsoft.AspNetCore.Mvc" Version="1.1.3" /&gt;
&lt;PackageReference Include="Microsoft.AspNetCore.StaticFiles" Version="1.1.2" /&gt;
&lt;PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="1.1.2" PrivateAssets="All" /&gt;
&lt;PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="1.1.2" /&gt;
&lt;PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer.Design" Version="1.1.2" PrivateAssets="All" /&gt;
&lt;PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="1.1.1" PrivateAssets="All" /&gt;
&lt;PackageReference Include="Microsoft.Extensions.Configuration.UserSecrets" Version="1.1.2" /&gt;
&lt;PackageReference Include="Microsoft.Extensions.Logging.Debug" Version="1.1.2" /&gt;
&lt;PackageReference Include="Microsoft.VisualStudio.Web.CodeGeneration.Design" Version="1.1.1" PrivateAssets="All" /&gt;
&lt;PackageReference Include="Microsoft.VisualStudio.Web.BrowserLink" Version="1.1.2" /&gt;</pre>

Por otro lado en el proyecto de NetCore 2.0 solo tenemos el _metapaquete_ y dos referencias adicionales (la CLI de EF y el paquete para generar código usado por EF). Ninguna de esas dos referencias adicionales se necesita en ejecución.

<pre class="EnlighterJSRAW" data-enlighter-language="null">&lt;PackageReference Include="Microsoft.AspNetCore.All" Version="2.0.3" /&gt;
&lt;PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="2.0.1" PrivateAssets="All" /&gt;
&lt;PackageReference Include="Microsoft.VisualStudio.Web.CodeGeneration.Design" Version="2.0.1" PrivateAssets="All" /&gt;
</pre>

Luego publico con _dotnet publish -c release_ ambas versiones. La carpeta de la publicación de NetCore 1.1 **contiene 98 ensamblados del sistema** (sin incluir el de la propia aplicación), mientras que la carpeta de NetCore 2.0 **no contiene ningún ensamblado**
  
[<img class="alignnone size-medium wp-image-1962" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/12/netcore1vs2-300x189.png" alt="Comparación carpetas publicación" width="300" height="189" />][3]
  
Por supuesto, la carpeta de publicación de NetCore 2.0 confía, precisamente, en que todos los ensamblados estarán disponibles... gracias a la _default __runtime store_. Por supuesto **los ensamblados contenidos en la _default runtime store_ se alinean con los del _metapaquete_ Microsoft.AspNetCore.All**_._
  
Por lo tanto: **usando el _metapaquete_ de Microsoft.AspNetCore.All no estamos yendo &#8220;hacia atrás&#8221;**, en el sentido de que no pasamos a depender de &#8220;un único paquete&#8221;. Nuestro código sigue, realmente, dependiendo de muchos paquetes NuGet, solo que esos ya vienen preinstalados en cualquier sistema que tenga NetCore 2.0.
  
Ahora **si añades un paquete cualquiera, como Autofac, observarás como este paquete si que se publica junto a tu aplicación:**
  
[<img class="alignnone size-medium wp-image-1967" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/12/autofac-300x246.png" alt="autofac publicado" width="300" height="246" />][4]
  
**Creando tus propias runtime store**
  
El concepto de _runtime store_ es extensible y de hecho te puedes crear las tuyas propias. Imagina que tienes un paquete NuGet que usas constantemente. Puede ser Autofac, puede ser Moq, puede ser un paquete propio. Da igual. El tema está en que en los ordenadores de destino este paquete va a estar instalado en la _runtime store _ y por lo tanto **no quieres incluirla en cada aplicación publicada**. Vamos a ver como puedes hacer esto.
  
Para ello necesitas un _manifiesto de package store_. P. ej. aquí tienes un ejemplo de _manifiesto de package store_ que incluyte Autofac:

<pre class="EnlighterJSRAW" data-enlighter-language="xml">&lt;Project Sdk="Microsoft.NET.Sdk"&gt;
  &lt;ItemGroup&gt;
    &lt;PackageReference Include="Autofac" Version="4.6.2" /&gt;
  &lt;/ItemGroup&gt;
&lt;/Project&gt;
</pre>

Como puedes observar **el formato del fichero de manifiesto es, de hecho, compatible con el csproj**. En mi caso he llamado al fichero &#8220;autofac_manifest.xml&#8221;.
  
Ahora puedo usar el comando _dotnet store_ para crear una _runtime store_:

<pre class="EnlighterJSRAW" data-enlighter-language="shell">dotnet store -m autofac_manifest.xml --framework netcoreapp2.0 --framework-version 2.0.0 --runtime win10-x64</pre>

Eso me creará una _runtime store__ _(por defecto ubicada en %USERPROFILE%&#46;dotnet\store, se puede modificar usando _&#8211;output_)
  
[<img class="alignnone size-medium wp-image-1965" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/12/store_propia-300x81.png" alt="store propia" width="300" height="81" />][5]
  
Con eso creamos la _runtime store _en la máquina. Es interesante ver que, por supuesto, _dotnet store_ resuelve dependencias y si un paquete depende de otro, al final tu store tendrá todos los paquetes (los indicados en el manifiesto y sus dependencias).
  
El siguiente paso, es **publicar un proyecto contra un manifiesto de _runtime store._**_ _Cuando hemos creado la _runtime store_ nos ha creado un fichero &#8220;artifact.xml&#8221;. Este fichero es el manifiesto de la runtime store que podemos pasarle a _dotnet publish._

<pre class="EnlighterJSRAW" data-enlighter-language="shell">dotnet publish -c release -o out --manifest C:\Users\etoma\.dotnet\store\x64\netcoreapp2.0\artifact.xml</pre>

Si lo pruebas verás **como a pesar de que nuestro fichero csproj tiene una referencia a Autofac, ahora la DLL de Autofac no se publica**.
  
Puedes usar la etiqueta <TargetManifestFiles> para incluir ficheros de manifiesto directamente en el csproj. De ese modo evitas tener que usar el parámetro &#8220;&#8211;manifest&#8221; en el comando _dotnet publish_. Efectivamente se llama _TargetManifestFiles_ en plural porque puedes poner varios (separados por punto y coma):

<pre class="EnlighterJSRAW" data-enlighter-language="xml">&lt;PropertyGroup&gt;
  &lt;TargetManifestFiles&gt;path/a/manifest1.xml;path/a/manifest2.xml&lt;/TargetManifestFiles&gt;
&lt;/PropertyGroup&gt;
</pre>

De este modo puedes tener los distintos &#8220;artifact.xml&#8221; publicados en tu repositorio de código fuente y directamente usarlos en los csproj.
  
**Publicar sin incluir la default runtime store**
  
Es posible que quieras publicar &#8220;a la antigua usanza&#8221;, es decir con todas las dependencias de tu proyecto. Para ello te basta con usar la propiedad _PublishWithAspNetCoreTargetManifest_ del csproj y ponerla a _false_:

<pre class="EnlighterJSRAW" data-enlighter-language="xml">&lt;PropertyGroup&gt;
  &lt;PublishWithAspNetCoreTargetManifest&gt;false&lt;/PublishWithAspNetCoreTargetManifest&gt;
&lt;/PropertyGroup&gt;</pre>

Con eso indicas que **no quieres publicar contra la _default runtime store_, por lo que ahora, al publicar se incluirán todas las referencias de tu aplicación** (por cierto, en mi caso, usando la plantilla por defecto son 192 ensamblados, ¡ahí es nada!).  Es importante notar que puedes usar  _PublishWithAspNetCoreTargetManifest_ a la vez que _TargetManifestFiles_ (o la opción &#8211;manifest en _dotnet publish_) y así darse el caso de publicar los ensamblados por defecto de ASP.NET Core pero no otros que tu quieras. Hay una flexibilidad total.
  
**Despliegues FDD vs despliegues SCD**
  
Este apartado es simplemente aclaratorio. No tiene que ver directamente con el uso de _metapaquetes_ pero ya que hemos tocado el tema de publicaciones, considero interesante añadirlo. Llamamos un despliegue FDD (Framework-dependent deployment) al despliegue que contiene solo tu aplicación y las dependencias de terceros, pero no las de NetCore. **Un despliegue FDD usará la versión de NetCore instalada en la máquina de destino**. La ventaja principal es que tu aplicación ocupa menos, la desventaja es que el ordenador de destino debe tener instalado la versión de NetCore contra la que se compiló tu aplicación o una posterior (eso podría llegar a causar problemas en el caso de que una versión posterior de NetCore rompiese la compatibilidad). **No confundas un despliegue FDD con usar la _default runtime store_. **Son dos cosas distintas:

  * Puedes tener un despliegue FDD con todos los ensamblados (sin usar la _default runtime store_)
  * Puedes tener un despliegue FDD con solo tus ensamblados (usando la _default runtime store_)

En NetCore 1.x por defecto tenemos el primer caso, y en NetCore 2, el segundo. Pero en ambos casos usamos FDD.
  
El segundo tipo de despliegue es SCD (Self-contained deployment) **que es cuando desplegamos NetCore junto con nuestra aplicación**. De este modo nos aseguramos que nuestra aplicación funcione en cualquier máquina, ya que viene con &#8220;su propio NetCore&#8221;. El único requisito es que la máquina **tenga las dependencias nativas de NetCore instaladas** (SCD incluye NetCore pero no sus dependencias nativas).
  
A diferencia de una publicación FDD donde todos los ensamblados publicados son en formato PE y compatibles con cualquier sistema (los mismos binarios te funcionan en Windows, Mac y Linux), **una publicación SCD debe ser para una plataforma en concreto** (ya que debe incluirse la versión de NetCore para la plataforma), por lo que debemos tener tantas publicaciones como plataformas queramos soportar.
  
Bueno... espero que eso os haya ayudado a aclarar las dudas que, quizá, teníais sobre este &#8220;mágico&#8221; paquete de NuGet que lo incluye todo... ¡pero realmente no incluye nada!

 [1]: https://www.nuget.org/packages/Microsoft.AspNetCore.all
 [2]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/12/runtimestore.png
 [3]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/12/netcore1vs2.png
 [4]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/12/autofac.png
 [5]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/12/store_propia.png