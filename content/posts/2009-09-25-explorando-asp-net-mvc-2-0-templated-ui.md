---
title: Explorando ASP.NET MVC 2.0… Templated UI
author: eiximenis

date: 2009-09-25T09:59:09+00:00
geeks_url: /?p=1469
geeks_visits:
  - 2025
geeks_ms_views:
  - 756
categories:
  - Uncategorized

---
Hola a todos!! Este post continua la _mini-serie_ de posts que [empecé con el que trataba sobre las áreas][1].

En este caso he estado jugando un poco con lo que llaman _UI helper templating support_ y que me parece bastante interesante…

En el HtmlHelper han aparecido básicamente dos métodos nuevos: EditorFor<TM, TV> y DisplayFor<TM, TV>, que sirven para _renderizar_ un editor o un visualizador para el tipo de datos TV del modelo cuyo tipo es TM.

Supongamos que tengo una clase _Persona_ en mi modelo tal como:

<div class="csharpcode">
  <pre class="alt"><span class="kwrd">public</span> <span class="kwrd">class</span> Persona</pre>
  
  <pre>{</pre>
  
  <pre class="alt">    <span class="kwrd">public</span> <span class="kwrd">string</span> Nombre { get; set; }</pre>
  
  <pre>    <span class="kwrd">public</span> <span class="kwrd">int</span> Edad { get; set; }</pre>
  
  <pre class="alt">}</pre>
</div>

Bien, ahora en el controlador Home, en la acción Index, obtenemos un lista de objetos de dicha clase y la pasamos a la vista:

<div class="csharpcode">
  <pre class="alt"><span class="kwrd">public</span> ActionResult Index()</pre>
  
  <pre>{</pre>
  
  <pre class="alt">    var matusalen = <span class="kwrd">new</span> Persona()</pre>
  
  <pre>    {</pre>
  
  <pre class="alt">        Nombre = <span class="str">"Matusalén"</span>,</pre>
  
  <pre>        Edad = 350</pre>
  
  <pre class="alt">    };</pre>
  
  <pre>    var jacob = <span class="kwrd">new</span> Persona()</pre>
  
  <pre class="alt">    {</pre>
  
  <pre>        Nombre = <span class="str">"Jacob"</span>,</pre>
  
  <pre class="alt">        Edad = 32</pre>
  
  <pre>    };</pre>
  
  <pre class="alt">    ViewData.Model = <span class="kwrd">new</span> List&lt;Persona&gt;() { matusalen,jacob};</pre>
  
  <pre>    <span class="kwrd">return</span> View();</pre>
  
  <pre class="alt">}</pre>
</div>

En ASP.NET MVC 1.0 para mostrar los elementos tendríamos un código en la vista similar a (suponiendo una vista tipada a IEnumerable<Persona>):

<div class="csharpcode">
  <pre class="alt"><span class="kwrd">&lt;</span><span class="html">table</span><span class="kwrd">&gt;</span></pre>
  
  <pre><span class="kwrd">&lt;</span><span class="html">tr</span><span class="kwrd">&gt;&lt;</span><span class="html">th</span><span class="kwrd">&gt;</span>Nombre<span class="kwrd">&lt;/</span><span class="html">th</span><span class="kwrd">&gt;&lt;</span><span class="html">th</span><span class="kwrd">&gt;</span>Edad<span class="kwrd">&lt;/</span><span class="html">th</span><span class="kwrd">&gt;&lt;/</span><span class="html">tr</span><span class="kwrd">&gt;</span></pre>
  
  <pre class="alt"><span class="asp">&lt;%</span> <span class="kwrd">foreach</span> (var persona <span class="kwrd">in</span> <span class="kwrd">this</span>.Model)</pre>
  
  <pre>   { <span class="asp">%&gt;</span></pre>
  
  <pre class="alt">    <span class="kwrd">&lt;</span><span class="html">tr</span><span class="kwrd">&gt;</span></pre>
  
  <pre>        <span class="kwrd">&lt;</span><span class="html">td</span><span class="kwrd">&gt;</span><span class="asp">&lt;%</span>= Html.Encode(persona.Nombre) <span class="asp">%&gt;</span><span class="kwrd">&lt;/</span><span class="html">td</span><span class="kwrd">&gt;</span></pre>
  
  <pre class="alt">        <span class="kwrd">&lt;</span><span class="html">td</span><span class="kwrd">&gt;</span><span class="asp">&lt;%</span>= Html.Encode(persona.Edad) <span class="asp">%&gt;</span><span class="kwrd">&lt;/</span><span class="html">td</span><span class="kwrd">&gt;</span></pre>
  
  <pre>    <span class="kwrd">&lt;/</span><span class="html">tr</span><span class="kwrd">&gt;</span></pre>
  
  <pre class="alt"><span class="asp">&lt;%</span> } <span class="asp">%&gt;</span></pre>
  
  <pre><span class="kwrd">&lt;/</span><span class="html">table</span><span class="kwrd">&gt;</span></pre>
