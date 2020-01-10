---
title: ASP.NET WebApi y X-HTTP-Method-Override

author: eiximenis

date: 2012-11-21T18:13:16+00:00
geeks_url: /?p=1617
geeks_visits:
  - 1645
geeks_ms_views:
  - 1142
categories:
  - Uncategorized

---
Muy buenas! Después de largo tiempo vuelvo a la carga con otro post sobre ASP.Net WebApi. En un post anterior vimos como WebApi <a href="http://geeks.ms/blogs/etomas/archive/2012/02/19/explorando-asp-net-mvc4-webapi-2-enrutamiento-y-verbos-http-propios.aspx" target="_blank" rel="noopener noreferrer">usaba automáticamente el verbo http usado para invocar el método correspondiente del controlador</a>. Eso está muy bien en aquellos casos en que el cliente es una aplicación de escritorio y tiene acceso a todos los verbos http posibles. Pero si el cliente es una aplicación web _es posible_ que tan solo tenga acceso a dos de los verbos http: GET y POST.

En una aproximación REST, habitualmente GET se usa para recuperar datos y POST para modificar datos existentes. Se usa DELETE para eliminar datos y PUT para insertar datos nuevos. Si tenemos un controlador WebApi con el siguiente código:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: blue">public</span> <span style="color: blue">class</span> <span style="color: #2b91af">PlayerController</span> : <span style="color: #2b91af">ApiController</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">public</span> <span style="color: blue">string</span> GetById(<span style="color: blue">string</span> id)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: green">// Devolver el jugador con el id indicado</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">return</span> id;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">public</span> <span style="color: blue">string</span> Put(<span style="color: blue">string</span> name)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: green">// Crear un jugador nuevo y devolver su id</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">return</span> <span style="color: #a31515">"new player created "</span> + name;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Si queremos invocar el método put debemos usar el verbo HTTP PUT pero eso, en navegadores puede no ser posible (puedes ver que métodos soporta tu navegador a través de XmlHttpRequest en [http://www.mnot.net/javascript/xmlhttprequest/][1]).

Para seguir permitiendo el enrutamiento según verbo http pero a la vez dar soporte a navegadores que no soporten otros verbos http salvo GET y POST se inventó la cabecera X-HTTP-Method-Override. La idea es guardar en esta cabecera el verbo http que se querría haber usado (p.ej. PUT) y usar el verbo http POST para realizar la llamada. Esta cabecera HTTP no es un estandard reconocido, comenzó a ser utilizada por Google creo y su uso se ha extendido (hay variantes de dicha cabcera, p.ej. la especificación de OData usa X-HTTP-Method en lugar de X-HTTP-Method-Override pero la idea es la misma).

Lamentablemente ASP.NET WebApi no tiene soporte para X-HTTP-Method-Override (ni para X-HTTP-Method ni ninguna otra variante). Si usas el verbo POST todas las llamadas se enrutarán a alguno de los métodos cuyo nombre empiece por Post con independencia del valor de estas cabeceras.

**DelegatingHandlers**

Por suerte dentro de ASP.NET WebApi no se incluye tan solo la idea de “haz controladores que devuelvan objetos y deja que se enruten a través del verbo http”. También se incluyen un conjunto de clases nuevas para interaccionar con la pipeline de Http… y entre esas encontramos los DelegatingHandlers.

Si tienes experiencia con los HttpModules, la idea es similar: un DelegatingHandler intercepta una petición http (también puede interceptar la respuesta si fuese necesario). A nivel de WebApi existe una cadena de DelegatingHandlers y la petición va viajando de uno a otro. Cada DelegatingHandler puede modificar los datos de la petición acorde a lo que necesite y pasar la petición al siguiente DelegatingHandler de la cadena. Lo mismo aplica a la respuesta http por supuesto.

¿Ya tienes la solución, verdad? 😉

Efectivamente, basta con crear un DelegatingHandler que recoja la petición, examine las cabeceras, mire si existe X-HTTP-Method-Override y establezca como nuevo verbo HTTP el valor que se indique en dicha cabecera.

