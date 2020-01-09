---
title: 'ASP.NET MVC: Edición de colecciones usando Ajax'
author: eiximenis

date: 2012-07-01T17:55:01+00:00
geeks_url: /?p=1600
geeks_visits:
  - 2857
geeks_ms_views:
  - 1443
categories:
  - Uncategorized

---
Buenas! El otro día me enviaron la siguiente duda:

_<font color="#0000ff">Imaginate esto:</font>_

_<font color="#0000ff">class Direccion{</font>_

_<font color="#0000ff">string Calle {get;set;}</font>_

_<font color="#0000ff">int Piso {get;set;}</font>_

_<font color="#0000ff">}</font>_

_<font color="#0000ff">class Cliente {</font>_

_<font color="#0000ff">string Nombre {get;set;}</font>_

_<font color="#0000ff">List<Direccion> Direcciones {get;set;}</font>_

_<font color="#0000ff">}</font>_

_<font color="#0000ff">Imagina que tiene mas propiedades cada clase pero para el ejemplo sirve.</font>_

_<font color="#0000ff">Entonces tengo una Vista para definir la información de Cliente:</font>_

_<font color="#0000ff">Donde van a aparecer los campos para rellenar el cliente. Dentro voy a tener un boton que va a ir agregandome vistas&#160; parciales con la info de Direcciones.</font>_

_<font color="#0000ff">Actualmente la parte de ediccion de Direcciones estaría en una Vista Parcial (con su correspondiente Form).</font>_

_<font color="#0000ff">El problema que tengo es que necesito Enviar todo en el submit de Cliente y realmente es como si no tuviese la información de las direcciones.</font>_

Vale, de lo que se trata es que des de la vista para crear un Cliente, podamos añadir objetos Direccion, tantos como sea necesario, usando una vista parcial que iremos cargando via Ajax. El problema era que no se recibía la información de las direcciones.

Hace algún tiempo hablé de las peculiaridades del binding de colecciones en ASP.NET MVC ([parte 1][1], [parte 2][2] y [parte 3][3]). Si os miráis la primera parte de la serie, allí comentábamos que nombre debían tener los campos que se enlazaban a una colección. Básicamente si queremos enlazar una colección que está en la propiedad Direcciones de nuestro viewmodel, los campos deben llamarse Direcciones[0], Direcciones[1] y así sucesivamente.

Sabiendo esto, queda claro lo que hay que hacer:

  1. Una vista para crear un Cliente. Dicha vista contendrá un <div> en el cual iremos cargando las vistas parciales para crear direcciones 
  2. Una vista parcial para crear un objeto direccion 

Tan solo debemos tener cuidado de incrustar las vistas parciales dentro del <form> de la vista principal y de seguir la convención de nombres del ModelBinder. Asi pues, empecemos por la vista para crear un Cliente.

Partimos del código que nos genera VS al crear una nueva vista para crear objetos de tipo Cliente:

