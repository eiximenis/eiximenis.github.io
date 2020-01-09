---
title: Creating middlewares de asp.net core
author: eiximenis

date: 2016-02-29T17:08:12+00:00
geeks_url: /?p=1729
geeks_ms_views:
  - 3079
categories:
  - asp.net 5
  - asp.net vNext

---
Asp.net core se basa en el concepto de **middleware**. En este modelo la petición web viaja a través de un conjunto de componentes. Cada componente recibe la petición y puede:

  1. Modificar la petición **y enviarla al siguiente componente**
  2. O bien, generar una respuesta **y enviarla de vuelta al componente anterior**.

<!--more-->


  
De la misma manera, cuando un componente recibe una respuesta puede modificarla (o no) y enviarla al componente anterior. Cuando la respuesta llega al primer componente, es enviado de vuelta al cliente (navegador):
  
[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/02/image_thumb.png" alt="image" width="640" height="210" border="0" />][1]
  
Esta imagen muestra dos casos. En el primero, la petición pasa por los cuatro componentes. Cada componente _puede_ modificar la petición con lo que considere oportuno antes de pasarla al siguiente componente. La petición llega al último componente que genera una respuesta, que es enviada “para atrás” pasando por todos los componentes, hasta llegar al cliente.
  
En el segundo caso, el segundo componente ha decidido “cortocircuitar” la petición **y ya no la envía al siguiente componente**. En su lugar genera una respuesta, y la envía “hacia atrás” hasta que llega de nuevo al cliente.
  
A nivel de nomenclatura, a cada uno de esos componentes les llamamos _middleware_ y al conjunto de componentes lo llamamos el _pipeline_ de la aplicación.
  
**Ventajas del uso de middlewares**
  
La ventaja principal de los middlewares es que componentizan el framework y permiten reaprovechar muchas partes de él. Supongamos un framework que no use este concepto, tal y como es ASP.NET 4.5 **sin usar OWIN**. En este caso montamos un framework sobre ASP.NET 4.5, tal y como puede ser MVC5. Este framework debe encargarse de todo, incluyendo la autenticación y la autorización. Ahora alguien quiere montar otro framework distinto, pongamos NancyFx. Pues de nuevo NancyFx debe tener sus mecanismos de autenticación y autorización propios. Harán exactamente lo mismo (validar una cookie) pero son códigos distintos.
  
¿No sería mucho mejor poder tener un sistema de validación de cookies único? ¿Y que cualquier framework que se montase sobre él pudiese reaprovecharlo? Pues eso es lo que trae el uso de middlewares. En el ejemplo anterior el segundo middleware (el recuadro naranja) podría ser un framework de autenticación, que solo haría eso: validar la cookie. En el caso de no existir, cortocircuita la petición y envía una respuesta de error.
  
En este modelo, tanto MVC como NancyFx se pueden despreocupar de la autenticación de los usuarios: esa tarea recae en un middleware externo, que se sitúa antes en el pipeline de la aplicación.
  
**Nota:** En ASP.NET tradicional, pre-OWIN se pueden crear también componentes parecidos a los middlewares (a través de HttpModules) pero el concepto no es ni tan práctico, ni tan sencillo como el concepto de middleware en OWIN o en ASP.NET Core.
  
Esa idea apareció en ASP.NET de la mano de OWIN y en ASP.NET Core se ha mantenido. Aunque **los middlewares ASP.NET Core no son middlewares OWIN**, la idea es la misma.
  
**¿Qué es un middleware ASP.NET Core?**
  
Hemos visto, a nivel conceptual, qué es un middleware. Veamos ahora en detalle qué es un middleware ASP.NET Core. Pues en el fondo un middleware ASP.NET Core se puede conceptualizar como un par de funciones:

  1. f(req) = req’
  2. f(res) = res’

Es decir, una función toma una petición y genera otra petición (igual o modificada) y otra toma una respuesta y genera otra respuesta (igual o modificada). Viendo esto uno podría pensar que existe una interfaz tipo IMiddleware o algo así, pero no. Se usa más un sistema basado en convenciones. Veamos como definir un middleware en C#:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:b03a2e03-6cf4-425b-ba57-08f71a53ef69" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px;" start="1">
        <li>
          <span style="background: #ffffff; color: #0000ff;">public</span> <span style="background: #ffffff; color: #0000ff;">class</span> <span style="background: #ffffff; color: #2b91af;">CustomMiddleware</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">{</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #0000ff;">private</span> <span style="background: #ffffff; color: #0000ff;">readonly</span> <span style="background: #ffffff; color: #2b91af;">RequestDelegate</span><span style="background: #ffffff; color: #000000;"> _next;</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #0000ff;">public</span><span style="background: #ffffff; color: #000000;"> CustomMiddleware(</span><span style="background: #ffffff; color: #2b91af;">RequestDelegate</span><span style="background: #ffffff; color: #000000;"> next)</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">{</span>
        </li>
        <li>
                  <span style="background: #ffffff; color: #000000;">_next = next;</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">}</span>
        </li>
        <li>
        </li>
        <li>
              <span style="background: #ffffff; color: #0000ff;">public</span> <span style="background: #ffffff; color: #0000ff;">async</span> <span style="background: #ffffff; color: #2b91af;">Task</span><span style="background: #ffffff; color: #000000;"> Invoke(</span><span style="background: #ffffff; color: #2b91af;">HttpContext</span><span style="background: #ffffff; color: #000000;"> context)</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">{</span>
        </li>
        <li>
                  <span style="background: #ffffff; color: #0000ff;">await</span><span style="background: #ffffff; color: #000000;"> _next.Invoke(context);</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">}</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Esto define un middleware ASP.NET Core. Observa que la clase no hereda de ninguna clase, ni implementa interfaz alguna. Las convenciones son:

  * Que exista un constructor que acepte un RequestDelegate
  * Que exista un método Invoke que acepte un HttpContext

Observa que es nuestra responsabilidad llamar al siguiente middleware. Eso es lo que nos permite cortocircuitar la petición (simplemente no lo llamamos y listo).
  
**Nota:** De hecho es posible incluso tener un middleware que no sea ni una clase. A nivel del framework un middleware es realmente una Func<RequestDelegate, RequestDelegate> o una Func<HttpContext, Func<Task>, Task> pero, en mi opinión, es más claro (al menos en C#) usar una clase con esas convenciones.
  
Este middleware no hace nada, ni modifica la respuesta. Pero ya podríamos hacer lo que quisieramos. Veamos un ejemplo: si la petición contiene una cabecera X-User vamos a autenticar la petición con este usuario:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:fbda7f33-04ad-4ac0-9102-b023846ceb9e" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px;" start="1">
        <li>
          <span style="background: #ffffff; color: #0000ff;">public</span> <span style="background: #ffffff; color: #0000ff;">async</span> <span style="background: #ffffff; color: #2b91af;">Task</span><span style="background: #ffffff; color: #000000;"> Invoke(</span><span style="background: #ffffff; color: #2b91af;">HttpContext</span><span style="background: #ffffff; color: #000000;"> context)</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">{</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #0000ff;">if</span><span style="background: #ffffff; color: #000000;"> (context.Request.Headers.Keys.Contains(</span><span style="background: #ffffff; color: #a31515;">&#8220;X-User&#8221;</span><span style="background: #ffffff; color: #000000;">))</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">{</span>
        </li>
        <li>
                  <span style="background: #ffffff; color: #0000ff;">var</span><span style="background: #ffffff; color: #000000;"> user = context.Request.Headers[</span><span style="background: #ffffff; color: #a31515;">&#8220;X-User&#8221;</span><span style="background: #ffffff; color: #000000;">].ToString();</span>
        </li>
        <li>
                  <span style="background: #ffffff; color: #0000ff;">var</span><span style="background: #ffffff; color: #000000;"> claim = </span><span style="background: #ffffff; color: #0000ff;">new</span> <span style="background: #ffffff; color: #2b91af;">Claim</span><span style="background: #ffffff; color: #000000;">(</span><span style="background: #ffffff; color: #2b91af;">ClaimTypes</span><span style="background: #ffffff; color: #000000;">.Name, user);</span>
        </li>
        <li>
                  <span style="background: #ffffff; color: #0000ff;">var</span><span style="background: #ffffff; color: #000000;"> principal = </span><span style="background: #ffffff; color: #0000ff;">new</span> <span style="background: #ffffff; color: #2b91af;">ClaimsPrincipal</span><span style="background: #ffffff; color: #000000;">(</span><span style="background: #ffffff; color: #0000ff;">new</span> <span style="background: #ffffff; color: #2b91af;">ClaimsIdentity</span><span style="background: #ffffff; color: #000000;">(</span><span style="background: #ffffff; color: #0000ff;">new</span><span style="background: #ffffff; color: #000000;">[] { claim }, </span><span style="background: #ffffff; color: #a31515;">&#8220;xuser&#8221;</span><span style="background: #ffffff; color: #000000;">));</span>
        </li>
        <li>
                  <span style="background: #ffffff; color: #000000;">context.User = principal;</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">}</span>
        </li>
        <li>
        </li>
        <li>
              <span style="background: #ffffff; color: #0000ff;">await</span><span style="background: #ffffff; color: #000000;"> _next.Invoke(context);</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Nos falta una cosa, que es **como añadir nuestro middleware al pipeline** de la aplicación. Para ello basta con usar el método UseMiddleware<T> donde T es el tipo de nuestro middleware:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:2cd55dbf-5efd-4a7c-a1eb-68a265c78b7c" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px;" start="1">
        <li>
          <span style="background: #ffffff; color: #0000ff;">public</span> <span style="background: #ffffff; color: #0000ff;">void</span><span style="background: #ffffff; color: #000000;"> Configure(</span><span style="background: #ffffff; color: #2b91af;">IApplicationBuilder</span><span style="background: #ffffff; color: #000000;"> app)</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">{</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">app.UseIISPlatformHandler();</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">app.UseMiddleware<</span><span style="background: #ffffff; color: #2b91af;">CustomMiddleware</span><span style="background: #ffffff; color: #000000;">>();</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">app.UseMvcWithDefaultRoute();</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

En este ejemplo tengo también MVC6 instalado en el pipeline (MVC6 es simplemente otro middleware) y ahora puedo tener un controlador como el que sigue:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:d454276d-edc3-4a62-b3db-8b8f8fc74f41" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px;" start="1">
        <li>
          <span style="background: #ffffff; color: #0000ff;">public</span> <span style="background: #ffffff; color: #0000ff;">class</span> <span style="background: #ffffff; color: #2b91af;">HomeController</span><span style="background: #ffffff; color: #000000;"> : </span><span style="background: #ffffff; color: #2b91af;">Controller</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">{</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">[</span><span style="background: #ffffff; color: #2b91af;">Authorize</span><span style="background: #ffffff; color: #000000;">]</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #0000ff;">public</span> <span style="background: #ffffff; color: #2b91af;">IActionResult</span><span style="background: #ffffff; color: #000000;"> Index()</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">{</span>
        </li>
        <li>
                  <span style="background: #ffffff; color: #0000ff;">return</span><span style="background: #ffffff; color: #000000;"> Ok(</span><span style="background: #ffffff; color: #0000ff;">new</span><span style="background: #ffffff; color: #000000;"> { user = User.Identity.Name });</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">}</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Observa que la acción Index está protegida por [Authorize]. Pero [Authorize] lo único que hace es validar que haya una identidad autenticada en el contexto… que es precisamente lo que ha dejado nuestro CustomMiddleware. El resultado, ya te lo puedes imaginar, es que si realizamos una petición con X-User se puede acceder a la acción, como muestra esta captura de Postman:
  
[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/02/image_thumb-1.png" alt="image" width="640" height="253" border="0" />][2]
  
Si no envías la cabecera X-User, entonces recibes el 401, que es lo que envía Authorize, por defecto:
  
[<img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/02/image_thumb-2.png" alt="image" width="640" height="244" border="0" />][3]
  
Hemos visto como nuestro middleware puede modificar la petición (o el contexto) antes de mandarlo al siguiente middleware (que en nuestro caso es MVC6). Pero hagamos otra cosa. Veamos como podemos modificar la respuesta antes de enviarla al cliente.
  
Para ello, cambiaremos el StatusCode. Si es 401, mandaremos un 500. El método Invoke ahora queda:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:c74921b9-b8fc-4545-8d2c-901c5a88d5bf" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px;" start="1">
        <li>
          <span style="background: #ffffff; color: #0000ff;">public</span> <span style="background: #ffffff; color: #0000ff;">async</span> <span style="background: #ffffff; color: #2b91af;">Task</span><span style="background: #ffffff; color: #000000;"> Invoke(</span><span style="background: #ffffff; color: #2b91af;">HttpContext</span><span style="background: #ffffff; color: #000000;"> context)</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">{</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #0000ff;">if</span><span style="background: #ffffff; color: #000000;"> (context.Request.Headers.Keys.Contains(</span><span style="background: #ffffff; color: #a31515;">&#8220;X-User&#8221;</span><span style="background: #ffffff; color: #000000;">))</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">{</span>
        </li>
        <li>
                  <span style="background: #ffffff; color: #0000ff;">var</span><span style="background: #ffffff; color: #000000;"> user = context.Request.Headers[</span><span style="background: #ffffff; color: #a31515;">&#8220;X-User&#8221;</span><span style="background: #ffffff; color: #000000;">].ToString();</span>
        </li>
        <li>
                  <span style="background: #ffffff; color: #0000ff;">var</span><span style="background: #ffffff; color: #000000;"> claim = </span><span style="background: #ffffff; color: #0000ff;">new</span> <span style="background: #ffffff; color: #2b91af;">Claim</span><span style="background: #ffffff; color: #000000;">(</span><span style="background: #ffffff; color: #2b91af;">ClaimTypes</span><span style="background: #ffffff; color: #000000;">.Name, user);</span>
        </li>
        <li>
                  <span style="background: #ffffff; color: #0000ff;">var</span><span style="background: #ffffff; color: #000000;"> principal = </span><span style="background: #ffffff; color: #0000ff;">new</span> <span style="background: #ffffff; color: #2b91af;">ClaimsPrincipal</span><span style="background: #ffffff; color: #000000;">(</span><span style="background: #ffffff; color: #0000ff;">new</span> <span style="background: #ffffff; color: #2b91af;">ClaimsIdentity</span><span style="background: #ffffff; color: #000000;">(</span><span style="background: #ffffff; color: #0000ff;">new</span><span style="background: #ffffff; color: #000000;">[] { claim }, </span><span style="background: #ffffff; color: #a31515;">&#8220;xuser&#8221;</span><span style="background: #ffffff; color: #000000;">));</span>
        </li>
        <li>
                  <span style="background: #ffffff; color: #000000;">context.User = principal;</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">}</span>
        </li>
        <li>
        </li>
        <li>
              <span style="background: #ffffff; color: #0000ff;">await</span><span style="background: #ffffff; color: #000000;"> _next.Invoke(context);</span>
        </li>
        <li>
        </li>
        <li>
              <span style="background: #ffffff; color: #0000ff;">if</span><span style="background: #ffffff; color: #000000;"> (context.Response.StatusCode == 401)</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">{</span>
        </li>
        <li>
                  <span style="background: #ffffff; color: #000000;">context.Response.StatusCode = 500;</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">}</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Observa el patrón: lo que va **antes** del await _next.Invoke permite modificar el contexto o la petición antes de mandarla al siguiente middleware. Y lo que va después del await, permite modificar la respuesta antes de mandarla para atrás al middleware anterior.
  
Si ahora ejecutas de nuevo la petición con postman, y no le pasas el X-User verás que ahora recibes un 500:
  
[<img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/02/image_thumb-3.png" alt="image" width="639" height="246" border="0" />][4]
  
Nuestro middleware transforma el 401 que devuelve MVC6 en un 500 antes de mandarlo al cliente.
  
Por supuesto, cuando uno crea un middleware suele crear un método de extensión sobre IApplicationBuilder que permita usarlo de forma más sencilla:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:25a2098e-496b-48d0-9480-6eb025a263cb" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px;" start="1">
        <li>
          <span style="background: #ffffff; color: #0000ff;">namespace</span><span style="background: #ffffff; color: #000000;"> Microsoft.AspNet.Builder</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">{</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #0000ff;">public</span> <span style="background: #ffffff; color: #0000ff;">static</span> <span style="background: #ffffff; color: #0000ff;">class</span> <span style="background: #ffffff; color: #2b91af;">CustomMiddlewareAppBuilderExtensions</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">{</span>
        </li>
        <li>
                  <span style="background: #ffffff; color: #0000ff;">public</span> <span style="background: #ffffff; color: #0000ff;">static</span> <span style="background: #ffffff; color: #2b91af;">IApplicationBuilder</span><span style="background: #ffffff; color: #000000;"> UseCustomMiddleware(</span><span style="background: #ffffff; color: #0000ff;">this</span> <span style="background: #ffffff; color: #2b91af;">IApplicationBuilder</span><span style="background: #ffffff; color: #000000;"> app)</span>
        </li>
        <li>
                  <span style="background: #ffffff; color: #000000;">{</span>
        </li>
        <li>
                      <span style="background: #ffffff; color: #0000ff;">return</span><span style="background: #ffffff; color: #000000;"> app.UseMiddleware<</span><span style="background: #ffffff; color: #2b91af;">CustomMiddleware</span><span style="background: #ffffff; color: #000000;">>();</span>
        </li>
        <li>
                  <span style="background: #ffffff; color: #000000;">}</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">}</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Y así nuestro método Startup queda “más profesional” 😛

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:81cc9644-b835-447c-b90e-499f596a13e1" class="wlWriterEditableSmartContent" style="float: none; margin: 0px; display: inline; padding: 0px;">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt;">
    <div style="background: #ddd; max-height: 300px; overflow: auto;">
      <ol style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px;" start="1">
        <li>
          <span style="background: #ffffff; color: #0000ff;">public</span> <span style="background: #ffffff; color: #0000ff;">void</span><span style="background: #ffffff; color: #000000;"> Configure(</span><span style="background: #ffffff; color: #2b91af;">IApplicationBuilder</span><span style="background: #ffffff; color: #000000;"> app)</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">{</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">app.UseIISPlatformHandler();</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">app.UseCustomMiddleware();</span>
        </li>
        <li>
              <span style="background: #ffffff; color: #000000;">app.UseMvcWithDefaultRoute();</span>
        </li>
        <li>
          <span style="background: #ffffff; color: #000000;">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Bueno… vamos a dejarlo aquí en este post. Espero que haya quedado un poco más claro el concepto de middleware en ASP.NET Core y ¡lo sencillo que es crear uno!

 [1]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/02/image.png
 [2]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/02/image-1.png
 [3]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/02/image-2.png
 [4]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/02/image-3.png