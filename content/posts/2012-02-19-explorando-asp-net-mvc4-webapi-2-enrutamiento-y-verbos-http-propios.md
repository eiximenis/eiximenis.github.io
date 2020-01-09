---
title: 'Explorando ASP.NET MVC4 WebAPI – 2: Enrutamiento y verbos HTTP propios'
author: eiximenis

date: 2012-02-19T10:46:31+00:00
geeks_url: /?p=1587
geeks_visits:
  - 4077
geeks_ms_views:
  - 2922
categories:
  - Uncategorized

---
Buenas! Este es el segundo post de la serie que trata sobre ASP.NET Web API una de las grandes novedades que vienen con ASP.NET MVC. El primer post de la serie fue la [introducción][1]. Lo que quiero comentar antes que nada es que esta serie la estoy escribiendo no como un tutorial de ASP.NET Web API desde el punto de vista de un experto (porque no lo soy) sino desde el punto de vista de alguien que conoce ASP.NET MVC y está empezando a explorar Web API. 

Hoy vamos a tratar un poco más el tema del enrutamiento, y el uso de verbos HTTP propios.

En el post anterior vimos como simplemente llamando a los métodos como el verbo HTTP a usar, el sistema enrutaba las peticiones perfectamente:

<div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
  <div style="background: #ddd; max-height: 300px; overflow: auto">
    <ol style="padding-bottom: 0px; margin: 0px 0px 0px 2em; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
      <li>
        <span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> <span style="color: #2b91af">ValuesController</span> : <span style="color: #2b91af">ApiController</span>
      </li>
      <li style="background: #f3f3f3">
        {
      </li>
      <li>
        &#160;&#160;&#160; <span style="color: #008000">// GET /api/values/5</span>
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Get(<span style="color: #0000ff">int</span> id)
      </li>
      <li>
        &#160;&#160;&#160; {
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #0000ff">return</span> <span style="color: #a31515">"Hola: "</span> + id.ToString();
      </li>
      <li>
        &#160;&#160;&#160; }
      </li>
      <li style="background: #f3f3f3">
        }
      </li>
    </ol>
  </div></p>
</div>

El método Get se llama a través dela URL /api/values/xxx y el verbo HTTP GET. Si el método se llamase Post() entonces se accedería a él a través del verbo HTTP POST. Vamos a ver ahora algunas variantes, porque esto se va a poner… interesante 🙂

<div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
  <div style="background: #ddd; max-height: 300px; overflow: auto">
    <ol style="padding-bottom: 0px; margin: 0px 0px 0px 2.5em; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
      <li>
        <span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> <span style="color: #2b91af">ValuesController</span> : <span style="color: #2b91af">ApiController</span>
      </li>
      <li style="background: #f3f3f3">
        {
      </li>
      <li>
        &#160;&#160;&#160; <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> GetAlgo(<span style="color: #0000ff">int</span> id)
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; {
      </li>
      <li>
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #0000ff">return</span> <span style="color: #a31515">"Get Algo: "</span> + id.ToString();
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; }
      </li>
      <li>
        &#160;
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> GetAlgoDistinto(<span style="color: #0000ff">int</span> id)
      </li>
      <li>
        &#160;&#160;&#160; {
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #0000ff">return</span> <span style="color: #a31515">"Get Algo Distinto: "</span> + id.ToString();
      </li>
      <li>
        &#160;&#160;&#160; }
      </li>
      <li style="background: #f3f3f3">
        &#160;
      </li>
      <li>
        }
      </li>
    </ol>
  </div></p>
</div>

Bueno… La primera pregunta es… ¿Y eso funciona? Ahora nuestros métodos no se llaman Get pero empiezan por Get. Será suficiente esto? Veamos:

[<img style="background-image: none; border-right-width: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_5ED33358.png" width="644" height="176" />][2]

