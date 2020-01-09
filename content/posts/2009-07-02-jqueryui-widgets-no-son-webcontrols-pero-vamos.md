---
title: jQueryUI widgets… no son webcontrols pero vamos…
author: eiximenis

date: 2009-07-02T15:49:00+00:00
geeks_url: /?p=1460
geeks_visits:
  - 2561
geeks_ms_views:
  - 1709
categories:
  - Uncategorized

---
&iexcl;&iexcl;Hola a todos!! ¿Como vamos?

Una de los argumentos que más usan aquellos a quienes no les gusta ASP.NET MVC és &ldquo;que hemos vuelto a los 90&rdquo;, refiriendose, entre otras cosas, a que en ASP.NET MVC no existe el concepto de &ldquo;controles&rdquo; y que continuamente estamos &ldquo;mezclando&rdquo; código de cliente y de servidor, lo que lleva a una [tag-soup][1] que _recuerda_ peligrosamente a la ASP clásica...

No quiero discutir en este post si gusta más el modelo MVC que el clásico de webforms (para quien le interese esta discusión puede pasarse por [este post en el blog de jersson][2]), sinó decir que ambos argumentos en mi opinión no tienen validez.

Las críticas al tag-soup generalmente vienen porque se asocia la mezcla de etiquetas de cliente y servidor con el código spaghetti que teníamos en las páginas ASP clásicas. Pero no es ni mucho menos lo mismo: una vista MVC tiene sólamente etiquetas de servidor que gestionan **lógica de presentación**. Esto implica que deberían ser muchas menos etiquetas de servidor que en las clásicas páginas ASP. Si, aún así, nos aparecen muchas y nuestro código nos sigue recordando al clásico ASP, hay varias maneras de minimizarlo:

  1. Utilizar un motor de vistas alternativo (hay varios _por ahí afuera_) como p.ej. [Brial][3], [NHaml][4], [NVelocity][5] y [XSLT][6], solo por citar los cuatro que vienen con [MVCContrib][7]. 
  2. Extender los dos helpers (HtmlHelper y AjaxHelper) con funciones propias que generen código HTML. Pasaros por este post de Rob Conery ([How to avoid tag soup][8]) para ver un ejemplo práctico. 
  3. Utilizar jQuery. Ya lo he comentado varias veces y no me cansaré de decirlo: el poder de jQuery es tremebundo. Muchas cosas que tendemos a hacer con código de servidor se pueden hacer con un par de líneas de jQuery... La contrapartida a pagar es obvia: javascript debe estar habilitado en el navegador, pero vamos... en el 99% de los casos esto no es problema alguno. 

Un ejemplo clásico de este punto tres... quieres mostrar una tabla con varios registros, y que las filas pares tengan un estilo y las impares otro? Puedes generar código con tags <% if %> para que te genere un estilo u otro:

<pre class="code"><span style="color: blue">&lt;</span><span style="color: #a31515">table</span><span style="color: blue">&gt;
    </span><span style="background: #ffee62">&lt;%</span> int idx = 0; <span style="background: #ffee62">%&gt;
</span>    <span style="background: #ffee62">&lt;%</span> foreach (var item in Model) { <span style="background: #ffee62">%&gt;
</span>        <span style="background: #ffee62">&lt;%</span> if ((idx++ % 2) == 0) {<span style="background: #ffee62">%&gt;
</span>        <span style="color: blue">&lt;</span><span style="color: #a31515">tr </span><span style="color: red">class</span><span style="color: blue">="odd"&gt;
        </span><span style="background: #ffee62">&lt;%</span> } else { <span style="background: #ffee62">%&gt;
</span>        <span style="color: blue">&lt;</span><span style="color: #a31515">tr </span><span style="color: red">class</span><span style="color: blue">="even"&gt;
        </span><span style="background: #ffee62">&lt;%</span> } <span style="background: #ffee62">%&gt;
</span>            <span style="color: blue">&lt;</span><span style="color: #a31515">td</span><span style="color: blue">&gt;
                </span><span style="background: #ffee62">&lt;%</span><span style="color: blue">= </span>Html.Encode(item.ID) <span style="background: #ffee62">%&gt;
</span>            <span style="color: blue">&lt;/</span><span style="color: #a31515">td</span><span style="color: blue">&gt;
            &lt;</span><span style="color: #a31515">td</span><span style="color: blue">&gt;
                </span><span style="background: #ffee62">&lt;%</span><span style="color: blue">= </span>Html.Encode(item.Name) <span style="background: #ffee62">%&gt;
</span>            <span style="color: blue">&lt;/</span><span style="color: #a31515">td</span><span style="color: blue">&gt;
        &lt;/</span><span style="color: #a31515">tr</span><span style="color: blue">&gt;
    </span><span style="background: #ffee62">&lt;%</span> } <span style="background: #ffee62">%&gt;
