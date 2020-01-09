---
title: 'Explorando ASP.NET vNext: Routing'
description: 'Explorando ASP.NET vNext: Routing'
author: eiximenis

date: 2014-06-25T12:24:44+00:00
geeks_url: /?p=1671
geeks_visits:
  - 1154
geeks_ms_views:
  - 1303
categories:
  - Uncategorized

---
Lentamente se van desgranando las novedades que incorporará ASP.NET vNext. Recordad que podéis ya jugar un poco con él, descargandoos la CTP de VS14 🙂

Vamos a ver en este post cuatro cosillas sobre el routing que incorpora ASP.NET vNext. Vamos a partir de una aplicación web vNext vacía y agregaremos tan solo lo justo para tener MVC y una tabla de rutas. Lo primero será editar el project.json y añadir la referencia a asp.net MVC:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:6e41be64-de2a-4ce2-9a9f-918c5ed91911" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#d7ba7d">"dependencies"</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#d7ba7d">"Helios"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"0.1-alpha-build-0585"</span><span style="background:#1e1e1e;color:#b4b4b4">,</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#d7ba7d">"Microsoft.AspNet.Mvc"</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"0.1-alpha-build-1268"</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">,</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Una vez hecho esto ya podemos editar el método _Configure_ de la clase _Startup_ para agregar MVC y configurar una tabla de rutas:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:c24c0125-130d-47ca-9cfb-12c3fec54176" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">app</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">UseServices(s </span><span style="background:#1e1e1e;color:#b4b4b4">=></span><span style="background:#1e1e1e;color:#dcdcdc"> s</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">AddMvc());</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">app</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">UseMvc(r </span><span style="background:#1e1e1e;color:#b4b4b4">=></span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">r</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">MapRoute(</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc">name: </span><span style="background:#1e1e1e;color:#d69d85">"default"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
        </li>
        <li style="background: #111111">
                      <span style="background:#1e1e1e;color:#dcdcdc">template: </span><span style="background:#1e1e1e;color:#d69d85">"{controller}/{action}/{id?}"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc">defaults: </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> { controller </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"Home"</span><span style="background:#1e1e1e;color:#dcdcdc">, action </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"Index"</span><span style="background:#1e1e1e;color:#dcdcdc"> });</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">});</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Con esto tenemos la “clásica” tabla de rutas por defecto. Una diferencia respecto a MVC clásico (MVC5 y anteriores) es el uso de id? para indicar un parámetro opcional (en lugar de colocar id = UrlParameter.Optional en el defaults). Esta nomenclatura es la misma que usábamos en el routing attribute de webapi 2 y es mucho más concisa.

Si añadimos un controlador HomeController para que nos devuelva una vista (Index.cshtml) veremos que todo funciona correctamente… Hasta ahí nada nuevo bajo el sol.

Pero, recordad: WebApi y MVC están ahora unificados y a nivel de routing funcionaban de forma relativamente distinta. Pero… ¿como funcionará el routing de vNext? Hagamos un experimento:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:f63d9071-db1c-4ac4-9bd1-51428a9b3125" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IActionResult</span><span style="background:#1e1e1e;color:#dcdcdc"> Index()</span>
        </li>
        <li style="background: #111111">
           <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
               <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> View();</span>
        </li>
        <li style="background: #111111">
           <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li style="background: #111111">
           <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IActionResult</span><span style="background:#1e1e1e;color:#dcdcdc"> Index(</span><span style="background:#1e1e1e;color:#569cd6">int</span><span style="background:#1e1e1e;color:#dcdcdc"> id)</span>
        </li>
        <li>
           <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li style="background: #111111">
               <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> View();</span>
        </li>
        <li>
           <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

En MVC clásico **este cód
  
igo no funciona** (MVC se queja de una ambigüedad) ya que dos métodos de un controlador **no pueden** implementar la misma acción (para el mismo verbo HTTP). Pero en WebApi esto funciona. Pues en vNext también.

Si invocamos a la acción **sin pasar el parámetro id** (p. ej. con / o con /Home/Index) se ejecutará el primer método. Si invocamos la acción pasando el parámetro id (p. ej. con /Home/Index/2) se ejecutará el segundo método.

Prosigamos nuestro experimento… Modifiquemos el método _Startup.Configure_ para tener la siguiente tabla de rutas:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:ef366ece-bb9d-4a4c-a8bc-ce6ac9e32ee3" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">app</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">UseMvc(r </span><span style="background:#1e1e1e;color:#b4b4b4">=></span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">r</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">MapRoute(</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">name: </span><span style="background:#1e1e1e;color:#d69d85">"default"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">template: </span><span style="background:#1e1e1e;color:#d69d85">"{controller}/{action}/{id?}"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">defaults: </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> { controller </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"Home"</span><span style="background:#1e1e1e;color:#dcdcdc">, action </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"Index"</span><span style="background:#1e1e1e;color:#dcdcdc"> });</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">r</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">MapRoute(</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">name: </span><span style="background:#1e1e1e;color:#d69d85">"second"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">template: </span><span style="background:#1e1e1e;color:#d69d85">"Details/{id?}"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">defaults: </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> { controller </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"Product"</span><span style="background:#1e1e1e;color:#dcdcdc">, action</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#d69d85">"Display"}</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">});</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div></p> 

