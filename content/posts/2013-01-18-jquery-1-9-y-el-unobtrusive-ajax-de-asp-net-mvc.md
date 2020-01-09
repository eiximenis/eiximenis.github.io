---
title: jQuery 1.9 y el “unobtrusive ajax” de ASP.NET MVC
description: jQuery 1.9 y el “unobtrusive ajax” de ASP.NET MVC
author: eiximenis

date: 2013-01-18T12:08:02+00:00
geeks_url: /?p=1626
geeks_visits:
  - 5109
geeks_ms_views:
  - 2912
categories:
  - Uncategorized

---
Hace justo casi nada que ha salido la nueva versión de jQuery 1.9 y he aprovechado para actualizar una aplicación web que tenía a medias.

Pues bien, si actualizas una aplicación ASP.NET MVC que use unobtrusive ajax y actualizas a ASP.NET MVC… deja de funcionar. Pero vayamos por partes…

**Reproducción del problema**

Abre VS2012 y crea un nuevo proyecto ASP.NET MVC4. Usa la plantilla Basic (no la Empty). Podemos ver como por defecto nos ha agregado, entre otros, el fichero jQuery-unobstrusive-ajax. Este fichero es el que da soporte para el unobtrusive ajax de MVC. Ya hablé en este blog hace un tiempecillo sobre <a href="http://geeks.ms/blogs/etomas/archive/2010/11/09/unobtrusive-ajax-en-mvc3.aspx" target="_blank" rel="noopener noreferrer">el unobstrusive ajax de MVC</a>. 

Para ver el problema en acción, añadimos un controlador HomeController con la acción Index por defecto y una acción que he llamado PostData:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">HomeController</span> : <span style="color: #4ec9b0">Controller</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #4ec9b0">ActionResult</span> <span style="color: white">Index</span>()
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: white">View</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; [<span style="color: #4ec9b0">HttpPost</span>]
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #4ec9b0">ActionResult</span> <span style="color: white">PostData</span>(<span style="color: #569cd6">string</span> <span style="color: white">name</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">ViewBag</span><span style="color: #b4b4b4">.</span><span style="color: white">DataName</span> <span style="color: #b4b4b4">=</span> <span style="color: white">name</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: white">PartialView</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