¡Muy interesante! Se nos queja diciendo que ha encontrado **más de un método** para invocar dada la URL /Api/Values/2 (y el método GET). Eso significa una cosa: la parte del nombre del método que sucede a Get **es ignorada** por el framework. Eso puede parecer una tontería, pero no lo es en absoluto, ya que me permite que los nombres de los métodos de mi controlador sean más claros:

<div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
  <div style="background: #ddd; max-height: 300px; overflow: auto">
    <ol style="padding-bottom: 0px; margin: 0px 0px 0px 2.5em; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
      <li>
        <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> GetAll()
      </li>
      <li style="background: #f3f3f3">
        {
      </li>
      <li>
        &#160;&#160;&#160; <span style="color: #0000ff">return</span> <span style="color: #a31515">"Get All"</span>;
      </li>
      <li style="background: #f3f3f3">
        }
      </li>
      <li>
        &#160;
      </li>
      <li style="background: #f3f3f3">
        <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> GetById(<span style="color: #0000ff">int</span> id)
      </li>
      <li>
        {
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; <span style="color: #0000ff">return</span> <span style="color: #a31515">"Get Algo Distinto: "</span> + id.ToString();
      </li>
      <li>
        }
      </li>
    </ol>
  </div></p>
</div>

La URL /api/values invocará al método GetAll y la url /api/values/10 invocará al método GetById pasándole el 10. 

Bueno… usando esta convención un controlador representa un recurso (p.ej. Values) y las acciones que pueden efectuarse sobre él, a través de los distintos verbos HTTP y parámetros de la URL. Esta es una visión muy REST, de hecho en REST generalmente se usa un patrón de URLs parecidos a:

  * /recursos (p.ej. /api/values) 
      * GET: Obtiene la colección de recursos 
      * PUT: Sustitye todos la colección de recursos por otra pasada (raro) 
      * POST: Añade un recurso nuevo a la colección 
      * DELETE: Elimina todos los recursos 
  * /recursos/id (p.ej. /api/values/5) 
      * GET: Obtiene este recurso 
      * PUT: Crea el recurso con este ID, o si ya existe lo sustituye 
      * POST: Añade información al elemento con este ID (modiificación) 
      * DELETE: Elimina el elemento con este ID 

Este patrón de URLs REST es el que nos da por defecto el routing de ASP.NET Web API

Pero, por supuesto este patrón puede no ser útil siempre, a nosotros nos puede interesar tener soporte para una URL del tipo /api/values/GreaterThan/10 que nos devuelva solo los valores superiores a 10 (por decir algo). Pues bien, eso podemos conseguirlo usando un mapeo similar al que tendríamos en ASP.NET MVC,
   
usando el route value _action_. En este caso, este route value se mapea al nombre del método (al igual que los controladores clásicos) y si queremos especificar un verbo HTTP distinto de GET debemos decorar el método con el atributo específico. Además ambos métodos de routing se pueden combinar en una tabla de rutas como esta:

<div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
  <div style="background: #ddd; max-height: 300px; overflow: auto">
    <ol style="padding-bottom: 0px; margin: 0px 0px 0px 2.5em; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
      <li>
        routes.MapHttpRoute(
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; name: <span style="color: #a31515">"DefaultApi"</span>,
      </li>
      <li>
        &#160;&#160;&#160; routeTemplate: <span style="color: #a31515">"api/{controller}/{id}"</span>,
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; defaults: <span style="color: #0000ff">new</span> { id = <span style="color: #2b91af">RouteParameter</span>.Optional }
      </li>
      <li>
        );
      </li>
      <li style="background: #f3f3f3">
        routes.MapHttpRoute(
      </li>
      <li>
        &#160;&#160;&#160; name: <span style="color: #a31515">"DefaultApiRouted"</span>,
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; routeTemplate: <span style="color: #a31515">"api/{controller}/{action}/{param}"</span>,
      </li>
      <li>
        &#160;&#160;&#160; defaults: <span style="color: #0000ff">new</span> { param = <span style="color: #2b91af">RouteParameter</span>.Optional }
      </li>
      <li style="background: #f3f3f3">
        );
      </li>
    </ol>
  </div></p>
