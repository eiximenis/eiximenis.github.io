---
title: 'ASP.NET MVC – Helper Html.LabelFor<T> parametrizable'

author: eiximenis

date: 2011-06-29T23:27:00+00:00
geeks_url: /?p=1570
geeks_visits:
  - 5078
geeks_ms_views:
  - 1833
categories:
  - Uncategorized

---
Buenas! La verdad es que llevo algunos días sin actualizar mucho el blog... Ya se sabe trabajo y tal 🙂

Hoy quiero comentaros algo rapidito y que se ha preguntado varias veces en los foros y que es como poder asignar un ID al <label /> generado por el helper Html.LabelFor<T>. En este caso vamos a hacer que se le puedan añadir todos los atributos que se quieran a la etiqueta <label />

<!--more-->

Aunque use este helper, la técnica aplicada debería serviros para ver como ampliar los helpers, en caso que lo necesitéis.

Por ejemplo, dado un viewmodel que tenga una propiedad _Address_ el siguiente código:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">@Html.LabelFor(x =&gt; x.Address);</pre>
</div>

Nos genera el HTML siguiente:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">&lt;</span><span style="color: #800000">label</span> <span style="color: #ff0000">for</span><span style="color: #0000ff">="Address"</span><span style="color: #0000ff">&gt;</span>Address<span style="color: #0000ff">&lt;/</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span></pre>
</div>

Si quisiéramos parametrizar el <label /> generado nos encontramos conque ninguna de las dos sobrecargas que ofrece el helper nos es de ayuda (la otra simplemente nos permite especificar el texto de la cadena).

En nuestro caso queremos poder hacer una llamada como la siguiente:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">@Html.LabelFor(x =&gt; x.Address, <span style="color: #0000ff">new</span> { id = <span style="color: #006080">"lblEiximenis"</span>, width = <span style="color: #006080">"100"</span> })&lt;br /&gt;</pre>
</div>

y que el código HTML generado sea:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">&lt;</span><span style="color: #800000">label</span> <span style="color: #ff0000">for</span><span style="color: #0000ff">="Address"</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="lblEiximenis"</span> <span style="color: #ff0000">width</span><span style="color: #0000ff">="100"</span><span style="color: #0000ff">&gt;</span>Address<span style="color: #0000ff">&lt;/</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span></pre>
</div>

Bien! Por suerte es muy sencillito, basta con crearnos un método extensor sobre HtmlHelper como el que sigue:

<span style="color: #0000ff;">public static class HtmlExtensions<br /> <br />{<br /> <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; public static MvcHtmlString LabelFor<TModel, TValue>(this HtmlHelper<TModel> html, Expression<Func<TModel, TValue>> expression, object htmlAttributes)<br /> <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {<br /> <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ModelMetadata meta = ModelMetadata.FromLambdaExpression (expression, html.ViewData);<br /> <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; var htmlFullName = ExpressionHelper.GetExpressionText(expression);<br /> <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; var labelText = meta.DisplayName ?? meta.PropertyName;<br /> <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; if (String.IsNullOrEmpty(labelText))<br /> <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {<br /> <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; return MvcHtmlString.Empty;<br /> <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }<br /> <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; var tag = new TagBuilder(&#8220;label&#8221;);<br /> <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; tag.Attributes.Add(&#8220;for&#8221;, html.ViewContext.ViewData.TemplateInfo.GetFullHtmlFieldId(htmlFullName));<br /> <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; if (htmlAttributes != null)<br /> <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {<br /> <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; tag.MergeAttributes(new RouteValueDictionary(htmlAttributes));<br /> <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }<br /> <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; tag.SetInnerText(labelText);<br /> <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; return MvcHtmlString.Create(tag.ToString(TagRenderMode.Normal));<br /> <br />&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }</span>

<span style="color: #0000ff;">&nbsp;&nbsp;&nbsp; }</span>

La verdad es que el código es muy sencillito, hay 3 puntos a destacar:

  1. El uso de _ModelMetadata_ para de esta manera poder cojer el valor de los meta datos del modelo (usualmente las Data Annotations) y de este modo poder hacer caso de ellas. En este caso respetaríamos el atrbuto [DisplayName] si el usuario lo usara. 
  2. El uso de _ExpressionHelper.GetExpressionText_. Este método lo que devuelve es el texto de una expresión lambda. Es decir, si yo tengo x=>x.Customer.Name, este método me devolverá &ldquo;Customer.Name&rdquo;. Esto lo necesitamos porque el valor del atributo _for_ **es todo el texto** del cuerpo de la lambda expression. Si al atributo _for_ le pasase el nombre de la propiedad en lugar de **todo** el texto de la expresión lambda no podría usar propiedades anidadas (es decir me funcionaría para x=>x.Name pero NO para x => x.Customer.Name). 
  3. El método MergeAttributes del TagBuilder, que lo que hace es añadir a la etiqueta que se está construyendo todos aquellos atributos basándose en el IDictionary<string, object> que recibe como parámetro. Como en nuestro método los atributos los pasamos como objeto anónimo, nos aprovechamos de la clase RouteValueDictionary (que tiene un constructor que acepta un object). Eso nos evita tener que usar reflection directamente 😉 

Y listos! Ya podemos personalizar al máximo las <label>s generadas!

Un saludo a todos!

PD: El código de este méotdo está copiado casi directamente del código fuente de ASP.NET MVC. Así que el consejo final de este post es: **mirad el código fuente de ASP.NET MVC aprendereis mucho de él**.

PD2: Una búsqueda en Google me rebela que Imran Baloch se me ha avanzado por algunos días: [http://weblogs.asp.net/imranbaloch/archive/2010/07/03/asp-net-mvc-labelfor-helper-with-htmlattributes.aspx][1]

 [1]: http://weblogs.asp.net/imranbaloch/archive/2010/07/03/asp-net-mvc-labelfor-helper-with-htmlattributes.aspx "http://weblogs.asp.net/imranbaloch/archive/2010/07/03/asp-net-mvc-labelfor-helper-with-htmlattributes.aspx"