---
title: 'ASP.NET MVC3: Un helper Repeater'
description: 'ASP.NET MVC3: Un helper Repeater'
author: eiximenis

date: 2011-01-26T09:35:23+00:00
geeks_url: /?p=1555
geeks_visits:
  - 3944
geeks_ms_views:
  - 1400
categories:
  - Uncategorized

---
Muy buenas! En el post anterior comenté la característica de los <a href="http://geeks.ms/blogs/etomas/archive/2011/01/25/asp-net-mvc3-razor-templates.aspx" target="_blank" rel="noopener noreferrer">templates de Razor</a> y hoy vamos a ver como podríamos crear un _helper_ que emule un poco el <a href="http://msdn.microsoft.com/es-es/library/6weyd81h(VS.80).aspx" target="_blank" rel="noopener noreferrer">control Repeater</a> que hay en webforms (salvando las distancias, claro).

Vamos a crear un _helper externo_, es decir que sea reutilizable en distintos proyectos: para ello nuestro helper va a residir en una clase (en mi ejemplo en el propio proyecto web, pero se podría situar en una librería de clases para ser reutilizable).

**Esqueleto inicial**

A nuestro helper se le pasará lo siguiente:

  1. Una cabecera: template Razor que se renderizará una sola vez al principio. 
  2. Un cuerpo: template Razor que se renderizará una vez por cada elemento 
  3. Un pie: template Razor que se renderizará una sola vez al final. 
  4. Una colección de elementos (por cada elemento se renderizará el cuerpo). 

Recordáis que ayer comentamos que los templates Razor son Func<T, HelperResult>? 

Bien, pues vamos a declarar nuestro helper:

&#160;

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> IHtmlString Repeat&lt;TItem&gt;(<br />    IEnumerable&lt;TItem&gt; items,<br />    Func&lt;dynamic, HelperResult&gt; header,<br />    Func&lt;TItem, HelperResult&gt; body,<br />    Func&lt;dynamic, HelperResult&gt; footer<br />    )<br />{<br />    var builder = <span style="color: #0000ff">new</span> StringBuilder();<br />    <span style="color: #0000ff">if</span> (header != <span style="color: #0000ff">null</span>)<br />    {<br />        builder.Append(header(<span style="color: #0000ff">null</span>).ToHtmlString());<br />    }<br />    <br />    <span style="color: #0000ff">foreach</span> (var item <span style="color: #0000ff">in</span> items)<br />    {<br />        builder.Append(body(item).ToHtmlString());<br />    }<br />    <span style="color: #0000ff">if</span> (footer != <span style="color: #0000ff">null</span>)<br />    {<br />        builder.Append(footer(<span style="color: #0000ff">null</span>).ToHtmlString());<br />    }<br /><br />    <span style="color: #0000ff">return</span> MvcHtmlString.Create(builder.ToString());<br />}<br /></pre>
  
  <p>
    </div> 
    
    <p>
      Los templates siempre están declarados como Func<T, HelperResult>, en este caso declaramos tres templates:
    </p>
    
    <ol>
      <li>
        header, cuyo parámetro @item es dynamic.
      </li>
      <li>
        body, cuyo parámetro @item es TItem, donde TItem es el tipo del enumerable que se le pasa al helper
      </li>
      <li>
        footer, cuyo parámetro @item es dynamic
      </li>
    </ol>
    
    <p>
      Para invocar los templates y obtener el html asociado simplemente invocamos el Func (pasándole el parámetro del tipo correcto) y sobre el resultado llamamos a <strong>ToHtmlString</strong>: Este método nos devuelve la cadena HTML que ha parseado Razor.
    </p>
    
    <p>
      Ahora p.ej. me creo una acción tal en el controlador:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> ActionResult List()<br />{<br />    <span style="color: #0000ff">return</span> View(<span style="color: #0000ff">new</span> List&lt;Product&gt;<br />                    {<br />                        <span style="color: #0000ff">new</span> Product() {Nombre=<span style="color: #006080">"PS3"</span>, Precio=300},<br />                        <span style="color: #0000ff">new</span> Product() {Nombre=<span style="color: #006080">"XBox360"</span>, Precio=150},<br />                        <span style="color: #0000ff">new</span> Product() {Nombre=<span style="color: #006080">"Wii"</span>, Precio=100}<br />                    });<br />}</pre>
      
      <p>
        </div> 
        
        <p>
          Y en la vista asociada:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">@using MvcApplication14.Helpers<br />@model IEnumerable<span style="color: #0000ff">&lt;</span><span style="color: #800000">MvcApplication14.Models.Product</span><span style="color: #0000ff">&gt;</span><br /><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">table</span><span style="color: #0000ff">&gt;</span><br />@Repeater.Repeat(Model, <br />    @<span style="color: #0000ff">&lt;</span><span style="color: #800000">thead</span><span style="color: #0000ff">&gt;&lt;</span><span style="color: #800000">tr</span><span style="color: #0000ff">&gt;&lt;</span><span style="color: #800000">td</span><span style="color: #0000ff">&gt;</span>Nombre<span style="color: #0000ff">&lt;/</span><span style="color: #800000">td</span><span style="color: #0000ff">&gt;&lt;</span><span style="color: #800000">td</span><span style="color: #0000ff">&gt;</span>Precio<span style="color: #0000ff">&lt;/</span><span style="color: #800000">td</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">tr</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">thead</span><span style="color: #0000ff">&gt;</span>, <br />    @<span style="color: #0000ff">&lt;</span><span style="color: #800000">tr</span><span style="color: #0000ff">&gt;&lt;</span><span style="color: #800000">td</span><span style="color: #0000ff">&gt;</span>@item.Nombre<span style="color: #0000ff">&lt;/</span><span style="color: #800000">td</span><span style="color: #0000ff">&gt;&lt;</span><span style="color: #800000">td</span><span style="color: #0000ff">&gt;</span>@item.Precio<span style="color: #0000ff">&lt;/</span><span style="color: #800000">td</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">tr</span><span style="color: #0000ff">&gt;</span>, <br />    null)<br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">table</span><span style="color: #0000ff">&gt;</span></pre>
          
          <p>
            </div> 
            
            <p>
              Fijaos en la llamada a Repeater.Repeat, donde le paso el modelo que recibe la vista (que es un IEnumerable de Product) y los tres templates Razor (en este caso no le paso footer y por ello pongo null).
            </p>
            
            <p>
              En el segundo template (body) puedo acceder al elemento que se está renderizando mediante @item. Además, dado que en helper he declarado el template body de tipo Func<TItem, HelperResult>, tengo soporte de Intellisense:
            </p>
            
            <p>
              <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_3B6BD963.png"><img style="background-image: none; border-bottom: 0px; border-left: 0px; margin: ; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_7984F414.png" width="244" height="107" /></a>
            </p>
            
            <p>
              Si teclease @item en el template header o footer no tendría intellisense porque los he declarado dynamic en el helper (además ojo, que en el helper le paso null, por lo que rebentaría si usamos @item en el template header o footer).
            </p>
            
            <p>
              Y listos! El código HTML generado es:
            </p>
            
            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">&lt;</span><span style="color: #800000">thead</span><span style="color: #0000ff">&gt;&lt;</span><span style="color: #800000">tr</span><span style="color: #0000ff">&gt;&lt;</span><span style="color: #800000">td</span><span style="color: #0000ff">&gt;</span>Nombre<span style="color: #0000ff">&lt;/</span><span style="color: #800000">td</span><span style="color: #0000ff">&gt;&lt;</span><span style="color: #800000">td</span><span style="color: #0000ff">&gt;</span>Precio<span style="color: #0000ff">&lt;/</span><span style="color: #800000">td</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">tr</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">thead</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">tr</span><span style="color: #0000ff">&gt;&lt;</span><span style="color: #800000">td</span><span style="color: #0000ff">&gt;</span>PS3<span style="color: #0000ff">&lt;/</span><span style="color: #800000">td</span><span style="color: #0000ff">&gt;&lt;</span><span style="color: #800000">td</span><span style="color: #0000ff">&gt;</span>300<span style="color: #0000ff">&lt;/</span><span style="color: #800000">td</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">tr</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">tr</span><span style="color: #0000ff">&gt;&lt;</span><span style="color: #800000">td</span><span style="color: #0000ff">&gt;</span>XBox360<span style="color: #0000ff">&lt;/</span><span style="color: #800000">td</span><span style="color: #0000ff">&gt;&lt;</span><span style="color: #800000">td</span><span style="color: #0000ff">&gt;</span>150<span style="color: #0000ff">&lt;/</span><span style="color: #800000">td</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">tr</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">tr</span><span style="color: #0000ff">&gt;&lt;</span><span style="color: #800000">td</span><span style="color: #0000ff">&gt;</span>Wii<span style="color: #0000ff">&lt;/</span><span style="color: #800000">td</span><span style="color: #0000ff">&gt;&lt;</span><span style="color: #800000">td</span><span style="color: #0000ff">&gt;</span>100<span style="color: #0000ff">&lt;/</span><span style="color: #800000">td</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">tr</span><span style="color: #0000ff">&gt;</span></pre>
              
              <p>
                </div> 
                
                <p>
                  Guay, no?
                </p>
                
                <p>
                  Podríamos “complicar” un poco el helper, para que desde los templates supiesemos si estamos en una fila par o impar y así aplicar clases… Para ello nos basta con declarar una clase adicional en nuestor helper:
                </p>
                
                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> RepeaterItem&lt;TItem&gt;<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">int</span> Index { get; <span style="color: #0000ff">private</span> set; }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">bool</span> IsEven { get { <span style="color: #0000ff">return</span> Index % 2 == 0; } }<br />    <span style="color: #0000ff">public</span> TItem Item { get; <span style="color: #0000ff">private</span> set; }<br /><br />    <span style="color: #0000ff">public</span> RepeaterItem(TItem item, <span style="color: #0000ff">int</span> index)<br />    {<br />        Index = index;<br />        Item = item;<br />    }<br />}</pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Esa clase simplemente contiene el índice del elemento actual, el propio elemento y una propiedad que indica si es par o no.
                    </p>
                    
                    <p>
                      Ahora modificamos el helper para pasarle al template body un objeto RepeaterItem<TItem>:
                    </p>
                    
                    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">class</span> Repeater<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> IHtmlString Repeat&lt;TItem&gt;(<br />        IEnumerable&lt;TItem&gt; items,<br />        Func&lt;dynamic, HelperResult&gt; header,<br />        Func&lt;RepeaterItem&lt;TItem&gt;, HelperResult&gt; body,<br />        Func&lt;dynamic, HelperResult&gt; footer<br />        )<br />    {<br />        var builder = <span style="color: #0000ff">new</span> StringBuilder();<br />        <span style="color: #0000ff">if</span> (header != <span style="color: #0000ff">null</span>)<br />        {<br />            builder.Append(header(<span style="color: #0000ff">null</span>).ToHtmlString());<br />        }<br /><br />        var count = 0;<br />        <span style="color: #0000ff">foreach</span> (var item <span style="color: #0000ff">in</span> items)<br />        {<br />            var repeaterItem = <span style="color: #0000ff">new</span> RepeaterItem&lt;TItem&gt;(item, count++);<br />            builder.Append(body(repeaterItem).ToHtmlString());<br />        }<br />        <span style="color: #0000ff">if</span> (footer != <span style="color: #0000ff">null</span>)<br />        {<br />            builder.Append(footer(<span style="color: #0000ff">null</span>).ToHtmlString());<br />        }<br /><br />        <span style="color: #0000ff">return</span> MvcHtmlString.Create(builder.ToString());<br />    }<br />}</pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          Los cambios básicamente son declarar el template domo Func<RepeaterItem<Titem>> y cuando invocamos el template, crear antes un objeto RepeaterItem y pasárselo como parámetro.
                        </p>
                        
                        <p>
                          Finalmente ahora debemos modificar la vista, ya que el parámetro @item de nuestro template es ahora un RepeaterItem<TItem>:
                        </p>
                        
                        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">@Repeater.Repeat(Model, <br />    @<span style="color: #0000ff">&lt;</span><span style="color: #800000">thead</span><span style="color: #0000ff">&gt;&lt;</span><span style="color: #800000">tr</span><span style="color: #0000ff">&gt;&lt;</span><span style="color: #800000">td</span><span style="color: #0000ff">&gt;</span>Nombre<span style="color: #0000ff">&lt;/</span><span style="color: #800000">td</span><span style="color: #0000ff">&gt;&lt;</span><span style="color: #800000">td</span><span style="color: #0000ff">&gt;</span>Precio<span style="color: #0000ff">&lt;/</span><span style="color: #800000">td</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">tr</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">thead</span><span style="color: #0000ff">&gt;</span>, <br />    @<span style="color: #0000ff">&lt;</span><span style="color: #800000">tr</span> <span style="color: #ff0000">style</span><span style="color: #0000ff">="@(item.IsEven ? "</span><span style="color: #ff0000">background-color:</span> <span style="color: #ff0000">white</span><span style="color: #0000ff">" : "</span><span style="color: #ff0000">background-color:pink</span><span style="color: #0000ff">")"</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">td</span><span style="color: #0000ff">&gt;</span>@item.Item.Nombre<span style="color: #0000ff">&lt;/</span><span style="color: #800000">td</span><span style="color: #0000ff">&gt;&lt;</span><span style="color: #800000">td</span><span style="color: #0000ff">&gt;</span>@item.Item.Precio<span style="color: #0000ff">&lt;/</span><span style="color: #800000">td</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">tr</span><span style="color: #0000ff">&gt;</span>, <br />    null)</pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              Fijaos como dentro del template body puedo preguntar si el elemento actual es par (@item.IsEven y acceder al propio elemento @item.Item). En este caso uso @item.IsEven para cambiar el color de fondo de la fila:
                            </p>
                            
                            <p>
                              <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_634ED8C2.png"><img style="background-image: none; border-bottom: 0px; border-left: 0px; margin: ; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_5099DF0B.png" width="145" height="133" /></a>
                            </p>
                            
                            <p>
                              Y listos! Espero que esos dos posts sobre templates Razor os hayan parecido interesantes!
                            </p>
                            
                            <p>
                              Un saludo a todos! 😉
                            </p>