</span>    <span style="color: blue">&lt;/</span><span style="color: #a31515">table</span><span style="color: blue">&gt;</span></pre>

[][9]

O bien generar la tabla &ldquo;tal cual&rdquo;:

<pre class="code"><span style="color: blue">&lt;</span><span style="color: #a31515">table</span><span style="color: blue">&gt;
    </span><span style="background: #ffee62">&lt;%</span> <span style="color: blue">foreach </span>(<span style="color: blue">var </span>item <span style="color: blue">in </span>Model) { <span style="background: #ffee62">%&gt;
</span>        <span style="color: blue">&lt;</span><span style="color: #a31515">tr</span><span style="color: blue">&gt;
            &lt;</span><span style="color: #a31515">td</span><span style="color: blue">&gt;
                </span><span style="background: #ffee62">&lt;%</span><span style="color: blue">= </span>Html.Encode(item.ID) <span style="background: #ffee62">%&gt;
</span>            <span style="color: blue">&lt;/</span><span style="color: #a31515">td</span><span style="color: blue">&gt;
            &lt;</span><span style="color: #a31515">td</span><span style="color: blue">&gt;
                </span><span style="background: #ffee62">&lt;%</span><span style="color: blue">= </span>Html.Encode(item.Name) <span style="background: #ffee62">%&gt;
</span>            <span style="color: blue">&lt;/</span><span style="color: #a31515">td</span><span style="color: blue">&gt;
        &lt;/</span><span style="color: #a31515">tr</span><span style="color: blue">&gt;
    </span><span style="background: #ffee62">&lt;%</span> } <span style="background: #ffee62">%&gt;
</span>    <span style="color: blue">&lt;/</span><span style="color: #a31515">table</span><span style="color: blue">&gt;
</span></pre>

[][9]

y&nbsp; dejar que jQuery haga el trabajo:

<pre class="code"><span style="color: blue">&lt;</span><span style="color: #a31515">script </span><span style="color: red">type</span><span style="color: blue">="text/javascript"&gt;
    </span>$(document).ready(<span style="color: blue">function</span>() {
        $(<span style="color: #a31515">"tr:odd"</span>).addClass(<span style="color: #a31515">"odd"</span>);
        $(<span style="color: #a31515">"tr:even"</span>).addClass(<span style="color: #a31515">"even"</span>);
    });
<span style="color: blue">&lt;/</span><span style="color: #a31515">script</span><span style="color: blue">&gt;</span></pre>

[][9]

El argumento de que no hay controles tampoco me es válido. Mi respuesta a quien me lo formula suele ser: ¿Y para que los necesito? Dime un webcontrol y muy probablemente te podré decir un plugin de jQuery que hace lo mismo (sinó mejor)... con la ventaja de que el código final que se genera es mucho más limpio. De acuerdo, no hay diseño RAD con un editor visual de controles, pero honestamente creo que vale la pena pagar este precio. P.ej. para comparar la productividad de ASP.NET MVC versus la productividad de webforms muchas veces se menciona el control [GridView][10]. Evidentemente implementar un GridView &ldquo;a mano&rdquo; no es trivial, pero por ejemplo... ¿habéis echado un vistazo al [plugin de jQuery para grids][11]? ¿No? Hacedlo y vereis que es brutal.

Hoy os quiero enseñar una mini-demo de &ldquo;estos controles jQuery&rdquo; por decirlo de algún modo (widgets en terminología jQuery UI). He escogido el DatePicker que forma parte de la suite [jQuery UI][12] que es un conjunto de &ldquo;controles&rdquo; (así entre comillas) implementados como plug-in de jQuery, además de otros detalles (como nuevos efectos de animaciones y soporte para drag-and-drop).