Creamos el controlador ProductController con una acción Dispay:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:e315184c-b946-4b7d-902c-0c35bf3cb7fd" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IActionResult</span><span style="background:#1e1e1e;color:#dcdcdc"> Display(</span><span style="background:#1e1e1e;color:#569cd6">int</span><span style="background:#1e1e1e;color:#b4b4b4">?</span><span style="background:#1e1e1e;color:#dcdcdc"> id)</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (id </span><span style="background:#1e1e1e;color:#b4b4b4">==</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc">) ViewBag</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Pid </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"None"</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">else</span><span style="background:#1e1e1e;color:#dcdcdc"> ViewBag</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Pid </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> id;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> View();</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Si ahora vamos a /Details/100.. ¿qué es lo que obtenemos? En ASP.NET MVC clásico obtendremos un 404 porque **las rutas se evalúan en orden y la primera que es posible aplicar se aplica**. Así la url /Details/100 encaja dentro de la primera ruta con:

  * controller = Details
  * action = 100
  * id = sin valor

Y como no existe el DetailsController de ahí el 404 que obtendríamos con MVC clásico. De ahí que la tabla de rutas tenga que configurarse con las rutas en orden _desde la más específica a la más general_.

Pero en vNext eso ya no es así. **Las rutas siguen evaluándose en orden pero si una ruta devuelve un 404 el control pasa a la siguiente ruta**. Así, en este caso se intenta evaluar la primera ruta con los parámetros indicados anteriormente (al igual que en MVC clásico). Dicha ruta devuelve un 404 así que el control pasa a la siguiente ruta. La URL /Details/10 encaja con el patrón de la siguiente ruta así que se intenta aplicar dicha ruta con los parámetros:

  * controller = Product (sacado de defaults)
  * action = Display (scado de
  
    defaults)
  * id = 100

Ahora como el controlador y la acción existen se invocan y vemos la página de detalles del producto 100.

Hagamos una tercera modificación a la tabla de rutas para que quede así:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:820367dd-5fea-4620-8753-db94424b98e5" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">app</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">UseMvc(r </span><span style="background:#1e1e1e;color:#b4b4b4">=></span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">r</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">MapRoute(</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">name: </span><span style="background:#1e1e1e;color:#d69d85">"default"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">template: </span><span style="background:#1e1e1e;color:#d69d85">"{controller}/{action}/{id?}"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">defaults: </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> { controller </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"Home"</span><span style="background:#1e1e1e;color:#dcdcdc">, action </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"Index"</span><span style="background:#1e1e1e;color:#dcdcdc"> });</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">r</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">MapRoute(</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">name: </span><span style="background:#1e1e1e;color:#d69d85">"second"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">template: </span><span style="background:#1e1e1e;color:#d69d85">"Details/{id?}"</span><span style="background:#1e1e1e;color:#dcdcdc">,</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">defaults: </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> { controller </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"Product"}</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">});</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Es la misma tabla de rutas que teníamos **con una excepción importante: la ruta “second” no define acción**.

Si ahora navegamos de nuevo a /Details/100 obtenemos un 404. Era de esperar, puesto que no hay acción alguna… Pero el routing de vNext nos tiene **reservada una última sorpresa: si no hay acción definida se enrutará por el verbo HTTP**.

Para hacer la prueba modifica el controlador Product:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:2088a9f1-529c-488c-8c3d-e0cdde47d5f1" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IActionResult</span><span style="background:#1e1e1e;color:#dcdcdc"> Get(</span><span style="background:#1e1e1e;color:#569cd6">int</span><span style="background:#1e1e1e;color:#b4b4b4">?</span><span style="background:#1e1e1e;color:#dcdcdc"> id)</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (id </span><span style="background:#1e1e1e;color:#b4b4b4">==</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc">) ViewBag</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Pid </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"None"</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">else</span><span style="background:#1e1e1e;color:#dcdcdc"> ViewBag</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Pid </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> id;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> View(</span><span style="background:#1e1e1e;color:#d69d85">"Display"</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Si ahora navegas a /Details/100 deberías ver, de nuevo, los detalles del producto con id 100. Al no haber acción vNext ha enrutado por el verbo HTTP y nuestro controlador se ha enrutado como si fuese un controlador de webapi.

Bueno… hemos visto brevemente algunas de las diferencias entre el routing de vNext y el de MVC tradicional. Para más detalles os recomiendo este post <a href="http://blogs.msdn.com/b/webdev/archive/2014/06/20/asp-net-vnext-routing-overview.aspx" target="_blank" rel="noopener noreferrer">del equipo de ASP.NET</a>.

Saludos!