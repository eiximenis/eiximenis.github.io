---
title: WebApi 2–Leer datos desde los headers
author: eiximenis

date: 2016-07-07T11:12:30+00:00
geeks_url: /?p=1777
geeks_ms_views:
  - 3481
  - 3481
categories:
  - webapi

---
En un curso que he impartido sobre WebApi 2 me han comentado un escenario en el que mandaban un conjunto de datos en varias cabeceras HTTP propias y querían leer esos datos desde los controladores.
  
La verdad es que hay varias maneras de hacer eso en WebApi 2 y vamos a analizar algunas de ellas en este post. Eso nos servirá como excusa para recorrer algunos de los mecanismos de extensibilidad del framework.
  
<!--more-->


  
**Para los ejemplos vamos a suponer que se envían dos cabeceras X-lang y X-version** **que son las que deseamos leer**.
  
Antes de nada, vamos a crear un poco de infraestructura que usaremos en todas las aproximaciones. Primero la clase “CustomHeaders” que contendrá los valores de los headers:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:a88a5cde-9cb7-4227-98b4-f3b93a2e9488" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap;" start="1">
        <li>
          <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">class</span> <span style="color: #2b91af;">CustomHeaders</span>
        </li>
        <li>
          <span style="color: #000000;">{</span>
        </li>
        <li>
              <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">string</span><span style="color: #000000;"> Lang { </span><span style="color: #0000ff;">get</span><span style="color: #000000;">; </span><span style="color: #0000ff;">set</span><span style="color: #000000;">; }</span>
        </li>
        <li>
              <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">int</span><span style="color: #000000;"> Version { </span><span style="color: #0000ff;">get</span><span style="color: #000000;">; </span><span style="color: #0000ff;">set</span><span style="color: #000000;">; }</span>
        </li>
        <li>
        </li>
        <li>
              <span style="color: #0000ff;">public</span><span style="color: #000000;"> CustomHeaders()</span>
        </li>
        <li>
              <span style="color: #000000;">{</span>
        </li>
        <li>
                  <span style="color: #000000;">Version = -1;</span>
        </li>
        <li>
              <span style="color: #000000;">}</span>
        </li>
        <li>
          <span style="color: #000000;">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Y luego un par de métodos de extensión sobre HttpRequestMessage para leer los headers y para obtener una propiedad de la petición (donde alguien debe haber guardado un objeto CustomHeaders):

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:f75901c7-d54f-4506-9704-00235fe7cc04" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap;" start="1">
        <li>
          <span style="color: #0000ff;">static</span> <span style="color: #0000ff;">class</span> <span style="color: #2b91af;">HttpRequestMessageExtensions</span>
        </li>
        <li>
          <span style="color: #000000;">{</span>
        </li>
        <li>
              <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">const</span> <span style="color: #0000ff;">string</span><span style="color: #000000;"> CustomHeadersKey = </span><span style="color: #0000ff;">nameof</span><span style="color: #000000;">(</span><span style="color: #2b91af;">CustomHeaders</span><span style="color: #000000;">);</span>
        </li>
        <li>
              <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">static</span> <span style="color: #2b91af;">CustomHeaders</span><span style="color: #000000;"> ReadCustomHeaders(</span><span style="color: #0000ff;">this</span> <span style="color: #2b91af;">HttpRequestMessage</span><span style="color: #000000;"> request)</span>
        </li>
        <li>
              <span style="color: #000000;">{</span>
        </li>
        <li>
                  <span style="color: #0000ff;">var</span><span style="color: #000000;"> customHeaders = </span><span style="color: #0000ff;">new</span> <span style="color: #2b91af;">CustomHeaders</span><span style="color: #000000;">();</span>
        </li>
        <li>
                  <span style="color: #0000ff;">var</span><span style="color: #000000;"> headers = request.Headers;</span>
        </li>
        <li>
        </li>
        <li>
                  <span style="color: #0000ff;">if</span><span style="color: #000000;"> (headers.Contains(</span><span style="color: #a31515;">&#8220;X-lang&#8221;</span><span style="color: #000000;">))</span>
        </li>
        <li>
                  <span style="color: #000000;">{</span>
        </li>
        <li>
                      <span style="color: #000000;">customHeaders.Lang = headers.GetValues(</span><span style="color: #a31515;">&#8220;X-lang&#8221;</span><span style="color: #000000;">).FirstOrDefault();</span>
        </li>
        <li>
                  <span style="color: #000000;">}</span>
        </li>
        <li>
                  <span style="color: #0000ff;">if</span><span style="color: #000000;"> (headers.Contains(</span><span style="color: #a31515;">&#8220;X-version&#8221;</span><span style="color: #000000;">))</span>
        </li>
        <li>
                  <span style="color: #000000;">{</span>
        </li>
        <li>
                      <span style="color: #0000ff;">var</span><span style="color: #000000;"> headerVersion = headers.GetValues(</span><span style="color: #a31515;">&#8220;X-version&#8221;</span><span style="color: #000000;">).FirstOrDefault();</span>
        </li>
        <li>
                      <span style="color: #0000ff;">if</span><span style="color: #000000;"> (!</span><span style="color: #0000ff;">string</span><span style="color: #000000;">.IsNullOrEmpty(headerVersion))</span>
        </li>
        <li>
                      <span style="color: #000000;">{</span>
        </li>
        <li>
                          <span style="color: #0000ff;">int</span><span style="color: #000000;"> version;</span>
        </li>
        <li>
                          <span style="color: #0000ff;">if</span><span style="color: #000000;"> (</span><span style="color: #0000ff;">int</span><span style="color: #000000;">.TryParse(headerVersion, </span><span style="color: #0000ff;">out</span><span style="color: #000000;"> version))</span>
        </li>
        <li>
                          <span style="color: #000000;">{</span>
        </li>
        <li>
                              <span style="color: #000000;">customHeaders.Version = version;</span>
        </li>
        <li>
                          <span style="color: #000000;">}</span>
        </li>
        <li>
                      <span style="color: #000000;">}</span>
        </li>
        <li>
        </li>
        <li>
                  <span style="color: #000000;">}</span>
        </li>
        <li>
                  <span style="color: #0000ff;">return</span><span style="color: #000000;"> customHeaders;</span>
        </li>
        <li>
              <span style="color: #000000;">}</span>
        </li>
        <li>
              <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">static</span> <span style="color: #0000ff;">void</span><span style="color: #000000;"> StoreCustomHeaders(</span><span style="color: #0000ff;">this</span> <span style="color: #2b91af;">HttpRequestMessage</span><span style="color: #000000;"> request, </span><span style="color: #2b91af;">CustomHeaders</span><span style="color: #000000;"> customHeaders)</span>
        </li>
        <li>
              <span style="color: #000000;">{</span>
        </li>
        <li>
                  <span style="color: #0000ff;">if</span><span style="color: #000000;"> (request.Properties.ContainsKey(CustomHeadersKey))</span>
        </li>
        <li>
                  <span style="color: #000000;">{</span>
        </li>
        <li>
                      <span style="color: #000000;">request.Properties[CustomHeadersKey] = customHeaders;</span>
        </li>
        <li>
                  <span style="color: #000000;">}</span>
        </li>
        <li>
                  <span style="color: #0000ff;">else</span>
        </li>
        <li>
                  <span style="color: #000000;">{</span>
        </li>
        <li>
                      <span style="color: #000000;">request.Properties.Add(CustomHeadersKey, customHeaders);</span>
        </li>
        <li>
                  <span style="color: #000000;">}</span>
        </li>
        <li>
              <span style="color: #000000;">}</span>
        </li>
        <li>
          <span style="color: #000000;">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Ahora ya podemos empezar a ver opciones para leer esos headers y usarlos en los controladores.
  
**1. MessageHandler que lea los headers y deje los datos en el contexto de la petición** 
  
En esta opción un MessageHandler procesa **todas las peticiones a WebApi,** lee las cabeceras y coloca sus datos en el contexto de la petición de WebApi:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:b25b7b31-6934-42df-8b6f-057bcf7090a1" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap;" start="1">
        <li>
          <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">class</span> <span style="color: #2b91af;">HeaderMessageHandler</span><span style="color: #000000;"> : </span><span style="color: #2b91af;">DelegatingHandler</span>
        </li>
        <li>
          <span style="color: #000000;">{</span>
        </li>
        <li>
              <span style="color: #0000ff;">protected</span> <span style="color: #0000ff;">override</span> <span style="color: #2b91af;">Task</span><span style="color: #000000;"><</span><span style="color: #2b91af;">HttpResponseMessage</span><span style="color: #000000;">> SendAsync(</span><span style="color: #2b91af;">HttpRequestMessage</span><span style="color: #000000;"> request, </span><span style="color: #2b91af;">CancellationToken</span><span style="color: #000000;"> cancellationToken)</span>
        </li>
        <li>
              <span style="color: #000000;">{</span>
        </li>
        <li>
                  <span style="color: #0000ff;">var</span><span style="color: #000000;"> customHeaders = request.ReadCustomHeaders();</span>
        </li>
        <li>
                  <span style="color: #000000;">request.StoreCustomHeaders(customHeaders);</span>
        </li>
        <li>
                  <span style="color: #0000ff;">return</span> <span style="color: #0000ff;">base</span><span style="color: #000000;">.SendAsync(request, cancellationToken);</span>
        </li>
        <li>
              <span style="color: #000000;">}</span>
        </li>
        <li>
          <span style="color: #000000;">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Luego necesitamos leer en los controladores el valor de los headers desde el contexto de la petición, donde los ha guardado el MessageHandler:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:721b83ff-5b17-4f44-af5a-8b71e112c461" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap;" start="1">
        <li>
          <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">class</span> <span style="color: #2b91af;">DemoController</span><span style="color: #000000;"> : </span><span style="color: #2b91af;">ApiController</span>
        </li>
        <li>
          <span style="color: #000000;">{</span>
        </li>
        <li>
              <span style="color: #000000;">[</span><span style="color: #2b91af;">Route</span><span style="color: #000000;">(</span><span style="color: #a31515;">&#8220;api/demo&#8221;</span><span style="color: #000000;">)]</span>
        </li>
        <li>
              <span style="color: #000000;">[</span><span style="color: #2b91af;">HttpGet</span><span style="color: #000000;">]</span>
        </li>
        <li>
              <span style="color: #000000;">[</span><span style="color: #2b91af;">ResponseType</span><span style="color: #000000;">(</span><span style="color: #0000ff;">typeof</span><span style="color: #000000;">(</span><span style="color: #2b91af;">CustomHeaders</span><span style="color: #000000;">))]</span>
        </li>
        <li>
              <span style="color: #0000ff;">public</span> <span style="color: #2b91af;">IHttpActionResult</span><span style="color: #000000;"> GetAll()</span>
        </li>
        <li>
              <span style="color: #000000;">{</span>
        </li>
        <li>
                  <span style="color: #0000ff;">return</span><span style="color: #000000;"> Ok(Request.Properties[</span><span style="color: #2b91af;">HttpRequestMessageExtensions</span><span style="color: #000000;">.CustomHeadersKey] </span><span style="color: #0000ff;">as</span> <span style="color: #2b91af;">CustomHeaders</span><span style="color: #000000;">);</span>
        </li>
        <li>
              <span style="color: #000000;">}</span>
        </li>
        <li>
          <span style="color: #000000;">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Por supuesto, si ves que es muy pesado hacer esto en cada acción siempre puedes derivar de un controlador base que haga eso de forma automática:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:0f654bc8-b573-4a5f-9b76-b4ac63698f19" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap;" start="1">
        <li>
          <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">class</span> <span style="color: #2b91af;">BaseController</span><span style="color: #000000;"> : </span><span style="color: #2b91af;">ApiController</span>
        </li>
        <li>
          <span style="color: #000000;">{</span>
        </li>
        <li>
              <span style="color: #0000ff;">protected</span> <span style="color: #2b91af;">CustomHeaders</span><span style="color: #000000;"> CustomHeaders {</span><span style="color: #0000ff;">get</span><span style="color: #000000;">; </span><span style="color: #0000ff;">private</span> <span style="color: #0000ff;">set</span><span style="color: #000000;">;}</span>
        </li>
        <li>
              <span style="color: #0000ff;">protected</span> <span style="color: #0000ff;">override</span> <span style="color: #0000ff;">void</span><span style="color: #000000;"> Initialize(</span><span style="color: #2b91af;">HttpControllerContext</span><span style="color: #000000;"> controllerContext)</span>
        </li>
        <li>
              <span style="color: #000000;">{</span>
        </li>
        <li>
                  <span style="color: #000000;">CustomHeaders = controllerContext.Request.Properties[</span><span style="color: #2b91af;">HttpRequestMessageExtensions</span><span style="color: #000000;">.CustomHeadersKey] </span><span style="color: #0000ff;">as</span> <span style="color: #2b91af;">CustomHeaders</span><span style="color: #000000;">;</span>
        </li>
        <li>
                  <span style="color: #0000ff;">base</span><span style="color: #000000;">.Initialize(controllerContext);</span>
        </li>
        <li>
              <span style="color: #000000;">}</span>
        </li>
        <li>
          <span style="color: #000000;">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Así nuestros controladores tienen siempre la propiedad “CustomHeaders” lista para ser leída:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:7ea7f808-3e13-4bdc-84ce-70a32ca909a9" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap;" start="1">
        <li>
          <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">class</span> <span style="color: #2b91af;">DemoController</span><span style="color: #000000;"> : </span><span style="color: #2b91af;">BaseController</span>
        </li>
        <li>
          <span style="color: #000000;">{</span>
        </li>
        <li>
              <span style="color: #000000;">[</span><span style="color: #2b91af;">Route</span><span style="color: #000000;">(</span><span style="color: #a31515;">&#8220;api/demo&#8221;</span><span style="color: #000000;">)]</span>
        </li>
        <li>
              <span style="color: #000000;">[</span><span style="color: #2b91af;">HttpGet</span><span style="color: #000000;">]</span>
        </li>
        <li>
              <span style="color: #000000;">[</span><span style="color: #2b91af;">ResponseType</span><span style="color: #000000;">(</span><span style="color: #0000ff;">typeof</span><span style="color: #000000;">(</span><span style="color: #2b91af;">CustomHeaders</span><span style="color: #000000;">))]</span>
        </li>
        <li>
              <span style="color: #0000ff;">public</span> <span style="color: #2b91af;">IHttpActionResult</span><span style="color: #000000;"> GetAll()</span>
        </li>
        <li>
              <span style="color: #000000;">{</span>
        </li>
        <li>
                  <span style="color: #0000ff;">return</span><span style="color: #000000;"> Ok(CustomHeaders);</span>
        </li>
        <li>
              <span style="color: #000000;">}</span>
        </li>
        <li>
          <span style="color: #000000;">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Por supuesto, incluso podríamos prescindir del MessageHandler y leer los headers en el _Initialize_ de nuestro controlador. Ten presente que el MessageHanlder actúa para cualquier petición a WebApi, **incluso aquellas que no terminan en un controlador** (porque no se enrutan correctamente). Por su parte colocarlo en el controlador tan solo lee los headers si la llamada llega a un controlador. Por lo tanto, si en lugar de leer headers debes hacer algo más costoso, eso debes tenerlo en consideración.
  
**2 – Filtros de Acción**
  
Un filtro de acción es otra posibilidad que tienes para procesar esos datos antes de que se invoque la acción:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:4039cba3-7e7d-4f31-b0a8-d5bb74fd244e" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap;" start="1">
        <li>
          <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">class</span> <span style="color: #2b91af;">ReadCustomHeadersAttribute</span><span style="color: #000000;"> : </span><span style="color: #2b91af;">ActionFilterAttribute</span>
        </li>
        <li>
          <span style="color: #000000;">{</span>
        </li>
        <li>
              <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">override</span> <span style="color: #2b91af;">Task</span><span style="color: #000000;"> OnActionExecutingAsync(</span><span style="color: #2b91af;">HttpActionContext</span><span style="color: #000000;"> actionContext, </span><span style="color: #2b91af;">CancellationToken</span><span style="color: #000000;"> cancellationToken)</span>
        </li>
        <li>
              <span style="color: #000000;">{</span>
        </li>
        <li>
                  <span style="color: #0000ff;">var</span><span style="color: #000000;"> request = actionContext.Request;</span>
        </li>
        <li>
                  <span style="color: #0000ff;">var</span><span style="color: #000000;"> customHeaders = request.ReadCustomHeaders();</span>
        </li>
        <li>
                  <span style="color: #000000;">request.StoreCustomHeaders(customHeaders);</span>
        </li>
        <li>
                  <span style="color: #0000ff;">return</span> <span style="color: #0000ff;">base</span><span style="color: #000000;">.OnActionExecutingAsync(actionContext, cancellationToken);</span>
        </li>
        <li>
              <span style="color: #000000;">}</span>
        </li>
        <li>
          <span style="color: #000000;">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Luego basta con aplicar el fitro a cualquier acción:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:fea983d2-e1ec-4952-bba0-064a3073bd7b" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap;" start="1">
        <li>
          <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">class</span> <span style="color: #2b91af;">DemoController</span><span style="color: #000000;"> : </span><span style="color: #2b91af;">ApiController</span>
        </li>
        <li>
          <span style="color: #000000;">{</span>
        </li>
        <li>
              <span style="color: #000000;">[</span><span style="color: #2b91af;">Route</span><span style="color: #000000;">(</span><span style="color: #a31515;">&#8220;api/demo&#8221;</span><span style="color: #000000;">)]</span>
        </li>
        <li>
              <span style="color: #000000;">[</span><span style="color: #2b91af;">HttpGet</span><span style="color: #000000;">]</span>
        </li>
        <li>
              <span style="color: #000000;">[</span><span style="color: #2b91af;">ReadCustomHeaders</span><span style="color: #000000;">]</span>
        </li>
        <li>
              <span style="color: #000000;">[</span><span style="color: #2b91af;">ResponseType</span><span style="color: #000000;">(</span><span style="color: #0000ff;">typeof</span><span style="color: #000000;">(</span><span style="color: #2b91af;">CustomHeaders</span><span style="color: #000000;">))]</span>
        </li>
        <li>
              <span style="color: #0000ff;">public</span> <span style="color: #2b91af;">IHttpActionResult</span><span style="color: #000000;"> GetAll()</span>
        </li>
        <li>
              <span style="color: #000000;">{</span>
        </li>
        <li>
                  <span style="color: #0000ff;">return</span><span style="color: #000000;"> Ok(Request.Properties[</span><span style="color: #2b91af;">HttpRequestMessageExtensions</span><span style="color: #000000;">.CustomHeadersKey] </span><span style="color: #0000ff;">as</span> <span style="color: #2b91af;">CustomHeaders</span><span style="color: #000000;">);</span>
        </li>
        <li>
              <span style="color: #000000;">}</span>
        </li>
        <li>
          <span style="color: #000000;">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Este mecanismo es mejor si solo hay algunas acciones que requieran leer los headers especiales (o hacer la acción correspondiente). Además queda explícito viendo el código qué acciones dependen de estos headers especiales (las decoradas con el atributo). Por supuesto podríamos añadir este filtro en los filtros globales de WebApi y entonces todas las acciones leerían los headers especiales.
  
**3 – Value provider**
  
Aquí usamos otro enfoque. Hasta ahora leíamos los headers y metíamos un objeto CustomHeaders en el contexto de la petición WebApi. Ahora vamos a añadir un parámetro de tipo “CustomHeaders” a las acciones que lo requieran y dejaremos que sea el proceso de model binding que se encargue de leerlo. Para ello debemos crear un ValueProvider para leer de los headers:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:263ebaf6-cfb5-49ef-8384-a83bf1ccec88" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap;" start="1">
        <li>
          <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">class</span> <span style="color: #2b91af;">HeaderValueProviderFactory</span><span style="color: #000000;"> : </span><span style="color: #2b91af;">ValueProviderFactory</span>
        </li>
        <li>
          <span style="color: #000000;">{</span>
        </li>
        <li>
              <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">override</span> <span style="color: #2b91af;">IValueProvider</span><span style="color: #000000;"> GetValueProvider(</span><span style="color: #2b91af;">HttpActionContext</span><span style="color: #000000;"> actionContext)</span>
        </li>
        <li>
              <span style="color: #000000;">{</span>
        </li>
        <li>
                  <span style="color: #0000ff;">return</span> <span style="color: #0000ff;">new</span> <span style="color: #2b91af;">HeaderValueProvider</span><span style="color: #000000;">(actionContext.Request);</span>
        </li>
        <li>
              <span style="color: #000000;">}</span>
        </li>
        <li>
          <span style="color: #000000;">}</span>
        </li>
        <li>
          <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">class</span> <span style="color: #2b91af;">HeaderValueProvider</span><span style="color: #000000;"> : </span><span style="color: #2b91af;">IValueProvider</span>
        </li>
        <li>
          <span style="color: #000000;">{</span>
        </li>
        <li>
              <span style="color: #0000ff;">private</span> <span style="color: #0000ff;">readonly</span> <span style="color: #2b91af;">HttpRequestMessage</span><span style="color: #000000;"> _request;</span>
        </li>
        <li>
              <span style="color: #0000ff;">private</span> <span style="color: #2b91af;">CustomHeaders</span><span style="color: #000000;"> _customHeaders;</span>
        </li>
        <li>
              <span style="color: #0000ff;">public</span><span style="color: #000000;"> HeaderValueProvider(</span><span style="color: #2b91af;">HttpRequestMessage</span><span style="color: #000000;"> request)</span>
        </li>
        <li>
              <span style="color: #000000;">{</span>
        </li>
        <li>
                  <span style="color: #000000;">_customHeaders = request.ReadCustomHeaders();</span>
        </li>
        <li>
              <span style="color: #000000;">}</span>
        </li>
        <li>
              <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">bool</span><span style="color: #000000;"> ContainsPrefix(</span><span style="color: #0000ff;">string</span><span style="color: #000000;"> prefix)</span>
        </li>
        <li>
              <span style="color: #000000;">{</span>
        </li>
        <li>
                  <span style="color: #0000ff;">return</span> <span style="color: #0000ff;">true</span><span style="color: #000000;">;</span>
        </li>
        <li>
              <span style="color: #000000;">}</span>
        </li>
        <li>
              <span style="color: #0000ff;">public</span> <span style="color: #2b91af;">ValueProviderResult</span><span style="color: #000000;"> GetValue(</span><span style="color: #0000ff;">string</span><span style="color: #000000;"> key)</span>
        </li>
        <li>
              <span style="color: #000000;">{</span>
        </li>
        <li>
                  <span style="color: #0000ff;">if</span><span style="color: #000000;"> (key == </span><span style="color: #0000ff;">nameof</span><span style="color: #000000;">(</span><span style="color: #2b91af;">CustomHeaders</span><span style="color: #000000;">.Lang) || key.EndsWith(</span><span style="color: #a31515;">&#8220;.&#8221;</span><span style="color: #000000;"> + </span><span style="color: #0000ff;">nameof</span><span style="color: #000000;">(</span><span style="color: #2b91af;">CustomHeaders</span><span style="color: #000000;">.Lang)))</span>
        </li>
        <li>
                  <span style="color: #000000;">{</span>
        </li>
        <li>
                      <span style="color: #0000ff;">return</span> <span style="color: #0000ff;">new</span> <span style="color: #2b91af;">ValueProviderResult</span><span style="color: #000000;">(_customHeaders.Lang, _customHeaders.Lang.ToString(), </span><span style="color: #2b91af;">CultureInfo</span><span style="color: #000000;">.InvariantCulture);</span>
        </li>
        <li>
                  <span style="color: #000000;">}</span>
        </li>
        <li>
                  <span style="color: #0000ff;">if</span><span style="color: #000000;"> (key == </span><span style="color: #0000ff;">nameof</span><span style="color: #000000;">(</span><span style="color: #2b91af;">CustomHeaders</span><span style="color: #000000;">.Version) || key.EndsWith(</span><span style="color: #a31515;">&#8220;.&#8221;</span><span style="color: #000000;"> + </span><span style="color: #0000ff;">nameof</span><span style="color: #000000;">(</span><span style="color: #2b91af;">CustomHeaders</span><span style="color: #000000;">.Version)))</span>
        </li>
        <li>
                  <span style="color: #000000;">{</span>
        </li>
        <li>
                      <span style="color: #0000ff;">return</span> <span style="color: #0000ff;">new</span> <span style="color: #2b91af;">ValueProviderResult</span><span style="color: #000000;">(_customHeaders.Version, _customHeaders.Version.ToString(), </span><span style="color: #2b91af;">CultureInfo</span><span style="color: #000000;">.InvariantCulture);</span>
        </li>
        <li>
                  <span style="color: #000000;">}</span>
        </li>
        <li>
                  <span style="color: #0000ff;">return</span> <span style="color: #0000ff;">null</span><span style="color: #000000;">;</span>
        </li>
        <li>
              <span style="color: #000000;">}</span>
        </li>
        <li>
          <span style="color: #000000;">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Ahora debemos añadir el parámetro de tipo “CustomHeaders” en las acciones que lo necesiten:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:f0540101-468d-432d-9e24-5275eb605fd4" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap;" start="1">
        <li>
          <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">class</span> <span style="color: #2b91af;">DemoController</span><span style="color: #000000;"> : </span><span style="color: #2b91af;">ApiController</span>
        </li>
        <li>
          <span style="color: #000000;">{</span>
        </li>
        <li>
              <span style="color: #000000;">[</span><span style="color: #2b91af;">Route</span><span style="color: #000000;">(</span><span style="color: #a31515;">&#8220;api/demo&#8221;</span><span style="color: #000000;">)]</span>
        </li>
        <li>
              <span style="color: #000000;">[</span><span style="color: #2b91af;">HttpGet</span><span style="color: #000000;">]</span>
        </li>
        <li>
              <span style="color: #000000;">[</span><span style="color: #2b91af;">HttpPost</span><span style="color: #000000;">]</span>
        </li>
        <li>
              <span style="color: #000000;">[</span><span style="color: #2b91af;">ResponseType</span><span style="color: #000000;">(</span><span style="color: #0000ff;">typeof</span><span style="color: #000000;">(</span><span style="color: #2b91af;">CustomHeaders</span><span style="color: #000000;">))]</span>
        </li>
        <li>
              <span style="color: #0000ff;">public</span> <span style="color: #2b91af;">IHttpActionResult</span><span style="color: #000000;"> GetAll([</span><span style="color: #2b91af;">ValueProvider</span><span style="color: #000000;">(</span><span style="color: #0000ff;">typeof</span><span style="color: #000000;">(</span><span style="color: #2b91af;">HeaderValueProviderFactory</span><span style="color: #000000;">))]</span><span style="color: #2b91af;">CustomHeaders</span><span style="color: #000000;"> ch)</span>
        </li>
        <li>
              <span style="color: #000000;">{</span>
        </li>
        <li>
                  <span style="color: #0000ff;">return</span><span style="color: #000000;"> Ok(ch);</span>
        </li>
        <li>
              <span style="color: #000000;">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Observa que la clave es que el parámetro está decorado con el atributo ValueProvider que le indica el Value Provider a utilizar. Así el model binder de WebApi sabe que debe usar este Value Provider para este parámetro.
  
Si quieres ver **una implementación genérica de un ValueProvider para enlazar parámetros desde los headers mira** <a href="http://www.codeproject.com/Tips/597804/Web-API-Custom-header-value-provider" target="_blank" rel="noopener noreferrer"><strong>este enlace</strong></a>**.**
  
**4 – Usar un Model Binder**
  
Esta opción es **conceptualmente errónea**, pero vamos a discutirla de todos modos. Digo que es conceptualmente errónea porque un model binder debería utilizarse cuando es el “formato de los datos” y no su “úbicación dentro de la petición” lo que cambia respecto a lo esperado por Web Api. P. ej. usaríamos un model binder para enlazar una query string de tipo ?fibo=1,1,2,3,5,8 a un IEnumerable<int>, porque WebApi no sabe por defecto enlazar IEnumerable<int> si vienen como una lista de enteros separados por comas. En este caso, como el formato de los datos (enteros separados por comas) es distinto de lo que espera Web Api es correcto usar un model binder. En nuestro caso lo suyo es usar un Value Provider como ya se ha visto en el punto anterior.
  
Aclarado esto, veamos como sería usando un model binder:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:947ec8b6-e630-4089-a386-f12d429b0ae9" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap;" start="1">
        <li>
          <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">class</span> <span style="color: #2b91af;">CustomHeadersModelBinder</span><span style="color: #000000;"> : </span><span style="color: #2b91af;">IModelBinder</span>
        </li>
        <li>
          <span style="color: #000000;">{</span>
        </li>
        <li>
              <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">bool</span><span style="color: #000000;"> BindModel(</span><span style="color: #2b91af;">HttpActionContext</span><span style="color: #000000;"> actionContext, </span><span style="color: #2b91af;">ModelBindingContext</span><span style="color: #000000;"> bindingContext)</span>
        </li>
        <li>
              <span style="color: #000000;">{</span>
        </li>
        <li>
                  <span style="color: #0000ff;">if</span><span style="color: #000000;"> (bindingContext.ModelType == </span><span style="color: #0000ff;">typeof</span><span style="color: #000000;">(</span><span style="color: #2b91af;">CustomHeaders</span><span style="color: #000000;">))</span>
        </li>
        <li>
                  <span style="color: #000000;">{</span>
        </li>
        <li>
                      <span style="color: #000000;">bindingContext.Model = actionContext.Request.ReadCustomHeaders();</span>
        </li>
        <li>
                      <span style="color: #0000ff;">return</span> <span style="color: #0000ff;">true</span><span style="color: #000000;">;</span>
        </li>
        <li>
                  <span style="color: #000000;">}</span>
        </li>
        <li>
                  <span style="color: #0000ff;">return</span> <span style="color: #0000ff;">false</span><span style="color: #000000;">;</span>
        </li>
        <li>
              <span style="color: #000000;">}</span>
        </li>
        <li>
          <span style="color: #000000;">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Luego en el controlador debemos indicar que el parámetro de tipo CustomHeaders usa dicho Model Binder:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:0c92bbea-73f2-4897-9654-4214aca3e59d" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap;" start="1">
        <li>
          <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">class</span> <span style="color: #2b91af;">DemoController</span><span style="color: #000000;"> : </span><span style="color: #2b91af;">ApiController</span>
        </li>
        <li>
          <span style="color: #000000;">{</span>
        </li>
        <li>
              <span style="color: #000000;">[</span><span style="color: #2b91af;">Route</span><span style="color: #000000;">(</span><span style="color: #a31515;">&#8220;api/demo&#8221;</span><span style="color: #000000;">)]</span>
        </li>
        <li>
              <span style="color: #000000;">[</span><span style="color: #2b91af;">HttpGet</span><span style="color: #000000;">]</span>
        </li>
        <li>
              <span style="color: #000000;">[</span><span style="color: #2b91af;">HttpPost</span><span style="color: #000000;">]</span>
        </li>
        <li>
              <span style="color: #000000;">[</span><span style="color: #2b91af;">ResponseType</span><span style="color: #000000;">(</span><span style="color: #0000ff;">typeof</span><span style="color: #000000;">(</span><span style="color: #2b91af;">CustomHeaders</span><span style="color: #000000;">))]</span>
        </li>
        <li>
              <span style="color: #0000ff;">public</span> <span style="color: #2b91af;">IHttpActionResult</span><span style="color: #000000;"> GetAll([</span><span style="color: #2b91af;">ModelBinder</span><span style="color: #000000;">(</span><span style="color: #0000ff;">typeof</span><span style="color: #000000;">(</span><span style="color: #2b91af;">CustomHeadersModelBinder</span><span style="color: #000000;">))]</span><span style="color: #2b91af;">CustomHeaders</span><span style="color: #000000;"> ch)</span>
        </li>
        <li>
              <span style="color: #000000;">{</span>
        </li>
        <li>
                  <span style="color: #0000ff;">return</span><span style="color: #000000;"> Ok(ch);</span>
        </li>
        <li>
              <span style="color: #000000;">}</span>
        </li>
        <li>
          <span style="color: #000000;">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Por supuesto podemos usar un ModelBinderProvider para el tipo CustomHeaders y así evitarnos poner el typeof en el [ModelBinder]:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:ebe7b3c9-ba87-4977-9047-259ec8da677e" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap;" start="1">
        <li>
          <span style="color: #0000ff;">class</span> <span style="color: #2b91af;">CustomHeadersModelBinderProvider</span><span style="color: #000000;"> : </span><span style="color: #2b91af;">ModelBinderProvider</span>
        </li>
        <li>
          <span style="color: #000000;">{</span>
        </li>
        <li>
              <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">override</span> <span style="color: #2b91af;">IModelBinder</span><span style="color: #000000;"> GetBinder(</span><span style="color: #2b91af;">HttpConfiguration</span><span style="color: #000000;"> configuration, </span><span style="color: #2b91af;">Type</span><span style="color: #000000;"> modelType)</span>
        </li>
        <li>
              <span style="color: #000000;">{</span>
        </li>
        <li>
                  <span style="color: #0000ff;">if</span><span style="color: #000000;"> (modelType == </span><span style="color: #0000ff;">typeof</span><span style="color: #000000;">(</span><span style="color: #2b91af;">CustomHeaders</span><span style="color: #000000;">))</span>
        </li>
        <li>
                  <span style="color: #000000;">{</span>
        </li>
        <li>
                      <span style="color: #0000ff;">return</span> <span style="color: #0000ff;">new</span> <span style="color: #2b91af;">CustomHeadersModelBinder</span><span style="color: #000000;">();</span>
        </li>
        <li>
                  <span style="color: #000000;">}</span>
        </li>
        <li>
                  <span style="color: #0000ff;">return</span> <span style="color: #0000ff;">null</span><span style="color: #000000;">;</span>
        </li>
        <li>
              <span style="color: #000000;">}</span>
        </li>
        <li>
          <span style="color: #000000;">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Luego debemos registrarlo en la configuración de WebApi:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:b1a8a4d6-3025-471f-893c-34739596a5a6" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap;" start="1">
        <li>
          <span style="color: #000000;">config.Services.Insert(</span><span style="color: #0000ff;">typeof</span><span style="color: #000000;">(</span><span style="color: #2b91af;">ModelBinderProvider</span><span style="color: #000000;">),0, </span><span style="color: #0000ff;">new</span> <span style="color: #2b91af;">CustomHeadersModelBinderProvider</span><span style="color: #000000;">());</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Y ahora el controlador nos queda de la siguiente forma:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:fc081b5c-d47f-4e47-b97e-e2dccf1ebd0b" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap;" start="1">
        <li>
          <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">class</span> <span style="color: #2b91af;">DemoController</span><span style="color: #000000;"> : </span><span style="color: #2b91af;">ApiController</span>
        </li>
        <li>
          <span style="color: #000000;">{</span>
        </li>
        <li>
              <span style="color: #000000;">[</span><span style="color: #2b91af;">Route</span><span style="color: #000000;">(</span><span style="color: #a31515;">&#8220;api/demo&#8221;</span><span style="color: #000000;">)]</span>
        </li>
        <li>
              <span style="color: #000000;">[</span><span style="color: #2b91af;">HttpGet</span><span style="color: #000000;">]</span>
        </li>
        <li>
              <span style="color: #000000;">[</span><span style="color: #2b91af;">HttpPost</span><span style="color: #000000;">]</span>
        </li>
        <li>
              <span style="color: #000000;">[</span><span style="color: #2b91af;">ResponseType</span><span style="color: #000000;">(</span><span style="color: #0000ff;">typeof</span><span style="color: #000000;">(</span><span style="color: #2b91af;">CustomHeaders</span><span style="color: #000000;">))]</span>
        </li>
        <li>
              <span style="color: #0000ff;">public</span> <span style="color: #2b91af;">IHttpActionResult</span><span style="color: #000000;"> GetAll([</span><span style="color: #2b91af;">ModelBinder</span><span style="color: #000000;">] </span><span style="color: #2b91af;">CustomHeaders</span><span style="color: #000000;"> ch)</span>
        </li>
        <li>
              <span style="color: #000000;">{</span>
        </li>
        <li>
                  <span style="color: #0000ff;">return</span><span style="color: #000000;"> Ok(ch);</span>
        </li>
        <li>
              <span style="color: #000000;">}</span>
        </li>
        <li>
          <span style="color: #000000;">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Observa que sigue siendo necesario decorar el parámetro con [ModelBinder] pero no es necesario especificar el tipo, ya que ya lo hemos dado de alta en la configuración.
  
**En resúmen…**
  
Hemos visto cuatro estrategias para leer datos desde los headers de una petición y acceder a ellos desde nuestros controladores. Como digo el uso de un custom model binder para esto no es conceptualmente correcto y de las otras tres cada una tiene sus ventajas e incovenientes:

  * Message Handler: Bueno para centralizar tareas a realizar en todas las peticiones de Web Api, lleguen estas a un controlador o no.
  * Filtro de acción: Bueno para centralizar tareas a realizar en algunas acciones de uno o más controladores. Si son globales pueden afectar a todas las acciones de todos los controladores.
  * Value Provider: Bueno para añadir orígenes de datos alternativos a los existentes por defecto en Web Api, para enlazar parámetros. De todos modos, asegúrate de entender las reglas del model binding de Web Api que no son, precisamente, sencillas.

Espero que el post te sea interesante!
  
Saludos!