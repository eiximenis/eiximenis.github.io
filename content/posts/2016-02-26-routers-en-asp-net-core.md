---
title: Routers en asp.net core
description: Routers en asp.net core
author: eiximenis

date: 2016-02-26T11:47:29+00:00
geeks_url: /?p=1717
geeks_ms_views:
  - 2230
categories:
  - asp.net 5
  - asp.net vNext

---
Cuando hablamos del _routing_ solemos referirnos al proceso por el cual una petición es enrutada hacia una acción concreta de un controlador. Esa definición es cierta en el contexto de una aplicación ASP.NET MVC (y/o WebApi) pero en ASP.NET Core, el concepto de routing es una parte integral del framework.
  
Por ello, en ASP.NET Core entendemos el routing como el proceso mediante el cual una petición web es enrutada hacia donde tenga que ser tratada. El destino puede ser una acción de un controlador, pero no tiene por qué (el middleware de MVC6 podría no estar instalado en el pipeline de la aplicación). En este post vamos a ver como funciona este proceso de enrutado.
  
<!--more-->

**Routers**
  
Un router es un objeto que implementa la interfaz IRouter. Cuando una petición entra en el pipeline, el framework pregunta a todos los routers cual puede tratar esa petición. La petición se envía a los routers en el orden en que esos son añadidos en el pipeline de la aplicación (a través del método _UseRouter_). Si ningún router declara que puede tratar la petición, el framework devuelve un 404.
  
