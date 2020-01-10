---
title: ¿Están tus servicios REST en otro servidor?

author: eiximenis

date: 2013-01-22T13:08:59+00:00
geeks_url: /?p=1627
geeks_visits:
  - 5658
geeks_ms_views:
  - 2129
categories:
  - Uncategorized

---
Muy buenas, en este post vamos a hablar de lo que ocurre si los servicios REST de tu aplicación están en otro servidor distinto al de tu aplicación web… Terminaremos hablando de CORS, pero antes lo haremos de JSONP y empezaremos por el…

**… Orígen**

No, no me refiero a la onírica película con Di Caprio, aunque muchas veces el desarrollo web se parezca a una pesadilla, si no a lo que en el mundo web entendemos como el _origen_ de una web.

Es un concepto muy sencillo pero que debemos tener siempre presente: protocolo, host y número de puerto conforman un orígen. Así las urls

  * <http://foo/bar> y <http://foo/baz> pertenecen al mismo orígen (mismo protocolo (http), mismo puerto (80) y mismo servidor (foo). 
  * <http://foo/bar> y <https://foo/baz> pertenecen a orígenes distintos (el protocolo es distinto). 
  * <http://foo/bar> y <http://m.foo/bar> pertenecen a orígenes distintos (el host es distinto) 
  * <http://foo:8080/bar> y <http://foo/bar> pertenecen a orígenes distintos (el puerto es distinto) 

¿Es simple no? Bueno pues ahora la regla de oro: **Los navegadores tienen prohibido procesar una llamada Ajax (con XmlHttpRequest) desde una página que esté en un orígen hacia una URL que sea de otro origen**.

Punto.

Vale, que nos impidan hacer peticiones ajax a otro dominio es una muy buena medida de seguridad, pero a veces es una necesidad legítima: Imagina que tienes la API REST de tu aplicación publicada en <http://api.foo.com> y tu aplicación web en <http://foo.com>. Pues si desde tu web quieres hacer una petición ajax a alguno de tus servicios: mala suerte.

Para hacer algunas demos voy a crear una solución de vs2012 con **dos** proyectos web:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_75088397.png" width="240" height="89" />][1]

