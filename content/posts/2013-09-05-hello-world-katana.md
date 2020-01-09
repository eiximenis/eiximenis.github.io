---
title: ¡Hello World Katana!
author: eiximenis

date: 2013-09-05T16:10:11+00:00
geeks_url: /?p=1651
geeks_visits:
  - 1953
geeks_ms_views:
  - 908
categories:
  - Uncategorized

---
Buenas! En este post vamos a ver como empezar a trabajar con Katana. En un <a href="http://geeks.ms/blogs/etomas/archive/2013/07/15/katana-cortando-el-framework.aspx" target="_blank" rel="noopener noreferrer">post anterior</a> hablé un poco de Katana y **mis fantasías** (más o menos húmedas) de lo que podría ser un un futuro.

Antes que nada hagamos un repaso rápido:

  1. <a href="http://owin.org/" target="_blank" rel="noopener noreferrer">OWIN</a>: Open Web Interface for .NET. **Especificación** que define un **estándard** para comunicar servidores web y aplicaciones web (en tecnología .NET). 
  2. <a href="http://katanaproject.codeplex.com/" target="_blank" rel="noopener noreferrer">Katana</a>: Implementación de Microsoft de la especificación OWIN. 

¿Cuál es la ventaja principal de hacer que una aplicación web sea compatible con OWIN? Pues simplemente que desacoplas esta aplicación web del servidor usado. Cualquier servidor (que sea compatible con OWIN) podrá hospedar tu aplicación. Esto abre la puerta a tener aplicaciones web _self-hosted._

**Empezando con Owin y Visual Studio 2012**

En este primer post vamos a realizar la aplicación más posible sencilla (un hello world, originalidad a tope).

Para empezar abre VS2012 (o VS2013 si lo tienes, para este post da igual) y crea una aplicación de consola. Luego añade con NuGet los siguientes paquetes:

  1. Microsoft.Owin.Hosting 
  2. Microsoft.Owin.Host.HttpListener 
  3. Microsoft.Owin.Diagnostics 
  4. Owin.Extensions 

Actualmente están en _prerelase_ así que incluye el flag –IncludePreRelase&#160; cuando lances el comando Install-Package desde la consola de NuGet.

