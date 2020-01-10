---
title: ASP.NET MVC, Controles Chart y Ajax…

author: eiximenis

date: 2009-04-02T14:34:00+00:00
geeks_url: /?p=1444
geeks_visits:
  - 5185
geeks_ms_views:
  - 1878
categories:
  - Uncategorized

---
Supongo que la gran mayoría de vosotros, conoceréis los controles de gráficos de ASP.NET. José M. Aguilar hizo un excelente post sobre ellos [aquí (http://geeks.ms/blogs/jmaguilar/archive/2008/12/14/microsoft-chart-control-para-asp-net-3-5-sp1.aspx)][1].

<!--more-->

Utilizarlos es realmente simple... basta con que os los descargueis de la web de Microsoft y después de instalarlos agregueis las siguiente líneas en el `web.config`:

<pre class="code"><span style="color: blue">&lt;</span><span style="color: #a31515">add </span><span style="color: red">path</span><span style="color: blue">=</span>"<span style="color: blue">ChartImg.axd</span>" <span style="color: red">verb</span><span style="color: blue">=</span>"<span style="color: blue">GET,HEAD</span>" <br /><span style="color: red">type</span><span style="color: blue">=</span>"<span style="color: blue">System.Web.UI.DataVisualization.Charting.ChartHttpHandler, <br />System.Web.DataVisualization, Version=3.5.0.0, Culture=neutral, <br />PublicKeyToken=31bf3856ad364e35</span>" <span style="color: red">validate</span><span style="color: blue">=</span>"<span style="color: blue">false</span>"<span style="color: blue">/&gt;</span></pre>

[][2]

En la sección <httpHandlers> y la siguiente:

<pre class="code"><span style="color: blue">&lt;</span><span style="color: #a31515">add </span><span style="color: red">tagPrefix</span><span style="color: blue">=</span>"<span style="color: blue">asp</span>" <br /><span style="color: red">namespace</span><span style="color: blue">=</span>"<span style="color: blue">System.Web.UI.DataVisualization.Charting</span>" <br /><span style="color: red">assembly</span><span style="color: blue">=</span>"<span style="color: blue">System.Web.DataVisualization, Version=3.5.0.0, <br />Culture=neutral, PublicKeyToken=31bf3856ad364e35</span>"<span style="color: blue">/&gt;</span></pre>

[][2]

En la sección <controls>.

Despues ya podeis arrastrar un Chart control desde la toolbox a vuestra página ASP.NET y empezar a trabajar con él.

Si, como yo, os encanta ASP.NET MVC sabed que podeis usar este control sin ningún problema ([http://code-inside.de/blog-in/2008/11/27/howto-use-the-new-aspnet-chart-controls-with-aspnet-mvc/][3]).

El único temilla a tener en cuenta es si se quiere actualizar sólo el gráfico mediante Ajax (usando ASP.NET MVC).

Suponed una vista parcial (Chart.ascx) con el siguiente código que muestra un gráfico con contenido _random_:

<pre class="code"><span style="background: #ffee62">&lt;%</span><span style="color: blue">@ </span><span style="color: #a31515">Control </span><span style="color: red">Language</span><span style="color: blue">="C#" </span><span style="color: red">Inherits</span><span style="color: blue">="System.Web.Mvc.ViewUserControl" </span><span style="background: #ffee62">%&gt;
</span><span style="color: blue">&lt;</span><span style="color: #a31515">asp</span><span style="color: blue">:</span><span style="color: #a31515">Chart </span><span style="color: red">ID</span><span style="color: blue">="chart" </span><span style="color: red">runat</span><span style="color: blue">="server" </span><span style="color: red">Palette</span><span style="color: blue">="Fire" &gt;
    &lt;</span><span style="color: #a31515">Series</span><span style="color: blue">&gt;
        &lt;</span><span style="color: #a31515">asp</span><span style="color: blue">:</span><span style="color: #a31515">Series </span><span style="color: red">Name</span><span style="color: blue">="D1" </span><span style="color: red">ChartType</span><span style="color: blue">="StackedColumn" /&gt;
</span><span style="color: blue">        &lt;</span><span style="color: #a31515">asp</span><span style="color: blue">:</span><span style="color: #a31515">Series </span><span style="color: red">Name</span><span style="color: blue">="D2" </span><span style="color: red">ChartType</span><span style="color: blue">="StackedColumn" /&gt;
</span><span style="color: blue">    &lt;/</span><span style="color: #a31515">Series</span><span style="color: blue">&gt;
    &lt;</span><span style="color: #a31515">ChartAreas</span><span style="color: blue">&gt;
        &lt;</span><span style="color: #a31515">asp</span><span style="color: blue">:</span><span style="color: #a31515">ChartArea </span><span style="color: red">Name</span><span style="color: blue">="ChartArea1"&gt;
        &lt;/</span><span style="color: #a31515">asp</span><span style="color: blue">:</span><span style="color: #a31515">ChartArea</span><span style="color: blue">&gt;
    &lt;/</span><span style="color: #a31515">ChartAreas</span><span style="color: blue">&gt;
&lt;/</span><span style="color: #a31515">asp</span><span style="color: blue">:</span><span style="color: #a31515">Chart</span><span style="color: blue">&gt;
&lt;</span><span style="color: #a31515">script </span><span style="color: red">runat</span><span style="color: blue">="server"&gt;
    protected void </span>Page_Load(<span style="color: blue">object </span>sender, <span style="color: #2b91af">EventArgs </span>e)
    {
        <span style="color: #2b91af">Random </span>r = <span style="color: blue">new </span><span style="color: #2b91af">Random</span>();
        <span style="color: blue">this</span>.chart.Series[<span style="color: #a31515">"D1"</span>].Points.Add(r.Next(100));
        <span style="color: blue">this</span>.chart.Series[<span style="color: #a31515">"D2"</span>].Points.Add(r.Next(100));
    }
<span style="color: blue">&lt;/</span><span style="color: #a31515">script</span><span style="color: blue">&gt;</span></pre>

Y otra vista (Victories.aspx) que contiene el siguiente código (entre otro):

<pre class="code"><span style="background: #ffee62">&lt;%</span><span style="color: blue">=</span>Ajax.ActionLink(<span style="color: #a31515">"Actualizar"</span>, <span style="color: #a31515">"Victories"</span>, <br /><span style="color: blue">new </span><span style="color: #2b91af">RouteValueDictionary</span>(<span style="color: blue">new </span>{ Days = 7, Interval = 1 }), <br /><span style="color: blue">new </span><span style="color: #2b91af">AjaxOptions</span>() { UpdateTargetId = <span style="color: #a31515">"chart" </span>}) <span style="background: #ffee62">%&gt;
</span>    <span style="color: blue">&lt;</span><span style="color: #a31515">div </span><span style="color: red">id</span><span style="color: blue">="chart" /&gt;</span></pre>

[][2]

El enlace &ldquo;Actualizar&rdquo; envia una petición Ajax al controlador actual para que invoque la acción &ldquo;Victories&rdquo; y con el resultado actualice el div &ldquo;chart&rdquo;.

La acción &ldquo;Victories&rdquo; está implementada en el controlador tal como sigue:

<pre class="code"><span style="color: blue">public </span><span style="color: #2b91af">ActionResult </span>Victories(<span style="color: blue">int</span>? days, <span style="color: blue">int</span>? interval)
{
<span style="color: blue">    </span><span style="color: blue">return </span>PartialView(<span style="color: #a31515">"Chart"</span>);
}</pre>

[][2]

De este modo a cada click del enlace se genera un nuevo gráfico aleatorio y se actualiza via Ajax la página...

... en teoria, porque en la práctica no se ve nada. Analizando con [firebug][4] lo que ha sucedido se puede observar que se lanza una excepción:

<pre>[HttpException (0x80004005): Error executing child request for ChartImg.axd.]</pre>

La solución? Caer en la cuenta de que las peticiones Ajax usan POST por defecto, así que o bien cambiamos la línea que añadimos en el web.config para que soporte POST:

<pre class="code"><span style="color: blue">&lt;</span><span style="color: #a31515">add </span><span style="color: red">path</span><span style="color: blue">=</span>"<span style="color: blue">ChartImg.axd</span>" <span style="color: red">verb</span><span style="color: blue">=</span>"<span style="color: blue">GET,HEAD, <strong><span style="text-decoration: underline;">POST</span></strong></span>" <br /><span style="color: red">type</span><span style="color: blue">=</span>"<span style="color: blue">System.Web.UI.DataVisualization.Charting.ChartHttpHandler, <br />System.Web.DataVisualization, Version=3.5.0.0, Culture=neutral, <br />PublicKeyToken=31bf3856ad364e35</span>" <span style="color: red">validate</span><span style="color: blue">=</span>"<span style="color: blue">false</span>"<span style="color: blue">/&gt;</span></pre>

[][2]

o bien le indicamos&nbsp; a la petición Ajax que sea usando &ldquo;GET&rdquo;:

<pre class="code"><span style="background: #ffee62">&lt;%</span><span style="color: blue">=</span>Ajax.ActionLink(<span style="color: #a31515">"Last Week"</span>, <span style="color: #a31515">"Victories"</span>, <br /><span style="color: blue">new </span><span style="color: #2b91af">RouteValueDictionary</span>(<span style="color: blue">new </span>{ Days = 7, Interval = 1 }), <br /><span style="color: blue">new </span><span style="color: #2b91af">AjaxOptions</span>() { <strong><span style="text-decoration: underline;">HttpMethod=<span style="color: #a31515">"GET"</span></span></strong>, UpdateTargetId = <span style="color: #a31515">"chart" </span>}) <span style="background: #ffee62">%&gt;</span></pre>

[][2]

Saludos!

 [1]: /blogs/jmaguilar/archive/2008/12/14/microsoft-chart-control-para-asp-net-3-5-sp1.aspx
 [2]: http://11011.net/software/vspaste
 [3]: http://code-inside.de/blog-in/2008/11/27/howto-use-the-new-aspnet-chart-controls-with-aspnet-mvc/ "http://code-inside.de/blog-in/2008/11/27/howto-use-the-new-aspnet-chart-controls-with-aspnet-mvc/"
 [4]: https://addons.mozilla.org/es-ES/firefox/addon/1843