Uno será mi aplicación web y el otro será mi API. El proyecto WebApi (que yo he llamado CORSDemo) tan solo tiene un controlador, que responde a las peticiones de tipo /Beer/id:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">BeersController</span> : <span style="color: #4ec9b0">ApiController</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #4ec9b0">Beer</span> <span style="color: white">Get</span>(<span style="color: #569cd6">int</span> <span style="color: white">id</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">Beer</span> {<span style="color: white">Name</span> <span style="color: #b4b4b4">=</span> <span style="color: #d69d85">"Cerveza "</span> <span style="color: #b4b4b4">+</span> <span style="color: white">id</span>, <span style="color: white">Id</span> <span style="color: #b4b4b4">=</span> <span style="color: white">id</span>};
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Por otro lado la aplicación web (que he llamado CORSDemo.Web) tiene un solo controlador (Home) que devuelve una vista. Dicha vista intenta hacer una llamada Ajax al servicio REST:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: gray"><</span><span style="color: #569cd6">h2</span><span style="color: gray">></span>Index<span style="color: gray"></</span><span style="color: #569cd6">h2</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    <span style="background: #ffffb3; color: black">@section scripts</span>
  </p>
  
  <p style="margin: 0px">
    <span style="background: #ffffb3; color: black">{</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">script</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #b4b4b4">(</span><span style="color: #569cd6">function</span><span style="color: #b4b4b4">()</span> <span style="color: #b4b4b4">{</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">xhr</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">new</span> <span style="color: white">XMLHttpRequest</span><span style="color: #b4b4b4">();</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">console</span><span style="color: #b4b4b4">.</span><span style="color: white">log</span><span style="color: #b4b4b4">(</span><span style="color: #d69d85">&#8216;Invocando servicio REST&#8217;</span><span style="color: #b4b4b4">);</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">xhr</span><span style="color: #b4b4b4">.</span><span style="color: white">open</span><span style="color: #b4b4b4">(</span><span style="color: #d69d85">&#8216;GET&#8217;</span><span style="color: #b4b4b4">,</span> <span style="color: #d69d85">&#8216;http://localhost:2614/Beers/10&#8217;</span><span style="color: #b4b4b4">,</span> <span style="color: #569cd6">true</span><span style="color: #b4b4b4">);</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">xhr</span><span style="color: #b4b4b4">.</span><span style="color: white">setRequestHeader</span><span style="color: #b4b4b4">(</span><span style="color: #d69d85">"Accept"</span><span style="color: #b4b4b4">,</span> <span style="color: #d69d85">"application/json"</span><span style="color: #b4b4b4">);</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">xhr</span><span style="color: #b4b4b4">.</span><span style="color: white">addEventListener</span><span style="color: #b4b4b4">(</span><span style="color: #d69d85">&#8216;readystatechange&#8217;</span><span style="color: #b4b4b4">,</span> <span style="color: #569cd6">function</span> <span style="color: #b4b4b4">(</span><span style="color: white">e</span><span style="color: #b4b4b4">)</span> <span style="color: #b4b4b4">{</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">console</span><span style="color: #b4b4b4">.</span><span style="color: white">log</span><span style="color: #b4b4b4">(</span><span style="color: #d69d85">&#8216;readyState: &#8216;</span> <span style="color: #b4b4b4">+</span> <span style="color: white">xhr</span><span style="color: #b4b4b4">.</span><span style="color: white">readyState</span><span style="color: #b4b4b4">)</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">console</span><span style="color: #b4b4b4"
>.</span><span style="color: white">log</span><span style="color: #b4b4b4">(</span><span style="color: #d69d85">&#8216;status: &#8216;</span> <span style="color: #b4b4b4">+</span> <span style="color: white">xhr</span><span style="color: #b4b4b4">.</span><span style="color: white">status</span><span style="color: #b4b4b4">);</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">console</span><span style="color: #b4b4b4">.</span><span style="color: white">log</span><span style="color: #b4b4b4">(</span><span style="color: #d69d85">&#8216;response: &#8216;</span> <span style="color: #b4b4b4">+</span> <span style="color: white">xhr</span><span style="color: #b4b4b4">.</span><span style="color: white">responseText</span><span style="color: #b4b4b4">);</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #b4b4b4">});</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">xhr</span><span style="color: #b4b4b4">.</span><span style="color: white">send</span><span style="color: #b4b4b4">();</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #b4b4b4">})();</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"></</span><span style="color: #569cd6">script</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="background: #ffffb3; color: black">}</span>
  </p></p>
</div>

Si pongo en marcha el proyecto, efectivamente en mi IIS Express tengo dos aplicaciones web:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_2B29FBE7.png" width="244" height="74" />][2]

En mi caso la aplicación WebApi está en localhost:2614 y la aplicación web está en localhost:2628. Y si navego a localhost:2628 veo lo que ya me esperaba:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_0F38C6EF.png" width="244" height="130" />][3]

El navegador me muestra el error que no puede realizar la llamada ya que el servicio REST está en otro orígen.

**Rompiendo la barrera – jsonp**

Por suerte (y por desgracia también, todo tiene las dos caras de la moneda), hay muchas cabezas pensantes por ahí y algunas de ellas se dedicaron a ver si existía alguna posible manera de saltarse esta medida de seguridad. Y dieron con una. Ciertamente no abrieron un boquete en la muralla de seguridad, pero sí una brecha y durante vario tiempo nos hemos estado aprovechando de ella. Esta brecha es la técnica conocida como jsonp. Veamos muy brevemente en que consiste…

El objetivo final es conseguir llamar al servicio REST que tenemos y recuperar los datos. Eso debemos hacerlo de forma asíncrona al igual que hace XMLHttpRequest, que ya hemos visto que no podemos usar.

La técnica de jsonp es muy simple, pero requiere eso sí que los servicios REST devuelvan json (no sirve si devuelven algún otro tipo de datos como XML). 

Consiste básicamente en sustuir la llamada AJAX por un tag <script>. El tag <script> permite sin ningún problema incluir scripts de otros orígenes (si no, no podríamos usar CDNs p. ej.). Asi en nuestro caso vamos a añadir un tag <script> como el siguiente:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: gray"><</span><span style="color: #569cd6">script</span> <span style="color: #9cdcfe">src</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"http://localhost:2614/Beers/10"</span><span style="color: gray">></</span><span style="color: #569cd6">script</span><span style="color: gray">></span>
  </p></p>
</div>

Ahora el navegador realiza la llamada web y obtiene el JSON pero… por supuesto ahora tenemos un error de javascript:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_371BC64E.png" width="244" height="165" />][4]

Eso es debido a que el navegador está intentando interpretar el JSON como si fuese código javascript y por supuesto <font face="Courier New">{"Name":"Cerveza 10","Id":10}</font> no es un código javascript válido. No lo es, pero le falta muy, muy poco para serlo.

Ahora toca que el servicio REST colabore un poco. Que nos devuelva los datos directamente en JSON no nos sirve ya que hemos visto que el navegador no puede interpretarlos. Pero… y si en lugar de devolvernos los datos en JSON el servicio REST nos devuelve algo como:

<font face="Courier New">func_callback({"Name":"Cerveza 10","Id":10});</font>

Ah! Esto sí que es javascript válido. A este código lo llamamos el código jsonp.

Tan solo falta que func_callback esté definida y de eso ya se encargaría la aplicación web.

Veamos como modificar el servicio en WebApi para soportar jsonp. Para ello nos basaremos en la querystring.

**Soportando JSONP en WebApi**

Que yo sepa WebApi NO tiene soporte directo para jsonp. Por suerte añadirlo es trivial. Basta con usar un MediaTypeFormatter nuevo:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">JsonpMediaFormatter</span> : <span style="color: #4ec9b0">JsonMediaTypeFormatter</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: white">JsonpMediaFormatter</span>()
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; : <span style="color: #569cd6">base</span>()
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">SupportedMediaTypes</span><span style="color: #b4b4b4">.</span><span style="color: white">Add</span>(<span style="color: white">DefaultMediaType</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">SupportedMediaTypes</span><span style="color: #b4b4b4">.</span><span style="color: white">Add</span>(<span style="color: #569cd6">new</span> <span style="color: #4ec9b0">MediaTypeHeaderValue</span>(<span style="color: #d69d85">"text/javascript"</span>));
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">MediaTypeMappings</span><span style="color: #b4b4b4">.</span><span style="color: white">Add</span>(<span style="color: #569cd6">new</span> <span style="color: #4ec9b0">QueryStringMapping</span>(<br /> <span style="color: #d69d85">"jsonp"</span>, <span style="color: #d69d85">"true"</span>,<span style="color: white">DefaultMediaType</span>));
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">override</span> <span style="color: #4ec9b0">Task</span> <span style="color: white">WriteToStreamAsync</span>(<span style="color: #4ec9b0">Type</span> <span style="color: white">type</span>, <span style="color: #569cd6">object</span> <span style="color: white">value</span>, <span style="color: #4ec9b0">Stream</span> <span style="color: white">writeStream</span>, <span style="color: white">System</span><span style="color: #b4b4b4">.</span><span style="color: white">Net</span><span style="color: #b4b4b4">.</span><span style="color: white">Http</span><span style="color: #b4b4b4">.</span><span style="color: #4ec9b0">HttpContent</span> <span style="color: white">content</span>, <span style="color: #4ec9b0">TransportContext</span> <span style="color: white">transportContext</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">callback</span> <span style="color: #b4b4b4">=</span> <span style="color: white">GetJsonpCallback</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">if</span> (<span style="color: #569cd6">string</span><span style="color: #b4b4b4">.</span><span style="color: white">IsNullOrEmpty</span>(<span style="color: white">callback</span>))
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: #569cd6">base</span><span style="color: #b4b4b4">.</span><span style="color: white">WriteToStreamAsync</span>(<span style="color: white">type</span>, <span style="color: white">value</span>, <span style="color: white">writeStream</span>, <span style="color: white">content</span>, <span style="color: white">transportContext</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #4ec9b0">Encoding</span> <span style="color: white">encoding</span> <span style="color: #b4b4b4">=</span> <span style="color: white">SelectCharacterEncoding</span>(<span style="color: white">content</span><span style="color: #b4b4b4">.</span><span style="color: white">Headers</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: #4ec9b0">Task</span><span style="color: #b4b4b4">.</span><span style="color: white">Factory</span><span style="color: #b4b4b4">.</span><span style="color: white">StartNew</span>(() <span style="color: #b4b4b4">=></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">bytes</span> <span style="color: #b4b4b4">=</span> <span style="color: white">encoding</span><span style="color: #b4b4b4">.</span><span style="color: white">GetBytes</span>(<span style="color: #569cd6">string</span><span style="color: #b4b4b4">.</span><span style="color: white">Format</span>(<span style="color: #d69d85">"</span><span style="color: #80ff80">{0}</span><span style="color: #d69d85">("</span>, <span style="color: white">callback</span>));
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">writeStream</span><span style="color: #b4b4b4">.</span><span style="color: white">Write</span>(<span style="color: white">bytes</span>, <span style="color: #b5cea8"></span>, <span style="color: white">bytes</span><span style="color: #b4b4b4">.</span><span style="color: white">Length</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; })<span style="color: #b4b4b4">.</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">ContinueWith</span>(<span style="color: white">task</span> <span style="color: #b4b4b4">=></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">base</span><span style="color: #b4b4b4">.</span><span style="color: white">WriteToStreamAsync</span>(<span style="color: white">type</span>, <span style="color: white">value</span>, <span style="color: white">writeStream</span>, <span style="color: white">content</span>, <span style="color: white">transportContext</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; })<span style="color: #b4b4b4">.</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">ContinueWith</span>(<span style="color: white">task</span> <span style="color: #b4b4b4">=></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">bytes</span> <span style="color: #b4b4b4">=</span> <span style="color: white">encoding</span><span style="color: #b4b4b4">.</span><span style="color: white">GetBytes</span>(<span style="color: #d69d85">");"</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">writeStream</span><span style="color: #b4b4b4">.</span><span style="color: white">Write</span>(<span style="color: white">bytes</span>, <span style="color: #b5cea8"></span>, <span style="color: white">bytes</span><span style="color: #b4b4b4">.</span><span style="color: white">Length</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; });
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">protected</span> <span style="color: #569cd6">string</span> <span style="color: white">GetJsonpCallback</span>()
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">if</span> (<span style="color: #4ec9b0">HttpContext</span><span style="color: #b4b4b4">.</span><span style="color: white">Current</span><span style="color: #b4b4b4">.</span><span style="color: white">Request</span><span style="color: #b4b4b4">.</span><span style="color: white">HttpMethod</span> <span style="color: #b4b4b4">!=</span> <span style="color: #d69d85">"GET"</span>) <span style="color: #569cd6">return</span> <span style="color: #569cd6">null</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">if</span> (<span style="color: #4ec9b0">HttpContext</span><span style="color: #b4b4b4">.</span><span style="color: white">Current</span><span style="color: #b4b4b4">.</span><span style="color: white">Request</span><span style="color: #b4b4b4">.</span><span style="color: white">QueryString</span>[<sp an style="color: #d69d85">"jsonp"</span>] <span style="color: #b4b4b4">!=</span> <span style="color: #d69d85">"true"</span>) <span style="color: #569cd6">return</span> <span style="color: #569cd6">null</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: #4ec9b0">HttpContext</span><span style="color: #b4b4b4">.</span><span style="color: white">Current</span><span style="color: #b4b4b4">.</span><span style="color: white">Request</span><span style="color: #b4b4b4">.</span><span style="color: white">QueryString</span>[<span style="color: #d69d85">"callback"</span>] <span style="color: #b4b4b4">??</span> <span style="color: #d69d85">"func_callback"</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Para registrar este MediaTypeFormatter debemos añadir la siguiente línea en el Application_Start:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #4ec9b0">GlobalConfiguration</span><span style="color: #b4b4b4">.</span><span style="color: white">Configuration</span><span style="color: #b4b4b4">.</span><span style="color: white">Formatters</span><span style="color: #b4b4b4">.</span><span style="color: white">Add</span>(<span style="color: #569cd6">new</span> <span style="color: #4ec9b0">JsonpMediaFormatter</span>());
  </p></p>
</div>

Ahora nuestro JsonpMediaFormatter actuará si la petición tiene un parámetro querystring llamado jsonp y cuyo valor sea true. Además admite otro parámetro llamado callback con el valor de la función callback. Por lo tanto modificamos ahora el tag <script> para que pase esos dos parámetros y también definimos antes la función show_beer:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: gray"><</span><span style="color: #569cd6">script</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">function</span> <span style="color: white">show_beer</span><span style="color: #b4b4b4">(</span><span style="color: white">data</span><span style="color: #b4b4b4">)</span> <span style="color: #b4b4b4">{</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">alert</span><span style="color: #b4b4b4">(</span><span style="color: white">data</span><span style="color: #b4b4b4">.</span><span style="color: white">Name</span><span style="color: #b4b4b4">);</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #b4b4b4">}</span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray"></</span><span style="color: #569cd6">script</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray"><</span><span style="color: #569cd6">script</span> <span style="color: #9cdcfe">src</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"http://localhost:2614/Beers/10?jsonp=true&callback=show_beer"</span><span style="color: gray">></</span><span style="color: #569cd6">script</span><span style="color: gray">></span>
  </p></p>
</div>

¡Y ya hemos terminado! Si ahora ejecutamos la página vemos que efectivamente nos hemos saltado la restricción de orígen:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_3AD96B1E.png" width="244" height="131" />][5]

¿Por qué digo que JSONP es una brecha en lugar de un agujero en la seguridad? Muy simple… porque está basado en el tag <script> lo que implica que tan solo funciona para el verbo http GET.

Así, aunque JSONP es un parche que nos puede sacar de muchos apuros, era evidente que necesitábamos una manera **segura** de poder llamar a servicios REST que estuviesen en otro dominio… y la W3C se puso manos a la obra y definió CORS.

**CORS**

Las ventajas de CORS sobre JSONP son enormes: CORS funciona para todos los verbos HTTP, permite usar XMLHttpRequest así que no tenemos que andar con trapicheos como en JSONP y además es un estándard y no una técnica salida de una mente calenturienta.

Por supuesto tiene sus inconvenientes: tiene que estar soportado por el servidor **y por el navegador**. Si os vais a [http://caniuse.com/#search=CORS][6] podeis ver como p.ej. **IE NO** **SOPORTA CORS hasta la versión 10** (En la versión 8 y 9 soporta un pseudo-CORS a través del objeto XDomainRequest). Es la historia de siempre… 🙁

CORS se basa en las cabeceras HTTP. Básicamente la idea es que el navegador envía una petición con la cabecera http “Origin” que contiene el origen de la aplicación web. El servidor recibe esta petición y si admite dicho orígen devuelve en la respuesta la cabecera “Access-Control-Allow-Origin” con el nombre de los orígenes admitidos.

Volvamos de nuevo al código original que teníamos antes de ver jsonp. Si abro una ventana de Chrome y navego a la aplicación web, donde se hace la llamada con XMLHttpRequest y miro las cabeceras enviadas:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_5EB21CAB.png" width="244" height="73" />][7]

Fijaos como el navegador envía la cabecera “Origin”. Por lo tanto Chrome ya está intentando iniciar una negociación CORS, pero como el servidor no le responde con la cabecera Access-Control-Allow-Origin Chrome no procesa la petición y no podemos acceder a la respuesta.

En este punto voy a dejar una cosa bien clara: La petición es enviada por el navegador y por lo tanto es RECIBIDA por el servidor. Lo podéis comprobar poniendo un Breakpoint en el controlador de WebApi y veréis que se llega a él. Pero la respuesta NO ES PROCESADA por el navegador. Otra forma de verlo es usando fiddler:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_7AFF5ECB.png" width="244" height="133" />][8]

Como podemos ver hay una petición (con su cabecera Origin) y una respuesta. Solo que el navegador nos ignora la respuesta debido a que no hay la cabecera CORS Access-Control-Allow-Origin.

**Soporte para CORS en WebApi**

De nuevo, que yo sepa, no hay soporte out-of-the-box en WebApi para CORS, aunque por suerte añadir uno básico es trivial. Si para JSONP usábamos un MediaTypeFormatter, para CORS usaremos un Message Handler para conseguirlo:

> **Nota**: El Message Handler aquí mostrado implementa una parte muy
  
> pequeña de CORS. Está pensado a modo de información y no para poner en producción. Solo por citar una de sus limitaciones no está preparado para lidiar con las peticiones “preflight” de CORS que se dan en según que escenarios y que no tratamos en este post.

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">CorsMessageHandler</span> : <span style="color: #4ec9b0">DelegatingHandler</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">protected</span> <span style="color: #569cd6">async</span> <span style="color: #569cd6">override</span> <span style="color: white">System</span><span style="color: #b4b4b4">.</span><span style="color: white">Threading</span><span style="color: #b4b4b4">.</span><span style="color: white">Tasks</span><span style="color: #b4b4b4">.</span><span style="color: #4ec9b0">Task</span><span style="color: #b4b4b4"><</span><span style="color: #4ec9b0">HttpResponseMessage</span><span style="color: #b4b4b4">></span> <span style="color: white">SendAsync</span>(<span style="color: #4ec9b0">HttpRequestMessage</span> <span style="color: white">request</span>, <span style="color: white">System</span><span style="color: #b4b4b4">.</span><span style="color: white">Threading</span><span style="color: #b4b4b4">.</span><span style="color: #4ec9b0">CancellationToken</span> <span style="color: white">cancellationToken</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">if</span> (<span style="color: white">request</span><span style="color: #b4b4b4">.</span><span style="color: white">Headers</span><span style="color: #b4b4b4">.</span><span style="color: white">Contains</span>(<span style="color: #d69d85">"Origin"</span>))
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">response</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">await</span> <span style="color: #569cd6">base</span><span style="color: #b4b4b4">.</span><span style="color: white">SendAsync</span>(<span style="color: white">request</span>, <span style="color: white">cancellationToken</span>);&#160;&#160;&#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">response</span><span style="color: #b4b4b4">.</span><span style="color: white">Headers</span><span style="color: #b4b4b4">.</span><span style="color: white">Add</span>(<span style="color: #d69d85">"Access-Control-Allow-Origin"</span>,
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">request</span><span style="color: #b4b4b4">.</span><span style="color: white">Headers</span><span style="color: #b4b4b4">.</span><span style="color: white">GetValues</span>(<span style="color: #d69d85">"Origin"</span>));
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: white">response</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: #569cd6">await</span> <span style="color: #569cd6">base</span><span style="color: #b4b4b4">.</span><span style="color: white">SendAsync</span>(<span style="color: white">request</span>, <span style="color: white">cancellationToken</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Este MessageHandler es muy sencillo: si la petición contiene la cabecera “Origin” añade la cabecera “Access-Control-Allow-Origin” con el mismo valor que la cabecera Origin.

Tenemos que registrar este Message Handler, en el Application_Start:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #4ec9b0">GlobalConfiguration</span><span style="color: #b4b4b4">.</span><span style="color: white">Configuration</span><span style="color: #b4b4b4">.</span><span style="color: white">MessageHandlers</span><span style="color: #b4b4b4">.</span><span style="color: white">Add</span>(<span style="color: #569cd6">new</span> <span style="color: #4ec9b0">CorsMessageHandler</span>());
  </p></p>
</div>

¡Listos! Hemos terminado. Si ahora navegamos de nuevo a la aplicación web vemos que la llamada Ajax se efectúa sin problemas:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_57168771.png" width="244" height="130" />][9]

Y si miramos en la pestaña network veremos la cabecera Access-Control-Allow-Origin que ahora envía el servidor como respuesta:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_457676D9.png" width="244" height="115" />][10]

Para más información, <a href="http://www.w3.org/TR/cors/" target="_blank" rel="noopener noreferrer">aquí tenéis la especificación de CORS del W3C</a>.

Si buscáis un soporte de CORS realmente completo para WebAPi echad un vistazo a este post de brocakllen: [http://brockallen.com/2012/06/28/cors-support-in-webapi-mvc-and-iis-with-thinktecture-identitymodel/][11]

Saludos!

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_16682934.png
 [2]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_7E81420D.png
 [3]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_09CA564B.png
 [4]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_58E79EDF.png
 [5]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_0E30B145.png
 [6]: http://caniuse.com/#search=CORS "http://caniuse.com/#search=CORS"
 [7]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_0730FEC0.png
 [8]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_7CB02A9F.png
 [9]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_3918797D.png
 [10]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_3C69EB58.png
 [11]: http://brockallen.com/2012/06/28/cors-support-in-webapi-mvc-and-iis-with-thinktecture-identitymodel/ "http://brockallen.com/2012/06/28/cors-support-in-webapi-mvc-and-iis-with-thinktecture-identitymodel/"