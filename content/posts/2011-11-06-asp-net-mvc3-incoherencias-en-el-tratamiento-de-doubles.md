---
title: 'ASP.NET MVC3: Incoherencias en el tratamiento de doubles'
author: eiximenis

date: 2011-11-06T11:29:44+00:00
geeks_url: /?p=1583
geeks_visits:
  - 1652
geeks_ms_views:
  - 834
categories:
  - Uncategorized

---
Muy, muy, muy molesto…&#160; ASP.NET MVC3 corriendo sobre un servidor web configurado en español (cultura es-ES).

Con la tabla de rutas estándar, cuatro acciones como las siguientes

<div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
  <div style="background: #ddd; max-height: 300px; overflow: auto">
    <ol style="padding-bottom: 0px; margin: 0px 0px 0px 2.5em; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
      <li>
        [<span style="color: #2b91af">HttpPost</span>]
      </li>
      <li style="background: #f3f3f3">
        <span style="color: #0000ff">public</span> <span style="color: #2b91af">ActionResult</span> Index(<span style="color: #2b91af">DoubleModel</span> model)
      </li>
      <li>
        {
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; ViewBag.Valor = model.Valor;
      </li>
      <li>
        &#160;
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; <span style="color: #0000ff">return</span> View(<span style="color: #a31515">"Resultado"</span>);
      </li>
      <li>
        }
      </li>
      <li style="background: #f3f3f3">
        &#160;
      </li>
      <li>
        [<span style="color: #2b91af">HttpPost</span>]
      </li>
      <li style="background: #f3f3f3">
        <span style="color: #0000ff">public</span> <span style="color: #2b91af">ActionResult</span> IndexSoloDouble(<span style="color: #0000ff">double</span>? valor)
      </li>
      <li>
        {
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; ViewBag.Valor = valor;
      </li>
      <li>
        &#160;
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; <span style="color: #0000ff">return</span> View(<span style="color: #a31515">"Resultado"</span>);
      </li>
      <li>
        }
      </li>
      <li style="background: #f3f3f3">
        &#160;
      </li>
      <li>
        &#160;
      </li>
      <li style="background: #f3f3f3">
        <span style="color: #0000ff">public</span> <span style="color: #2b91af">ActionResult</span> IndexRuta(<span style="color: #0000ff">double</span>? id)
      </li>
      <li>
        {
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; ViewBag.Valor = id;
      </li>
      <li>
        &#160;&#160;&#160; <span style="color: #0000ff">return</span> View(<span style="color: #a31515">"Resultado"</span>);
      </li>
      <li style="background: #f3f3f3">
        }
      </li>
      <li>
        &#160;
      </li>
      <li style="background: #f3f3f3">
        <span style="color: #0000ff">public</span> <span style="color: #2b91af">ActionResult</span> IndexGet(<span style="color: #0000ff">double</span>? valor)
      </li>
      <li>
        {
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; ViewBag.Valor = valor;
      </li>
      <li>
        &#160;&#160;&#160; <span style="color: #0000ff">return</span> View(<span style="color: #a31515">"Resultado"</span>);
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160;&#160;
      </li>
      <li>
        }
      </li>
    </ol>
  </div></p>
</div>

La clase DoubleMode simplemente tiene una propiedad double llamada Valor.

  1. La primera acción recibe un double via POST&#160; dentro de un modelo 
  2. La segunda recibe un double via POST sólo 
  3. La tercera acción recibe un double como parámetro de ruta 
  4. La cuarta acción recibe un double via querystring 

Pues bien:

  1. La primera acción acepta 12,10 pero no 12.10 (usa la cultura del servidor web) 
  2. La segunda acción acepta 12,10 pero no 12.10 (usa la cultura del servidor web) 
  3. La tercera acción acepta 12.10 pero no 12,10 (usa la cultura ¿invariante?) 
  4. La cuarta acción acepta 12.10 pero no 12, 10 (como la tercera, parece usar la invariante). 

Aquí tenéis el código de la vista usada para enviar los datos:

<div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
  <div style="background: #ddd; max-height: 300px; overflow: auto">
    <ol style="padding-bottom: 0px; margin: 0px 0px 0px 2.5em; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
      <li>
        <span style="background: #ffff00">@</span><span style="color: #0000ff">using</span> MvcApplication1.Models
      </li>
      <li style="background: #f3f3f3">
        <span style="background: #ffff00">@model </span><span style="color: #2b91af">DoubleModel</span>
      </li>
      <li>
        &#160;
      </li>
      <li style="background: #f3f3f3">
        <span style="background: #ffff00">@{</span>
      </li>
      <li>
        &#160;&#160;&#160; ViewBag.Title = <span style="color: #a31515">"Index"</span>;
      </li>
      <li style="background: #f3f3f3">
        }
      </li>
      <li>
        &#160;
      </li>
      <li style="background: #f3f3f3">
        <span style="color: #0000ff"><</span><span style="color: #800000">h2</span><span style="color: #0000ff">></span>Index<span style="color: #0000ff"></</span><span style="color: #800000">h2</span><span style="color: #0000ff">></span>
      </li>
      <li>
        &#160;
      </li>
      <li style="background: #f3f3f3">
        <span style="background: #ffff00">@{</span>
      </li>
      <li>
        &#160;&#160;&#160; Html.EnableClientValidation(<span style="color: #0000ff">false</span>);
      </li>
      <li style="background: #f3f3f3">
        }
      </li>
      <li>
        &#160;
      </li>
      <li style="background: #f3f3f3">
        <span style="background: #ffff00">@</span><span style="color: #0000ff">using</span> (Html.BeginForm()) {
      </li>
      <li>
        &#160;&#160;&#160; <span style="color: #0000ff"><</span><span style="color: #800000">div</span><span style="color: #0000ff">></span>
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; Introduce el valor:
      </li>
      <li>
        &#160;&#160;&#160; <span style="color: #0000ff"><</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="hidden"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="Valor"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="12,10"</span> <span style="color: #0000ff">/></span>
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; <span style="color: #0000ff"></</span><span style="color: #800000">div</span><span style="color: #0000ff">></span>
      </li>
      <li>
        &#160;&#160;&#160; <span style="color: #0000ff"><</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="submit"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="enviar 12,10 via POST"</span> <span style="color: #0000ff">/></span>
      </li>
      <li style="background: #f3f3f3">
        }
      </li>
      <li>
        &#160;
      </li>
      <li style="background: #f3f3f3">
        <span style="background: #ffff00">@</span><span style="color: #0000ff">using</span> (Html.BeginForm()) {
      </li>
      <li>
        &#160;&#160;&#160; <span style="color: #0000ff"><</span><span style="color: #800000">div</span><span style="color: #0000ff">></span>
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; <span style="color: #0000ff"><</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="hidden"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="Valor"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="12.10"</span> <span style="color: #0000ff">/></span>
      </li>
      <li>
        &#160;&#160;&#160; <span style="color: #0000ff"></</span><span style="color: #800000">div</span><span style="color: #0000ff">></span>
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; <span style="color: #0000ff"><</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="submit"</span> <span style="color: #ff0000">value< /span><span style="color: #0000ff">="enviar 12.10 via POST"</span> <span style="color: #0000ff">/></span> </li> 
        
        <li>
          }
        </li>
        <li style="background: #f3f3f3">
          &#160;
        </li>
        <li>
          &#160;
        </li>
        <li style="background: #f3f3f3">
          <span style="background: #ffff00">@</span><span style="color: #0000ff">using</span> (Html.BeginForm(<span style="color: #a31515">"IndexSoloDouble"</span>,<span style="color: #a31515">"Home"</span>))
        </li>
        <li>
          {
        </li>
        <li style="background: #f3f3f3">
          &#160;&#160;&#160; <span style="color: #0000ff"><</span><span style="color: #800000">div</span><span style="color: #0000ff">></span>
        </li>
        <li>
          &#160;&#160;&#160; <span style="color: #0000ff"><</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="hidden"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="Valor"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="12,10"</span> <span style="color: #0000ff">/></span>
        </li>
        <li style="background: #f3f3f3">
          &#160;&#160;&#160; <span style="color: #0000ff"></</span><span style="color: #800000">div</span><span style="color: #0000ff">></span>
        </li>
        <li>
          &#160;&#160;&#160; <span style="color: #0000ff"><</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="submit"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="enviar 12,10 via POST (solo double)"</span> <span style="color: #0000ff">/></span>
        </li>
        <li style="background: #f3f3f3">
          }
        </li>
        <li>
          &#160;
        </li>
        <li style="background: #f3f3f3">
          &#160;
        </li>
        <li>
          <span style="background: #ffff00">@</span><span style="color: #0000ff">using</span> (Html.BeginForm(<span style="color: #a31515">"IndexSoloDouble"</span>, <span style="color: #a31515">"Home"</span>))
        </li>
        <li style="background: #f3f3f3">
          {
        </li>
        <li>
          &#160;&#160;&#160; <span style="color: #0000ff"><</span><span style="color: #800000">div</span><span style="color: #0000ff">></span>
        </li>
        <li style="background: #f3f3f3">
          &#160;&#160;&#160; <span style="color: #0000ff"><</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="hidden"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="Valor"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="12.10"</span> <span style="color: #0000ff">/></span>
        </li>
        <li>
          &#160;&#160;&#160; <span style="color: #0000ff"></</span><span style="color: #800000">div</span><span style="color: #0000ff">></span>
        </li>
        <li style="background: #f3f3f3">
          &#160;&#160;&#160; <span style="color: #0000ff"><</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="submit"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="enviar 12.10 via POST (solo double)"</span> <span style="color: #0000ff">/></span>
        </li>
        <li>
          }
        </li>
        <li style="background: #f3f3f3">
          &#160;
        </li>
        <li>
          &#160;
        </li>
        <li style="background: #f3f3f3">
          &#160;
        </li>
        <li>
          <span style="background: #ffff00">@</span>Html.ActionLink(<span style="color: #a31515">"Enviar 12,10 via GET"</span>, <span style="color: #a31515">"IndexGet"</span>, <span style="color: #0000ff">new</span> { valor = <span style="color: #a31515">"12,10"</span> }); | <span style="background: #ffff00">@</span>Html.ActionLink(<span style="color: #a31515">"Enviar 12.10 via GET"</span>, <span style="color: #a31515">"IndexGet"</span>, <span style="color: #0000ff">new</span> { valor = <span style="color: #a31515">"12.10"</span> });
        </li>
        <li style="background: #f3f3f3">
          <span style="color: #0000ff"><</span><span style="color: #800000">br</span><span style="color: #0000ff">/></span>
        </li>
        <li>
          <span style="background: #ffff00">@</span>Html.ActionLink(<span style="color: #a31515">"Enviar 12,10 en ruta"</span>, <span style="color: #a31515">"IndexRuta"</span>, <span style="color: #0000ff">new</span> { id = <span style="color: #a31515">"12,10"</span> }); | <span style="background: #ffff00">@</span>Html.ActionLink(<span style="color: #a31515">"Enviar 12.10 en ruta"</span>, <span style="color: #a31515">"IndexRuta"</span>, <span style="color: #0000ff">new</span> { id = <span style="color: #a31515">"12.10"</span> });
        </li></ol></div> </p></div> 
        
        <p>
          El problema parece estar en los ValueProviders de URL (RouteDataValueProvider y QueryStringVaueProvider), que parecen ignorar la cultura, mientras que el ValueProvider de POST (FormValueProvider) la respeta.
        </p>
        
        <p>
          Lo dicho… muy molesto 🙁
        </p>
        
        <p>
          Saludos!
        </p>
        
        <p>
          <em><strong>Editado 16:43.</strong></em> PD: <a href="http://www.iloire.com/">Iván Loire</a> (<a href="http://twitter.com/#!/ivanloire">@ivanloire</a>) me comenta que este comportamiento es por diseño: los locales NO se aplican en los parámetros de URL para mantener las URLs canónicas. Si usáramos los locales del usuario podríamos tener URLs distintas que sirviesen el mismo contenido, y URLs válidas para un usuario podrían no serlo para otro. Es por ello que en MVC han optado por usar la cultura invariante en los ValueProviders de la URL. Dejo dos enlaces que me ha pasado Iván a dos posts donde se explica esto: <a href="http://bit.ly/tIZX64">http://bit.ly/tIZX64</a> y <a title="http://bit.ly/vCgbmf" href="http://bit.ly/vCgbmf">http://bit.ly/vCgbmf</a>
        </p>