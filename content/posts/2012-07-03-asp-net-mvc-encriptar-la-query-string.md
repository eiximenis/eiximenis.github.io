---
title: 'ASP.NET MVC: Encriptar la query string'

author: eiximenis

date: 2012-07-03T21:09:16+00:00
geeks_url: /?p=1602
geeks_visits:
  - 4722
geeks_ms_views:
  - 2623
categories:
  - Uncategorized

---
Buenas! Este post surge debido a [esta pregunta del foro de ASP.NET MVC][1]. El usuario se pregunta si existe en el framework una manera built-in de encriptar la query string. Y la realidad es que no, no la hay, pero añadir una es muy sencillo y me da una excusa perfecta para poner un buen ejemplo del poder de los value providers.

ASP.NET MVC está construído de una forma bastante flexible, pero en el pipeline de una petición hay más o menos 4 pasos:

  1. Procesar la url para generar unos valores de ruta 
  2. En base a dichos valores seleccionar una acción de un controlador 
  3. Explorar la petición http en busca de parámetros (que pueden estar en la querystring, en formdata, etc) 
  4. A partir de esos parámetros rellenar los parámetros de la acción del controlador 

De todos esos pasos, el tercero es responsabilidad de los value providers: ellos inspeccionan la petición http en busca de parámetros y los dejan en “un sitio común”. Luego, en el paso cuatro, el model binder recoge los parámetros de este sitio común y a partir de ellos instancia los parámetros de la acción del controlador.

Ahora supongamos que nuestra querystring tiene algún parámetro encriptado. Para este post he supuesto que _algunos_ parámetros de la querystring pueden ir encriptados, mientras que otros no (no se encripta toda la querystring, aunque la técnica seria la misma). Para el ejemplo supongo que los parámetros encriptados tienen un nombre que empieza por un subrayado (y no, el nombre en si mismo no se encripta).

Así, p.ej. voy a tener una querystring del tipo:

/accion/controlador?_foo=aIGf0UYsNARiBaGZr3blRg%3D%3D 

Donde el valor del parámetro _foo está encriptado.

Ahora bien, a mi me gustaría evitar tener que colocar el código para desencriptar dichos valores en cada acción:

<pre style="overflow: auto; border-top: #cecece 1px solid; border-right: #cecece 1px solid; border-bottom: #cecece 1px solid; padding-bottom: 5px; padding-top: 5px; padding-left: 5px; min-height: 40px; border-left: #cecece 1px solid; padding-right: 5px; width: 500px; background-color: #fbfbfb"><pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"><span style="color: #0000ff">public</span> ActionResult Test(<span style="color: #0000ff">string</span> _foo)
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">{
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">    <span style="color: #008000">// Codigo a evitar:</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">    var realfoo = Crypto.DecryptValue(_foo);
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">    <span style="color: #008000">// ...</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">}
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"></pre>


<p>
  Aquí es donde entran los value providers. ASP.NET MVC trae de serie varios value providers que “inspeccionan” una petición http y miran los sitios más habituales donde se pueden encontrar datos:
</p>


<p>
  <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_58CA2867.png"><img title="image" style="border-top: 0px; border-right: 0px; border-bottom: 0px; border-left: 0px; display: inline" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_1271E3B4.png" width="504" height="148" /></a> 
</p>


<p>
  Los value providers se registran en el sistema a través de una factoría, y esta captura muestra las factorías de value providers registradas por defecto en ASP.NET MVC4 RC. Cada factoría devuelve un tipo de value provider, es decir un value provider que mira <em>en una parte</em> de la petición http y por el nombre se puede más o menos deducir:
</p>


<ol>
  <li>
    ChildActionValueProviderFactory: Se usa cuando se invocan acciones hijas (a través de Html.Action o de Html.RenderAction). Es más de infraestructura de ASP.NET MVC que otra cosa.
  </li>
  
  
  <li>
    FormValueProviderFactory: Devuelve un value provider encargado de inspeccionar los parámetros de formdata (básicamente formularios enviados via POST).
  </li>
  
  
  <li>
    JsonValueProviderFactory: Introducido en MVC3 devuelve un value provider que inspecciona la petición http, buscando datos en el cuerpo de ésta que estén en formato json. Es el responsable de que en MVC3 puedas hacer un POST enviando datos en formato json y que el model binder los entienda.
  </li>
  
  
  <li>
    RouteDataValueProviderFactory: Devuelve un value provider que inspecciona los route values, o sea los parámetros establecidos a través de la tabla de rutas.
  </li>
  
  
  <li>
    QueryStringValueProviderFactory: Devuelve un value provider que inspecciona la querystring.
  </li>
  
  
  <li>
    HttpFileCollectionValueProviderFactory: Devuelve un value provider que inspecciona los datos de los ficheros subidos (usando <input type=”file”>).
  </li>
  
