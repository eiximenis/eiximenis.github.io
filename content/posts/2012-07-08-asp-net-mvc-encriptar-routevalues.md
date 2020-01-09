---
title: 'ASP.NET MVC: Encriptar RouteValues'
description: 'ASP.NET MVC: Encriptar RouteValues'
author: eiximenis

date: 2012-07-08T18:36:37+00:00
geeks_url: /?p=1603
geeks_visits:
  - 3858
geeks_ms_views:
  - 3074
categories:
  - Uncategorized

---
Muy buenas! El otro día publicaba en mi blog una solución para [encriptar la querystring en ASP.NET MVC][1]. A raiz de este post, me preguntaron si era posible hacer lo mismo pero en el caso de que tengamos URL amigables y como se podría hacer.

La respuesta es que sí, que se puede hacer y que a diferencia del caso de la querystring tenemos dos opciones.

**Opción 1 – Value Provider**

En este caso el planteamiento es el mismo que en el post anterior, debemos crear un value provider que recoja los datos y los desencripte.

Así, empezamos por crear la factoría de nuestro nuevo value provider:

<pre style="overflow: auto; border-top: #cecece 1px solid; border-right: #cecece 1px solid; border-bottom: #cecece 1px solid; padding-bottom: 5px; padding-top: 5px; padding-left: 5px; min-height: 40px; border-left: #cecece 1px solid; padding-right: 5px; width: 500px; background-color: #fbfbfb"><pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> RouteValuesEncriptedValueProviderFactory : ValueProviderFactory
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">{
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">    <span style="color: #0000ff">public</span> <span style="color: #0000ff">override</span> IValueProvider GetValueProvider(ControllerContext controllerContext)
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">    {
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">        <span style="color: #0000ff">return</span> <span style="color: #0000ff">new</span> RouteValuesEncriptedValueProvider(controllerContext);
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">    }
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">}</pre>


<p>
  Y el código de nuestro value provider, muy parecido al del post anterior. La diferencia está en que en lugar de mirar en la request los valores de la querystring, miramos en RouteData que es donde están guardados todos los valores de ruta. Los valores de ruta se extraen a partir del PathInfo, es decir la parte de la URL que va después del host hasta el inicio de la querystring. El código es muy sencillo:
</p>


<pre style="overflow: auto; border-top: #cecece 1px solid; border-right: #cecece 1px solid; border-bottom: #cecece 1px solid; padding-bottom: 5px; padding-top: 5px; padding-left: 5px; min-height: 40px; border-left: #cecece 1px solid; padding-right: 5px; width: 500px; background-color: #fbfbfb"><pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"><span style="color: #0000ff">class</span> RouteValuesEncriptedValueProvider : DictionaryValueProvider&lt;<span style="color: #0000ff">string</span>&gt;
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">{
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"></pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">    <span style="color: #0000ff">public</span> RouteValuesEncriptedValueProvider(System.Web.Mvc.ControllerContext controllerContext)
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">        : <span style="color: #0000ff">base</span>(GetRouteValueDictionary(controllerContext), Thread.CurrentThread.CurrentCulture)
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">    {
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">    }
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"></pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">    <span style="color: #0000ff">private</span> <span style="color: #0000ff">static</span> IDictionary&lt;<span style="color: #0000ff">string</span>, <span style="color: #0000ff">string</span>&gt; GetRouteValueDictionary(ControllerContext controllerContext)
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">    {
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">        var dict = <span style="color: #0000ff">new</span> Dictionary&lt;<span style="color: #0000ff">string</span>, <span style="color: #0000ff">string</span>&gt;();
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">        <span style="color: #0000ff">foreach</span> (var key <span style="color: #0000ff">in</span> controllerContext.RouteData.Values.Keys.Where(x =&gt; x.First() == '_'))
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">        {
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">            var <span style="color: #0000ff">value</span> = controllerContext.RouteData.GetRequiredString(key);
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">            var decripted = Crypto.DecryptValue(<span style="color: #0000ff">value</span>);
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">            dict.Add(key.Substring(1), decripted);
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">        }
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">        <span style="color: #0000ff">return</span> dict;
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">    }
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">}
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"></pre>


