---
title: Saca tus scripts de tu código HTML
description: Saca tus scripts de tu código HTML
author: eiximenis

date: 2010-11-12T12:42:00+00:00
geeks_url: /?p=1541
geeks_visits:
  - 4871
geeks_ms_views:
  - 2230
categories:
  - Uncategorized

---
Buenas! En el post anterior os comenté el soporte de [Unobtrusive Ajax en ASP.NET MVC3][1]. Hoy quiero mostraros que esa técnica **ni** es exclusiva de MVC3, **ni**&nbsp; requiere HTML5 para nada. En fin, que podéis empezar a usarla ya, con independencia de la tecnología que uséis. Lo que contaré en este artículo no es nada &ldquo;revolucionario&rdquo; ni una &ldquo;técnica nueva&rdquo;...

De hecho, el ejemplo va a ser una página HTML, nada de ASP.NET 🙂

Veamos, la técnica de Unobtrusive Javascript, se refiere a **no tener mezclado código javascript con código de marcado HTML**. Es decir, **no** queremos algo como:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff;">&lt;</span><span style="color: #800000;">input</span> <span style="color: #ff0000;">type</span><span style="color: #0000ff;">="text"</span> <span style="color: #ff0000;">id</span><span style="color: #0000ff;">="txtName"</span> <span style="color: #ff0000;">onkeypress</span><span style="color: #0000ff;">="checkKey();"</span> <span style="color: #0000ff;">/&gt;</span></pre>
</div>

Aquí estamos mezclando código HTML con el código javascript (la llamada checkKey en el _onkeypress_).

Imaginemos que queremos que nuestros **textboxes sólo acepten números**. Y recordad que el objetivo es no tener código javascript mezclado con nuestro código HTML.

Eso lo podemos conseguir fácilmente, ya con jQuery:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">&lt;html xmlns=<span style="color: #006080;">"http://www.w3.org/1999/xhtml"</span>&gt;<br />&lt;head&gt;<br />    &lt;title&gt;Demo Unobtrusive Javascript&lt;/title&gt;<br />    &lt;script src=<span style="color: #006080;">"jquery-1.4.1.js"</span> type=<span style="color: #006080;">"text/javascript"</span>&gt;&lt;/script&gt;<br />&lt;/head&gt;<br />&lt;body&gt;<br />    &lt;script type=<span style="color: #006080;">"text/javascript"</span>&gt;<br />        $(document).ready(<span style="color: #0000ff;">function</span> () {<br />            $(<span style="color: #006080;">'input:text'</span>).keypress(<span style="color: #0000ff;">function</span> (<span style="color: #0000ff;">event</span>) {<br />                <span style="color: #0000ff;">if</span> (<span style="color: #0000ff;">event</span>.keyCode &lt; 47 || <span style="color: #0000ff;">event</span>.keyCode &gt; 58) {<br />                    <span style="color: #0000ff;">event</span>.preventDefault();<br />                }<br />            });<br />        });<br />    &lt;/script&gt;<br /><br />    Introduce sólo números: &lt;br /&gt;<br />    &lt;input type=<span style="color: #006080;">"text"</span> /&gt;<br />&lt;/body&gt;<br />&lt;/html&gt;</pre>
</div>

Incluso, si no queréis que haya el tag <script> con todo el código, podemos moverlo a un .js separado y usarlo desde nuestra página HTML que entonces quedaría como:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">&lt;html xmlns=<span style="color: #006080;">"http://www.w3.org/1999/xhtml"</span>&gt;<br />&lt;head&gt;<br />    &lt;title&gt;Demo Unobtrusive Javascript&lt;/title&gt;<br />    &lt;script src=<span style="color: #006080;">"jquery-1.4.1.js"</span> type=<span style="color: #006080;">"text/javascript"</span>&gt;&lt;/script&gt;<br />    &lt;script src=<span style="color: #006080;">"myscript.js"</span> type=<span style="color: #006080;">"text/javascript"</span>&gt;&lt;/script&gt;<br />&lt;/head&gt;<br />&lt;body&gt;<br />    Introduce sólo números: &lt;br /&gt;<br />    &lt;input type=<span style="color: #006080;">"text"</span> /&gt;<br />&lt;/body&gt;<br />&lt;/html&gt;</pre>
</div>

Por lo tanto vemos que con jQuery es muy fácil asignar comportamiento a objetos DOM, sin necesidad de andar con los handlers onXXXX.

Ahora bien, el código jQuery selecciona **todos** los <input type=&rdquo;text&rdquo;>, que passa si sólo quiero seleccionar _algunos?_ Como le indico a mi código jQuery que sólo algunos textboxes son numéricos?

