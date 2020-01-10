---
title: Creando formateadores de salida en asp.net core

author: eiximenis

date: 2016-04-21T07:51:29+00:00
geeks_url: /?p=1771
geeks_ms_views:
  - 2341
categories:
  - asp.net 5
  - asp.net MVC

---
Cuando salió WebApi lo hizo con la negociación de contenido incorporada de serie en el framework. Eso venía a significar, básicamente, que el framework intentaba suministrar los datos en el formato en que el cliente los había pedido. La negociación de contenido se basa (generalmente) en el uso de la cabecera _accept_ de HTTP: el cliente manda en esa cabecera cual, o cuales, son sus formatos de respuesta preferidos. WebApi soporta de serie devolver datos en JSON y XML y el sistema es extensible para crear nuestros propios formatos.

“MVC clásico” (es decir hasta MVC5) no incluye soporte de negociación de contenido: en MVC si queremos devolver datos en formato JSON, debemos devolver explícitamente un JsonResult y si los queremos devolver en XML debemos hacerlo también explícitamente.

En ASP.NET Core tenemos a MVC6 que unifica a WebApi y MVC clásico en un solo framework. ¿Como queda el soporte para negociación de contenido en MVC6? Pues bien, existe soporte para ella, pero dependiendo de que IActionResult devolvamos en nuestros controladores. Así, si en WebApi la negociación de contenido se usaba siempre y en MVC clásico nunca, **en MVC6 la negociación de contenido aplica solo si la acción del controlador devuelve un ObjectResult (o derivado)**. Esto nos permite como desarrolladores decidir sobre qué acciones de qué controladores queremos aplicar la negociación de contenido. Es evidente que aplicarla siempre no tiene sentido: si devolvemos una vista Razor su resultado debe ser sí o sí un HTML que se envía al cliente. No tendría sentido aplicar negociación de contenido sobre una acción que devolviese una vista. De hecho la negociación de contenido tiene sentido en APIs que devuelvan datos (no vistas) y en MVC6 para devolver datos tenemos a ObjectResult, así que es lógico que sea sobre este resultado donde se aplique la negociación de contenido.

En WebApi la negociación de contenido estaba gestionada por los formateadores (_formatters_). Básicamente a cada _content-type_ se le asociaba un formateador. Si el cliente pedía datos en un determinado content-type se miraba que formateador podía devolver datos en dicho formato. Si no existía se usaba por defecto el formateador de JSON. En MVC6 se ha mantenido básicamente dicho esquema.

<!--more-->

La principal diferencia es que en WebApi los formateadores se encargaban realmente de dos tareas totalmente separadas: por un lado procesaban (leían) los datos de entrada (es decir definían que tipos de content-types aceptaba el servidor) y también procesaban (serializaban) los datos de salida. El problema es fácil de ver: el hecho de que una API devuelva datos en un determinado formato (pongamos XML) no significa que deba aceptar datos (p. ej. un POST) en dicho formato. Pero en WebApi al implementar el formateador de XML debíamos implementar tanto el método para leer datos en XML como para escribirlos. En MVC6 se ha solucionado este detalle separando los formateadores en dos: los de entrada (leen los datos enviados por el cliente) y los de salida (envían los datos al cliente). Esto separa mejor las responsabilidades.

Vamos a ver como podemos crear un formateador que nos permita devolver datos en CSV. Dado que CSV es un formato de tipo “tabular”, solo vamos a aceptar devolver datos en este formato, siempre que esos datos sean un IEnumerable.

**Creando un formateador de salida**