</div>

Notad como la segunda entrada se parece mucho, pero mucho, a la entrada por defecto de ASP.NET MVC.

En este caso, las URLS (y suponiendo el verbo HTTP GET):

  * /api/values –> Invocará al método Get() o GetXXX() (sin parámetros) 
  * /api/values/x –> Invocará al método Get(id) o GetXXX(id) (con un parámetro) 
  * /api/values/GreaterThan/x –> Invocará al método GreaterThan(param) 

Fijaos que el parámetro se llama distinto (id en la primera ruta y param en la segunda). Eso debe ser así, si usáis el mismo nombre de parámetro en ambas rutas os va a dar un error de ambigüedad en la tabla de rutas.

**Verbos propios HTTP**

Aunque el protocolo HTTP define un conjunto de verbos estándard (tales como GET, POST, PUT, DELETE) no hay nada que impida crearse verbos propios. En el protocolo HTTP el verbo usado no es nada más que la primera palabra que se envía en la petición. P.ej. esa es una petición GET:

<font face="Courier New">GET </font>[<font face="Courier New">http://www.fiddler2.com/fiddler2/updatecheck.asp?isBeta=False</font>][3] <font face="Courier New">HTTP/1.1 <br />User-Agent: Fiddler/2.3.9.0 (.NET 2.0.50727.5448; Microsoft Windows NT 6.1.7601 Service Pack 1; AMD64) <br />Pragma: no-cache <br />Accept-Language: es-ES <br />Referer: </font>[<font face="Courier New">http://fiddler2.com/client/2.3.9.0</font>][4]   
<font face="Courier New">Host: www.fiddler2.com <br />Connection: Close</font> 

De todos esos datos que envía el navegador si nos fijamos en la primera línea vemos que empieza por GET: eso es el verbo HTTP. Nada nos impide usar un verbo propio (otra cosa es que el servidor que está al otro lado lo entienda claro).

Quizá te preguntes por la necesidad de definir verbos nuevos HTTP. Bueno, imagina una situación como la siguiente: estás creando un juego, que expone una interfaz REST para ser consumida desde cualquier sitio. Sigue suponiendo que en dicho juego se puede atacar casillas. Una forma de hacerlo usando los verbos estándard sería una petición PUT a /Attacks/id (donde ID fuese un ID del nuevo ataque, p.ej.). Es una visión muy CRUD: atacar una casilla es _crear un nuevo ataque y insertarlo_. Pero otra visión podría ser una petición a /Tile/id_casilla con el verbo HTTP Attack. En esta otra visión el recurso al que accedemos es la casilla y la acción que realizamos sobre ella es atacarla. Esta puede ser una razón por la que queremos crearnos nuestros propios verbos HTTP.

Ahora la duda es si podemos gestionar peticiones que lleguen con un verbo HTTP propio usando ASP.NET Web API. La primera prueba es fácil: consiste en crear un método con un nombre determinado, p.ej. Attack y crear una petición usando este verbo HTTP. Así que creamos el siguiente método en nuestro controlador:

<div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
  <div style="background: #ddd; max-height: 300px; overflow: auto">
    <ol style="padding-bottom: 0px; margin: 0px 0px 0px 2em; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
      <li>
        <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Attack()
      </li>
      <li style="background: #f3f3f3">
        {
      </li>
      <li>
        &#160;&#160;&#160; <span style="color: #0000ff">return</span> <span style="color: #a31515">"Verbo attack!"</span>;
      </li>
      <li style="background: #f3f3f3">
        }
      </li>
    </ol>
  </div></p>
</div>

Y luego creamos una petición a api/values usando el verbo Attack. Para usar un verbo HTTP inventado lo más rápido es usar fiddler. Abrimos fiddler, le damos a la pestaña “Composer” (en versiones anteriores era “Request Builder”) y ponemos los datos:

[<img style="background-image: none; border-right-width: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_38612A40.png" width="644" height="143" />][5]

El host que veis es simplemente un alias (definido en hosts) para localhost, porque fiddler no soporta localhost.

Y el resultado que nos muestra el propio fiddler es este:

<font face="Courier New">HTTP/1.1 405 Method Not Allowed <br />Server: ASP.NET Development Server/10.0.0.0 <br />Date: Sun, 19 Feb 2012 09:23:13 GMT <br />X-AspNet-Version: 4.0.30319 <br />Cache-Control: no-cache <br />Pragma: no-cache <br />Expires: -1 <br />Content-Type: application/json; charset=utf-8 <br />Connection: Close</font>

<font face="Courier New">"The requested resource does not support http method &#8216;ATTACK&#8217;."</font>

Bueno… en cierto modo es lo que nos podíamos esperar… Hubiese sido demasiado bonito que funcionase así 🙂

Si queremos soportar verbos HTTP propios lo que tenemos que hacer es decorar el método apropiado con AcceptVerbs y pasar el nombre del verbo HTTP que queremos:

<div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
  <div style="background: #ddd; max-height: 300px; overflow: auto">
    <ol style="padding-bottom: 0px; margin: 0px 0px 0px 2em; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
      <li>
        [<span style="color: #2b91af">AcceptVerbs</span>(<span style="color: #a31515">"Attack"</span>)]
      </li>
      <li style="background: #f3f3f3">
        <span sty
le="color: #0000ff">public</span> <span style="color: #0000ff">string</span> PerformAttack()
      </li>
      <li>
        {
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; <span style="color: #0000ff">return</span> <span style="color: #a31515">"Verbo attack!"</span>;
      </li>
      <li>
        }
      </li>
    </ol>
  </div></p>
</div>

Pasamos a AcceptVerbs el verbo HTTP a soportar y luego el nombre del método ya es indiferente. Si ahora repetimos la petición con fiddler, en lugar de un error 405 obtenemos un 200 (petición correcta):

<font face="Courier New">HTTP/1.1 200 OK <br />Server: ASP.NET Development Server/10.0.0.0 <br />Date: Sun, 19 Feb 2012 09:27:54 GMT <br />X-AspNet-Version: 4.0.30319 <br />Cache-Control: no-cache <br />Pragma: no-cache <br />Expires: -1 <br />Content-Type: application/json; charset=utf-8 <br />Connection: Close <br />Content-Length: 15</font>

<font face="Courier New">"Verbo attack!"</font>

Recuerda que los verbos HTTP propios no pueden ser usados por lo general desde un navegador (ni usando javascript). Así que si los usas debes ofrecer una alternativa con verbos HTTP estándard (y si quieres ser totalmente compatible con cualquier navegador limitándote a GET y POST) si tu API debe ser invocable desde un navegador. Algunas APIs REST ofrecen mecanismos built-in para esto, como el uso de [X-Http-Method-Override][6] (usado en las APIs de Google) o de [X-Http-Method][7] (usado en OData) pero ninguno de los dos está directamente soportado en ASP.NET Web API.

Bueno… espero que os haya sido interesante. En siguientes posts seguiremos explorando ASP.NET Web API que nos quedan varias cosas intersantes para ver!!

Un saludo a todos!

 [1]: http://geeks.ms/blogs/etomas/archive/2012/02/17/explorando-asp-net-mvc4-webapis-1-introducci-243-n.aspx
 [2]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_01E3A4C9.png
 [3]: http://www.fiddler2.com/fiddler2/updatecheck.asp?isBeta=False
 [4]: http://fiddler2.com/client/2.3.9.0
 [5]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_67FF48CC.png
 [6]: http://code.google.com/intl/es-ES/apis/gdata/docs/2.0/basics.html
 [7]: http://msdn.microsoft.com/en-us/library/dd541471(v=prot.10).aspx