</div>

Algunos otros a quienes no les gustan tantos tags de servidor mezclados con código cliente se crearían una vista parcial tipada a _Persona_, para mostrar los datos de UNA persona:

<div class="csharpcode">
  <pre class="alt">&lt;%@ Control Language=<span class="str">"C#"</span> </pre>
  
  <pre>Inherits=<span class="str">"<font size="1">System.Web.Mvc.ViewUserControl&lt;MvcApplication1.Models.Persona&gt;</font>"</span> %&gt;</pre>
  
  <pre class="alt">&lt;tr&gt;</pre>
  
  <pre>    &lt;td&gt;&lt;%= Html.Encode(Model.Nombre) %&gt;&lt;/td&gt;</pre>
  
  <pre class="alt">    &lt;td&gt;&lt;%= Html.Encode(Model.Edad) %&gt;&lt;/td&gt;</pre>
  
  <pre>&lt;/tr&gt;</pre>
</div>

Y una vez tienen esta vista parcial (llamada p.ej. PersonaView) en la vista principal tendrían un código mucho más simple:

<div class="csharpcode">
  <pre class="alt"><span class="kwrd">&lt;</span><span class="html">table</span><span class="kwrd">&gt;</span></pre>
  
  <pre><span class="kwrd">&lt;</span><span class="html">tr</span><span class="kwrd">&gt;&lt;</span><span class="html">th</span><span class="kwrd">&gt;</span>Nombre<span class="kwrd">&lt;/</span><span class="html">th</span><span class="kwrd">&gt;&lt;</span><span class="html">th</span><span class="kwrd">&gt;</span>Edad<span class="kwrd">&lt;/</span><span class="html">th</span><span class="kwrd">&gt;&lt;/</span><span class="html">tr</span><span class="kwrd">&gt;</span></pre>
  
  <pre class="alt"><span class="asp">&lt;%</span> <span class="kwrd">foreach</span> (var persona <span class="kwrd">in</span> <span class="kwrd">this</span>.Model)</pre>
  
  <pre>       Html.RenderPartial(<span class="str">"PersonaView"</span>, persona);</pre>
  
  <pre class="alt"><span class="asp">%&gt;</span></pre>
  
  <pre><span class="kwrd">&lt;/</span><span class="html">table</span><span class="kwrd">&gt;</span></pre>
</div>

Esta técnica (de usar una vista parcial) es la que está recomendada (especialmente si nuestro modelo tiene muchos campos)…

… y ASP.NET MVC 2, “integra” esa forma de pensar con los UI templates.

En ASP.NET MVC 2, en lugar de utilizar directamente Html.Encode para mostrar una propiedad, podemos utilizar Html.DisplayFor, que nos renderizará un “visualizador” para una propiedad de nuestro modelo. Y que tipo de visualizador renderizará? Pues para propiedades simples (tales como bools, enteros, strings) es capaz de generar el control HTML correspondiente.

P.ej. el primer paso que podríamos hacer es cojer la vista parcial PersonaView que tenemos y cambiar las llamadas a Html.Encode por llamadas a Html.DisplayFor:

<pre class="csharpcode"><span class="asp">&lt;%@ Control Language="C#"
Inherits="<font size="1">System.Web.Mvc.ViewUserControl&lt;MvcApplication1.Models.Persona&gt;</font>" %&gt;</span>
<span class="kwrd">&lt;</span><span class="html">tr</span><span class="kwrd">&gt;</span>
    <span class="kwrd">&lt;</span><span class="html">td</span><span class="kwrd">&gt;</span><span class="asp">&lt;%</span>= Html.DisplayFor(x =&gt; x.Nombre) <span class="asp">%&gt;</span><span class="kwrd">&lt;/</span><span class="html">td</span><span class="kwrd">&gt;</span>
    <span class="kwrd">&lt;</span><span class="html">td</span><span class="kwrd">&gt;</span><span class="asp">&lt;%</span>= Html.DisplayFor(x =&gt; x.Edad) <span class="asp">%&gt;</span><span class="kwrd">&lt;/</span><span class="html">td</span><span class="kwrd">&gt;</span>
<span class="kwrd">&lt;/</span><span class="html">tr</span><span class="kwrd">&gt;</span></pre>