Para crear un formateador de salida basta con implementar la interfaz IOutputFormatter, que define dos métodos:

  1. **CanWriteResult**: Que debe indicar si el formateador puede enviar el resultado al cliente 
      * **WriteAsync**: Que debe enviar los datos formateados</ol> 
    Una posible implementación podría ser tal y como sigue:
    
    <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:61cb7d01-465c-43db-ace1-2e9160a1983f" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
      <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
        <div style="background: #ddd; max-height: 300px; overflow: auto">
          <ol start="1" style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
            <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">public</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">class</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#2b91af">CsvOutputFormatter</span><span style="background:#ffffff;color:#000000"> : </span><span style="background:#ffffff;color:#2b91af">IOutputFormatter</span>
            </li>
            <li>
              <span style="background:#ffffff;color:#000000">{</span>
            </li>
            <li>
                  <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">public</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">bool</span><span style="background:#ffffff;color:#000000"> CanWriteResult(</span><span style="background:#ffffff;color:#2b91af">OutputFormatterCanWriteContext</span><span style="background:#ffffff;color:#000000"> context)</span>
            </li>
            <li>
                  <span style="background:#ffffff;color:#000000">{</span>
            </li>
            <li>
                      <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">if</span><span style="background:#ffffff;color:#000000"> (context.ContentType.MediaType != </span><span style="background:#ffffff;color:#a31515">"text/csv"</span><span style="background:#ffffff;color:#000000">)</span>
            </li>
            <li>
                      <span style="background:#ffffff;color:#000000">{</span>
            </li>
            <li>
                          <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">false</span><span style="background:#ffffff;color:#000000">;</span>
            </li>
            <li>
                      <span style="background:#ffffff;color:#000000">}</span>
            </li>
            <li>
                      <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> type = context.ObjectType.GetTypeInfo();</span>
            </li>
            <li>
                      <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">if</span><span style="background:#ffffff;color:#000000"> (type.ImplementedInterfaces.Any(ii => ii == </span><span style="background:#ffffff;color:#0000ff">typeof</span><span style="background:#ffffff;color:#000000">(</span><span style="background:#ffffff;color:#2b91af">IEnumerable</span><span style="background:#ffffff;color:#000000">)))</span>
            </li>
            <li>
                      <span style="background:#ffffff;color:#000000">{</span>
            </li>
            <li>
                          <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">true</span><span style="background:#ffffff;color:#000000">;</span>
            </li>
            <li>
                      <span style="background:#ffffff;color:#000000">}</span>
            </li>
            <li>
                      <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">false</span><span style="background:#ffffff;color:#000000">;</span>
            </li>
            <li>
                  <span style="background:#ffffff;color:#000000">}</span>
            </li>
            <li>
              &nbsp;
            </li>
            <li>
                  <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">public</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;
color:#0000ff">async</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#2b91af">Task</span><span style="background:#ffffff;color:#000000"> WriteAsync(</span><span style="background:#ffffff;color:#2b91af">OutputFormatterWriteContext</span><span style="background:#ffffff;color:#000000"> context)</span>
            </li>
            <li>
                  <span style="background:#ffffff;color:#000000">{</span>
            </li>
            <li>
                      <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> response = context.HttpContext.Response;</span>
            </li>
            <li>
                      <span style="background:#ffffff;color:#000000">response.ContentType = </span><span style="background:#ffffff;color:#a31515">"text/csv"</span><span style="background:#ffffff;color:#000000">;</span>
            </li>
            <li>
                      <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">using</span><span style="background:#ffffff;color:#000000"> (</span><span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> writer = context.WriterFactory(response.Body, </span><span style="background:#ffffff;color:#2b91af">Encoding</span><span style="background:#ffffff;color:#000000">.UTF8))</span>
            </li>
            <li>
                      <span style="background:#ffffff;color:#000000">{</span>
            </li>
            <li>
                          <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">await</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#2b91af">CsvSerializer</span><span style="background:#ffffff;color:#000000">.SerializeAsync(context.Object, writer);</span>
            </li>
            <li>
                          <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">await</span><span style="background:#ffffff;color:#000000"> writer.FlushAsync();</span>
            </li>
            <li>
                      <span style="background:#ffffff;color:#000000">}</span>
            </li>
            <li>
                  <span style="background:#ffffff;color:#000000">}</span>
            </li>
            <li>
              <span style="background:#ffffff;color:#000000">}</span>
            </li>
          </ol>
        </div></p>
      </div></p>
    </div>
    
    La implementación es muy sencilla, simplemente en el _CanWriteResult_ miramos que el valor de la cabecera _accept_ sea “text/csv” y que el objeto a serializar implemente IEnumerable (en la realidad quizás haríamos aquí más comprobaciones).
    
    En el método _WriteAsync_ simplemente serializamos los datos en formato CSV. Para ello usamos una clase CsvSerializer, cuya implementación es trivial:
    
    <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:125b8e15-5d75-4566-b524-7ca318287999" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
      <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
        <div style="background: #ddd; max-height: 300px; overflow: auto">
          <ol start="1" style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
            <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">static</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">class</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#2b91af">CsvSerializer</span>
            </li>
            <li>
              <span style="background:#ffffff;color:#000000">{</span>
            </li>
            <li>
                  <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">public</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">static</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">async</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#2b91af">Task</span><span style="background:#ffffff;color:#000000"> SerializeAsync(</span><span style="background:#ffffff;color:#0000ff">object</span><span style="background:#ffffff;color:#000000"> obj, </span><span style="background:#ffffff;color:#2b91af">TextWriter</span><span style="background:#ffffff;color:#000000"> writer)</span>
            </li>
            <li>
                  <span style="background:#ffffff;color:#000000">{</span>
            </li>
            <li>
                      <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> collection = obj </span><span style="background:#ffffff;color:#0000ff">as</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#2b91af">IEnumerable</span><span style="background:#ffffff;color:#000000">;</span>
            </li>
            <li>
                      <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">if</span><span style="background:#ffffff;color:#000000"> (collection != </span><span style="background:#ffffff;color:#0000ff">null</span><span style="background:#ffffff;color:#000000">)</span>
            </li>
            <li>
                      <span style="background:#ffffff;color:#000000">{</span>
            </li>
            <li>
                          <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> properties = GetItemsProperties(collection);</span>
            </li>
            <li>
                          <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">await</span><span style="background:#ffffff;color:#000000"> WriteHeadersAsync(properties, writer);</span>
            </li>
            <li>
                          <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">await</span><span style="background:#ffffff;color:#000000"> WriteItemsAsync(collection, properties, writer);</span>
            </li>
            <li>
                      <span style="background:#ffffff;color:#000000">}</span>
            </li>
            <li>
                  <span style="background:#ffffff;color:#000000">}</span>
            </li>
            <li>
              &nbsp;
            </li>
            <li>
                  <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">private</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">static</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">async</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#2b91af">Task</span><span style="background:#ffffff;color:#000000"> WriteItemsAsync(</span><span style="background:#ffffff;color:#2b91af">IEnumerable</span><span style="background:#ffffff;color:#000000"> collection, </span><span style="background:#ffffff;color:#2b91af">PropertyInfo</span><span style="background:#ffffff;color:#000000">[] properties, </span><span style="background:#ffffff;color:#2b91af">TextWriter</span><span style="background:#ffffff;color:#000000"> writer)</span>
            </li>
            <li>
                  <span style="background:#ffffff;color:#000000">{</span>
            </li>
            <li>
                      <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">foreach</span><span style="background:#ffffff;color:#000000"> (</span><span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> item </span><span style="background:#ffffff;color:#0000ff">in</span><span style="background:#ffffff;color:#000000"> collection)</span>
            </li>
            <li>
                      <span style="background:#ffffff;color:#000000">{</span>
            </li>
            <li>
                          <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">await</span><span style="background:#ffffff;color:#000000"> writer.WriteLineAsync(</span><span style="background:#ffffff;color:#0000ff">string</span><span style="background:#ffffff;color:#000000">.Join(</span><span style="background:#fffff
f;color:#a31515">","</span><span style="background:#ffffff;color:#000000">,</span>
            </li>
            <li>
                              <span style="background:#ffffff;color:#000000">properties.Select(pi => </span><span style="background:#ffffff;color:#2b91af">Convert</span><span style="background:#ffffff;color:#000000">.ToString(pi.GetValue(item), </span><span style="background:#ffffff;color:#2b91af">CultureInfo</span><span style="background:#ffffff;color:#000000">.InvariantCulture))));</span>
            </li>
            <li>
                      <span style="background:#ffffff;color:#000000">}</span>
            </li>
            <li>
                  <span style="background:#ffffff;color:#000000">}</span>
            </li>
            <li>
                  <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">private</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">static</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#2b91af">Task</span><span style="background:#ffffff;color:#000000"> WriteHeadersAsync(</span><span style="background:#ffffff;color:#2b91af">IEnumerable</span><span style="background:#ffffff;color:#000000"><</span><span style="background:#ffffff;color:#2b91af">PropertyInfo</span><span style="background:#ffffff;color:#000000">> properties, </span><span style="background:#ffffff;color:#2b91af">TextWriter</span><span style="background:#ffffff;color:#000000"> writer)</span>
            </li>
            <li>
                  <span style="background:#ffffff;color:#000000">{</span>
            </li>
            <li>
                      <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> writer.WriteLineAsync(</span><span style="background:#ffffff;color:#0000ff">string</span><span style="background:#ffffff;color:#000000">.Join(</span><span style="background:#ffffff;color:#a31515">","</span><span style="background:#ffffff;color:#000000">,</span>
            </li>
            <li>
                          <span style="background:#ffffff;color:#000000">properties.Select(pi => pi.Name)));</span>
            </li>
            <li>
                  <span style="background:#ffffff;color:#000000">}</span>
            </li>
            <li>
                  <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">private</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">static</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#2b91af">PropertyInfo</span><span style="background:#ffffff;color:#000000">[] GetItemsProperties(</span><span style="background:#ffffff;color:#2b91af">IEnumerable</span><span style="background:#ffffff;color:#000000"> collection)</span>
            </li>
            <li>
                  <span style="background:#ffffff;color:#000000">{</span>
            </li>
            <li>
                      <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">object</span><span style="background:#ffffff;color:#000000"> first = </span><span style="background:#ffffff;color:#0000ff">null</span><span style="background:#ffffff;color:#000000">;</span>
            </li>
            <li>
                      <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">foreach</span><span style="background:#ffffff;color:#000000"> (</span><span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> item </span><span style="background:#ffffff;color:#0000ff">in</span><span style="background:#ffffff;color:#000000"> collection)</span>
            </li>
            <li>
                      <span style="background:#ffffff;color:#000000">{</span>
            </li>
            <li>
                          <span style="background:#ffffff;color:#000000">first = item;</span>
            </li>
            <li>
                          <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">break</span><span style="background:#ffffff;color:#000000">;</span>
            </li>
            <li>
                      <span style="background:#ffffff;color:#000000">}</span>
            </li>
            <li>
                      <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> type = first.GetType().GetTypeInfo();</span>
            </li>
            <li>
                      <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> type.DeclaredProperties.ToArray();</span>
            </li>
            <li>
                  <span style="background:#ffffff;color:#000000">}</span>
            </li>
            <li>
              <span style="background:#ffffff;color:#000000">}</span>
            </li>
          </ol>
        </div></p>
      </div></p>
    </div>
    
    Ahora solo nos falta un punto, que es añadir nuestro formateador de salida a MVC6. Para ello cuando añadimos los servicios de MVC6 en Startup, debemos agregar el formateador:
    
    <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:8053130a-70d0-430a-866d-c9c734662cbb" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
      <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
        <div style="background: #ddd; max-height: 300px; overflow: auto">
          <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
            <li>
              <span style="background:#ffffff;color:#000000">services.AddMvc(opt =></span>
            </li>
            <li>
              <span style="background:#ffffff;color:#000000">{</span>
            </li>
            <li>
                  <span style="background:#ffffff;color:#000000">opt.OutputFormatters.Add(</span><span style="background:#ffffff;color:#0000ff">new</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#2b91af">CsvOutputFormatter</span><span style="background:#ffffff;color:#000000">());</span>
            </li>
            <li>
              <span style="background:#ffffff;color:#000000">});</span>
            </li>
          </ol>
        </div></p>
      </div></p>
    </div>
    
    ¡Y listos! Ya no debemos hacer nada más. Ahora nos podemos crear una acción en un controlador:
    
    <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:8c032606-ea66-4d86-9356-f2bb1a44f66d" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
      <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
        <div style="background: #ddd; max-height: 300px; overflow: auto">
          <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
            <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">public</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#2b91af">IActionResult</span><span style="background:#ffffff;color:#000000"> Data()</span>
            </li>
            <li>
              <span style="background:#ffffff;color:#000000">{</span>
            </li>
            <li>
                  <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> beers = </span><span style="background:#ffffff;color:#0000ff">new</span><span style="background:#ffffff;color:#000000">[] {</span>
            </li>
            <li>
                      <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">new</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#2b91af">Beer</span><span style="background:#ffffff;color:#000000">() { Id=1, Name = </span><span style="background:#ffffff;color:#a31515">"Punk IPA"</span><span style="background:#ffffff;color:#000000">, Abv = 4.5 },</span>
            </li>
            <li>
                      <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">new</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#2b91af">Beer</span><span style="background:#ffffff;color:#000000">() { Id=2, Name = </span><span style="background:#ffffff;color:#a31515">"Mahou"</span>< span style="background:#ffffff;color:#000000">, Abv = 4.0 }</span>
            </li>
            <li>
                      <span style="background:#ffffff;color:#000000">};</span>
            </li>
            <li>
                  <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">new</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#2b91af">ObjectResult</span><span style="background:#ffffff;color:#000000">(beers);</span>
            </li>
            <li>
              <span style="background:#ffffff;color:#000000">}</span>
            </li>
          </ol>
        </div></p>
      </div></p>
    </div>
    
    Y si llamamos a esta acción con el valor “text/csv” en la cabecera _accept_ obtendremos los datos en CSV:
    
    [<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/04/image_thumb-6.png" width="640" height="153" />][1]
    
    Observa como el valor que colocamos en _accept_ es “text/csv, text/html” (el valor de _accept_ puede ser compuesto) y que a pesar de nosotros comprobamos con “text/csv” funciona igualmente, ya que MVC6 parsea la cabecera por nosotros (de hecho si en lugar de usar “text/csv,text/html” usaras “text/html,text/cvs”, primero MVC6 intentaria encontrar un formateador de HTML y si no lo encuentra intentaría encontrar del de CSV (si colocas un breakpoint en el método _CanWriteResult_ verías que pasa dos veces).
    
    Y eso es todo… como puedes ver es muy sencillo adaptar MVC6 para que use tus propios formatos de salida 😉_&nbsp;_
    
    Saludos!

 [1]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/04/image-6.png