El código puede ser algo parecido a:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: blue">public</span> <span style="color: blue">class</span> <span style="color: #2b91af">OverrideMethodDelegatingHandler</span> : <span style="color: #2b91af">DelegatingHandler</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">protected</span> <span style="color: blue">override</span> <span style="color: #2b91af">Task</span><<span style="color: #2b91af">HttpResponseMessage</span>> SendAsync(<span style="color: #2b91af">HttpRequestMessage</span> request, <span style="color: #2b91af">CancellationToken</span> cancellationToken)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">if</span> (request.Method == <span style="color: #2b91af">HttpMethod</span>.Post && request.Headers.Contains(<span style="color: #a31515">"X-HTTP-Method-Override"</span>))
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">var</span> newMethod = request.Headers.GetValues(<span style="color: #a31515">"X-HTTP-Method-Override"</span>).FirstOrDefault();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">if</span> (!<span style="color: blue">string</span>.IsNullOrEmpty(newMethod)) {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; request.Method = <span style="color: blue">new</span> <span style="color: #2b91af">HttpMethod</span>(newMethod);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">return</span> <span style="color: blue">base</span>.SendAsync(request, cancellationToken);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

¿Creo que no necesita mucha explicación verdad? Simplemente validamos que la petición sea POST (si alguien envía X-HTTP-Method-Override a través de GET lo ignoramos) y si la cabecera existe modificamos el verbo http de la petición.

Ahora tan solo nos queda el punto final: registrar nuestro DelegatingHandler. Como todo en WebApi (y ASP.NET MVC) se sigue la filosofia de usar una clase estática con la configuración:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: #2b91af">GlobalConfiguration</span>.Configuration.MessageHandlers.Add(<span style="color: blue">new</span> <span style="color: #2b91af">OverrideMethodDelegatingHandler</span>());
  </p></p>
</div>

Y listos! Con esto ya podemos usar X-HTTP-Method-Override en nuestras APIs WebApi. Para probarlo nos basta con el siguiente código:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: blue"><!</span><span style="color: maroon">DOCTYPE</span> <span style="color: red">html</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"><</span><span style="color: maroon">html</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">head</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">title</span><span style="color: blue">></span>title<span style="color: blue"></</span><span style="color: maroon">title</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">script</span> <span style="color: red">src</span><span style="color: blue">="http://ajax.aspnetcdn.com/ajax/jQuery/jquery-1.8.2.min.js"></</span><span style="color: maroon">script</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue"></</span><span style="color: maroon">head</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">body</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">script</span> <span style="color: red">type</span><span style="color: blue">="text/javascript"></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; $(document).ready(<span style="color: blue">function</span>() {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; $(<span style="color: #a31515">"#myfrm"</span>).submit(<span style="color: blue">function</span> (evt) {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">var</span> url = <span style="color: #a31515">&#8216;</span><span style="background: yellow">@</span>Url.RouteUrl(<span style="color: #a31515">"DefaultApi"</span>, <span style="color: blue">new</span> {httproute=<span style="color: #a31515">""</span>, controller=<span style="color: #a31515">"Player"</span>, name=<span style="color: #a31515">"xxx"</span>})<span style="color: #a31515">&#8216;</span>.
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; replace(<span style="color: #a31515">"xxx"</span>, $(<span style="color: #a31515">"#name"</span>).val());
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; $.ajax({
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; url: url,
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; type: <span style="color: #a31515">&#8216;post&#8217;</span>, <span style="color: green">// usamos http post</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; headers: {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #a31515">"X-HTTP-Method-Override"</span> : <span style="color: #a31515">&#8216;PUT&#8217;</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; });
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; evt.preventDefault();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; });
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; });
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"></</span><span style="color: maroon">script</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    <p style="margin: 0px">
      <span style="color: blue"></span>
    </p>
    
    <p>
      &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">div</span><span style="color: blue">></span>
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">form</span> <span style="color: red">method</span><span style="color: blue">="POST"</span> <span style="color: red">id</span><span style="color: blue">="myfrm"></span>
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">input</span> <span style="color: red">type</span><span style="color: blue">="text"</span> <span style="color: red">name</span><span style="color: blue">="name"</span> <span style="color: red">id</span><span style="color: blue">="name"</span> <span style="color: blue">/></span>
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">input</span> <span style="color: red">type</span><span style="color: blue">="submit"/></span>
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"></</span><span style="color: maroon">form</span><span style="color: blue">></span>
    </p>
    
    <p style="margin: 0px">
      &#160;
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"></</span><span style="color: maroon">div</span><span style="color: blue">></span>
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160; <span style="color: blue"></</span><span style="color: maroon">body</span><span style="color: blue">></span>
    </p>
    
    <p style="margin: 0px">
      <span style="color: blue"></</span><span style="color: maroon">html</span><span style="color: blue">></span>
    </p>
  </p>
</div>

Y si colocais un breakpoint en el método Put del controlador vereis como la llamada llega a pesar de estar usando POST 😉

Un saludo!

 [1]: http://www.mnot.net/javascript/xmlhttprequest/ "http://www.mnot.net/javascript/xmlhttprequest/"