Fijaos en la sintaxis de DisplayFor: se pasa una lambda expresion que indica _que_ se debe renderizar (puede ser un objeto o una propiedad del modelo). Pasar la lambda tiene un par de ventajas respecto a pasar el _valor_ (como le pasábamos en el Encode), y es que entonces podemos utilizar _reflection_ para inspeccionar la propiedad o clase que recibimos y ver si está decorada con determinados atributos… esto es lo que se llama DataAnnotation Support y sería tema para otro post 😉

Bueno… la verdad es que de momento, alguien quizá se está preguntando que mejora nos aporta esto, porque más o menos tenemos un código muy parecido. La mejora viene _ahora_. Hasta aquí hemos utilizado DisplayFor para tipos sencillos, pero también podemos utilizarlo para tipos _complejos_… Es decir podemos pasar todo un objeto Persona en el DisplayFor y dejar que ASP.NET MVC decida que “visualizador” debe utilizar. Podemos modificar la vista principal para que quede:

<div class="csharpcode">
  <pre class="alt"><span class="kwrd">&lt;</span><span class="html">table</span><span class="kwrd">&gt;</span></pre>
  
  <pre><span class="kwrd">&lt;</span><span class="html">tr</span><span class="kwrd">&gt;&lt;</span><span class="html">th</span><span class="kwrd">&gt;</span>Nombre<span class="kwrd">&lt;/</span><span class="html">th</span><span class="kwrd">&gt;&lt;</span><span class="html">th</span><span class="kwrd">&gt;</span>Edad<span class="kwrd">&lt;/</span><span class="html">th</span><span class="kwrd">&gt;&lt;/</span><span class="html">tr</span><span class="kwrd">&gt;</span></pre>
  
  <pre class="alt"><span class="asp">&lt;%</span> <span class="kwrd">foreach</span> (var persona <span class="kwrd">in</span> <span class="kwrd">this</span>.Model)</pre>
  
  <pre>       Html.DisplayFor(x =&gt; persona);</pre>
  
  <pre class="alt"> <span class="asp">%&gt;</span></pre>
  
  <pre><span class="kwrd">&lt;/</span><span class="html">table</span><span class="kwrd">&gt;</span></pre>
</div>

Fijaos que ahora en cada llamada a DisplayFor, no le estamos pasando una propiedad del modelo (en este caso mi modelo es un IEnumerable<Persona>), sinó que le pasamos el objeto persona a renderizar. Si ahora ejecutamos nuestro código… primera decepción, ya que vemos algo como:

[<img style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_442EB594.png" width="244" height="214" />][2] 

¿Que ha pasado? Pues simple: estamos viendo el comportamiento por defecto de DisplayFor cuando recibe un tipo completo: Itera sobre todas las propiedades y va mostrando su nombre y su valor… Útil, lo que se dice útil no lo es mucho… Nos queda un último paso que es _configurar un UI Template_ para el tipo Persona, es decir indicarle a ASP.NET MVC como debe renderizar un visualizador para un objeto de tipo Persona.

Los visualizadores son siempre vistas parciales normales y corrientes (no deben implementar ninguna interfaz ni nada). En nuestro caso vamos a utilizar la vista parcial PersonaView que creamos antes. Los UI Templates en ASP.NET MVC siguen la regla de “convención antes que configuración”, eso significa que no debemos “registrarlos” en ningun sitio, sinó que implemente ASP.NET MVC va a buscarlos en una carpeta determinada… que se llama de DisplayTemplates y que está dentro de Views/Shared (para templates compartidos entre todos los controladores) o Views/<Controler> (para templates que aplican a un solo controlador).

En nuestro caso creamos la carpeta “DisplayTemplates” dentro de la carpeta Views/Home y movemos la vista PersonaView.ascx dentro de la carpeta DisplayTemplates y **la renombramos como Persona.ascx (para que se llame igual que el tipo que renderiza)**.

Ahora si ejecutamos de nuevo la aplicación, vemos que ya se muestran bien los datos: ASP.NET MVC ha encontrado el UI Template para el tipo persona y lo ha utilizado!

Si no queremos renombrar la vista (y que se siga llamando PersonasView) entonces cuando utilizamos el DisplayFor debemos indicarle que template se debe utilizar:

<div class="csharpcode">
  <pre class="alt">Html.DisplayFor(x =<span class="kwrd">&gt;</span> persona, "PersonaView");</pre>
</div></p> 

En el caso de Html.EditorFor todo funciona exactamente igual, salvo que la carpeta para los templates no se llama DisplayTemplates sinó EditorTemplates.

Un saludo!!!

 [1]: http://geeks.ms/blogs/etomas/archive/2009/09/22/explorando-asp-net-mvc-2-0-225-reas.aspx
 [2]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_52D44017.png