</ol>


<p>
  Por supueso nosotros podemos crearnos nuestras propias factorías de value providers, y eso es justamente lo que haremos. En nuestro caso vamos a hacer un value provider que:
</p>


<ol>
  <li>
    Inspeccione la query string
  </li>
  
  
  <li>
    Seleccione aquellos parámetros que empiezan por un subrayado.
  </li>
  
  
  <li>
    Desencripte cada uno de esos parámetros y guarde en “este sitio común” dicho parámetro, pero con el valor desencriptado. Además el nombre del parámetro en el “sitio común” lo modificaremos quitándole el subrayado.
  </li>
  
</ol>


<p>
  Veamos primero el código de la factoría:
</p>


<pre style="overflow: auto; border-top: #cecece 1px solid; border-right: #cecece 1px solid; border-bottom: #cecece 1px solid; padding-bottom: 5px; padding-top: 5px; padding-left: 5px; min-height: 40px; border-left: #cecece 1px solid; padding-right: 5px; width: 500px; background-color: #fbfbfb"><pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> QSEncriptedValueProviderFactory : ValueProviderFactory
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">{
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">    <span style="color: #0000ff">public</span> <span style="color: #0000ff">override</span> IValueProvider GetValueProvider(ControllerContext controllerContext)
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">    {
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">        <span style="color: #0000ff">return</span> <span style="color: #0000ff">new</span> QSEncriptedValueProvider(controllerContext);
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">    }
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">}</pre>


<p>
  Simple no? 🙂 Nos limitamos a devolver una instancia de nuestro value provider propio. Veamos su código:
</p>


<p>
  &#160;
</p>


<pre style="overflow: auto; border-top: #cecece 1px solid; border-right: #cecece 1px solid; border-bottom: #cecece 1px solid; padding-bottom: 5px; padding-top: 5px; padding-left: 5px; min-height: 40px; border-left: #cecece 1px solid; padding-right: 5px; width: 500px; background-color: #fbfbfb"><pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> QSEncriptedValueProvider : DictionaryValueProvider&lt;<span style="color: #0000ff">string</span>&gt;
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">{
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">    <span style="color: #0000ff">public</span> QSEncriptedValueProvider(ControllerContext controllerContext) : <span style="color: #0000ff">base</span>(GetQSDictionary(controllerContext), Thread.CurrentThread.CurrentCulture)
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">    {
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">    }
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"></pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">    <span style="color: #0000ff">private</span> <span style="color: #0000ff">static</span> IDictionary&lt;<span style="color: #0000ff">string</span>, <span style="color: #0000ff">string</span>&gt; GetQSDictionary(ControllerContext controllerContext)
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">    {
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">        var dict = <span style="color: #0000ff">new</span> Dictionary&lt;<span style="color: #0000ff">string</span>, <span style="color: #0000ff">string</span>&gt;();
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">        var req = controllerContext.HttpContext.Request;
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">        <span style="color: #0000ff">foreach</span> (var key <span style="color: #0000ff">in</span> req.QueryString.AllKeys.Where(x =&gt; x.First() == '_'))
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">        {
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">            var <span style="color: #0000ff">value</span> = req.QueryString[key];
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">            dict.Add(key.Substring(1),Crypto.DecryptValue(<span style="color: #0000ff">value</span>));
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">        }
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"></pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">        <span style="color: #0000ff">return</span> dict;
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">    }
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"></pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">}</pre>


<p>
  Derivamos de la clase base DictionaryValueProvider<string>. Esto lo que significa es que este value providers, tan solo dejará strings en “este sitio común”. Bueno, esto es lógico ya que en la querystring tan solo hay strings. Pero p.ej. otros value providers podrían dejar objetos enteros en este “sitio común” (como el que inspecciona la petición buscando datos en json).
</p>


<p>
  Al derivar de esta clase obtenemos la facilidad de trabajar con un diccionario de cadenas (en nuestro caso de <string,string> ya que las claves siempre son cadenas). Básicamente, lo que dejemos en este diccionario es lo que se colocará en este “sitio común”. Y eso es lo que hace el método GetQSDictionary.
