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

```xml
<add path="ChartImg.axd" verb="GET,HEAD" 
type="System.Web.UI.DataVisualization.Charting.ChartHttpHandler, 
System.Web.DataVisualization, Version=3.5.0.0, Culture=neutral, 
PublicKeyToken=31bf3856ad364e35" validate="false"/>
```


En la sección `<httpHandlers>` y la siguiente:

```xml
<add tagPrefix="asp" 
namespace="System.Web.UI.DataVisualization.Charting" 
assembly="System.Web.DataVisualization, Version=3.5.0.0, 
Culture=neutral, PublicKeyToken=31bf3856ad364e35"/>
```

En la sección `<controls>`.

Despues ya podeis arrastrar un Chart control desde la toolbox a vuestra página ASP.NET y empezar a trabajar con él.

Si, como yo, os encanta ASP.NET MVC sabed que podeis usar este control sin ningún problema ([http://code-inside.de/blog-in/2008/11/27/howto-use-the-new-aspnet-chart-controls-with-aspnet-mvc/][2]).

El único temilla a tener en cuenta es si se quiere actualizar sólo el gráfico mediante Ajax (usando ASP.NET MVC).

Suponed una vista parcial (Chart.ascx) con el siguiente código que muestra un gráfico con contenido _random_:

```html
<%@ Control Language="C#" Inherits="System.Web.Mvc.ViewUserControl" %>
<asp:Chart ID="chart" runat="server" Palette="Fire" >
    <Series>
        <asp:Series Name="D1" ChartType="StackedColumn" />
        <asp:Series Name="D2" ChartType="StackedColumn" />
    </Series>
    <ChartAreas>
        <asp:ChartArea Name="ChartArea1">
        </asp:ChartArea>
    </ChartAreas>
</asp:Chart>
<script runat="server">
    protected void Page_Load(object sender, EventArgs e)
    {
        Random r = new Random();
        this.chart.Series["D1"].Points.Add(r.Next(100));
        this.chart.Series["D2"].Points.Add(r.Next(100));
    }
</script>
```

Y otra vista (Victories.aspx) que contiene el siguiente código (entre otro):

```html
<%=Ajax.ActionLink("Actualizar", "Victories", 
new RouteValueDictionary(new { Days = 7, Interval = 1 }), 
new AjaxOptions() { UpdateTargetId = "chart" }) %>
    <div id="chart" />
```

El enlace &ldquo;Actualizar&rdquo; envia una petición Ajax al controlador actual para que invoque la acción &ldquo;Victories&rdquo; y con el resultado actualice el div &ldquo;chart&rdquo;.

La acción &ldquo;Victories&rdquo; está implementada en el controlador tal como sigue:

```cs
public ActionResult Victories(int? days, int? interval)
{
    return PartialView("Chart");
}
```

De este modo a cada click del enlace se genera un nuevo gráfico aleatorio y se actualiza via Ajax la página...

... en teoria, porque en la práctica no se ve nada. Analizando con [firebug][3] lo que ha sucedido se puede observar que se lanza una excepción:

```
[HttpException (0x80004005): Error executing child request for ChartImg.axd.]
```

La solución? Caer en la cuenta de que las peticiones Ajax usan POST por defecto, así que o bien cambiamos la línea que añadimos en el web.config para que soporte POST:

```xml
<add path="ChartImg.axd" verb="GET,HEAD, POST" 
type="System.Web.UI.DataVisualization.Charting.ChartHttpHandler, 
System.Web.DataVisualization, Version=3.5.0.0, Culture=neutral, 
PublicKeyToken=31bf3856ad364e35" validate="false"/>
``` 

o bien le indicamos&nbsp; a la petición Ajax que sea usando &ldquo;GET&rdquo;:

```html
<%=Ajax.ActionLink("Last Week", "Victories", 
new RouteValueDictionary(new { Days = 7, Interval = 1 }), 
new AjaxOptions() { HttpMethod="GET", UpdateTargetId = "chart" }) %>
```

Saludos!

 [1]: /blogs/jmaguilar/archive/2008/12/14/microsoft-chart-control-para-asp-net-3-5-sp1.aspx
 [2]: http://code-inside.de/blog-in/2008/11/27/howto-use-the-new-aspnet-chart-controls-with-aspnet-mvc/ "http://code-inside.de/blog-in/2008/11/27/howto-use-the-new-aspnet-chart-controls-with-aspnet-mvc/"
 [3]: https://addons.mozilla.org/es-ES/firefox/addon/1843