<pre style="overflow: auto; border-top: #cecece 1px solid; border-right: #cecece 1px solid; border-bottom: #cecece 1px solid; padding-bottom: 5px; padding-top: 5px; padding-left: 5px; min-height: 40px; border-left: #cecece 1px solid; padding-right: 5px; width: 500px; background-color: #fbfbfb"><pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">@model MvcAjaxCol.Models.Cliente
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"></pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">@{
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">    ViewBag.Title = "Nuevo cliente";
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">}
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"><span style="color: #0000ff">&lt;</span><span style="color: #800000">h2</span><span style="color: #0000ff">&gt;</span>Nuevo Cliente<span style="color: #0000ff">&lt;/</span><span style="color: #800000">h2</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"><span style="color: #0000ff">&lt;</span><span style="color: #800000">script</span> <span style="color: #ff0000">src</span>=<span style="color: #0000ff">"@Url.Content("</span>~/<span style="color: #ff0000">Scripts</span>/<span style="color: #ff0000">jquery</span>.<span style="color: #ff0000">validate</span>.<span style="color: #ff0000">min</span>.<span style="color: #ff0000">js</span>")" <span style="color: #ff0000">type</span>=<span style="color: #0000ff">"text/javascript"</span><span style="color: #0000ff">&gt;</span><span style="color: #0000ff">&lt;/</span><span style="color: #800000">script</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"><span style="color: #0000ff">&lt;</span><span style="color: #800000">script</span> <span style="color: #ff0000">src</span>=<span style="color: #0000ff">"@Url.Content("</span>~/<span style="color: #ff0000">Scripts</span>/<span style="color: #ff0000">jquery</span>.<span style="color: #ff0000">validate</span>.<span style="color: #ff0000">unobtrusive</span>.<span style="color: #ff0000">min</span>.<span style="color: #ff0000">js</span>")" <span style="color: #ff0000">type</span>=<span style="color: #0000ff">"text/javascript"</span><span style="color: #0000ff">&gt;</span><span style="color: #0000ff">&lt;/</span><span style="color: #800000">script</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">@using (Html.BeginForm()) {
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">    @Html.ValidationSummary(true)
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">    &lt;fieldset&gt;
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">        <span style="color: #0000ff">&lt;</span><span style="color: #800000">legend</span><span style="color: #0000ff">&gt;</span>Cliente<span style="color: #0000ff">&lt;/</span><span style="color: #800000">legend</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"></pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">        <span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">class</span>=<span style="color: #0000ff">"editor-label"</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">            @Html.LabelFor(model =&gt; model.Nombre)
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">        <span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">        <span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">class</span>=<span style="color: #0000ff">"editor-field"</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">            @Html.EditorFor(model =&gt; model.Nombre)
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">            @Html.ValidationMessageFor(model =&gt; model.Nombre)
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">        <span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">        <span style="color: #0000ff">&lt;</span><span style="color: #800000">p</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">            <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span>=<span style="color: #0000ff">"submit"</span> <span style="color: #ff0000">value</span>=<span style="color: #0000ff">"Create"</span> <span style="color: #0000ff">/&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">        <span style="color: #0000ff">&lt;/</span><span style="color: #800000">p</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">    <span style="color: #0000ff">&lt;/</span><span style="color: #800000">fieldset</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">}
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"></pre>


<p>
  El propio VS no nos genera código para la propiedad “Direcciones” de la clase Cliente porque es una colección. Bueno, el siguiente paso es crear un <div> vacío donde colocaremos via Ajax las vistas parciales para crear objetos del tipo dirección. Por supuesto dicho <div> debe estar dentro del <form>. Luego añadimos un botón y cargamos una vista parcial nueva cada vez que se pulse el botón:
</p>


<pre style="overflow: auto; border-top: #cecece 1px solid; border-right: #cecece 1px solid; border-bottom: #cecece 1px solid; padding-bottom: 5px; padding-top: 5px; padding-left: 5px; min-height: 40px; border-left: #cecece 1px solid; padding-right: 5px; width: 500px; background-color: #fbfbfb"><pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">@model MvcAjaxCol.Models.Cliente
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"></pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">@{
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">    ViewBag.Title = "Nuevo cliente";
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">}
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"></pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"><span style="color: #0000ff">&lt;</span><span style="color: #800000">script</span> <span style="color: #ff0000">type</span>=<span style="color: #0000ff">"text/javascript"</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">    var idx = 0;
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">    $(document).ready(function() {
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">        $("#cmdNewDir").click(function() {
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">            var uri = "@Html.Raw(Url.Action("NewDir", "Home", new {prefix="Direcciones", idx="xxx" }))";
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">            uri = uri.replace("xxx", idx);
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">            idx++;
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">            $.get(uri, function(data) {
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">                $("#direcciones").append(data);
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">            });
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">        });
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">    });
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"></pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"><span style="color: #0000ff">&lt;/</span><span style="color: #800000">script</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"><span style="color: #0000ff">&lt;</span><span style="color: #800000">h2</span><span style="color: #0000ff">&gt;</span>Nuevo Cliente<span style="color: #0000ff">&lt;/</span><span style="color: #800000">h2</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"><span style="color: #0000ff">&lt;</span><span style="color: #800000">script</span> <span style="color: #ff0000">src</span>=<span style="color: #0000ff">"@Url.Content("</span>~/<span style="color: #ff0000">Scripts</span>/<span style="color: #ff0000">jquery</span>.<span style="color: #ff0000">validate</span>.<span style="color: #ff0000">min</span>.<span style="color: #ff0000">js</span>")" <span style="color: #ff0000">type</span>=<span style="color: #0000ff">"text/javascript"</span><span style="color: #0000ff">&gt;</span><span style="color: #0000ff">&lt;/</span><span style="color: #800000">script</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"><span style="color: #0000ff">&lt;</span><span style="color: #800000">script</span> <span style="color: #ff0000">src</span>=<span style="color: #0000ff">"@Url.Content("</span>~/<span style="color: #ff0000">Scripts</span>/<span style="color: #ff0000">jquery</span>.<span style="color: #ff0000">validate</span>.<span style="color: #ff0000">unobtrusive</span>.<span style="color: #ff0000">min</span>.<span style="color: #ff0000">js</span>")" <span style="color: #ff0000">type</span>=<span style="color: #0000ff">"text/javascript"</span><span style="color: #0000ff">&gt;</span><span style="color: #0000ff">&lt;/</span><span style="color: #800000">script</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">@using (Html.BeginForm())
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">{
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">    @Html.ValidationSummary(true)
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">    <span style="color: #0000ff">&lt;</span><span style="color: #800000">fieldset</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">        <span style="color: #0000ff">&lt;</span><span style="color: #800000">legend</span><span style="color: #0000ff">&gt;</span>Cliente<span style="color: #0000ff">&lt;/</span><span style="color: #800000">legend</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"></pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">        <span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">class</span>=<span style="color: #0000ff">"editor-label"</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">            @Html.LabelFor(model =&gt; model.Nombre)
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">        <span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">        <span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">class</span>=<span style="color: #0000ff">"editor-field"</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">            @Html.EditorFor(model =&gt; model.Nombre)
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">            @Html.ValidationMessageFor(model =&gt; model.Nombre)
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">        <span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">        <span style="color: #0000ff">&lt;</span><span style="color: #800000">hr</span> <span style="color: #0000ff">/&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">        <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span>=<span style="color: #0000ff">"button"</span> <span style="color: #ff0000">value</span>=<span style="color: #0000ff">"Nueva Direccion"</span> <span style="color: #ff0000">id</span>=<span style="color: #0000ff">"cmdNewDir"</span><span style="color: #0000ff">/&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">        <span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">id</span>=<span style="color: #0000ff">"direcciones"</span><span style="color: #0000ff">&gt;</span><span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">        <span style="color: #0000ff">&lt;</span><span style="color: #800000">p</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">            <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span>=<span style="color: #0000ff">"submit"</span> <span style="color: #ff0000">value</span>=<span style="color: #0000ff">"Create"</span> <span style="color: #0000ff">/&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">        <span style="color: #0000ff">&lt;/</span><span style="color: #800000">p</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">    <span style="color: #0000ff">&lt;/</span><span style="color: #800000">fieldset</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">}</pre>


<p>
  Como se puede a cada click del botón llamamos a una acción del controlador llamada NewDir. Esta acción es la que devuelve la vista parcial que añadimos al div:
</p>


<pre style="overflow: auto; border-top: #cecece 1px solid; border-right: #cecece 1px solid; border-bottom: #cecece 1px solid; padding-bottom: 5px; padding-top: 5px; padding-left: 5px; min-height: 40px; border-left: #cecece 1px solid; padding-right: 5px; width: 500px; background-color: #fbfbfb"><pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"><span style="color: #0000ff">public</span> ActionResult NewDir(<span style="color: #0000ff">string</span> prefix, <span style="color: #0000ff">string</span> idx)
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">{
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">   ViewData["<span style="color: #8b0000">prefix</span>"] = prefix;
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">   ViewData["<span style="color: #8b0000">idx</span>"] = idx;
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">   <span style="color: #0000ff">return</span> PartialView("<span style="color: #8b0000">Direccion</span>");
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">}</pre>


<p>
  Y finalmente la vista Direccion, usa los parámetros prefix y idx para ir generando campos cuyo nombre sea prefix[idx].XXX:
</p>


<pre style="overflow: auto; border-top: #cecece 1px solid; border-right: #cecece 1px solid; border-bottom: #cecece 1px solid; padding-bottom: 5px; padding-top: 5px; padding-left: 5px; min-height: 40px; border-left: #cecece 1px solid; padding-right: 5px; width: 500px; background-color: #fbfbfb"><pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">@model MvcAjaxCol.Models.Direccion
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"><span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">class</span>=<span style="color: #0000ff">"editor-label"</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">    @Html.LabelFor(model =&gt; model.Calle)
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"><span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"><span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">class</span>=<span style="color: #0000ff">"editor-field"</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">    @Html.Editor(string.Format("{0}[{1}].{2}", ViewData["prefix"], ViewData["idx"], "Calle"))
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">    @Html.ValidationMessage(string.Format("{0}[{1}].{2}", ViewData["prefix"], ViewData["idx"], "Calle"))
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"><span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"></pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"><span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">class</span>=<span style="color: #0000ff">"editor-label"</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">    @Html.LabelFor(model =&gt; model.Piso)
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"><span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff"><span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">class</span>=<span style="color: #0000ff">"editor-field"</span><span style="color: #0000ff">&gt;</span>
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb">    @Html.Editor(string.Format("{0}[{1}].{2}", ViewData["prefix"], ViewData["idx"], "Piso"))
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #ffffff">    @Html.ValidationMessage(string.Format("{0}[{1}].{2}", ViewData["prefix"], ViewData["idx"], "Piso"))
</pre>


<pre style="font-size: 12px; font-family: consolas,&#39;Courier New&#39;,courier,monospace; margin: 0em; width: 100%; background-color: #fbfbfb"><span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span></pre>


<p>
  ¡Y listos! Con esto ya podemos ir añadiendo tantas direcciones como sea necesario a nuesro cliente!
</p>


<p>
  ¡Un saludo!
</p>

 [1]: http://geeks.ms/blogs/etomas/archive/2011/07/09/binding-de-colecciones-en-asp-net-mvc.aspx
 [2]: http://geeks.ms/blogs/etomas/archive/2011/07/15/binding-de-colecciones-en-asp-net-mvc-ii.aspx
 [3]: http://geeks.ms/blogs/etomas/archive/2011/08/03/binding-de-colecciones-en-asp-net-mvc-iii.aspx