El primer paso es descargarse jQuery UI desde [http://jqueryui.com/download][13]. Podeis bajaroslo todo o seleccionar solo aquellas partes que necesiteis (en mi caso tengo toda la versión 1.7.2, la última a la hora de escribir este post).

Crea un nuevo proyecto ASP.NET MVC (en mi caso he eliminado el AccountController y todas sus vistas (Views/Account/\*.\*) así como Views/Shared/LogOnUserControl.ascx). Como siempre las modificaciones las realizo en la página inicial (Views/Home/Index.aspx).

Lo primero es incluir jQuery y jQuery UI en la página master (para que sea usable desde cualquier página). Abrid Views/Shared/Site.master y añadid los archivos de jQuery, jQuery UI, que en mi caso son los siguientes:

<pre class="code"><span style="color: blue">&lt;</span><span style="color: #a31515">link </span><span style="color: red">href</span><span style="color: blue">="../../Content/jquery-ui-1.7.2.custom.css" <br /></span><span style="color: red">rel</span><span style="color: blue">="stylesheet" </span><span style="color: red">type</span><span style="color: blue">="text/css" /&gt;
&lt;</span><span style="color: #a31515">script </span><span style="color: red">type</span><span style="color: blue">="text/javascript" <br /></span><span style="color: red">src</span><span style="color: blue">="../../Scripts/jquery-1.3.2.js"&gt;&lt;/</span><span style="color: #a31515">script</span><span style="color: blue">&gt;
&lt;</span><span style="color: #a31515">script </span><span style="color: red">type</span><span style="color: blue">="text/javascript" <br /></span><span style="color: red">src</span><span style="color: blue">="../../Scripts/jquery-ui-1.7.2.custom.min.js"&gt;&lt;/</span><span style="color: #a31515">script</span><span style="color: blue">&gt;</span></pre>

[][9]

Copiad tambien la carpeta images que viene con &ldquo;jQuery UI&rdquo; dentro del directorio &ldquo;Content&rdquo; de ASP.NET MVC (el directorio &ldquo;images&rdquo; contiene imágenes usadas por los widgets de jQuery UI).

Ahora modificamos la vista (Views/Home/Index.aspx) para incluir un textbox:

<pre class="code"><span style="color: blue">&lt;</span><span style="color: #a31515">input </span><span style="color: red">type</span><span style="color: blue">="text" </span><span style="color: red">id</span><span style="color: blue">="dtPicker" /&gt;</span></pre>

[][9]

Y finalmente vinculamos este elemento con un DatePicker de jQuery UI:

<pre class="code"><span style="color: blue">&lt;</span><span style="color: #a31515">script </span><span style="color: red">type</span><span style="color: blue">="text/javascript"&gt;
    </span>$(document).ready(<span style="color: blue">function</span>() {
        $(<span style="color: #a31515">"#dtPicker"</span>).datepicker();
    });
<span style="color: blue">&lt;/</span><span style="color: #a31515">script</span><span style="color: blue">&gt;</span></pre>

[][9]

&iexcl;y listos! Nos aparece un textbox y cuando pulsamos sobre él se abre el DatePicker:

> [<img height="145" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_3C4591BC.png" alt="image" border="0" title="image" style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" />][14] 

&iexcl;Cómo podeis apreciar no puede ser más facil! Al pulsar sobre un día este se seleccionar y es introducido en el textbox.

Podemos personalizar el DatePicker usando opciones cuando lo invocamos. P.ej. podemos establecer un intervalo de fechas válidas

<pre class="code"><span style="color: green">// Solo valido desde el 02/Feb/2009 hasta 15/Feb/2009
</span>$(<span style="color: #a31515">"#dtPicker"</span>).datepicker({ maxDate: <span style="color: blue">new </span>Date(2009, 1, 15),
    minDate: <span style="color: blue">new </span>Date(2009,1,2) });</pre>

[][9][][9]

Y efectivamente podeis ver como el resto de fechas quedan desactivadas:

[<img height="134" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_2273EC3E.png" alt="image" border="0" title="image" style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" />][15] 

Esto ha sido sólo un pequeño ejemplo... hay muchos más widgets en jQuery UI!! Echadle un vistazo en[http://jqueryui.com/demos/][16]. Y luego no olvideis que hay otros plug-ins para jQuery que siendo también &ldquo;widgets&rdquo; no forman parte de jQuery UI!

Saludos!!!!!

 [1]: http://en.wikipedia.org/wiki/Tag_soup
 [2]: /blogs/jersson/archive/2009/06/21/asp-net-mvc-como-asp-tradicional.aspx
 [3]: http://www.codeplex.com/MVCContrib/Wiki/View.aspx?title=Brail&referringTitle=Documentation
 [4]: http://code.google.com/p/nhaml/
 [5]: http://www.castleproject.org/monorail/documentation/trunk/viewengines/nvelocity/index.html
 [6]: http://www.ohloh.net/accounts/maxtoroq/messages/33362
 [7]: http://www.codeplex.com/MVCContrib
 [8]: http://blog.wekeroad.com/blog/asp-net-mvc-avoiding-tag-soup/
 [9]: http://11011.net/software/vspaste
 [10]: http://msdn.microsoft.com/en-us/library/system.web.ui.webcontrols.gridview.aspx
 [11]: http://www.trirand.com/blog/
 [12]: http://jqueryui.com/
 [13]: http://jqueryui.com/download "http://jqueryui.com/download"
 [14]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_302186E4.png
 [15]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_27C2D2EF.png
 [16]: http://jqueryui.com/demos/ "http://jqueryui.com/demos/"