---
title: Catálogo en ASP.NET MVC3 sin usar BBDD
description: Catálogo en ASP.NET MVC3 sin usar BBDD
author: eiximenis

date: 2012-04-14T00:38:57+00:00
geeks_url: /?p=1591
geeks_visits:
  - 4011
geeks_ms_views:
  - 1527
categories:
  - Uncategorized

---
Bueno… este es un post por “encargo”… Hoy he recibido un <a href="http://twitter.com/#!/JanoRuiz/status/190777187899670528" target="_blank" rel="noopener noreferrer">tweet de @JanoRuiz</a> que decía lo siguiente: _Hola, Saludos, una Consulta, Como Hacer Un Catalogo En asp.net mvc3 Sin Usar BD, Hacer Altas, Bajas y Modificaciones_.

Bueno, vamos a explorar algunas “formas de hacerlo”… 😀

Vamos a utilizar el siguiente modelo para representar los productos:

<div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
  <div style="background: #ddd; max-height: 300px; overflow: auto">
    <ol style="padding-bottom: 0px; margin: 0px 0px 0px 2em; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
      <li>
        <span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> <span style="color: #2b91af">Producto</span>
      </li>
      <li style="background: #f3f3f3">
        {
      </li>
      <li>
        &#160;&#160;&#160; <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Nombre { <span style="color: #0000ff">get</span>; <span style="color: #0000ff">set</span>; }
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; <span style="color: #0000ff">public</span> <span style="color: #0000ff">int</span> Precio { <span style="color: #0000ff">get</span>; <span style="color: #0000ff">set</span>; }
      </li>
      <li>
        }
      </li>
    </ol>
  </div></p>
</div>

Y el catálogo en sí:

<div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
  <div style="background: #ddd; max-height: 300px; overflow: auto">
    <ol style="padding-bottom: 0px; margin: 0px 0px 0px 2.5em; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
      <li>
        <span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> <span style="color: #2b91af">Catalogo</span>
      </li>
      <li style="background: #f3f3f3">
        {
      </li>
      <li>
        &#160;&#160;&#160; <span style="color: #0000ff">private</span> <span style="color: #0000ff">readonly</span> <span style="color: #0000ff">string</span> _nombre;
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; <span style="color: #0000ff">private</span> <span style="color: #0000ff">readonly</span> <span style="color: #2b91af">List</span><<span style="color: #2b91af">Producto</span>> _productos;
      </li>
      <li>
        &#160;
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; <span style="color: #0000ff">public</span> Catalogo()
      </li>
      <li>
        &#160;&#160;&#160; {
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; _productos = <span style="color: #0000ff">new</span> <span style="color: #2b91af">List</span><<span style="color: #2b91af">Producto</span>>();
      </li>
      <li>
        &#160;&#160;&#160; }
      </li>
      <li style="background: #f3f3f3">
        &#160;
      </li>
      <li>
        &#160;&#160;&#160; <span style="color: #0000ff">public</span> <span style="color: #2b91af">IEnumerable</span><<span style="color: #2b91af">Producto</span>> Productos
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; {
      </li>
      <li>
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #0000ff">get</span> { <span style="color: #0000ff">return</span> _productos; }
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; }
      </li>
      <li>
        &#160;
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> AnyadirProducto(<span style="color: #2b91af">Producto</span> p)
      </li>
      <li>
        &#160;&#160;&#160; {
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; _productos.Add(p);
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

**Opción 1. Usar la sesión**

Bueno, la opción más sencilla es usar la sesión para mantener nuestro catálogo. Para ello directamente creamos el catálogo cuando se crea la sesión, añadiendo el siguiente código en global.asax.cs:

<div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
  <div style="background: #ddd; max-height: 300px; overflow: auto">
    <ol style="padding-bottom: 0px; margin: 0px 0px 0px 2em; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
      <li>
        <span style="color: #0000ff">protected</span>&#160; <span style="color: #0000ff">void</span> Session_Start()
      </li>
      <li style="background: #f3f3f3">
        {
      </li>
      <li>
        &#160;&#160;&#160; Session[<span style="color: #a31515">"catalogo"</span>] = <span style="color: #0000ff">new</span> <span style="color: #2b91af">Catalogo</span>();
      </li>
      <li style="background: #f3f3f3">
        }
      </li>
    </ol>
  </div></p>
</div>

La sesión se crea en la primera petición que realiza el usuario a nuestra web y está vinculada al usuario (usuario en este contexto = ventana de navegador, por norma general).

Ahora cuando queramos acceder al catálogo:

<div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
  <div style="background: #ddd; max-height: 300px; overflow: auto">
    <ol style="padding-bottom: 0px; margin: 0px 0px 0px 2em; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
      <li>
        [<span style="color: #2b91af">HttpPost</span>]
      </li>
      <li style="background: #f3f3f3">
        <span style="color: #0000ff">public</span> <span style="color: #2b91af">ActionResult</span> Nuevo(<span style="color: #2b91af">Producto</span> producto)
      </li>
      <li>
        {
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; <span style="color: #0000ff">var</span> catalogo = Session[<span style="color: #a31515">"Catalogo"</span>] <span style="color: #0000ff">as</span> <span style="color: #2b91af">Catalogo</span>;
      </li>
      <li>
        &#160;&#160;&#160; catalogo.AnyadirProducto(producto);
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; <span style="color: #0000ff">return</span> RedirectToAction(<span style="color: #a31515">"Index"</span>);
      </li>
      <li>
        }
      </li>
    </ol>
  </div></p>
</div>

Y listos!

Un par de consideraciones sobre el uso de sesión:

  1. No es persistente. Es decir, cuando el usuario cierre la ventana del navegador y vuelva más tarde a nuestra web, habremos perdido los datos. Si queremos hacer los datos persistentes hay que guardarlos en algún almacenamiento persistente, esto es… una base de datos 😛 (aunque podría ser un fichero en el servidor también). 
  2. Se crea incluso si el usuario no está autenticado. Si queremos que tan solo los usuarios autenticados tengan catálogos, podemos crear el catálogo justo cuando autenticamos al usuario (y por supuesto tener protegidos con [Authorize] las acciones correspondientes de los controladores que acceden a la sesión). 

Por lo tanto, la sesión nos evita el uso de una BBDD **solo mientras el usuario navega por nuestra aplicación**, pero si queremos que sea persistente deberemos usar un almacenamiento externo que persista (o sea, una BBDD).

**Opcion 2: Serializar los datos en cada petición**

De acuerdo, la sesión la podemos usar pero es un recurso digamos… _caro_. Tiene ciertas implicaciones en escalabilidad y tolerancia a fallos a
  
sí que quizá no queremos o podemos usarla. Hay alguna manera de mantener el estado como si tuviéramos sesión? Pues sí, _serializar_ los datos de la sesión en cada petición y colocarlas en un campo hidden. ¿Te suena a algo esto? ¿Cómo? ¿Quien ha dicho viewstate? Premio!

Vamos a ver una prueba de concepto de esto. Para empezar necesitamos una clase que dado un objeto nos devuelva su representación en Base64 y vicerversa:

<div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
  <div style="background: #ddd; max-height: 300px; overflow: auto">
    <ol style="padding-bottom: 0px; margin: 0px 0px 0px 2.5em; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
      <li>
        <span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> <span style="color: #2b91af">ObjectToBase64</span>
      </li>
      <li style="background: #f3f3f3">
        {
      </li>
      <li>
        &#160;
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; <span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">string</span> ToBase64(<span style="color: #0000ff">object</span> source)
      </li>
      <li>
        &#160;&#160;&#160; {
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #0000ff">var</span> base64 = <span style="color: #0000ff">string</span>.Empty;
      </li>
      <li>
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #0000ff">if</span> (source != <span style="color: #0000ff">null</span>)
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; {
      </li>
      <li>
        &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #0000ff">using</span> (<span style="color: #0000ff">var</span> ms = <span style="color: #0000ff">new</span> <span style="color: #2b91af">MemoryStream</span>())
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; {
      </li>
      <li>
        &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #0000ff">var</span> formatter = <span style="color: #0000ff">new</span> <span style="color: #2b91af">BinaryFormatter</span>();
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; formatter.Serialize(ms, source);
      </li>
      <li>
        &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #0000ff">var</span> buffer = ms.ToArray();
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; base64 = <span style="color: #2b91af">Convert</span>.ToBase64String(buffer);
      </li>
      <li>
        &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
      </li>
      <li>
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #0000ff">return</span> base64;
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; }
      </li>
      <li>
        &#160;
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; <span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">object</span> FromBase64 (<span style="color: #0000ff">string</span> base64)
      </li>
      <li>
        &#160;&#160;&#160; {
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #0000ff">if</span> (<span style="color: #0000ff">string</span>.IsNullOrEmpty(base64)) <span style="color: #0000ff">return</span> <span style="color: #0000ff">null</span>;
      </li>
      <li>
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #0000ff">var</span> buffer = <span style="color: #2b91af">Convert</span>.FromBase64String(base64);
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #0000ff">using</span> (<span style="color: #0000ff">var</span> ms = <span style="color: #0000ff">new</span> <span style="color: #2b91af">MemoryStream</span>(buffer))
      </li>
      <li>
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; {
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #0000ff">var</span> formatter = <span style="color: #0000ff">new</span> <span style="color: #2b91af">BinaryFormatter</span>();
      </li>
      <li>
        &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #0000ff">return</span> formatter.Deserialize(ms);
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
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

La idea ahora es que:

  1. Cada acción tiene un parámetro adicional (p.ej. llamado _viewstate). 
  2. Este parámetro debe ser pasado en todas las llamadas, ya sea via GET o via POST. 

Así, si p.ej. en una vista tenemos un formulario, podemos hacer:

<div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
  <div style="background: #ddd; max-height: 300px; overflow: auto">
    <ol style="padding-bottom: 0px; margin: 0px 0px 0px 2em; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
      <li>
        <span style="background: #ffff00">@</span><span style="color: #0000ff">if</span> (ViewBag.ViewState != <span style="color: #0000ff">null</span>)
      </li>
      <li style="background: #f3f3f3">
        {
      </li>
      <li>
        &#160;&#160;&#160; <span style="color: #0000ff"><</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="hidden"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="_viewstate"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="</span><span style="background: #ffff00">@</span><span style="color: #0000ff">ViewBag.ViewState"/></span>
      </li>
      <li style="background: #f3f3f3">
        }
      </li>
    </ol>
  </div></p>
</div>

Y si generamos un enlace:

<span style="background: #ffff00">@</span>Html.ActionLink(<span style="color: #a31515">"Nuevo"</span>, <span style="color: #a31515">"Nuevo"</span>, <span style="color: #0000ff">new</span> {_viewstate = ViewBag.ViewState}) 

Del mismo modo en el caso de una redirección de un controlador a otro:

<span style="color: #0000ff">return</span> RedirectToAction(<span style="color: #a31515">"Index"</span>, <span style="color: #0000ff">new</span> { _viewstate = <span style="color: #2b91af">ObjectToBase64</span>.ToBase64(catalogo) });

Bueno… creo que la idea se entiende, no? 😉

En el fondo lo que estamos haciendo es serializar la “sesión” cada vez y mandarla arriba y abajo. Hacerlo via GET (como en el ActionLink o en RedirectToAction) hace que esta sesión sea visible en la URL:

[<img style="background-image: none; border-right-width: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_7A66A6F9.png" width="644" height="28" />][1]

Lo ideal es usar siempre POST (que es lo que hace Webforms). Esta técnica se puede mejorar bastante
  
, pero bueno… como prueba de concepto no está mal. Si alguna vez queréis usar esta técnica echad un vistazo al post del Maestro: <http://www.variablenotfound.com/2010/07/un-viewstate-en-aspnet-mvc.html>

He de decir que a mi esta técnica no me gusta nada, salvo que no sea para casos muy, muy, muy puntuales. Y por supuesto tiene el mismo “problema” que la sesión: no es persistente.

Y así llegamos a la opción 3…

**Opción 3 – LocalStorage**

Bueno… en la Opción 1 se trata de mantener el estado en el servidor y la Opción 2 mantiene el estado en el cliente pasándolo a través de campos hidden o en la URL. Ahora toca la tercera opción: mantener el estado _en el cliente_. No solo eso, sino que además lo haremos persistente. ¿Y eso como? Pues… usaremos un almacenamiento persistente… En este caso el Local Storage.

Local Storage es básicamente un almacén de objetos en el navegador. Llámalo base de datos si quieres, pero ten presente que no hay tablas, ni filas, ni columnas. Se trataría más bien de un diccionario (clave, valor). ¿Y qué guardas? Pues cadenas. Simple y llanamente cadenas. 

La idea es la siguiente:

  1. El catálogo se guarda en el local storage del cliente 
  2. El cliente añade el producto en el almacén 

Todo se realiza en cliente, no hay comunicación en el servidor. Evidentemente en una aplicación real antes de añadir el producto en cliente, este preguntaría al servidor (mediante una llamada ajax) si puede añadirlo. Pero en nuestra prueba de concepto no lo haremos… 😛

La prueba de concepto es muy, sencilla, en la página de Index accedemos al localStorage, lo recorremos y ponemos su contenido en una lista:

<div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
  <div style="background: #ddd; max-height: 300px; overflow: auto">
    <ol style="padding-bottom: 0px; margin: 0px 0px 0px 2.5em; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
      <li>
        <span style="color: #0000ff"><</span><span style="color: #800000">h2</span><span style="color: #0000ff">></span>Index<span style="color: #0000ff"></</span><span style="color: #800000">h2</span><span style="color: #0000ff">></span>
      </li>
      <li style="background: #f3f3f3">
        <span style="color: #0000ff"><</span><span style="color: #800000">script</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text/javascript"></span>
      </li>
      <li>
        &#160;&#160;&#160; $(document).ready(<span style="color: #0000ff">function</span> () {
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #0000ff">for</span> (<span style="color: #0000ff">var</span> x = 0; x <= localStorage.length &#8211; 1; x++) {
      </li>
      <li>
        &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #0000ff">var</span> key = localStorage.key(x);
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #0000ff">var</span> value = JSON.parse(localStorage.getItem(key));
      </li>
      <li>
        &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; $(<span style="color: #800000">"#lstCatalog"</span>).append(
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; $(<span style="color: #800000">"<li>"</span>).append(value.nombre + <span style="color: #800000">" "</span> + value.precio)
      </li>
      <li>
        &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; );
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
      </li>
      <li>
        &#160;&#160;&#160; });
      </li>
      <li style="background: #f3f3f3">
        &#160;
      </li>
      <li>
        &#160;
      </li>
      <li style="background: #f3f3f3">
        <span style="color: #0000ff"></</span><span style="color: #800000">script</span><span style="color: #0000ff">></span>
      </li>
      <li>
        &#160;
      </li>
      <li style="background: #f3f3f3">
        <span style="color: #0000ff"><</span><span style="color: #800000">ul</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="lstCatalog"></span>
      </li>
      <li>
        &#160;&#160;&#160;&#160;
      </li>
      <li style="background: #f3f3f3">
        <span style="color: #0000ff"></</span><span style="color: #800000">ul</span><span style="color: #0000ff">></span>
      </li>
    </ol>
  </div></p>
</div>

Fijaos en el uso de JSON.parse para pasar la cadena JSON que guardamos en el localStorage a un objeto javascript. El resto del código se entiende bastante bien, ¿no?

Y la vista de alta? Bueno, pues se trata de un formulario de alta normal, salvo que el botón de enviar no es un botón de submit sino un botón normal. Y luego tenemos el siguiente código js:

<div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
  <div style="background: #ddd; max-height: 300px; overflow: auto">
    <ol style="padding-bottom: 0px; margin: 0px 0px 0px 2.5em; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
      <li>
        <span style="color: #0000ff"><</span><span style="color: #800000">script</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text/javascript"></span>
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; $(document).ready(<span style="color: #0000ff">function</span> () {
      </li>
      <li>
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; $(<span style="color: #800000">"#btnAdd"</span>).click(<span style="color: #0000ff">function</span> () {
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #0000ff">var</span> nombre = $(<span style="color: #800000">"#Nombre"</span>).val();
      </li>
      <li>
        &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #0000ff">var</span> precio = $(<span style="color: #800000">"#Precio"</span>).val();
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; localStorage.setItem(nombre, JSON.stringify({ nombre: nombre, precio: precio }));
      </li>
      <li>
        &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; location.href = <span style="color: #800000">"</span><span style="background: #ffff00; color: #800000">@</span>Url.Action(<span style="color: #a31515">"Index"</span>)<span style="color: #800000">"</span>;
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; });
      </li>
      <li>
        &#160;&#160;&#160; });
      </li>
      <li style="background: #f3f3f3">
        <span style="color: #0000ff"></</span><span style="color: #800000">script</span><span style="color: #0000ff">></span>
      </li>
    </ol>
  </div></p>
</div>

(btnAdd es el ID del botón del formulario).

Es simple: accedemos al valor del input “Nombre” y del “Precio”. Construimos un objeto javascript, lo pasamos a cadena con JSON.stringify y lo guardamos en el localStorage. Para la clave usamos el nombre del producto. Es aquí, donde antes de insertar el elemento en el localStorage, nos iriamos al servidor via Ajax para confirmar que podemos añadirlo.

Al final nos volvemos a la página de Index.

Y ya hemos terminado. Como prueba de concepto, no está mal: 5 minutos para tener una lista de productos en el cliente. El servidor ni se ha enterado, este es el código del controlador:

<div style="bo
rder-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
  </p> 
  
  <div style="background: #ddd; max-height: 300px; overflow: auto">
    <ol style="padding-bottom: 0px; margin: 0px 0px 0px 2.5em; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
      <li>
        <span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> <span style="color: #2b91af">LocalStorageController</span> : <span style="color: #2b91af">Controller</span>
      </li>
      <li style="background: #f3f3f3">
        {
      </li>
      <li>
        &#160;&#160;&#160; <span style="color: #0000ff">public</span> <span style="color: #2b91af">ActionResult</span> Index()
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; {
      </li>
      <li>
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #0000ff">return</span> View();
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; }
      </li>
      <li>
        &#160;
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; <span style="color: #0000ff">public</span> <span style="color: #2b91af">ActionResult</span> Nuevo()
      </li>
      <li>
        &#160;&#160;&#160; {
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #0000ff">return</span> View();
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

Por supuesto falta soporte para eliminar un elemento del localStorage, pero eso es trivial usando los métodos clear() (borra todo el localStorage) y removeItem (borra un elemento).

**Nota:** localStorage **es persistente**. Es decir, entre ejecución y ejecución de la aplicación (cerrar el browser y abrirlo de nuevo mañana) los datos _se mantienen_. Viene a ser como una cookie pero con esteroides. Si quieres usar un localStorage “no persistente” (que dure solo mientras la ventana está abierta” existe el **sessionStorage**. Además recuerda que localStorage, al estar en el cliente, es totalmente _hackeable_.

En resumen: Realizar un catálogo en MVC sin base de datos en el servidor es posible si queremos que los datos no sean persistentes. Pero si queremos que esos sean persistentes (que el usuario pueda cerrar el navegador, abrirlo y continuar viendo sus datos) debe usarse una BBDD (o algún otro almacenamiento persistente) sí o sí. Hasta antes de HTML5 este almacenamiento persistente tenía que estar en el servidor. Con HTML5 y localStorage puede estar en el cliente.

**Nota final:**

Además de esas tres opciones se me ocurren al menos DOS más para realizar lo mismo:

  1. Usar cookies (seria la versión pre-html5 de usar el localStorage, pero hay serias limitaciones en el tamaño de los datos a guardar).
  2. Usar indexedDB (sería parecido en filosofía a usar el localStorage).

**Algunos links**:

  1. WebStorage (localStorage y sessionStorage): <http://dev.w3.org/html5/webstorage/>
  2. Sesión en ASP.NET MVC (en mi blog): <http://geeks.ms/blogs/etomas/archive/2010/06/30/asp-net-mvc-q-amp-a-c-243-mo-usar-la-sesi-243-n.aspx>
  3. IndexedDB: <http://www.w3.org/TR/IndexedDB/>

Espero que el ejercicio os haya sido de interés! 😉

PD: Os dejo un zip con el código de las opciones 2 y 3 en [https://skydrive.live.com/redir.aspx?cid=6521c259e9b1bec6&resid=6521C259E9B1BEC6!233&parid=6521C259E9B1BEC6!167&authkey=!AHLp6PeH4Uw1hEg][2]

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_70622B9B.png
 [2]: https://skydrive.live.com/redir.aspx?cid=6521c259e9b1bec6&resid=6521C259E9B1BEC6!233&parid=6521C259E9B1BEC6!167&authkey=!AHLp6PeH4Uw1hEg "https://skydrive.live.com/redir.aspx?cid=6521c259e9b1bec6&resid=6521C259E9B1BEC6!233&parid=6521C259E9B1BEC6!167&authkey=!AHLp6PeH4Uw1hEg"