<p>
  El funcionamiento es análogo al del value provider del post anterior y actúa sobre los route values cuyo nombre empieze por un subrayado (aunque por supuesto esto va a gustos del consumidor). Dado que el nombre de los route values se establece en la tabla de rutas, debemos cambiar la definición para que allí los nombres empiecen por un subrayado:
</p>


<pre style="overflow: auto; border-top: #cecece 1px solid; border-right: #cecece 1px solid; border-bottom: #cecece 1px solid; padding-bottom: 5px; padding-top: 5px; padding-left: 5px; min-height: 40px; border-left: #cecece 1px solid; padding-right: 5px; width: 500px; background-color: #fbfbfb"><pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">routes.MapRoute(
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">    name: "<span style="color: #8b0000">Default</span>",
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">    url: "<span style="color: #8b0000">{controller}/{action}/{_id}</span>",
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">    defaults: <span style="color: #0000ff">new</span> { controller = "<span style="color: #8b0000">Home</span>", action = "<span style="color: #8b0000">Index</span>", _id = UrlParameter.Optional }
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">);
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"></pre>


<p>
  Al igual que en el value provider del post anterior, al añadir el valor desencriptado de un route value, lo hace eliminando el subrayado inicial, por lo que en nuestros controladores podemos declarar el parámetro como “id” y recibiremos el valor desencriptado (si lo declarásemos como _id lo recibiríamos encriptado).
</p>


<p>
  Pero, en este caso especial tenemos otra opción distinta para desencriptar los valores de ruta…
</p>


<p>
  <strong>Opción 2: Un custom route handler</strong>
</p>


<p>
  Hemos visto como este value provider nuevo no accede a la request, ni mira la URL. En su lugar accede a los valores de ruta (controllerContext.RouteData). Pero… ¿quien rellena esto? Pues el route handler.
</p>


<p>
  Cada vez que una ruta es seleccionada, se ejecuta su route handler, que es el encargado de rellenar los valores de ruta. ASP.NET MVC viene con un route handler por defecto, pero nos podemos crear el nuestro. En este caso, nada nos impide crear un route handler que desencripte los valores <em>antes</em> de añadirlos:
</p>


<pre style="overflow: auto; border-top: #cecece 1px solid; border-right: #cecece 1px solid; border-bottom: #cecece 1px solid; padding-bottom: 5px; padding-top: 5px; padding-left: 5px; min-height: 40px; border-left: #cecece 1px solid; padding-right: 5px; width: 500px; background-color: #fbfbfb"><pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> UnencriptedRouteHandler : MvcRouteHandler
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">{
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">    <span style="color: #0000ff">protected</span> <span style="color: #0000ff">override</span> IHttpHandler GetHttpHandler(System.Web.Routing.RequestContext requestContext)
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">    {
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">        <span style="color: #0000ff">foreach</span> (var rd <span style="color: #0000ff">in</span> requestContext.RouteData.Values.Where(x=&gt;x.Key.First()== '_').ToList())
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">        {
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">            var <span style="color: #0000ff">value</span> = rd.Value.ToString();
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">            var decrypted = !<span style="color: #0000ff">string</span>.IsNullOrEmpty(<span style="color: #0000ff">value</span>) ?
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">                Crypto.DecryptValue(<span style="color: #0000ff">value</span>) :
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">                <span style="color: #0000ff">string</span>.Empty;
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">            requestContext.RouteData.Values.Remove(rd.Key);
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">            requestContext.RouteData.Values.Add(rd.Key.Substring(1), decrypted);
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">        }
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"></pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"></pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">        <span style="color: #0000ff">return</span> <span style="color: #0000ff">base</span>.GetHttpHandler(requestContext);
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">    }
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">}
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"></pre>


<p>
  Si miramos el código, vemos como accedemos a los route values (que en este punto ya están relleanados por el route handler por defecto del cual derivamos) y buscamos aquellos que empiecen por un subrayado y en su lugar los colocamos de nuevo pero desencriptados (y sin subrayar).
