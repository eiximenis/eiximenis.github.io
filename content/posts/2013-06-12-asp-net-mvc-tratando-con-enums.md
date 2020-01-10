---
title: ASP.NET MVC – Tratando con enums.

author: eiximenis

date: 2013-06-12T09:04:59+00:00
geeks_url: /?p=1644
geeks_visits:
  - 2592
geeks_ms_views:
  - 1859
categories:
  - Uncategorized

---
En un proyecto ASP.NET MVC en el que estoy colaborando, surgió la necesidad de tratar con viewmodels que tenían propiedades cuyo tipo era un enum. Algo así como:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    [<span style="color: #4ec9b0">Flags</span>]
  </p>
  
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #569cd6">enum</span> <span style="color: #b8d7a3">TestEnum</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: white">None</span> <span style="color: #b4b4b4">=</span> <span style="color: #b5cea8"></span>,
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: white">One</span> <span style="color: #b4b4b4">=</span> <span style="color: #b5cea8">1</span>,
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: white">Two</span> <span style="color: #b4b4b4">=</span><span style="color: #b5cea8">2</span>,
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: white">Four</span> <span style="color: #b4b4b4">=</span><span style="color: #b5cea8">4</span>
  </p>
  
  <p style="margin: 0px">
    }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">FooModel</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #b8d7a3">TestEnum</span> <span style="color: white">TestData</span> { <span style="color: #569cd6">get</span>; <span style="color: #569cd6">set</span>; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Los valores de TestEnum son combinables a nivel de bits (de ahí que esté decorado con [Flags], es decir el valor de la propiedad TestData puede ser cualquiera de los cuatro o bien una combinación (p. ej. One y Two).

ASP.NET MVC no gestiona, por defecto, bien estos casos. Imaginemos que tenemos el siguiente código en el controlador:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #4ec9b0">ActionResult</span> <span style="color: white">Index</span>()
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">model</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">FooModel</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: white">model</span><span style="color: #b4b4b4">.</span><span style="color: white">TestData</span> <span style="color: #b4b4b4">=</span> <span style="color: #b8d7a3">TestEnum</span><span style="color: #b4b4b4">.</span><span style="color: white">One</span> <span style="color: #b4b4b4">|</span> <span style="color: #b8d7a3">TestEnum</span><span style="color: #b4b4b4">.</span><span style="color: white">Two</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: white">View</span>(<span style="color: white">model</span>);
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Y el código correspondiente en la vista:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="background: #ffffb3; color: black">@</span><span style="color: #569cd6">using</span> (<span style="color: white">Html</span><span style="color: #b4b4b4">.</span><span style="color: white">BeginForm</span>())
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="background: #ffffb3; color: black">@</span><span style="color: white">Html</span><span style="color: #b4b4b4">.</span><span style="color: white">EditorFor</span>(<span style="color: white">x</span> <span style="color: #b4b4b4">=></span> <span style="color: white">x</span><span style="color: #b4b4b4">.</span><span style="color: white">TestData</span>)&#160;&#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">input</span> <span style="color: #9cdcfe">type</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"submit"</span> <span style="color: #9cdcfe">value</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"enviar"</span> <span style="color: gray">/></span>
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Lo que nos generará será un TextBox con los valores del enum (en este caso One y Two) separados por comas:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; display: inline; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_78E86DA2.png" width="484" height="89" />][1] 

Es obvio que no es una buena manera que el usuario entre esos datos. Por su parte el enlazado funcionará correctamente (es decir en el método que recibe el POST se recibirá que la propiedad TestData vale TestEnum.One | TestEnum.Two, ya que el DefaultModelBinder sí que es capaz de tratar este caso

En el proyecto lo que se quería era que en lugar de mostrar este texto se mostraran tantas checkboxes como valores tiene el enum y que se pudiesen marcar una o varias.

Eso se consigue con relativa facilidad usando un EditorTemplate. En ASP.NET MVC los EditorTemplates (y sus equivalentes de visualización los DisplayTemplates) son vistas parciales que son renderizadas cuando se debe de editar o visualizar un valor de un tipo o propiedad en concreto. Los EditorTemplates se colocan por lo general en la carpeta Views/Shared/EditorTemplates (los Display Templates en la Views/Shared/DisplayTemplates).

Así pues creamos un Editor Template que nos cree tantas checkboxes como valores tiene el enum:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="background: #ffffb3; color: black">@model </span><span style="color: #4ec9b0">Enum</span>
  </p>
  
  <p style="margin: 0px">
    <span style="background: #ffffb3; color: black">@{</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">modelType</span> <span style="color: #b4b4b4">=</span> <span style="color: white">@Model</span><span style="color: #b4b4b4">.</span><span style="color: white">GetType</span>();
  </p>
  
  <p style="margin: 0px">
    <span style="background: #ffffb3; color: black">}</span>
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    <span style="background: #ffffb3; color: black">@</span><span style="color: #569cd6">foreach</span> (<span style="color: #569cd6">var</span> <span style="color: white">name</span> <span style="color: #569cd6">in</span> <span style="color: #4ec9b0">Enum</span><span style="color: #b4b4b4">.</span><span style="color: white">GetNames</span>(<span style="color: white">modelType</span>))
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">value</span> <span style="color: #b4b4b4">=</span> <span style="color: #4ec9b0">Convert</span><span style="color: #b4b4b4">.</span><span style="color: white">ToInt32</span>(<span style="color: #4ec9b0">Enum</span><span style="color: #b4b4b4">.</span><span style="color: white">Parse</span>(<span style="color: white">modelType</span>, <span style="color: white">name</span>));
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">if</span> (<span style="color: white">value</span> <span style="color: #b4b4b4">!=</span> <span style="color: #b5cea8"></span>)
  </p>
  
  <p style="margin
: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">isChecked</span> <span style="color: #b4b4b4">=</span> ((<span style="color: #4ec9b0">Convert</span><span style="color: #b4b4b4">.</span><span style="color: white">ToInt32</span>(<span style="color: white">Model</span>) <span style="color: #b4b4b4">&</span> <span style="color: white">value</span>) <span style="color: #b4b4b4">==</span> <span style="color: white">value</span>) <span style="color: #b4b4b4">?</span> <span style="color: #d69d85">"checked"</span> : <span style="color: #569cd6">null</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">input</span> <span style="color: #9cdcfe">type</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"checkbox"</span> <span style="color: #9cdcfe">name</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"</span><span style="background: #ffffb3; color: black">@</span><span style="color: white">ViewData</span><span style="color: #b4b4b4">.</span><span style="color: white">TemplateInfo</span><span style="color: #b4b4b4">.</span><span style="color: white">HtmlFieldPrefix</span><span style="color: #c8c8c8">"</span> <span style="color: #9cdcfe">value</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"</span><span style="background: #ffffb3; color: black">@</span><span style="color: white">name</span><span style="color: #c8c8c8">"</span> <span style="color: #9cdcfe">checked</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"</span><span style="background: #ffffb3; color: black">@</span><span style="color: white">isChecked</span><span style="color: #c8c8c8">"</span> <span style="color: gray">/></span>&#160; <span style="background: #ffffb3; color: black">@</span><span style="color: white">name</span><span style="color: gray"><</span><span style="color: #569cd6">br</span> <span style="color: gray">/></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

El código es relativamente sencillo y lo que hace es crear una checkbox por cada clave (valor) del enum y marcarla si es necesario (es decir, si un and a nivel a de bits entre el valor del enum y el valor de cada clave da distinto de cero). Hay una comprobación addicional para no renderizar una checkbox si el valor de la clave es 0 (en nuestro caso sería el None). Para usar este EditorTemplate (que yo he llamado FlagEnum.cshtml) una posibilidad es indicarle al viewmodel que lo use, decorando la propiedad con [UIHint]:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">FooModel</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; [<span style="color: #4ec9b0">UIHint</span>(<span style="color: #d69d85">"FlagEnum"</span>)]
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #b8d7a3">TestEnum</span> <span style="color: white">TestData</span> { <span style="color: #569cd6">get</span>; <span style="color: #569cd6">set</span>; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Dado que en nuestra vista ya usábamos EditorFor para generar el editor de la propiedad TestData, no es necesario ningún cambio más. El resultado es ahora más interesante:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; display: inline; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_02CD5F0E.png" width="129" height="145" />][2] 

Parece que hemos terminado, eh? Pues no… Aparece un problema. Si le damos a enviar tal cual, lo que ahora recibimos en el método que recibe los datos POST es:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; display: inline; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_0188C62F.png" width="484" height="45" />][3] 

¡Ahora no se nos enlaza bien el campo! Eso es debido a como funciona el Model Binder de ASP.NET MVC. La diferencia con antes (cuando había el textbox) es que antes teníamos un solo campo en la petición y ahora tenemos N (tantos como checkboxes marcadas). Si comparamos las peticiones enviadas, lo veremos mejor.

Esta es la petición que el navegador envía si NO usamos nuestro EditorTemplate:

<pre>POST http://localhost:19515/ HTTP/1.1
Accept: text/html
Referer: http://localhost:19515/
Origin: http://localhost:19515
Content-Type: application/x-www-form-urlencoded
<strong>TestData=One%2C+Two</strong></pre>

Y esta otra la petición que se envía si usamos el EditorTemplate:

<pre>POST http://localhost:19515/ HTTP/1.1
Accept: text/html
Referer: http://localhost:19515/
Origin: <a href="http://localhost:19515Content-Type">http://localhost:19515
Content-Type</a>: application/x-www-form-urlencoded
<strong>TestData=One&TestData=Two</strong></pre>

Fijaos en la diferencia (marcada en negrita). El Model Binder por defecto de ASP.NET MVC es capaz de gestionar el primer caso, pero no el segundo, de ahí que ahora no funcione y nos enlace tan solo el primer TestData que se encuentra (y que vale One).

¿Y la solución? Pues hacernos un Model Binder propio que sea capaz de tratar estos casos. Por suerte el código no es excesivamente complejo:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">EnumFlagModelBinder</span> : <span style="color: #4ec9b0">DefaultModelBinder</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">override</span> <span style="color: #569cd6">object</span> <span style="color: white">BindModel</span>(<span style="color: #4ec9b0">ControllerContext</span> <span style="color: white">controllerContext</span>, <span style="color: #4ec9b0">ModelBindingContext</span> <span style="color: white">bindingContext</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">if</span> (<span style="color: #b4b4b4">!</span>(<span style="color: white">bindingContext</span><span style="color: #b4b4b4">.</span><span style="color: white">Model</span>&#160; <span style="color: #569cd6">is</span> <span style="color: #4ec9b0">Enum</span>))
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: #569cd6">base</span><span style="color: #b4b4b4">.</span><span style="color: white">BindModel</span>(<span style="color: white">controllerContext</span>, <span style="color: white">bindingContext</span>);&#160;&#160;&#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">value</span> <span style="color: #b4b4b4">=</span> <span style="color: white">bindingContext</span><span style="color: #b4b4b4">.</span><span style="color: white">ValueProvider</span><span style="color: #b4b4b4">.</span><span style="color: white">GetValue</span>(<span style="color: white">bindingContext</span><span style="color: #b4b4b4">.</span><span style="color: white">ModelName</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">if</span> (<span style="color: white">value</span><span style="color: #b4b4b4">.</span><span style="color: white">RawValue</span> <span style="color: #569cd6">is</span> <span style="color: #569cd6">string</span>[])
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">tokens</span> <span style="color: #b4b4b4">=</span> (<span style="color: #569cd6">string</span>[]) <span style="color: white">value</span><span style="color: #b4b4b4">.</span><span style="color: white">RawValue</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">int</span> <span style="color: white">intValue</span> <span style="color: #b4b4b4">=</span> <span style="color: #b5cea8"></span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">foreach</span> (<span style="color: #569cd6">var</span> <span style="color: white">token</span> <span style="color: #569cd6">in</span> <span style="color: white">tokens</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">currentIntValue</span> <span style="color: #b4b4b4">=</span> <span style="color: #b5cea8"></span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">isIntValue</span> <span style="color: #b4b4b4">=</span> <span style="color: #4ec9b0">Int32</span><span style="color: #b4b4b4">.</span><span style="color: white">TryParse</span>(<span style="color: white">token</span>, <span style="color: #569cd6">out</span> <span style="color: white">currentIntValue</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">if</span> (<span style="color: white">isIntValue</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">intValue</span> <span style="color: #b4b4b4">|=</span> <span style="color: white">currentIntValue</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">else</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">modelType</span> <span style="color: #b4b4b4">=</span> <span style="color: white">bindingContext</span><span style="color: #b4b4b4">.</span><span style="color: white">Model</span><span style="color: #b4b4b4">.</span><span style="color: white">GetType</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">intValue</span> <span style="color: #b4b4b4">|=</span> <span style="color: #4ec9b0">Convert</span><span style="color: #b4b4b4">.</span><span style="color: white">ToInt32</span>(<span style="color: #4ec9b0">Enum</span><span style="color: #b4b4b4">.</span><span style="color: white">Parse</span>(<span style="color: white">modelType</span>, <span style="color: white">token</span>));
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: #4ec9b0">Enum</span><span style="color: #b4b4b4">.</span><span style="color: white">Parse</span>(<span style="color: white">bindingContext</span><span style="color: #b4b4b4">.</span><span style="color: white">Model</span><span style="color: #b4b4b4">.</span><span style="color: white">GetType</span>(), <span style="color: white">intValue</span><span style="color: #b4b4b4">.</span><span style="color: white">ToString</span>());
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: #569cd6">base</span><span style="color: #b4b4b4">.</span><span style="color: white">BindModel</span>(<span style="color: white">controllerContext</span>, <span style="color: white">bindingContext</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
</div>

El código es muy sencillo:

Si el tipo del modelo no es un Enum derivamos hacia el enlace por defecto de ASP.NET MVC. En caso contrario obtenemos el valor de la propiedad que se está enlazando, consultando a los Value Providers.

Este valor será un string[] con los diversos valores que el usuario ha entrado (p. ej. un string[] con dos elementos “One” y “Two”). Iteramos sobre este array de valores y por cada valor:

  1. Miramos si es un entero. Si lo es, hacemos un or a nivel de bits entre una variable (intValue inicialmente a cero) y este valor. 
  2. Si NO es un entero (p.ej. es “One”) obtenemos el valor entero con Enum.Parse y realizamos el mismo or a nivel de bits. 

Finalmente devolvemos el valor asociado al valor de entero que hemos obtenido.

P. ej. si el usuario ha marcado las checks “One” y “Two” el array (tokens) tendrá esos dos valores. Al final IntValue valdrá 3 (el resultado de hacer un or de bits entre 1 y 2) y devolveremos el resultado de hacer Enum.Parse de este 3 (que es precisamente One | Two).

**Nota:** El hecho de mirar primero si el valor de token es un entero, es un pequeño refinamiento para que nuestro EditorTemplate funcione también en aquellos casos en que nos envíen los valores numéricos asociados (que nos envien p. ej. TestData=1&TestData=2 en la petición. Usando nuestro EditorTemplate este caso no se da).

Ahora tan solo debemos indicarle a ASP.NET MC que use nuestro Model Binder para enlazar las propiedades de tipo TestEnum. Aunque nuestro Model Binder podría enlazar cualquier Enum (es genérico) ASP.NET MVC no nos permite asociar un Model Binder a “cualquier Enum”. Para indicarle a ASP.NET MVC que use nuestro Model Binder para las propiedades de tipo TestEnum basta con añadir en el Application_Start:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #4ec9b0">ModelBinders</span><span style="color: #b4b4b4">.</span><span style="color: white">Binders</span><span style="color: #b4b4b4">.</span><span style="color: white">Add</span>(<span style="color: #569cd6">typeof</span>(<span style="color: #b8d7a3">TestEnum</span>), <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">EnumFlagModelBinder</span>());
  </p>
</div>

Y ¡listos! Con esto hemos terminado.

Espero que os haya sido interesante.

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_52F2BD4C.png
 [2]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_5141F178.png
 [3]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_304E7ED1.png