---
title: El problema de la WebGrid con VS2012RC y ASP.NET MVC4

author: eiximenis

date: 2012-08-02T13:23:45+00:00
geeks_url: /?p=1608
geeks_visits:
  - 2660
geeks_ms_views:
  - 1185
categories:
  - Uncategorized

---
> **Nota:** Este post está basado en la versión RC de VS2012 y la versión RC de MVC4 y es posible (o eso espero, vaya!) que en la versión final no haya los problemas que este post menciona!

Buenas! Coje un VS2102RC y crea un nuevo proyecto ASP.NET MVC4, con la plantilla “Basic”.

Crea el HomeController, crea la acción Index y añádele un código tal como:

<div id="codeSnippetWrapper" style="overflow: auto; cursor: text; font-size: 8pt; border-top: silver 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; border-right: silver 1px solid; border-bottom: silver 1px solid; padding-bottom: 4px; direction: ltr; text-align: left; padding-top: 4px; padding-left: 4px; margin: 20px 0px 10px; border-left: silver 1px solid; line-height: 12pt; padding-right: 4px; max-height: 200px; width: 97.5%; background-color: #f4f4f4">
  <div id="codeSnippet" style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4">
    <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum1" style="color: #606060">   1:</span> <span style="color: #0000ff">public</span> ActionResult Index()</pre>
    
    <p>
      <!--CRLF-->
    </p>
    
    <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum2" style="color: #606060">   2:</span> {</pre>
    
    <p>
      <!--CRLF-->
    </p>
    
    <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum3" style="color: #606060">   3:</span>     <span style="color: #0000ff">var</span> data = <span style="color: #0000ff">new</span> List&lt;dynamic&gt;() { <span style="color: #0000ff">new</span> { Name = <span style="color: #006080">"Edu"</span>, Twitter = <span style="color: #006080">"eiximenis"</span> } };</pre>
    
    <p>
      <!--CRLF-->
    </p>
    
    <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum4" style="color: #606060">   4:</span>     <span style="color: #0000ff">return</span> View(data);</pre>
    
    <p>
      <!--CRLF-->
    </p>
    
    <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum5" style="color: #606060">   5:</span> }</pre>
    
    <p>
      <!--CRLF--></div> </div> 
      
      <p>
        Finalmente crea la vista Index.cshtml:
      </p>
      
      <div id="codeSnippetWrapper" style="overflow: auto; cursor: text; font-size: 8pt; border-top: silver 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; border-right: silver 1px solid; border-bottom: silver 1px solid; padding-bottom: 4px; direction: ltr; text-align: left; padding-top: 4px; padding-left: 4px; margin: 20px 0px 10px; border-left: silver 1px solid; line-height: 12pt; padding-right: 4px; max-height: 200px; width: 97.5%; background-color: #f4f4f4">
        <div id="codeSnippet" style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4">
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum1" style="color: #606060">   1:</span> @{</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum2" style="color: #606060">   2:</span>     <span style="color: #0000ff">var</span> grid = <span style="color: #0000ff">new</span> WebGrid(Model, <span style="color: #0000ff">new</span> [] {<span style="color: #006080">"Name"</span>, <span style="color: #006080">"Twitter"</span>});</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum3" style="color: #606060">   3:</span> }</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum4" style="color: #606060">   4:</span> @grid.GetHtml()</pre>
          
          <p>
            <!--CRLF--></div> </div> 
            
            <p>
              Seguro que esperas ver una grid con las dos columnas y una fila no? ¡Pues no! Lo que verás es:
            </p>
            
            <p>
              <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_789491CB.png"><img title="image" style="border-top: 0px; border-right: 0px; border-bottom: 0px; border-left: 0px; display: inline" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_0117F116.png" width="644" height="263" /></a>
            </p>
            
            <p>
              Obtendrás el error “<em>CS0246: The type or namespace name &#8216;WebGrid&#8217; could not be found (are you missing a using directive or an assembly reference?)</em>”.
            </p>
            
            <p>
              Si repites este mismo procedimiento pero cojes la plantilla “Empty” el código funcionará sin problemas 🙂
            </p>
            
            <p>
              <strong>¿Y donde está el problema?</strong>
            </p>
            
            <p>
              Pues eso me he estado preguntando un buen rato. Por supuesto antes de intentar hacer nada he usado el comodín de Google pero esta vez me ha fallado. La única referencia que he encontrado es un post en los blogs de ASP.NET (<a href="http://forums.asp.net/t/1823940.aspx/1?MVC4+WebGrid+problem+in+View+Razor">http://forums.asp.net/t/1823940.aspx/1?MVC4+WebGrid+problem+in+View+Razor+</a>) donde alguien más experimenta el problema pero no se llega a ninguna solución.
            </p>
            
            <p>
              Al final lo que he hecho para solucionar el problema ha sido:
            </p>
            
            <ol>
              <li>
                Desde NuGet <strong>desinstalar</strong> el paquete ASP.NET MVC4 RC 4.0.20505. Eso ha desinstalado también los paquetes Microsoft.AspNet.WebPages 2.0.20505.0 y Microsoft.AspNet.Razor 2.0.20505.0
              </li>
              <li>
                Volver a instalar el paquete ASP.NET MVC4 RC 4.0.20505 desde NuGet. Me ha dado un error y ha hecho un rollback.
              </li>
              <li>
                Intentar de nuevo volver a instalar el paquete ASP.NET MVC4 RC 4.0.20505. Ahora todo ha funcionado correctamente.
              </li>
            </ol>
            
            <p>
              Después del tercer punto, la WebGrid ya funcionaba correctamente.
            </p>
            
            <p>
              Honestamente desconozco la causa, pero bueno… si alguien se encuentra con ello, ya lo sabe. Que pruebe esto a ver si le funciona! 🙂
            </p>
            
            <p>
              Un saludo!
            </p>