</p>


<p>
  Ahora debemos indicarle a ASP.NET MVC que use este route handler nuestro, y eso se hace en la definición de la tabla de rutas:
</p>


<pre style="overflow: auto; border-top: #cecece 1px solid; border-right: #cecece 1px solid; border-bottom: #cecece 1px solid; padding-bottom: 5px; padding-top: 5px; padding-left: 5px; min-height: 40px; border-left: #cecece 1px solid; padding-right: 5px; width: 500px; background-color: #fbfbfb"><pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">routes.Add(
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">    "<span style="color: #8b0000">Default</span>",
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">    <span style="color: #0000ff">new</span> Route("<span style="color: #8b0000">{controller}/{action}/{_id}</span>",
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">                <span style="color: #0000ff">new</span> RouteValueDictionary(<span style="color: #0000ff">new</span> { controller = "<span style="color: #8b0000">Home</span>", action = "<span style="color: #8b0000">Index</span>", _id = UrlParameter.Optional }),
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">                <span style="color: #0000ff">new</span> UnencriptedRouteHandler())
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">);
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"></pre>


<p>
  En lugar de usar MapRoute, usamos el método Add, el cual espera un objeto Route al cual de lo podemos especificar cual será su route handler. Además fijaos en que hemos declarado _id en lugar de id, eso es porque nuestro route handler actuará tan solo sobre los route values que empiecen por el subrayado.
</p>


<p>
  Con esto ya hemos terminado. Ahora, si generamos URLs que tengan el _id encriptado:
</p>


<pre style="overflow: auto; border-top: #cecece 1px solid; border-right: #cecece 1px solid; border-bottom: #cecece 1px solid; padding-bottom: 5px; padding-top: 5px; padding-left: 5px; min-height: 40px; border-left: #cecece 1px solid; padding-right: 5px; width: 500px; background-color: #fbfbfb"><pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">@Html.ActionLink("<span style="color: #8b0000">Test</span>", "<span style="color: #8b0000">Test</span>",
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"><span style="color: #0000ff">new</span> {_id=MvcApplication4.Crypto.EncryptValue("<span style="color: #8b0000">bar value</span>")}, <span style="color: #0000ff">null</span>)</pre>


<p>
  Podemos declarar en el controlador la acción que espere un parámetro llamado id que tendrá el valor ya desencriptado. A diferencia de la opción anterior, ahora si usamos _id como parámetro recibiremos null. El valor encriptado se ha perdido y no existe a todos los efectos.
</p>


<p>
  Usando esta segunda opción no hay necesidad alguna de crear un value provider propio.
</p>


<p>
  <strong>¿Y las diferencias?</strong>
</p>


<p>
  Las diferencias entre las dos opciones son básicamente el momento en que se produce la desencriptación:
</p>


<ol>
  <li>
    En la primera opción (value provider), la desencriptación se produce tan solo en el momento de enlazar los parámetros al controlador. Y la desencriptación es tan solo visible para el model binder. Así p.ej. si hay filtros que actuen sobre los valores de ruta, estos verán los valores encriptados.
  </li>
  
  
  <li>
    En la segunda opción, justo al inicio de procesarse la request, antes incluso de decidirse que controlador y que acción deben ser invocados, los valores de ruta son desencriptados. El valor encriptado desaparece y todo el mundo (y no sólo el model binder) tiene acceso a los valores desencriptados.
  </li>
  
</ol>


<p>
  Si me preguntáis en mi caso que opción escogería… probablemente la segunda 🙂
</p>


<p>
  Un saludo!
</p>


<p>
  PD: En el caso de la querystring <strong>no</strong> existe la posibilidad de esta segunda opción. Los route handlers actúan tan solo sobre los valores de ruta, no sobre la querystring, por lo que el value provider es la única opción.
</p>

 [1]: http://geeks.ms/blogs/etomas/archive/2012/07/03/asp-net-mvc-encriptar-la-query-string.aspx