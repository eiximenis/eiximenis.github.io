---
title: 'Explorando ASP.NET MVC4 WebApis – 1: Introducción'
author: eiximenis

date: 2012-02-17T18:28:00+00:00
geeks_url: /?p=1586
geeks_visits:
  - 7115
geeks_ms_views:
  - 2142
categories:
  - Uncategorized

---
> 
Bueno... ayer se animó un poco el cotarro con la salida de la beta de ASP.NET MVC4. Y ayer mismo, el Maestro realizó un post fenómenal explicando un poco [todas las novedades del framework][1]. Echadle un vistazo al post, porque es realmente espectacular (para variar :p).

De todas las novedades que aparecen en esta versión 4, yo quiero centrarme en la llamada ASP.NET Web API. A pesar de su nombre no se trata de una API nueva. En concreto se trata _de un conjunto de herramientas para permitirnos a nosotros, la creación de APIs basadas en REST_. Quizá os preguntaréis: Ah, ¿pero es que ahora **no** se podía?

Pues la verdad es que... sí. Hasta ahora si queríamos crear una API REST teníamos dos opciones. La primera era usar ASP.NET MVC y crear una aplicación cuyos controladores en lugar de devolver vistas con contenido HTML devolvieran datos en JSON o XML. La verdad es que ASP.NET MVC con su sistema de rutas y el fácil soporte para los distintos verbos http es una herramienta ideal para crear servicios REST. Y ¿la otra opción? La otra opción se llama WCF. La verdad es que al principio WCF no estaba muy bien preparada para la creación de servicios REST (nació mucho más orientada a servicios tipo RPC como servicios Web SOAP), pero con el tiempo se le fueron poniendo añadidos que culminaron con la salida de WCF Web API (<http://wcf.codeplex.com/wikipage?title=WCF%20HTTP>) que simplificaba al máximo la creación de servicios REST usando WCF.

Así que, ¿si ya tenemos dos opciones para la creación de servicios REST para qué añadir otra? Pues muy sencillo: para integrar a las dos anteriores. ASP.NET Web Api, recoge lo mejor de WCF Web Api y lo integra dentro de ASP.NET MVC y de esta manera obtenemos un sistema muy sencillo y potente para la creación de servicios REST. Un sistema que de fábrica nos ofrece:

  1. Negociación de contenido automática: Que permite que el servicio devuelva los datos en el formato que prefiera el cliente (p. ej. XML o JSON), sin que nosotros nos hayamos de preocupar al respecto. 
  2. Soporte para consultas realizadas usando ODATA 
  3. Hosting autocontenido, es decir, podemos hospedar nuestra API REST usando IIS (sin ningún problema) pero también dentro de un ejecutable propio! 

Además obtenemos todos los beneficios de ASP.NET MVC: Model binding, rutas, action filters,... Y es que estamos desarrollando una aplicación ASP.NET MVC!

**Creación del proyecto**

Para crear un proyecto de Web API es muy sencillo, con la Beta de ASP.NET MVC 4 instalada, os aparecerá un template de proyecto nuevo llamado &ldquo;ASP.NET Web API&rdquo;, que os creará el esqueleto del proyecto base:

[<img height="244" width="230" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_2180BBE0.png" alt="image" border="0" title="image" style="background-image: none; border-right-width: 0px; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" />][2]

Como podéis ver en la figura no se diferencia en nada de un proyecto ASP.NET MVC normal... Veamos el código que nos ha generado VS para el controlador ValuesController (lo genera automáticamente):

<div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: 'Courier New', courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
  <div style="background: #ddd; max-height: 300px; overflow: auto">
    <ol style="padding-bottom: 0px; margin: 0px 0px 0px 2.5em; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
      <li>
        <span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> <span style="color: #2b91af">ValuesController</span> : <span style="color: #2b91af">ApiController</span>
      </li>
      <li style="background: #f3f3f3">
        {
      </li>
      <li>
        &nbsp;&nbsp;&nbsp; <span style="color: #008000">// GET /api/values</span>
      </li>
      <li style="background: #f3f3f3">
        &nbsp;&nbsp;&nbsp; <span style="color: #0000ff">public</span> <span style="color: #2b91af">IEnumerable</span><<span style="color: #0000ff">string</span>> Get()
      </li>
      <li>
        &nbsp;&nbsp;&nbsp; {
      </li>
      <li style="background: #f3f3f3">
        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #0000ff">return</span> <span style="color: #0000ff">new</span> <span style="color: #0000ff">string</span>[] { <span style="color: #a31515">&#8220;value1&#8221;</span>, <span style="color: #a31515">&#8220;value2&#8221;</span> };
      </li>
      <li>
        &nbsp;&nbsp;&nbsp; }
      </li>
      <li style="background: #f3f3f3">
        &nbsp;
      </li>
      <li>
        &nbsp;&nbsp;&nbsp; <span style="color: #008000">// GET /api/values/5</span>
      </li>
      <li style="background: #f3f3f3">
        &nbsp;&nbsp;&nbsp; <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Get(<span style="color: #0000ff">int</span> id)
      </li>
      <li>
        &nbsp;&nbsp;&nbsp; {
      </li>
      <li style="background: #f3f3f3">
        &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: #0000ff">return</span> <span style="color: #a31515">&#8220;value&#8221;</span>;
      </li>
      <li>
        &nbsp;&nbsp;&nbsp; }
      </li>
      <li style="background: #f3f3f3">
        &nbsp;
      </li>
      <li>
        &nbsp;&nbsp;&nbsp; <span style="color: #008000">// POST /api/values</span>
      </li>
      <li style="background: #f3f3f3">
        &nbsp;&nbsp;&nbsp; <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Post(<span style="color: #0000ff">string</span> value)
      </li>
      <li>
        &nbsp;&nbsp;&nbsp; {
      </li>
      <li style="background: #f3f3f3">
        &nbsp;&nbsp;&nbsp; }
      </li>
      <li>
        &nbsp;
      </li>
      <li style="background: #f3f3f3">
        &nbsp;&nbsp;&nbsp; <span style="color: #008000">// PUT /api/values/5</span>
      </li>
      <li>
        &nbsp;&nbsp;&nbsp; <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Put(<span style="color: #0000ff">int</span> id, <span style="color: #0000ff">string</span> value)
      </li>
      <li style="background: #f3f3f3">
        &nbsp;&nbsp;&nbsp; {
      </li>
      <li>
        &nbsp;&nbsp;&nbsp; }
      </li>
      <li style="background: #f3f3f3">
        &nbsp;
      </li>
      <li>
        &nbsp;&nbsp;&nbsp; <span style="color: #008000">// DELETE /api/values/5</span>
      </li>
      <li style="background: #f3f3f3">
        &nbsp;&nbsp;&nbsp; <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Delete(<span style="color: #0000ff">int</span> id)
      </li>
      <li>
        &nbsp;&nbsp;&nbsp; {
      </li>
      <li style="background: #f3f3f3">
        &nbsp;&nbsp;&nbsp; }
      </li>
      <li>
        }
      </li>
    </ol>
  </div>
</div>

Vale, hay **tres** puntos a destacar:

  1. El controlador no deriva de Controller, deriva de ApiController 
  2. Las acciones no devuelven ActionResult, en su lugar devuelven _datos_ tal cual. Fijaos que en ningún sitio indicamos si estos datos serán enviados en JSON, XML u otro formato. El cliente recibirá los datos automáticamente en el formato que haya pedido. 
  3. El nombre de los métodos es el nombre del verbo HTTP que se usa para acceder a ellos. Es decir en lugar de decorar un método con [HttpPost] para indicar que debe accederse a él usando POST, llamamos a este método Post. 

Si miramos la tabla de rutas veremos que el código generado por VS es el siguiente:

<div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: 'Courier New', courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
  <div style="background: #ddd; max-height: 300px; overflow: auto">
    <ol style="padding-bottom: 0px; margin: 0px 0px 0px 2.5em; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
      <li>
        routes.MapHttpRoute(
      </li>
      <li style="background: #f3f3f3">
        &nbsp;&nbsp;&nbsp; name: <span style="color: #a31515">&#8220;DefaultApi&#8221;</span>,
      </li>
      <li>
        &nbsp;&nbsp;&nbsp; routeTemplate: <span style="color: #a31515">&#8220;api/{controller}/{id}&#8221;</span>,
      </li>
      <li style="background: #f3f3f3">
        &nbsp;&nbsp;&nbsp; defaults: <span style="color: #0000ff">new</span> { id = <span style="color: #2b91af">RouteParameter</span>.Optional}
      </li>
      <li>
        );
      </li>
      <li style="background: #f3f3f3">
        &nbsp;
      </li>
      <li>
        routes.MapRoute(
      </li>
      <li style="background: #f3f3f3">
        &nbsp;&nbsp;&nbsp; name: <span style="color: #a31515">&#8220;Default&#8221;</span>,
      </li>
      <li>
        &nbsp;&nbsp;&nbsp; url: <span style="color: #a31515">&#8220;{controller}/{action}/{id}&#8221;</span>,
      </li>
      <li style="background: #f3f3f3">
        &nbsp;&nbsp;&nbsp; defaults: <span style="color: #0000ff">new</span> { controller = <span style="color: #a31515">&#8220;Home&#8221;</span>, action = <span style="color: #a31515">&#8220;Index&#8221;</span>, id = <span style="color: #2b91af">UrlParameter</span>.Optional }
      </li>
      <li>
        );
      </li>
    </ol>
  </div>
</div>

Vemos dos entradas, la primera es la que corresponde a nuestra API REST (mapeará todas las URLS tipo /api/{controlador}). La segunda entrada es una entrada estándard de ASP.NET MVC porque **podemos mezclar una API REST con controladores estándard que devuelvan vistas (o lo que sea)**.

Si quieremos soportar llamadas del tipo /api/{controlador}/id/texto entonces debemos modificar la tabla de rutas:

<div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: 'Courier New', courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
  <div style="background: #ddd; max-height: 300px; overflow: auto">
    <ol style="padding-bottom: 0px; margin: 0px 0px 0px 2em; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
      <li>
        routes.MapHttpRoute(
      </li>
      <li style="background: #f3f3f3">
        &nbsp;&nbsp;&nbsp; name: <span style="color: #a31515">&#8220;DefaultApi&#8221;</span>,
      </li>
      <li>
        &nbsp;&nbsp;&nbsp; routeTemplate: <span style="color: #a31515">&#8220;api/{controller}/{id}/{value}&#8221;</span>,
      </li>
      <li style="background: #f3f3f3">
        &nbsp;&nbsp;&nbsp; defaults: <span style="color: #0000ff">new</span> { id = <span style="color: #2b91af">RouteParameter</span>.Optional, value = <span style="color: #2b91af">RouteParameter</span>.Optional }
      </li>
      <li>
        );
      </li>
    </ol>
  </div>
</div>

Y ahora podemos añadir un método tal como el siguiente a nuestro controlador (ValuesController):

<div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: 'Courier New', courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
  <div style="background: #ddd; max-height: 300px; overflow: auto">
    <ol style="padding-bottom: 0px; margin: 0px 0px 0px 2em; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
      <li>
        <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Get(<span style="color: #0000ff">int</span> id, <span style="color: #0000ff">string</span> value)
      </li>
      <li style="background: #f3f3f3">
        {
      </li>
      <li>
        &nbsp;&nbsp;&nbsp; <span style="color: #0000ff">return</span> value + <span style="color: #a31515">&#8221; -> &#8220;</span> + id;
      </li>
      <li style="background: #f3f3f3">
        }
      </li>
    </ol>
  </div>
</div>

Y si llamamos a la url http://localhost:55603/api/values/3/hola obtenemos:

<img height="150" width="368" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_0352FE2C.png" alt="image" border="0" title="image" style="background-image: none; border-right-width: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" />

Fijaos como nos ha llamado a nuestro método y nos ha devuelto los datos en formato XML, de forma automática.

Los que hayáis desarrollado en ASP.NET MVC os dais cuenta de una cosa? **Eso funciona!** Fijaos que tenemos en nuestro controlador:

<div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: 'Courier New', courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
  <div style="background: #ddd; max-height: 300px; overflow: auto">
    <ol style="padding-bottom: 0px; margin: 0px 0px 0px 2.5em; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
      <li>
        <span style="color: #008000">// GET /api/values</span>
      </li>
      <li style="background: #f3f3f3">
        <span style="color: #0000ff">public</span> <span style="color: #2b91af">IEnumerable</span><<span style="color: #0000ff">string</span>> Get()
      </li>
      <li>
        {
      </li>
      <li style="background: #f3f3f3">
        &nbsp;&nbsp;&nbsp; <span style="color: #0000ff">return</span> <span style="color: #0000ff">new</span> <span style="color: #0000ff">string</span>[] { <span style="color: #a31515">&#8220;value1&#8221;</span>, <span style="color: #a31515">&#8220;value2&#8221;</span> };
      </li>
      <li>
        }
      </li>
      <li style="background: #f3f3f3">
        &nbsp;
      </li>
      <li>
        <span style="color: #008000">// GET /api/values/5</span>
      </li>
      <li style="background: #f3f3f3">
        <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Get(<span style="color: #0000ff">int</span> id)
      </li>
      <li>
        {
      </li>
      <li style="background: #f3f3f3">
        &nbsp;&nbsp;&nbsp; <span style="color: #0000ff">return</span> <span style="color: #a31515">&#8220;Hola: &#8220;</span> + id.ToString();
      </li>
      <li>
        }
      </li>
      <li style="background: #f3f3f3">
        &nbsp;
      </li>
      <li>
        <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Get(<span style="color: #0000ff">int</span> id, <span style="color: #0000ff">string</span> value)
      </li>
      <li style="background: #f3f3f3">
        {
      </li>
      <li>
        &nbsp;&nbsp;&nbsp; <span style="color: #0000ff">return</span> value + <span style="color: #a31515">&#8221; -> &#8220;</span> + id;
      </li>
      <li style="background: #f3f3f3">
        }
      </li>
    </ol>
  </div>
</div>

¿Os acordáis lo que pasaba cuando en un controlador clásico teníamos algo como lo siguiente?

<div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: 'Courier New', courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
  <div style="background: #ddd; max-height: 300px; overflow: auto">
    <ol style="padding-bottom: 0px; margin: 0px 0px 0px 2.5em; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
      <li>
        <span style="color: #0000ff">public</span> <span style="color: #2b91af">ActionResult</span> Index()
      </li>
      <li style="background: #f3f3f3">
        {
      </li>
      <li>
        &nbsp;&nbsp;&nbsp; <span style="color: #0000ff">return</span> View();
      </li>
      <li style="background: #f3f3f3">
        }
      </li>
      <li>
        &nbsp;
      </li>
      <li style="background: #f3f3f3">
        <span style="color: #0000ff">public</span> <span style="color: #2b91af">ActionResult</span> Index(<span style="color: #0000ff">int</span> id)
      </li>
      <li>
        {
      </li>
      <li style="background: #f3f3f3">
        &nbsp;&nbsp;&nbsp; <span style="color: #0000ff">return</span> View();
      </li>
      <li>
        }
      </li>
    </ol>
  </div>
</div>

&iexcl;Exacto! Pasaba (y sigue pasando en los controladores _&ldquo;normales_&rdquo;) esto:

[<img height="111" width="644" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_6F4945C8.png" alt="image" border="0" title="image" style="background-image: none; border-right-width: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" />][3]

En cambio en los controladores para APIs no ocurre esto: tenemos tres métodos (Get, Get(int) y Get(int, string)) y el sistema llamará correctamente a uno u otro en función de si en la URL le pasamos 0,1 ó 2 parámetros!

Bueno... Lo dejamos aquí en este post. En los siguientes iremos investigando más sobre todas las características de Web API: Iremos viendo como usar ODATA, distintos formatos de salida, autenticación y no sé... lo que se me vaya ocurriendo, sinceramente!

Un abrazo!

 [1]: http://www.variablenotfound.com/2012/02/aspnet-4-beta-disponible.html
 [2]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_0720C8C9.png
 [3]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_36428E93.png