La vista Index.cshtml es muy sencilla, tiene tan solo un Ajax.BeginForm:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="background: #ffffb3; color: black">@</span><span style="color: #569cd6">using</span> (<span style="color: white">Ajax</span><span style="color: #b4b4b4">.</span><span style="color: white">BeginForm</span>(<span style="color: #d69d85">"PostData"</span>, <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">AjaxOptions</span>() {<span style="color: white">HttpMethod</span> <span style="color: #b4b4b4">=</span> <span style="color: #d69d85">"Post"</span>, <span style="color: white">UpdateTargetId</span> <span style="color: #b4b4b4">=</span> <span style="color: #d69d85">"datadiv"</span>}))
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">label</span> <span style="color: #9cdcfe">for</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"name"</span><span style="color: gray">></span>Name: <span style="color: gray"></</span><span style="color: #569cd6">label</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">input</span> <span style="color: #9cdcfe">type</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"text"</span> <span style="color: #9cdcfe">name</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"name"</span> <span style="color: #9cdcfe">id</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"name"</span><span style="color: gray">/></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">input</span> <span style="color: #9cdcfe">type</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"submit"</span> <span style="color: #9cdcfe">value</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"Send"</span> <span style="color: gray">/></span>
  </p>
  
  <p style="margin: 0px">
    }
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray"><</span><span style="color: #569cd6">hr</span> <span style="color: gray">/></span>
  </p>
  
  <p style="margin: 0px">
    Aquí irá el resultado: <span style="color: gray"><</span><span style="color: #569cd6">p</span> <span style="color: gray">/></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray"><</span><span style="color: #569cd6">div</span> <span style="color: #9cdcfe">id</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"datadiv"</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray"></</span><span style="color: #569cd6">div</span><span style="color: gray">></span>
  </p></p>
</div>

Y finalmente nos queda la vista PostData.cshtml que contiene el código HTML que se incrustará en el div “datadiv”:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: gray"><</span><span style="color: #569cd6">h2</span><span style="color: gray">></span>Hola <span style="background: #ffffb3; color: black">@</span><span style="color: white">ViewBag</span><span style="color: #b4b4b4">.</span><span style="color: white">DataName</span><span style="color: gray"></</span><span style="color: #569cd6">h2</span><span style="color: gray">></span>
  </p></p>
</div>

Tnemos que hacer una última modificación a la página de _Layout.cshtml que es añadir el bundle que nos añade (entre otros) jQuery-unobstrusive-ajax.js. Para ello añadimos la siguiente línea **justo antes del RenderSection**:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="background: #ffffb3; color: black">@</span><span style="color: #4ec9b0">Scripts</span><span style="color: #b4b4b4">.</span><span style="color: white">Render</span>(<span style="color: #d69d85">"~/bundles/jqueryval"</span>)
  </p></p>
</div>

Listos, con esto tenemos un formulario que es enviado via Ajax al servidor y el resultado (la vista PostData.cshtml) se incrusta en el formulario.

Ahora simplemente reemplazamos el fichero de jQuery 1.7.1 por el de jQuery 1.9.0 y volvemos a probar la aplicación.

Veréis que el formulario **no** se envía por Ajax (en su lugar se envía por un POST normal) por lo que el resultado de PostData.cshtml no se incrusta en el div si no que pasa a ser toda la nueva página.

**La causa del error y la solución**

La causa es que jquery-unobtrusive-ajax.js que es quien se encarga de dar soporte al unobtrusive ajax de ASP.NET MVC usa el método live de jQuery. Pero **dicho método fue declarado obsoleto en jQuery 1.7 y se ha eliminado en 1.9**. Dicho método permitía asociarse a un evento de cualquier elemento del DOM actual o futuro.

El método que debe usarse actualmente en lugar de live es el método on. No obstante la sintaxis es un poco distinta, ya que el método on tiene más usos en jQuery.

<div st
yle="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  </p> 
  
  <p style="margin: 0px">
    <span style="color: white">$</span><span style="color: #b4b4b4">(</span><span style="color: #d69d85">"form[data-ajax=true]"</span><span style="color: #b4b4b4">).</span><span style="color: white">live</span><span style="color: #b4b4b4">(</span><span style="color: #d69d85">"submit"</span><span style="color: #b4b4b4">,</span> <span style="color: #569cd6">function</span> <span style="color: #b4b4b4">(</span><span style="color: white">evt</span><span style="color: #b4b4b4">)</span> <span style="color: #b4b4b4">{</span>
  </p></p>
</div>

Vamos a modificar la llamada a “live” a una llamada a “on”. Para que “on” actue como live debemos pasarle a “on” 3 parámetros:

  * El evento (igual que live, será “submit”) 
  * Un selector (elementos “hijos”) del selector base que es el que debe existir siempre 
  * La función gestora (igual que live). 

En este caso nuestra línea queda como:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    &#160;<span style="color: white">$</span><span style="color: #b4b4b4">(</span><span style="color: #d69d85">"body"</span><span style="color: #b4b4b4">).</span><span style="color: white">on</span><span style="color: #b4b4b4">(</span><span style="color: #d69d85">"submit"</span><span style="color: #b4b4b4">,</span> <span style="color: #d69d85">"form[data-ajax=true]"</span><span style="color: #b4b4b4">,</span><span style="color: #569cd6">function</span> <span style="color: #b4b4b4">(</span><span style="color: white">evt</span><span style="color: #b4b4b4">)</span> <span style="color: #b4b4b4">{</span>
  </p></p>
</div>

Fijaos: he movido el selector al segundo parámetro de “on” y he puesto como selector base “body” (no es el más óptimo, pero así estoy seguro de que existe siempre). La idea es que la función pasada se asocia a todos los elementos presentes y futuros de tipo “form[data-ajax=true]” que estén dentro del selector base (body).

Para el resto de llamadas a live (hay 3 más) hacemos la misma sustitución que nos quedarán así:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: white">$</span><span style="color: #b4b4b4">(</span><span style="color: #d69d85">"body"</span><span style="color: #b4b4b4">).</span><span style="color: white">on</span><span style="color: #b4b4b4">(</span><span style="color: #d69d85">"click"</span><span style="color: #b4b4b4">,</span> <span style="color: #d69d85">"form[data-ajax=true] :submit"</span><span style="color: #b4b4b4">,</span> <span style="color: #569cd6">function</span> <span style="color: #b4b4b4">(</span><span style="color: white">evt</span><span style="color: #b4b4b4">)</span> <span style="color: #b4b4b4">{</span>
  </p></p>
</div>

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: white">$</span><span style="color: #b4b4b4">(</span><span style="color: #d69d85">"body"</span><span style="color: #b4b4b4">).</span><span style="color: white">on</span><span style="color: #b4b4b4">(</span><span style="color: #d69d85">"click"</span><span style="color: #b4b4b4">,</span> <span style="color: #d69d85">"a[data-ajax=true]"</span><span style="color: #b4b4b4">,</span> <span style="color: #569cd6">function</span> <span style="color: #b4b4b4">(</span><span style="color: white">evt</span><span style="color: #b4b4b4">)</span> <span style="color: #b4b4b4">{</span>
  </p></p>
</div>

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: white">$</span><span style="color: #b4b4b4">(</span><span style="color: #d69d85">"body"</span><span style="color: #b4b4b4">).</span><span style="color: white">on</span><span style="color: #b4b4b4">(</span><span style="color: #d69d85">"click"</span><span style="color: #b4b4b4">,</span> <span style="color: #d69d85">"form[data-ajax=true] input[type=image]"</span><span style="color: #b4b4b4">,</span> <span style="color: #569cd6">function</span> <span style="color: #b4b4b4">(</span><span style="color: white">evt</span><span style="color: #b4b4b4">)</span> <span style="color: #b4b4b4">{</span>
  </p></p>
</div>

Y listos! Con esto ya hemos recuperado la funcionalidad del unobtrusive ajax de MVC y nuestra aplicación debería volver a funcionar correctamente!

Saludos!

> PD: Actualmente el fichero jquery-unobstrusive-ajax.js se distribuye a través de un paquete de NuGet llamado <a href="http://nuget.org/packages/Microsoft.jQuery.Unobtrusive.Ajax" target="_blank" rel="noopener noreferrer">Microsoft jQuery Unobtrusive Ajax</a>. Es de esperar que en breve lo actualizarán 😉