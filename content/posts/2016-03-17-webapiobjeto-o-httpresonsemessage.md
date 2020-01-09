---
title: WebApi–¿objeto o HttpResonseMessage?
description: WebApi–¿objeto o HttpResonseMessage?
author: eiximenis

date: 2016-03-17T12:32:13+00:00
geeks_url: /?p=1736
geeks_ms_views:
  - 4147
categories:
  - webapi

---
El otro día hablando con Carlos (aka <a href="https://twitter.com/3lcarry" target="_blank" rel="noopener noreferrer">@3lcarry</a>) surgió el tema de **como devolver datos en una WebApi**. En WebApi tenemos hasta tres maneras generales de devolver unos datos desde una acción de un controlador. Devolver un objeto, devolver un HttpResponseMessage o devolver un IHttpActionResult. (Una discusión equivalente aparece en MVC6 donde tenemos dos alternativas, devolver un objeto o bien un IActionResult.) ¿Cuál es la mejor?
  
<!--more-->

**Devolver un objeto**
  
En este caso nuestro controlador tiene un código como el siguiente:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:01d50b34-38e2-424a-b6a9-4499b5a2b9f4" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px;" start="1">
        <li>
          <span style="background: #ffffff; color: #0000ff;">public</span> <span style="background: #ffffff; color: #2b91af;">IEnumerable</span><span style="background: #ffffff; color: #000000;"><</span><span style="background: #ffffff; color: #0000ff;">string</span><span style="background: #ffffff; color: #000000;">> Get()</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">{</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #0000ff;">return</span> <span style="background: #ffffff; color: #0000ff;">new</span> <span style="background: #ffffff; color: #0000ff;">string</span><span style="background: #ffffff; color: #000000;">[] { </span><span style="background: #ffffff; color: #a31515;">&#8220;value1&#8221;</span><span style="background: #ffffff; color: #000000;">, </span><span style="background: #ffffff; color: #a31515;">&#8220;value2&#8221;</span><span style="background: #ffffff; color: #000000;"> };</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Cuando devolvemos un objeto, WebApi usa el serializador (formatter en la jerga de WebApi) correspondiente, serializa el objeto y lo envía de respuesta.
  
La ventaja principal de devolver un objeto es que simplemente viendo la firma del método de acción, ya se puede ver que tipo de datos se devuelven. Y el compilador, por supuesto, nos verificará que estemos devolviendo el tipo correcto. En este ejemplo el compilador nos deja devolver un array de cadenas, pero no podríamos devolver un array de enteros. También al tener la firma en el método de acción, herramientas como Swagger pueden generar documentación indicando que este método devuelve un conjunto de cadenas.
  
Parece la forma perfecta, ¿verdad? Pero, si lo fuese, no existiría este post, claro. El principal problema que pagamos **es que WebApi siempre devuelve un código HTTP 200**. No tenemos manera de indicar qué codigo HTTP queremos establecer.
  
Supongamos un código como el siguiente:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:48d06404-61d6-4e02-8cb0-df5ab12c5ee0" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px;" start="1">
        <li>
          <span style="background: #ffffff; color: #0000ff;">public</span> <span style="background: #ffffff; color: #0000ff;">string</span><span style="background: #ffffff; color: #000000;"> Get(</span><span style="background: #ffffff; color: #0000ff;">int</span><span style="background: #ffffff; color: #000000;"> id)</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">{</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #0000ff;">if</span><span style="background: #ffffff; color: #000000;"> (id > 1) { </span><span style="background: #ffffff; color: #0000ff;">return</span> <span style="background: #ffffff; color: #a31515;">&#8220;value&#8221;</span><span style="background: #ffffff; color: #000000;">; }</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #0000ff;">else</span><span style="background: #ffffff; color: #000000;"> { </span><span style="background: #ffffff; color: #0000ff;">return</span> <span style="background: #ffffff; color: #0000ff;">null</span><span style="background: #ffffff; color: #000000;">; }</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

En una aplicación real haríamos una consulta a nuestro repositorio de datos y buscaríamos por el id. La pregunta es… ¿qué hacemos si no existe el elemento? En este caso devolvemos null. No parece una mala opción, pero el problema es que nos estamos pasando REST por el forro:
  
[<img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/03/image_thumb.png" alt="image" width="640" height="232" border="0" />][1]
  
Observa que hacemos una petición pasando un 1 como id, y lo que obtenemos es un 200, con el null serializado. Esto no es para nada bonito y como digo antes rompe REST. En este caso lo suyo sería devolver un 404. Pero no tenemos manera de hacerlo, porque Web Api no nos deja establecer el código HTTP.
  
Vale, estoy mintiendo. Sí que tenemos una manera de hacerlo, que es la siguiente:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:ae79e8bf-05cd-41e8-acf3-421003c1019d" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px;" start="1">
        <li>
          <span style="background: #ffffff; color: #0000ff;">public</span> <span style="background: #ffffff; color: #0000ff;">string</span><span style="background: #ffffff; color: #000000;"> Get(</span><span style="background: #ffffff; color: #0000ff;">int</span><span style="background: #ffffff; color: #000000;"> id)</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">{</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #0000ff;">if</span><span style="background: #ffffff; color: #000000;"> (id > 1) { </span><span style="background: #ffffff; color: #0000ff;">return</span> <span style="background: #ffffff; color: #a31515;">&#8220;value&#8221;</span><span style="background: #ffffff; color: #000000;">; }</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #0000ff;">else</span><span style="background: #ffffff; color: #000000;"> { </span><span style="background: #ffffff; color: #0000ff;">throw</span> <span style="background: #ffffff; color: #0000ff;">new</span> <span style="background: #ffffff; color: #2b91af;">HttpResponseException</span><span style="background: #ffffff; color: #000000;">(</span><span style="background: #ffffff; color: #2b91af;">HttpStatusCode</span><span style="background: #ffffff; color: #000000;">.NotFound); }</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

La solución **pasa por lanzar una excepción para establecer el código HTTP**.  Ahora sí que si pedimos el id 1 obtenemos nuestro 404. He de decir que **no me gusta para nada esa aproximación**, las excepciones deberían ser para eso… cosas excepcionales. Buscar un id y no encontrarlo no tiene nada de excepcional. Es control de flujo.
  
Vale… alguien puede argumentar que tiene lógica tratar eso como un error (de hecho 404 es un código de error). Muy bien, pero entonces **imagina que quieres devolver un 201** (código que se usa para indicar que una entidad ha sido creada). De verdad ¿vas a lanzar una excepción para devolver un 201? El código 201 no es, para nada, un código de error. Es un código de éxito, que indica que la petición ha sido procesada, y el resultado de procesar dicha petición ha implicado la creación de alguna entidad (que se puede mandar en el cuerpo). Si usas una API y quieres ser realmente RESTful deberías mandar códigos de éxito un poco más específicos que el 200, que para eso existen.
  
Es en este punto donde entra la otra alternativa. Ya sea devolviendo un HttpResponseMessage o un IHttpActionResult (el segundo se introdujo en WebApi 2 y no es más que una interfaz que define una factoría para crear objetos del primero).
  
En este caso, al especificar un mensaje de respuesta, tenemos mucho más control que en el caso anterior:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:d121d55c-1584-4567-a69b-cf7c8d807432" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px;" start="1">
        <li>
          <span style="background: #ffffff; color: #0000ff;">public</span> <span style="background: #ffffff; color: #2b91af;">HttpResponseMessage</span><span style="background: #ffffff; color: #000000;"> Get(</span><span style="background: #ffffff; color: #0000ff;">int</span><span style="background: #ffffff; color: #000000;"> id)</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">{</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #0000ff;">if</span><span style="background: #ffffff; color: #000000;"> (id > 1) { </span><span style="background: #ffffff; color: #0000ff;">return</span><span style="background: #ffffff; color: #000000;"> Request.CreateResponse<</span><span style="background: #ffffff; color: #0000ff;">string</span><span style="background: #ffffff; color: #000000;">>(</span><span style="background: #ffffff; color: #a31515;">&#8220;value&#8221;</span><span style="background: #ffffff; color: #000000;">); }</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #0000ff;">else</span><span style="background: #ffffff; color: #000000;"> { </span><span style="background: #ffffff; color: #0000ff;">return</span><span style="background: #ffffff; color: #000000;"> Request.CreateErrorResponse(</span><span style="background: #ffffff; color: #2b91af;">HttpStatusCode</span><span style="background: #ffffff; color: #000000;">.NotFound, </span><span style="background: #ffffff; color: #a31515;">&#8220;invalid id&#8221;</span><span style="background: #ffffff; color: #000000;">); }</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

El método CreateResponse espera un objeto **y opcionalmente un código HTTP.** Si no lo colocamos manda el 200, pero podemos indicar el que queramos. Por otro lado el CreateErrorResponse, espera un código HTTP y un mensaje de error a mandar en el cuerpo.
  
No tenemos que usar excepciones para casos controlados pero ahora pagamos un precio: leyendo la firma no tenemos nada claro que devuelve esta acción, de hecho podría devolver cualquier cosa (incluso cosas distintas en distintos returns, lo que según cuando puede ser otra ventaja). Pero… ¿como podemos documentar que este método devuelve una cadena?
  
He habilitado **Swagger** y el resultado para este controlador es el siguiente:
  
[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/03/image_thumb-1.png" alt="image" width="633" height="480" border="0" />][2]
  
El primer es un método que devuelve un IEnumerable<string> y el segundo un método que devuelte un HttpResponseMessage. Se puede ver como en el primer caso Swagger es capaz de inferir el tipo de respuesta, pero en el segundo caso no. Tiene lógica. No es tan malo como parece, desde la propia página de Swagger podemos probar los métodos y ver realmente qué nos devuelven. Pero… ¿hay alguna manera de indicar que el segundo método devuelve una cadena?
  
Pues sí, podemos usar el atributo ResponseType:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:20c615cc-1a43-4738-8dec-ef14b76d4773" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px;" start="1">
        <li>
          <span style="background: #ffffff; color: #000000;">[</span><span style="background: #ffffff; color: #2b91af;">ResponseType</span><span style="background: #ffffff; color: #000000;">(</span><span style="background: #ffffff; color: #0000ff;">typeof</span><span style="background: #ffffff; color: #000000;">(</span><span style="background: #ffffff; color: #0000ff;">string</span><span style="background: #ffffff; color: #000000;">)]</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #0000ff;">public</span> <span style="background: #ffffff; color: #2b91af;">HttpResponseMessage</span><span style="background: #ffffff; color: #000000;"> Get(</span><span style="background: #ffffff; color: #0000ff;">int</span><span style="background: #ffffff; color: #000000;"> id)</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">{</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #0000ff;">if</span><span style="background: #ffffff; color: #000000;"> (id > 1) { </span><span style="background: #ffffff; color: #0000ff;">return</span><span style="background: #ffffff; color: #000000;"> Request.CreateResponse<</span><span style="background: #ffffff; color: #0000ff;">string</span><span style="background: #ffffff; color: #000000;">>(</span><span style="background: #ffffff; color: #a31515;">&#8220;value&#8221;</span><span style="background: #ffffff; color: #000000;">); }</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #0000ff;">else</span><span style="background: #ffffff; color: #000000;"> { </span><span style="background: #ffffff; color: #0000ff;">return</span><span style="background: #ffffff; color: #000000;"> Request.CreateErrorResponse(</span><span style="background: #ffffff; color: #2b91af;">HttpStatusCode</span><span style="background: #ffffff; color: #000000;">.NotFound, </span><span style="background: #ffffff; color: #a31515;">&#8220;invalid id&#8221;</span><span style="background: #ffffff; color: #000000;">); }</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

ResponseType añade metadatos que indican que la acción devuelve objetos de un determinado tipo. Entonces Swagger (u otra herramienta) puede usar esos metadatos para dar más información al generar la documentación.
  
Por supuesto es responsabilidad nuestra hacer que la acción devuelva realmente un objeto del tipo indicado en ResponseType, aquí el compilador no puede ayudarnos.
  
Personalmente pienso que devolver un HttpResponseMessage (o una IHttpActionResult) es mucho más elegante que devolver un objeto. Gracias al uso de ResponseType las APIs pueden estar igualmente documentadas, y a cambio tenemos mucho más control sobre los códigos HTTP que usamos y no debemos usar excepciones.
  
Pero… ¿Y tú, qué opinas?

 [1]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/03/image.png
 [2]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/03/image-1.png