</p>


<p>
  Mirando el código vemos que selecciona todas las claves de la querystring que empiecen por un subrayado, y por cada uno de ellas:
</p>


<ul>
  <li>
    Desencripta su contenido
  </li>
  
  
  <li>
    Guarda el valor desencriptado con la misma clave, pero quitando el subrayado del principio.
  </li>
  
</ul>


<p>
  ¡Ya estamos listos! Ahora tan solo debemos registrar la factoría de value providers. Para ello en el Application_Start podemos colocar la línea:
</p>


<pre style="overflow: auto; border-top: #cecece 1px solid; border-right: #cecece 1px solid; border-bottom: #cecece 1px solid; padding-bottom: 5px; padding-top: 5px; padding-left: 5px; min-height: 40px; border-left: #cecece 1px solid; padding-right: 5px; width: 500px; background-color: #fbfbfb"><pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">ValueProviderFactories.Factories.Add(<span style="color: #0000ff">new</span> QSEncriptedValueProviderFactory());</pre>


<p>
  ¡Ya podemos probarlo!
</p>


<p>
  Para ello me creo una vista con un link que tenga un parámetro encriptado:
</p>


<pre style="overflow: auto; border-top: #cecece 1px solid; border-right: #cecece 1px solid; border-bottom: #cecece 1px solid; padding-bottom: 5px; padding-top: 5px; padding-left: 5px; min-height: 40px; border-left: #cecece 1px solid; padding-right: 5px; width: 500px; background-color: #fbfbfb"><pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">@Html.ActionLink("<span style="color: #8b0000">Test</span>", "<span style="color: #8b0000">Test</span>",
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"><span style="color: #0000ff">new</span> {_foo=MvcApplication4.Crypto.EncryptValue("<span style="color: #8b0000">bar value</span>")}, <span style="color: #0000ff">null</span>)</pre>


<p>
  El código fuente generado es el siguiente:
</p>


<pre style="overflow: auto; border-top: #cecece 1px solid; border-right: #cecece 1px solid; border-bottom: #cecece 1px solid; padding-bottom: 5px; padding-top: 5px; padding-left: 5px; min-height: 40px; border-left: #cecece 1px solid; padding-right: 5px; width: 500px; background-color: #fbfbfb"><pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"><span style="color: #0000ff">&lt;</span><span style="color: #800000">a</span> <span style="color: #ff0000">href</span>=<span style="color: #0000ff">"/Home/Test?_foo=aIGf0UYsNARiBaGZr3blRg%3D%3D"</span><span style="color: #0000ff">&gt;</span>Test<span style="color: #0000ff">&lt;/</span><span style="color: #800000">a</span><span style="color: #0000ff">&gt;</span>
</pre>


<p>
  Podemos ver como el valor “bar value” ha sido encriptado. Y ahora viene lo bueno: en nuestro controlador declaramos la acción Test que recibe una cadena, y eso es lo que obtenemos:
</p>


<p>
  <img title="image" style="border-top: 0px; border-right: 0px; border-bottom: 0px; border-left: 0px; display: inline" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_0A7F62B4.png" width="419" height="57" /> 
</p>


<p>
  En el controlador obtenemos ya el valor desencriptado! Fijaos además como el parámetro de la acción se llama foo (en lugar de _foo). Eso es porque en el value provider hemos eliminado el subrayado inicial. Por supuesto dado que también existe el value provider que procesa la querystring de forma “normal”, si en la acción declaramos un parámetro _foo obtendremos el valor encriptado.
</p>


<p>
  Esta es una breve demostración del poder de los value providers. Por supuesto, deberíamos crearnos un conjunto de helpers para generar URLs encriptadas de forma más fácil, pero eso ya daría para otro post.
</p>


<p>
  Un saludo!
</p>


<p>
  PD: No he puesto el código de encriptar/desencriptar adrede porque hay muchas maneras de hacerlo. Yo en mi caso he usado uno que he encontrado en <a href="http://www.joshrharrison.com/archive/2009/01/28/c-encryption.aspx">http://www.joshrharrison.com/archive/2009/01/28/c-encryption.aspx</a> (aunque ello no es relevante para lo que cuenta este post).
</p>

 [1]: http://social.msdn.microsoft.com/Forums/es-ES/aspnetmvces/thread/c860aacd-d5b6-480d-8c36-45733c8cc813