Una vez tengos estos paquetes instalados, ya podemos desarrollar nuestra aplicación. Lo primero que necesitamos es una clase que ponga en marcha nuestra aplicación:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">Program</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">static</span> <span style="color: #569cd6">void</span> <span style="color: white">Main</span>(<span style="color: #569cd6">string</span>[] <span style="color: white">args</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">uri</span> <span style="color: #b4b4b4">=</span> <span style="color: #d69d85">"http://localhost:8080/"</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">using</span> (<span style="color: #4ec9b0">WebApp</span><span style="color: #b4b4b4">.</span><span style="color: white">Start</span><span style="color: #b4b4b4"><</span><span style="color: #4ec9b0">Startup</span><span style="color: #b4b4b4">></span>(<span style="color: white">uri</span>))
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #4ec9b0">Console</span><span style="color: #b4b4b4">.</span><span style="color: white">WriteLine</span>(<span style="color: #d69d85">"Started"</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #4ec9b0">Console</span><span style="color: #b4b4b4">.</span><span style="color: white">ReadKey</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #4ec9b0">Console</span><span style="color: #b4b4b4">.</span><span style="color: white">WriteLine</span>(<span style="color: #d69d85">"Stopping"</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Usamos la clase WebApp del paquete Microsoft.Owin.Hosting para poner en marcha nuestra aplicación. El parámetro genérico (Startup) es el nombre de una clase que será la que configurará nuestra aplicación.

Veamos el código:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">Startup</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">void</span> <span style="color: white">Configuration</span>(<span style="color: #b8d7a3">IAppBuilder</span> <span style="color: white">app</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">app</span><span style="color: #b4b4b4">.</span><span style="color: white">UseHandlerAsync</span>((<span style="color: white">req</span>, <span style="color: white">res</span>) <span style="color: #b4b4b4">=></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">res</span><span style="color: #b4b4b4">.</span><span style="color: white">ContentType</span> <span style="color: #b4b4b4">=</span> <span style="color: #d69d85">"text/plain"</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: white">res</span><span style="color: #b4b4b4">.</span><span style="color: white">WriteAsync</span>(<span style="color: #d69d85">"Hello Katana. You reached "</span> <span style="color: #b4b4b4">+</span> <span style="color: white">req</span><span style="color: #b4b4b4">.</span><span style="color: white">Uri</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; });
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

El método Configuration se invoca automáticamente y se recibe un parámetro IAppBuilder. Dicha interfaz estaba definida en la specificación de OWIN ([http://owin.org/spec/owin-0.12.0.html#\_2.10.\_IAppBuilder][1]) pero desapareció en la versión final. 

Katana usa esta interfaz para permitir a la aplicación web configurar el pipeline de procesamiento de peticiones. De momento nuestra aplicación es muy simple: Por cada petición, construirá una respuesta con el texto “Hello Katana. You reached “ seguido de la URL navegada.

Si ejecutamos el proyecto, y abrimos un navegador y nos vamos a localhost:8080, vemos que nuestra aplicación ya está en marcha:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; display: inline; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_3F611CDC.png" width="244" height="108" />][2] 

Fíjate que nuestra aplicación es un ejecutable. No hay servidor web, ni cassini, ni IIS Express, ni IIS, ni nada 🙂

**Agregando un módulo**

OWIN se define de forma totalmente modular. Por un lado tenemos un Host (en este caso es nuestro ejecutable a través del objeto WebApp del paquete Microsoft.Owin.Host), varios módulos y finalmente la aplicación en si.

Los módulos implementan un “delegado” que
   
se conoce como AppFunc (aunque no hay ningún delegado real con este nombre). AppFunc es realmente Func<IDictionary<string, object>, Task>, es decir recibir un IDictionary<string, object> y devolver un Task.

La idea es que un módulo recibe un diccionario (claves cadenas, valores objects) que es el entorno del servidor y devuelve una Task que es el código que este módulo debe ejecutar.

Los módulos están encadenados y cada módulo debe llamar al siguiente. El aspecto genérico de un módulo queda así:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">using</span> <span style="color: #4ec9b0">AppFunc</span> <span style="color: #b4b4b4">=</span> <span style="color: #4ec9b0">Func</span><span style="color: #b4b4b4"><</span><span style="color: #b8d7a3">IDictionary</span><span style="color: #b4b4b4"><</span><span style="color: #569cd6">string</span>, <span style="color: #569cd6">object</span><span style="color: #b4b4b4">></span>, <span style="color: #4ec9b0">Task</span><span style="color: #b4b4b4">></span>;
  </p>
  
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">OwinConsoleLog</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">private</span> <span style="color: #569cd6">readonly</span> <span style="color: #4ec9b0">AppFunc</span> <span style="color: white">_next</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: white">OwinConsoleLog</span>(<span style="color: #4ec9b0">AppFunc</span> <span style="color: white">next</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">_next</span> <span style="color: #b4b4b4">=</span> <span style="color: white">next</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #4ec9b0">Task</span> <span style="color: white">Invoke</span>(<span style="color: #b8d7a3">IDictionary</span><span style="color: #b4b4b4"><</span><span style="color: #569cd6">string</span>, <span style="color: #569cd6">object</span><span style="color: #b4b4b4">></span> <span style="color: white">environment</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #4ec9b0">Console</span><span style="color: #b4b4b4">.</span><span style="color: white">WriteLine</span>(<span style="color: #d69d85">"Path requested: </span><span style="color: #80ff80">{0}</span><span style="color: #d69d85">"</span>, <span style="color: white">environment</span>[<span style="color: #d69d85">"owin.RequestPath"</span>]);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: white">_next</span>(<span style="color: white">environment</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

El módulo define el método Invoke y recibe como parámetro el diccionario que contiene el entorno del servidor. Luego llama al siguiente módulo y le pasa el entorno. Fíjate que la clase OwinConsoleLog no implementa ninguna interfaz ni nada, pero **debe** tener un método llamado Invoke que sea conforme al “delegado” _AppFunc_ (es decir que devuelva un Task y reciba un diccionario).

Para añadir el módulo simplemente llamamos al método Use de IAppBuilder pasándole el Type de nuestro módulo:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #569cd6">void</span> <span style="color: white">Configuration</span>(<span style="color: #b8d7a3">IAppBuilder</span> <span style="color: white">app</span>)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: white">app</span><span style="color: #b4b4b4">.</span><span style="color: white">Use</span>(<span style="color: #569cd6">typeof</span> (<span style="color: #4ec9b0">OwinConsoleLog</span>));
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: white">app</span><span style="color: #b4b4b4">.</span><span style="color: white">UseHandlerAsync</span>((<span style="color: white">req</span>, <span style="color: white">res</span>) <span style="color: #b4b4b4">=></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">res</span><span style="color: #b4b4b4">.</span><span style="color: white">ContentType</span> <span style="color: #b4b4b4">=</span> <span style="color: #d69d85">"text/plain"</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: white">res</span><span style="color: #b4b4b4">.</span><span style="color: white">WriteAsync</span>(<span style="color: #d69d85">"Hello Katana. You reached "</span> <span style="color: #b4b4b4">+</span> <span style="color: white">req</span><span style="color: #b4b4b4">.</span><span style="color: white">Uri</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; });
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Si ahora ejecutas el proyecto y navegas a localhost:8080 verás como se imprimen las distintas peticiones recibidas:

[<img title="image" style="border-top: 0px; border-right: 0px; border-bottom: 0px; border-left: 0px; display: inline" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_79DC1CB0.png" width="484" height="90" />][3] 

¡Listos! Hemos creado nuestra primera aplicación web, compatible con OWIN y auto hospedada 🙂

En sucesivos posts iremos desgranando más cosillas sobre OWIN y Katana…

Saludos!

 [1]: http://owin.org/spec/owin-0.12.0.html#_2.10._IAppBuilder
 [2]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_684C31E5.png
 [3]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_4C5AFCED.png