Cuando un router recibe la petición puede examinarla para decidir si puede o no procesarla, y en caso de hacerlo debe generar la respuesta y establecer la propiedad IsHandled del HttpContext a true. Observa que el router **genera la respuesta que se envía al cliente**. Veamos un ejemplo muy sencillo. Vamos a crear un proyecto web con ASP.NET Core usando la plantilla Empty, para empezar desde cero. En el project.json agregamos la referencia al paquete _Microsoft.AspNet.Routing_. Y en la clase Startup colocamos el siguiente código:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:37049a09-da8b-4a7b-a20b-c00d857cb48c" class="wlWriterEditableSmartContent" style="float: none;margin: 0px;padding: 0px">
  <div style="border: #000080 1px solid;color: #000;font-family: 'Courier New', Courier, Monospace;font-size: 10pt">
    <div style="background: #ddd;max-height: 300px;overflow: auto">
      <ol style="background: #ffffff;margin: 0 0 0 2em;padding: 0 0 0 5px" start="1">
        <li>
          <span style="background: #ffffff;color: #000000">app.UseRouter(</span><span style="background: #ffffff;color: #0000ff">new</span> <span style="background: #ffffff;color: #2b91af">HandleAlwaysRouter</span><span style="background: #ffffff;color: #000000">());</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Ahora vamos a crear la clase HandleAlwaysRouter, que va a ser un router que siempre responda a cualquier petición:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:9fba0a12-211e-4000-9334-ea7d740c59a1" class="wlWriterEditableSmartContent" style="float: none;margin: 0px;padding: 0px">
  <div style="border: #000080 1px solid;color: #000;font-family: 'Courier New', Courier, Monospace;font-size: 10pt">
    <div style="background: #ddd;max-height: 300px;overflow: auto">
      <ol style="background: #ffffff;margin: 0 0 0 2.5em;padding: 0 0 0 5px" start="1">
        <li>
          <span style="background: #ffffff;color: #0000ff">public</span> <span style="background: #ffffff;color: #0000ff">class</span> <span style="background: #ffffff;color: #2b91af">HandleAlwaysRouter</span><span style="background: #ffffff;color: #000000"> : </span><span style="background: #ffffff;color: #2b91af">IRouter</span>
        </li>
        <li>
          <span style="background: #ffffff;color: #000000">{</span>
        </li>
        <li>
              <span style="background: #ffffff;color: #0000ff">public</span> <span style="background: #ffffff;color: #2b91af">VirtualPathData</span><span style="background: #ffffff;color: #000000"> GetVirtualPath(</span><span style="background: #ffffff;color: #2b91af">VirtualPathContext</span><span style="background: #ffffff;color: #000000"> context)</span>
        </li>
        <li>
              <span style="background: #ffffff;color: #000000">{</span>
        </li>
        <li>
                  <span style="background: #ffffff;color: #0000ff">return</span> <span style="background: #ffffff;color: #0000ff">new</span> <span style="background: #ffffff;color: #2b91af">VirtualPathData</span><span style="background: #ffffff;color: #000000">(</span><span style="background: #ffffff;color: #0000ff">this</span><span style="background: #ffffff;color: #000000">, </span><span style="background: #ffffff;color: #a31515">&#8220;&#8221;</span><span style="background: #ffffff;color: #000000">);</span>
        </li>
        <li>
              <span style="background: #ffffff;color: #000000">}</span>
        </li>
        <li>
        </li>
        <li>
              <span style="background: #ffffff;color: #0000ff">public</span> <span style="background: #ffffff;color: #0000ff">async</span> <span style="background: #ffffff;color: #2b91af">Task</span><span style="background: #ffffff;color: #000000"> RouteAsync(</span><span style="background: #ffffff;color: #2b91af">RouteContext</span><span style="background: #ffffff;color: #000000"> context)</span>
        </li>
        <li>
              <span style="background: #ffffff;color: #000000">{</span>
        </li>
        <li>
                  <span style="background: #ffffff;color: #0000ff">await</span><span style="background: #ffffff;color: #000000"> context.HttpContext.Response.WriteAsync(</span><span style="background: #ffffff;color: #a31515">&#8220;Hello world&#8221;</span><span style="background: #ffffff;color: #000000">);</span>
        </li>
        <li>
                  <span style="background: #ffffff;color: #000000">context.IsHandled = </span><span style="background: #ffffff;color: #0000ff">true</span><span style="background: #ffffff;color: #000000">;</span>
        </li>
        <li>
              <span style="background: #ffffff;color: #000000">}</span>
        </li>
        <li>
          <span style="background: #ffffff;color: #000000">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Si ahora ejecutas la aplicación verás que cualquier petición que envies devolverá siempre “Hello World”. Nuestro router siempre determina que puede devolver una respuesta.
  
**Nota:** Si generas una respuesta, es importante que establezcas la propiedad IsHandled a _true_. Si no lo haces el framework invocará el siguiente router. Si dos routers modifican la respuesta, todas las modificaciones son aplicadas (todos los routers reciben el mismo contexto que contiene la misma Response). Eso puede generar errores aunque también habilitar algunos escenarios interesantes.
  
Hagamos la prueba. Elimina la línea context.IsHandled del HandleAlwaysRouter y registra otra instancia de este router en la aplicación (duplica la línea app.UseRouter del Startup). Si ahora ejecutas de nuevo, verás que “Hello world” aparece dos veces.
  
Ahora bien, ten presente que combinar la respuesta de dos routers, puede generar errores. P. ej. si modificas el método RouteAsync con el siguiente código y ejecutas la aplicación, recibirás un error:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:a6b7c5f5-40ba-477c-bfd1-84432573483a" class="wlWriterEditableSmartContent" style="float: none;margin: 0px;padding: 0px">
  <div style="border: #000080 1px solid;color: #000;font-family: 'Courier New', Courier, Monospace;font-size: 10pt">
    <div style="background: #ddd;max-height: 300px;overflow: auto">
      <ol style="background: #ffffff;margin: 0 0 0 2em;padding: 0 0 0 5px" start="1">
        <li>
          <span style="background: #ffffff;color: #0000ff">public</span> <span style="background: #ffffff;color: #0000ff">async</span> <span style="background: #ffffff;color: #2b91af">Task</span><span style="background: #ffffff;color: #000000"> RouteAsync(</span><span style="background: #ffffff;color: #2b91af">RouteContext</span><span style="background: #ffffff;color: #000000"> context)</span>
        </li>
        <li>
          <span style="background: #ffffff;color: #000000">{</span>
        </li>
        <li>
              <span style="background: #ffffff;color: #000000">context.HttpContext.Response.Headers.Add(</span><span style="background: #ffffff;color: #a31515">&#8220;some-header&#8221;</span><span style="background: #ffffff;color: #000000">, </span><span style="background: #ffffff;color: #0000ff">new</span> <span style="background: #ffffff;color: #2b91af">StringValues</span><span style="background: #ffffff;color: #000000">(</span><span style="background: #ffffff;color: #a31515">&#8220;value&#8221;</span><span style="background: #ffffff;color: #000000">));</span>
        </li>
        <li>
              <span style="background: #ffffff;color: #0000ff">await</span><span style="background: #ffffff;color: #000000"> context.HttpContext.Response.WriteAsync(</span><span style="background: #ffffff;color: #a31515">&#8220;Hello world&#8221;</span><span style="background: #ffffff;color: #000000">);</span>
        </li>
        <li>
              <span style="background: #ffffff;color: #008000">//context.IsHandled = true;</span>
        </li>
        <li>
          <span style="background: #ffffff;color: #000000">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

La razón del error es que no se pueden modificar los _headers_ de la respuesta una vez se ha empezado a enviar datos, por lo que la segunda vez que se ejecuta el router, da un error porque no puede añadir la cabecera “some-header”, porque el primer router ya ha empezado a enviar datos.
  
**TemplateRoute**
  
TemplateRoute es un router, que viene con el framework. Este router lo que hace es verificar si la URL de la petición valida una plantilla, y si es así, ceder el control a otro router interno. Cuando creamos un TemplateRoute tenemos que decirle que router interno es el que va a procesar la petición. Observa el siguiente código en Startup:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:1532b2a0-23eb-47e7-84dc-19be752a5071" class="wlWriterEditableSmartContent" style="float: none;margin: 0px;padding: 0px">
  <div style="border: #000080 1px solid;color: #000;font-family: 'Courier New', Courier, Monospace;font-size: 10pt">
    <div style="background: #ddd;max-height: 300px;overflow: auto">
      <ol style="background: #ffffff;margin: 0 0 0 2.5em;padding: 0 0 0 5px" start="1">
        <li>
          <span style="background: #ffffff;color: #0000ff">public</span> <span style="background: #ffffff;color: #0000ff">void</span><span style="background: #ffffff;color: #000000"> ConfigureServices(</span><span style="background: #ffffff;color: #2b91af">IServiceCollection</span><span style="background: #ffffff;color: #000000"> services)</span>
        </li>
        <li>
          <span style="background: #ffffff;color: #000000">{</span>
        </li>
        <li>
              <span style="background: #ffffff;color: #000000">services.AddRouting();</span>
        </li>
        <li>
          <span style="background: #ffffff;color: #000000">}</span>
        </li>
        <li>
          <span style="background: #ffffff;color: #0000ff">public</span> <span style="background: #ffffff;color: #0000ff">void</span><span style="background: #ffffff;color: #000000"> Configure(</span><span style="background: #ffffff;color: #2b91af">IApplicationBuilder</span><span style="background: #ffffff;color: #000000"> app)</span>
        </li>
        <li>
          <span style="background: #ffffff;color: #000000">{</span>
        </li>
        <li>
              <span style="background: #ffffff;color: #000000">app.UseIISPlatformHandler();</span>
        </li>
        <li>
              <span style="background: #ffffff;color: #000000">app.UseRouter(</span><span style="background: #ffffff;color: #0000ff">new</span> <span style="background: #ffffff;color: #2b91af">TemplateRoute</span><span style="background: #ffffff;color: #000000">(</span><span style="background: #ffffff;color: #0000ff">new</span> <span style="background: #ffffff;color: #2b91af">HandleAlwaysRouter</span><span style="background: #ffffff;color: #000000">(), </span><span style="background: #ffffff;color: #a31515">&#8220;api/always&#8221;</span><span style="background: #ffffff;color: #000000">,</span>
        </li>
        <li>
                   <span style="background: #ffffff;color: #000000">app.ApplicationServices.GetService<</span><span style="background: #ffffff;color: #2b91af">IInlineConstraintResolver</span><span style="background: #ffffff;color: #000000">>()));</span>
        </li>
        <li>
          <span style="background: #ffffff;color: #000000">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Añadimos un TemplateRoute que usará un HandleAlwaysRouter pero solo si la url es api/always. El tercer parámetro del constructor es un IInlineConstraintResolver que se usa para resolver las _route constraints_ (tales como {id:int}). El propio middleware de routing coloca uno en el sistema de DI del framework (siempre y cuando llamemos a AddRouting en el ConfigureServices).
  
Ahora si vas a la url /api/always verás el “Hello world” y cualquier otra URL te dará un 404.
  
El TemplateRoute es muy útil para evitar tener que comprobar la URL en cada router, y además rellena los _route values_. De hecho, casi todos los registros routers, usan un TemplateRoute (y lo vinculan a un router interno que es el que hace el trabajo).
  
**RouteCollection**
  
RouteCollection es el segundo router que viene implementado en el framework y no es nada más que una coleccion de routers internos. Este router itera por sus propios routers internos y se para tan buen punto un router interno ha procesado la petición (es decir, ha establecido IsHandled a true). No tiene más intención que dar un lugar donde agrupar un conjunto de routers relacionados.
  
**IRouteBuilder**
  
Vamos a hablar de otra pieza en juego. La interfaz IRouteBuilder. Un IRouteBuilder no es nada más que un objeto que se usa para crear **un router**. Es pues una interfaz para definir factorías de routers.
  
No obstante esta interfaz tiene propiedades que van más allá de una simple factoría. Esa es la definición de la interfaz:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:2268961e-bb46-44de-99f5-753d6fbee7be" class="wlWriterEditableSmartContent" style="float: none;margin: 0px;padding: 0px">
  <div style="border: #000080 1px solid;color: #000;font-family: 'Courier New', Courier, Monospace;font-size: 10pt">
    <div style="background: #ddd;max-height: 300px;overflow: auto">
      <ol style="background: #ffffff;margin: 0 0 0 2em;padding: 0 0 0 5px" start="1">
        <li>
          <span style="background: #ffffff;color: #0000ff">public</span> <span style="background: #ffffff;color: #0000ff">interface</span> <span style="background: #ffffff;color: #2b91af">IRouteBuilder</span>
        </li>
        <li>
          <span style="background: #ffffff;color: #000000">{</span>
        </li>
        <li>
              <span style="background: #ffffff;color: #2b91af">IRouter</span><span style="background: #ffffff;color: #000000"> DefaultHandler { </span><span style="background: #ffffff;color: #0000ff">get</span><span style="background: #ffffff;color: #000000">; </span><span style="background: #ffffff;color: #0000ff">set</span><span style="background: #ffffff;color: #000000">; }</span>
        </li>
        <li>
              <span style="background: #ffffff;color: #2b91af">IServiceProvider</span><span style="background: #ffffff;color: #000000"> ServiceProvider { </span><span style="background: #ffffff;color: #0000ff">get</span><span style="background: #ffffff;color: #000000">; }</span>
        </li>
        <li>
              <span style="background: #ffffff;color: #2b91af">IList</span><span style="background: #ffffff;color: #000000"><</span><span style="background: #ffffff;color: #2b91af">IRouter</span><span style="background: #ffffff;color: #000000">> Routes { </span><span style="background: #ffffff;color: #0000ff">get</span><span style="background: #ffffff;color: #000000">; }</span>
        </li>
        <li>
              <span style="background: #ffffff;color: #2b91af">IRouter</span><span style="background: #ffffff;color: #000000"> Build();</span>
        </li>
        <li>
          <span style="background: #ffffff;color: #000000">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Me interesa especialmente la propiedad Routes. Esa propiedad es una Lista de IRouters. Es decir, un IRouteBuilder nos da un mecanismo para construir un router a partir de varios routers. El método Build() es el que devuelve el router configurado.
  
Si crear un router a partir de varios routers te parece algo raro… recuerda que tenemos la RouteCollection, que es nada más y nada menos que eso. De hecho, el resultado del método Build() suele ser una RouteCollection, aunque por supuesto eso dependerá de la implementación concreta del IRouteBuilder (la que usa MVC6 devuelve una RouteCollecion).
  
**Routers en MVC6**
  
Cuando creamos una aplicación MVC6 no debemos registrar ningún router en el sistema. Eso es porque el método UseMvc (o el UseMvcWithDefaultRoute) lo hace por nosotros… Pero, ¿qué registra exactamente este método?
  
El método UseMvc podemos llamarlo sin parámetros (app.UseMvc()) o con un parámetro. Este parámetro es una Action<IRouteBuilder>. Es decir, cuando tenemos un código como:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:541f8d43-244c-4085-918a-039e3e50a9fa" class="wlWriterEditableSmartContent" style="float: none;margin: 0px;padding: 0px">
  <div style="border: #000080 1px solid;color: #000;font-family: 'Courier New', Courier, Monospace;font-size: 10pt">
    <div style="background: #ddd;max-height: 300px;overflow: auto">
      <ol style="background: #ffffff;margin: 0 0 0 2em;padding: 0 0 0 5px" start="1">
        <li>
          <span style="background: #ffffff;color: #000000">app.UseMvc(routes =></span>
        </li>
        <li>
          <span style="background: #ffffff;color: #000000">{</span>
        </li>
        <li>
              <span style="background: #ffffff;color: #008000">// ...</span>
        </li>
        <li>
          <span style="background: #ffffff;color: #000000">});</span>
        </li>
      </ol>
    </div>
  </div>
</div>

El parámetro “routes” es un IRouteBuilder (que ha creado MVC6). Nosotros configuramos el IRouteBuilder y luego, MVC6 obtiene la lista de IRouter a partir de este IRouteBuilder y agrega esos routers al pipeline. Usando la propiedad Routes del IRouteBuilder podemos agregar un router al IRouteBuilder, pero también hay otros mecanismos como el método MapRoute (discutido más abajo).
  
Así que resumiendo podemos decir que el método UseMvc añade al pipeline el router resultado de configurar el IRouteBuilder. Este router es, en el caso de MVC6, un RouteCollection que contiene todos los routers que hemos agregado al IRouteBuilder. Además el método UseMvc agrega un IRoute adicional al RouteCollection. Este IRoute adicional es necesario para soportar el routing basado en atributos y lo discutimos más adelante.

> **Nota:** Aunque hablemos solo del método UseMvc, todo lo dicho aplica al método UseMvcWithDefaultRoute. De hecho, este método equivale a llamar a UseMvc y dentro de UseMvc usar MapRoute para registrar la ruta típica (“{controller}/{action}/{id?}”).

Si tenemos este código (que es básicamente equivalente a llamar UseMvcWithDefaultRoute)

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:f5895505-f92a-47d9-a217-ae52f44c8162" class="wlWriterEditableSmartContent" style="float: none;margin: 0px;padding: 0px">
  <div style="border: #000080 1px solid;color: #000;font-family: 'Courier New', Courier, Monospace;font-size: 10pt">
    <div style="background: #ddd;max-height: 300px;overflow: auto">
      <ol style="background: #ffffff;margin: 0 0 0 2em;padding: 0 0 0 5px" start="1">
        <li>
          <span style="background: #ffffff;color: #000000">app.UseMvc(routes =></span>
        </li>
        <li>
          <span style="background: #ffffff;color: #000000">{</span>
        </li>
        <li>
              <span style="background: #ffffff;color: #000000">routes.MapRoute(</span><span style="background: #ffffff;color: #a31515">&#8220;Default&#8221;</span><span style="background: #ffffff;color: #000000">, </span><span style="background: #ffffff;color: #a31515">&#8220;{controller}/{action}/{id?}&#8221;</span><span style="background: #ffffff;color: #000000">,</span>
        </li>
        <li>
                  <span style="background: #ffffff;color: #0000ff">new</span><span style="background: #ffffff;color: #000000"> { controller = </span><span style="background: #ffffff;color: #a31515">&#8220;Home&#8221;</span><span style="background: #ffffff;color: #000000">, action = </span><span style="background: #ffffff;color: #a31515">&#8220;Index&#8221;</span><span style="background: #ffffff;color: #000000"> });</span>
        </li>
        <li>
          <span style="background: #ffffff;color: #000000">});</span>
        </li>
      </ol>
    </div>
  </div>
</div>

El resultado será que en el pipeline se agregará una RouteCollection. Ese RouteCollection tendrá dos routers en su interior:

  1. Un objeto de tipo <a href="https://github.com/aspnet/mvc/blob/master/src/Microsoft.AspNet.Mvc.Core/Routing/AttributeRoute.cs" target="_blank" rel="noopener noreferrer">Microsoft.AspNet.Mvc.Routing.AttributeRoute</a>. Esta clase implementa IRouter y es la que da soporte al routing basado en atributos.
  2. Un objeto de tipo RouteTemplate, con el template “{controller}/{action}/{id?}”. Por supuesto este RouteTemplate da soporte a la convención básica de URLs de MVC. Este segundo es el que se agrega como resultado de la llamada al MapRoute.

Analicemos un poco ambas entradas generadas, empezando por la segunda. Recuerda que RouteTemplate lo único que hace es validar que la URL cumple una cierta plantilla **y delegar en un router interno**. RouteTemplate es genérico y no entiende de conceptos como “controller” o “action”. Esos conceptos son de MVC. Pero sí que entiende de los route values. Supongamos una URL /Home/Index/10.
  
En este caso el RouteTemplate añadido validará esta URL y asignará los siguientes route values:

  * controller = Home
  * action = Index
  * id = 10

Dado que la URL /Home/Index/10 valida la plantilla del RouteTemplate, este crea esos route values y invoca al router interno que tiene. Para el RouteTemplate el concepto de controller no tiene  ningún significado. Simplemente lo deja en las Route Values (dentro del RouteContext) y manda ese RouteContext al router interno. Es ese router interno el que sabe qué hacer con esas route values.
  
¿Y cuál es ese router interno? Pues, por supuesto, **uno propio de ASP.NET MVC**. Uno que sí que entiende de conceptos como “controller” y “action” y que, precisamente, usará esos route values para saber qué controlador y acción ejecutar. Insisto mucho en este punto porque es clave:

  * El RouteTemplate (IRouter genérico) no entiende de conceptos de MVC. Solo sabe validar plantillas de URL y asignar route values (que no tienen significado alguno para él). Si la plantilla se cumple invoca a un router interno.
  * El router propio de ASP.NET MVC6 no valida URLs, tan solo actúa con los route values. Ese router es el que selecciona el controlador y la acción a partir de los route values, pero no se preocupa de donde le vienen esos route values (lo normal será que le vengan rellenados, por un RouteTemplate que contenga al router de ASP.NET MVC6).

El router de ASP.NET MVC6 es un objeto de la clase MvcDefaultRouteHandler. Esa clase es el IRouter de ASP.NET MVC que usa los route values para llamar e invocar el controlador correspondiente.
  
Veamos un código que deje claros esos hechos. Para ello, añadimos el paquete del middleware de MVC y en el método Configure de la clase Startup ponemos lo siguiente:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:bf2bd003-a4ac-4368-af3b-86834768243b" class="wlWriterEditableSmartContent" style="float: none;margin: 0px;padding: 0px">
  <div style="border: #000080 1px solid;color: #000;font-family: 'Courier New', Courier, Monospace;font-size: 10pt">
    <div style="background: #ddd;max-height: 300px;overflow: auto">
      <ol style="background: #ffffff;margin: 0 0 0 2em;padding: 0 0 0 5px" start="1">
        <li>
          <span style="background: #ffffff;color: #000000">app.UseMvc(routes =></span>
        </li>
        <li>
          <span style="background: #ffffff;color: #000000">{</span>
        </li>
        <li>
              <span style="background: #ffffff;color: #000000">routes.Routes.Add(</span><span style="background: #ffffff;color: #0000ff">new</span> <span style="background: #ffffff;color: #2b91af">FakeMvcRouter</span><span style="background: #ffffff;color: #000000">(routes.DefaultHandler));</span>
        </li>
        <li>
          <span style="background: #ffffff;color: #000000">});</span>
        </li>
      </ol>
    </div>
  </div>
</div>

En este caso estamos agregando directamente un FakeMvcRouter al IRouteBuilder del método UseMvc. La clase FakeMvcRouter la definimos como en el siguiente código:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:44bf6040-deb0-4950-98cc-9f3ab1aa0ebe" class="wlWriterEditableSmartContent" style="float: none;margin: 0px;padding: 0px">
  <div style="border: #000080 1px solid;color: #000;font-family: 'Courier New', Courier, Monospace;font-size: 10pt">
    <div style="background: #ddd;max-height: 300px;overflow: auto">
      <ol style="background: #ffffff;margin: 0 0 0 2.5em;padding: 0 0 0 5px" start="1">
        <li>
          <span style="background: #ffffff;color: #0000ff">internal</span> <span style="background: #ffffff;color: #0000ff">class</span> <span style="background: #ffffff;color: #2b91af">FakeMvcRouter</span><span style="background: #ffffff;color: #000000"> : </span><span style="background: #ffffff;color: #2b91af">IRouter</span>
        </li>
        <li>
          <span style="background: #ffffff;color: #000000">{</span>
        </li>
        <li>
              <span style="background: #ffffff;color: #0000ff">private</span> <span style="background: #ffffff;color: #2b91af">IRouter</span><span style="background: #ffffff;color: #000000"> defaultHandler;</span>
        </li>
        <li>
        </li>
        <li>
              <span style="background: #ffffff;color: #0000ff">public</span><span style="background: #ffffff;color: #000000"> FakeMvcRouter(</span><span style="background: #ffffff;color: #2b91af">IRouter</span><span style="background: #ffffff;color: #000000"> defaultHandler)</span>
        </li>
        <li>
              <span style="background: #ffffff;color: #000000">{</span>
        </li>
        <li>
                  <span style="background: #ffffff;color: #0000ff">this</span><span style="background: #ffffff;color: #000000">.defaultHandler = defaultHandler;</span>
        </li>
        <li>
              <span style="background: #ffffff;color: #000000">}</span>
        </li>
        <li>
        </li>
        <li>
              <span style="background: #ffffff;color: #0000ff">public</span> <span style="background: #ffffff;color: #2b91af">VirtualPathData</span><span style="background: #ffffff;color: #000000"> GetVirtualPath(</span><span style="background: #ffffff;color: #2b91af">VirtualPathContext</span><span style="background: #ffffff;color: #000000"> context)</span>
        </li>
        <li>
              <span style="background: #ffffff;color: #000000">{</span>
        </li>
        <li>
                  <span style="background: #ffffff;color: #0000ff">return</span> <span style="background: #ffffff;color: #0000ff">null</span><span style="background: #ffffff;color: #000000">;</span>
        </li>
        <li>
              <span style="background: #ffffff;color: #000000">}</span>
        </li>
        <li>
        </li>
        <li>
              <span style="background: #ffffff;color: #0000ff">public</span> <span style="background: #ffffff;color: #0000ff">async</span> <span style="background: #ffffff;color: #2b91af">Task</span><span style="background: #ffffff;color: #000000"> RouteAsync(</span><span style="background: #ffffff;color: #2b91af">RouteContext</span><span style="background: #ffffff;color: #000000"> context)</span>
        </li>
        <li>
              <span style="background: #ffffff;color: #000000">{</span>
        </li>
        <li>
                  <span style="background: #ffffff;color: #000000">context.RouteData.Values[</span><span style="background: #ffffff;color: #a31515">&#8220;controller&#8221;</span><span style="background: #ffffff;color: #000000">] = </span><span style="background: #ffffff;color: #a31515">&#8220;home&#8221;</span><span style="background: #ffffff;color: #000000">;</span>
        </li>
        <li>
                  <span style="background: #ffffff;color: #000000">context.RouteData.Values[</span><span style="background: #ffffff;color: #a31515">&#8220;action&#8221;</span><span style="background: #ffffff;color: #000000">] = </span><span style="background: #ffffff;color: #a31515">&#8220;index&#8221;</span><span style="background: #ffffff;color: #000000">;</span>
        </li>
        <li>
                  <span style="background: #ffffff;color: #0000ff">await</span><span style="background: #ffffff;color: #000000"> defaultHandler.RouteAsync(context);</span>
        </li>
        <li>
                  <span style="background: #ffffff;color: #000000">context.IsHandled = </span><span style="background: #ffffff;color: #0000ff">true</span><span style="background: #ffffff;color: #000000">;</span>
        </li>
        <li>
              <span style="background: #ffffff;color: #000000">}</span>
        </li>
        <li>
          <span style="background: #ffffff;color: #000000">}</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Simplemente este FakeMvcRouter siempre coloca en el route value “controller” el valor de Home y en el route value “action” el valor de Index. Y luego cede el control al router interno (parámetro defaultHandler que se le pasa en el constructor). Si miramos el código del Startup en el constructor del FakeMvcRouter le pasamos routes.DefaultHandler que es nada más ni nada menos que el objeto MvcDefaultRouteHandler, es decir el router por defecto de ASP.NET MVC6.
  
¿Y qué consigue ese código? Pues que cualquier URL que metas termine llamando a la acción Index del controlador Home.
  
¿Y qué demuestra este código? Pues que el router de MVC6 (el MvcDefaultRouteHandler) no es quien parsea las URLs. Solo analiza los route values. En la configuración habitual és el TemplateRoute el router que analiza la URL y mira si cumple la plantilla deseada.
  
Vale, hemos empezado por la segunda entrada de la RouteCollection que nos generabe el método UseMvcWithDefaultRoute. Veamos ahora la primera (que es la única que tendríamos si usáramos UseMvc sin parámetros).
  
Esa entrada hemos comentado que es un objeto de tipo AttributeRoute. Lo que hace este router, es analizar la URL de la petición y ver si encaja en alguno de los atributos [Route] que tengamos en el código. Es decir, si la URL es /api/users y tenemos un controlador, llamado Usuarios con una acción Index que está decorada con [Route(“api/users”)] lo que hará el AttributeRoute es rellenar los route values controller, action y los route values adicionales que estén definidos dentro de la plantilla del [Route] seleccionado y terminar llamando a… un router interno.
  
Y por supuesto, ya te imaginas cual es el router interno que tiene el AttributeRoute, ¿no? Efectivamente, el MvcDefaultRouteHandler. El router de MVC6.
  
**El método MapRoute**
  
Vale, lo normal es que cuando agregues rutas en MVC6 lo hagas usando el método MapRoute:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:aa85930b-7295-4560-85e8-2acb036b5a4c" class="wlWriterEditableSmartContent" style="float: none;margin: 0px;padding: 0px">
  <div style="border: #000080 1px solid;color: #000;font-family: 'Courier New', Courier, Monospace;font-size: 10pt">
    <div style="background: #ddd;max-height: 300px;overflow: auto">
      <ol style="background: #ffffff;margin: 0 0 0 2em;padding: 0 0 0 5px" start="1">
        <li>
          <span style="background: #ffffff;color: #000000">app.UseMvc(routes =></span>
        </li>
        <li>
          <span style="background: #ffffff;color: #000000">{</span>
        </li>
        <li>
              <span style="background: #ffffff;color: #000000">routes.MapRoute(</span><span style="background: #ffffff;color: #a31515">&#8220;Default&#8221;</span><span style="background: #ffffff;color: #000000">, </span><span style="background: #ffffff;color: #a31515">&#8220;{controller}/{action}/{id?}&#8221;</span><span style="background: #ffffff;color: #000000">,</span>
        </li>
        <li>
                  <span style="background: #ffffff;color: #0000ff">new</span><span style="background: #ffffff;color: #000000"> { controller = </span><span style="background: #ffffff;color: #a31515">&#8220;Home&#8221;</span><span style="background: #ffffff;color: #000000">, action = </span><span style="background: #ffffff;color: #a31515">&#8220;Index&#8221;</span><span style="background: #ffffff;color: #000000"> });</span>
        </li>
        <li>
          <span style="background: #ffffff;color: #000000">});</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Este método es un método de extensión definido sobre IRouteBuilder que básicamente lo único que hace es agregar un RouteTemplate con la plantilla indicada y le coloca como router interno del RouteTemplate el valor de la propiedad DefaultHandler del IRouteBuilder. El IRouteBuilder que recibimos como el parámetro “routes” del método UseMvc tiene la propiedad DefaultHandler y su valor es, como ya se ha comentado, el MvcDefaultRouteHandler. Por lo tanto, si usamos el método MapRoute con el IRouteBuilder que recibe el método UseMvc entonces, cada MapRoute añade un RouteTemplate con el MvcDefaultRouteHandler.
  
Es importante ver que el método MapRoute no es de MVC6, es del sistema de routing en general. Cualquier otro framework podría tener un método UseXXX, dentro del cual el IRouteBuilder que usemos tuviese el DefaultHandler establecido al valor correspondiente.
  
**Resumen**
  
Bueno, vamos a dejarlo aquí por hoy. En esta entrada hemos “destripado” un poco como funciona el routing en asp.net core. Es importante entender que el routing es un elemento de asp.net core, no de MVC6 y que podemos tener routing sin necesidad de MVC6. Por defecto asp.net core nos proporciona dos routers: TemplateRoute y RouteCollection. El primero valida que una URL cumple una plantilla (y rellena route values) y si es el caso invoca un router interno que se le pasa en el constructor. RouteCollection por su parte es un router, que mantiene una lista de routers internos y los evalúa todos por oden hasta encontrar uno que gestione la petición. Podemos ver que para tener funcionalidad real, tenemos que implementar un Router, ya que tanto TemplateRoute como RouteCollection necesitan un router interno que sea el que gestione la petición.
  
Hemos visto también que MVC6 implementa dos routers (RouteAttribute y MvcDefaultRouteHandler) y hemos comentado como, a través de los métodos UseMvc y UseMvcWithDefaultRoute, se da de alta la infraestructura necesaria para soportar tanto el routing por atributos como basado en las convenciones y en la tabla de rutas.
  
¡Espero que este post te haya resultado interesante!