Una solución es _invertarnos_ un atributo que indique que elementos queremos como numéricos. De esta manera p.ej. la página HTML queda como:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">&lt;html xmlns=<span style="color: #006080;">"http://www.w3.org/1999/xhtml"</span>&gt;<br />&lt;head&gt;<br />    &lt;title&gt;Demo Unobtrusive Javascript&lt;/title&gt;<br />    &lt;script src=<span style="color: #006080;">"jquery-1.4.1.js"</span> type=<span style="color: #006080;">"text/javascript"</span>&gt;&lt;/script&gt;<br />    &lt;script src=<span style="color: #006080;">"myscript.js"</span> type=<span style="color: #006080;">"text/javascript"</span>&gt;&lt;/script&gt;<br />&lt;/head&gt;<br />&lt;body&gt;<br />    Introduce sólo números: &lt;br /&gt;<br />    &lt;input type=<span style="color: #006080;">"text"</span> datatype=<span style="color: #006080;">"numeric"</span> /&gt;  &lt;br /&gt;<br />    Aquí puedes introducir lo que quieras: &lt;br /&gt;<br />    &lt;input type=<span style="color: #006080;">"text"</span> /&gt;<br />&lt;/body&gt;<br />&lt;/html&gt;</pre>
</div>

Fijaos en el &ldquo;datatype=&rdquo;numeric&rdquo; que es el atributo que me va a servir para decidir que textboxes son numéricos.

Y el código de myscript.js queda como:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">$(document).ready(<span style="color: #0000ff;">function</span> () {<br />    $(<span style="color: #006080;">'input[datatype=numeric]'</span>).keypress(<span style="color: #0000ff;">function</span> (<span style="color: #0000ff;">event</span>) {<br />        <span style="color: #0000ff;">if</span> (<span style="color: #0000ff;">event</span>.keyCode &lt; 47 || <span style="color: #0000ff;">event</span>.keyCode &gt; 58) {<br />            <span style="color: #0000ff;">event</span>.preventDefault();<br />        }<br />    });<br />});</pre>
</div>

Y listos, simplemente incluyendo &ldquo;myscript.js&rdquo; en cualquier página ya podemos declarar que un textbox es numérico _simplemente_ poniendo el atributo datatype=&rdquo;numeric&rdquo;.

Ahora, si alguien hace otra librería javascript para textboxes numéricos **si también usa este atributo para indicarlos** (ahí está el quid de la cuestión) simplemente cambiando el <script> para que en lugar de ir a myscript.js vaya a la nueva librería, ya tengo todo el cambio hecho... es decir, me he independizado del framework javascript que use.

**Y por ahí por donde entra HTML5?** Pues bien, como eso de crearnos nuestros propios atributos está bien pero genera HTML que podríamos llamar _inválido_ (en el sentido de que estos atributos no forman parte de HTML), para HTML5 han decidido simplemente que todos estos atributos &ldquo;inventados&rdquo; empiecen por _data-_.

Lo &ldquo;único&rdquo; que dice al respecto HTML5 es: &ldquo;_Hey, si tienes que invertarte un atributo para lo que sea, haz que su nombre empiece por data-. Todos los atributos que empiecen por data- son atributos inventados por alquien y deben ser ignorados a todos los efectos (salvo para quien lo haya inventado que hará con él lo que le plazca, claro_). Ok, [también añade una API específica (element.dataset) para leer esos atributos][2] (pero eso de momento no nos importa ya que no está soportada por la mayoría de navegadores).

Por lo tanto, si en lugar de que mi atributo se llame datatype, hago que le llame data-datatype (p.ej. cualquier nombre que empiece por data-) ya lo tengo todo _HTML5 compliant!_

De hecho podéis hacer la prueba en <http://validator.w3.org/check>. Entráis el código HTML de la página y lo validáis contra:

  * HTML5 usando el atributo datatype=&rdquo;numeric&rdquo; y os dará **error** (Attribute not allowed)
  * HTML5 usando el atributo data-datatype=&rdquo;numeric&rdquo; y os validará correctamente.
  * Cualquier otra versión de HTML y os dará error en ambos casos.

Y listos! Por lo tanto fijaos que **desde ya** podeis empezar a aplicar técnicas de &ldquo;Unobtrusive Javascript&rdquo;: no necesitáis HTML5 para nada, ni MVC3 ni nada y la _recompensa_ es un HTML mucho más claro y sencillo de ver!

Mi opinión es que, gracias a que HTML5 ha definido un _espacio de nombres_ (data-) para los _atrbutos inventados_ empezaremos a ver, cada ves más, librerías de javascript que usarán esos atributos, y seguramente algunos de ellos terminarán siendo estándares de facto (si yo hago una librería de javascript para validación. pues intentaré usar los mismos atributos data- que use la librería que sea _líder_ en aquel momento, para compatibilizarme con ella).

Por cierto, si vais a usar muchos atributos data- en vuestras páginas web, echadle un vistazo a este plugin de jQuery: [HTML5 Dataset][3].

Un saludo!

**Nota:** El código de ese artículo lo he probado con IE9 y Firefox 3.6.10.

 [1]: /blogs/etomas/archive/2010/11/09/unobtrusive-ajax-en-mvc3.aspx
 [2]: http://www.w3.org/TR/2009/WD-html5-20090423/dom.html
 [3]: http://plugins.jquery.com/project/html5-dataset