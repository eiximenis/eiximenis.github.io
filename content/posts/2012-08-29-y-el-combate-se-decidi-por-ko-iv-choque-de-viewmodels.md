---
title: 'Y el combate se decidió por KO (iv): Choque de viewmodels'

author: eiximenis

date: 2012-08-29T18:22:00+00:00
geeks_url: /?p=1610
geeks_visits:
  - 1974
geeks_ms_views:
  - 947
categories:
  - Uncategorized

---
> **Disclaimer:** Este post es un poco _distinto_ al resto de posts de [esta serie sobre knockout][1]. De hecho no tenía presente escribirlo pero lo he hecho cuando he visto la cantidad de preguntas relativas a ello que aparecen por Google. Aunque lo he reescrito varias veces, entiendo que puede ser un post durillo de leer, especialmente si no se tiene experiencia previa en ASP.NET MVC. Si NO quieres leerte este post, **no te preocupes:** no es necesario para nada para entender el resto de posts de la serie, ni explica nada nuevo sobre knockout que no hayamos vistos en los 3 anteriores. Se trata simplemente de divagaciones sobre el siguiente tema: ¿_Se pueden usar los helpers de ASP.NET MVC para mostrar o editar datos junto con knockout_?

En los tres primeros posts de esta serie sobre Knockout hemos estado viendo algunos de los aspecto básicos de dicha librería (en sucesivos posts iremos viendo más). Pero hasta ahora nos hemos limitado a un modelo de aplicación en el que la vista obtenía los datos llamando a un servicio REST, y a partir de dichos datos creaba su viewmodel y usaba knockout para representarlo en pantalla.

Pero este modelo, extremadamente útil si hacemos uso intensivo de Ajax, no es el único posible. Existe otro, más &ldquo;clásico&rdquo; en el que las vistas reciben datos de sus controladores (en lugar de tener que realizar una llamada ajax a un servicio REST). En el contexto de ASP.NET MVC llamamos también viewmodel a los datos que un controlador manda a la vista (y que esta accede a través de su propiedad Model). Así pues ahora vamos a lidiar con dos viewmodels:

  1. Los datos que el controlador manda a la vista y que esta accede usando la propiedad Model. Es el viewmodel de ASP.NET MVC o el viewmodel de servidor. 
  2. El objeto javascript que usa knockout. Es el viewmodel de cliente. 

En este post veremos como el &ldquo;modelo clásico de ASP.NET MVC&rdquo; choca con el modelo knockout y daremos algunas indicaciones de como podemos solucionarlo. No llegaremos a una solución completa porque simple y llanamente no la hay.

**<span style="text-decoration: underline;">Escenario 1. Mostrando datos</span>**

Vamos a suponer que tenemos una clase Beer:

<div style="overflow: auto; cursor: text; font-size: 8pt; font-family: 'Courier New', courier, monospace; direction: ltr; text-align: left; margin: 20px 0px 10px; line-height: 12pt; max-height: 200px; width: 97.5%; background-color: #f4f4f4; border: silver 1px solid; padding: 4px;" id="codeSnippetWrapper">
  <div style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;" id="codeSnippet">
    <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum1">   1:</span> <span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> Beer</pre>
    
    <p>
      <!--CRLF-->
    </p>
    
    <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum2">   2:</span> {</pre>
    
    <p>
      <!--CRLF-->
    </p>
    
    <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum3">   3:</span>     <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Name { get; set; }</pre>
    
    <p>
      <!--CRLF-->
    </p>
    
    <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum4">   4:</span>     <span style="color: #0000ff">public</span> <span style="color: #0000ff">int</span> Ibu { get; set; }</pre>
    
    <p>
      <!--CRLF-->
    </p>
    
    <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum5">   5:</span> }</pre>
    
    <p>
      <!--CRLF--></div> </div> 
      
      <p>
        Y luego tenemos un controlador con una acción tal que:
      </p>
      
      <div style="overflow: auto; cursor: text; font-size: 8pt; font-family: 'Courier New', courier, monospace; direction: ltr; text-align: left; margin: 20px 0px 10px; line-height: 12pt; max-height: 200px; width: 97.5%; background-color: #f4f4f4; border: silver 1px solid; padding: 4px;" id="codeSnippetWrapper">
        <div style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;" id="codeSnippet">
          <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum1">   1:</span> <span style="color: #0000ff">public</span> ActionResult Index()</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum2">   2:</span> {</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum3">   3:</span>     var beers = <span style="color: #0000ff">new</span> List&lt;Beer&gt;</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum4">   4:</span>                     {</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum5">   5:</span>                         <span style="color: #0000ff">new</span> Beer { Name = <span style="color: #006080">"Estrella Damm"</span>, Ibu = 30},</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum6">   6:</span>                         <span style="color: #0000ff">new</span> Beer { Name = <span style="color: #006080">"Marina Devil's IPA"</span>, Ibu = 150}</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum7">   7:</span>                     };</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum8">   8:</span>     <span style="color: #0000ff">return</span> View(beers);</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum9">   9:</span> }</pre>
          
          <p>
            <!--CRLF--></div> </div> 
            
            <p>
              La vista recibe esta lista de cervezas en su propiedad Model. Ahora la pregunta es, podemos usar knockout para mostrar los datos?
            </p>
            
            <p>
              La respuesta es que sí, aunque para ello debemos crear el viewmodel de cliente <em>a partir de los datos del viewmodel de servidor</em>:
            </p>
            
            <div style="overflow: auto; cursor: text; font-size: 8pt; font-family: 'Courier New', courier, monospace; direction: ltr; text-align: left; margin: 20px 0px 10px; line-height: 12pt; max-height: 200px; width: 97.5%; background-color: #f4f4f4; border: silver 1px solid; padding: 4px;" id="codeSnippetWrapper">
              <div style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;" id="codeSnippet">
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum1">   1:</span> <span style="color: #cc6633">@using</span> System.Web.Script.Serialization</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum2">   2:</span> <span style="color: #cc6633">@model</span> IEnumerable&lt;MvcApplication1.Models.Beer&gt;</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum3">   3:</span> @{</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum4">   4:</span>     ViewBag.Title = <span style="color: #006080">"Index"</span>;</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum5">   5:</span>     <span style="color: #0000ff">var</span> ser = <span style="color: #0000ff">new</span> JavaScriptSerializer();</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum6">   6:</span>     <span style="color: #0000ff">var</span> jscode = Model != <span style="color: #0000ff">null</span> ? ser.Serialize(Model) : <span style="color: #0000ff">string</span>.Empty;</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum7">   7:</span> }</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum8">   8:</span>&nbsp; </pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum9">   9:</span> &lt;h2&gt;Index&lt;/h2&gt;</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum10">  10:</span>&nbsp; </pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum11">  11:</span> &lt;script type=<span style="color: #006080">"text/javascript"</span>&gt;</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum12">  12:</span>     $(document).ready(<span style="color: #0000ff">function</span> () {</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum13">  13:</span>         <span style="color: #0000ff">var</span> vm = JSON.parse(<span style="color: #006080">'@Html.Raw(jscode)'</span>);</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum14">  14:</span>         vm = completeViewModel(vm);</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum15">  15:</span>         ko.applyBindings(vm);</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum16">  16:</span>     });</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum17">  17:</span>&nbsp; </pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum18">  18:</span>     <span style="color: #0000ff">function</span> completeViewModel(vm) {</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum19">  19:</span>         <span style="color: #0000ff">return</span> { items: vm };</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum20">  20:</span>     }</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum21">  21:</span> &lt;/script&gt;</pre>
                
                <p>
                  <!--CRLF--></div> </div> 
                  
                  <p>
                    La vista contiene un bloque de código Razor y luego un tag <script> con código de cliente:
                  </p>
                  
                  <ol>
                    <li>
                      El código Razor recoje el ViewModel de ASP.NET MVC y lo serializa a una cadena JSON.
                    </li>
                    <li>
                      El código <script /> usa JSON.parse para obtener un objeto javascript a&nbsp; partir de la cadena en JSON obtenida en el código anterior.
                    </li>
                  </ol>
                  
                  <p>
                    Si ejecutamos esta vista y ponemos un breakpoint en el javascript cuando se ha obtenido el viewmodel de cliente (yo he usado las herramientas de Chrome para ello), vemos que efectivamente tenemos un objeto javascript con los datos del viewmodel de ASP.NET MVC (excepto que le he añadido la propiedad items que me servirá para el enlace con knockout):
                  </p>
                  
                  <p>
                    <a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_46D42D82.png"><img height="89" width="504" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_6F08F745.png" alt="image" border="0" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" title="image" /></a>
                  </p>
                  
                  <p>
                    Finalmente tan solo nos queda mostrar los datos:
                  </p>
                  
                  <div style="overflow: auto; cursor: text; font-size: 8pt; font-family: 'Courier New', courier, monospace; direction: ltr; text-align: left; margin: 20px 0px 10px; line-height: 12pt; max-height: 200px; width: 97.5%; background-color: #f4f4f4; border: silver 1px solid; padding: 4px;" id="codeSnippetWrapper">
                    <div style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;" id="codeSnippet">
                      <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum1">   1:</span> <span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="beers"</span> <span style="color: #ff0000">data-bind</span><span style="color: #0000ff">="foreach: items"</span><span style="color: #0000ff">&gt;</span></pre>
                      
                      <p>
                        <!--CRLF-->
                      </p>
                      
                      <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum2">   2:</span>     <span style="color: #0000ff">&lt;</span><span style="color: #800000">span</span> <span style="color: #ff0000">data-bind</span><span style="color: #0000ff">="text: Name"</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">span</span><span style="color: #0000ff">&gt;</span> - <span style="color: #0000ff">&lt;</span><span style="color: #800000">span</span> <span style="color: #ff0000">data-bind</span><span style="color: #0000ff">="text: Ibu"</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">span</span><span style="color: #0000ff">&gt;</span></pre>
                      
                      <p>
                        <!--CRLF-->
                      </p>
                      
                      <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum3">   3:</span>     <span style="color: #0000ff">&lt;</span><span style="color: #800000">br</span> <span style="color: #0000ff">/&gt;</span></pre>
                      
                      <p>
                        <!--CRLF-->
                      </p>
                      
                      <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum4">   4:</span> <span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span></pre>
                      
                      <p>
                        <!--CRLF--></div> </div> 
                        
                        <p>
                          En este punto es necesario hacer una mención importante: la cadena JSON resultado de convertir a JSON el viewmodel de ASP.NET MVC se está incluyendo en el código fuente de la página (a través del Html.Raw). Eso es lo que obtengo si hago un &ldquo;ver codigo fuente&rdquo;:
                        </p>
                        
                        <div style="overflow: auto; cursor: text; font-size: 8pt; height: 35px; font-family: 'Courier New', courier, monospace; direction: ltr; text-align: left; margin: 20px 0px 10px; line-height: 12pt; max-height: 200px; width: 97.5%; background-color: #f4f4f4; border: silver 1px solid; padding: 4px;" id="codeSnippetWrapper">
                          <div style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;" id="codeSnippet">
                            <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum1">   1:</span> <span style="color: #0000ff">var</span> vm = JSON.parse(<span style="color: #006080">'[{"Name":"Estrella Damm","Ibu":30},{"Name":"Marina Devilu0027s IPA","Ibu":150}]'</span>);</pre>
                            
                            <p>
                              <!--CRLF--></div> </div> 
                              
                              <p>
                                Por lo tanto, si mi viewmodel ASP.NET MVC es <em>grande</em> eso puede generar páginas grandes (en Kilobytes) y por lo tanto lentas.
                              </p>
                              
                              <p>
                                <strong>Pregunta: </strong><em>Puedo usar los helpers de ASP.NET MVC (DisplayFor y similares) para mostrar estos datos?</em>
                              </p>
                              
                              <p>
                                <strong>Respuesta:</strong> Si... y no. Me explico.
                              </p>
                              
                              <p>
                                Algunos helpers pueden <em>no generar tag html</em>. Por ejemplo DisplayFor para una propiedad de tipo string, no genera tag alguno (simplemente renderiza el texto tal cual). Si no hay tag HTML, no hay sitio donde poner el atributo data-bind para usar con knockout.
                              </p>
                              
                              <p>
                                Si usas un helper que genere un tag (p.ej. pones un Html.TextboxFor), entonces puedes usarlo, pero <strong>ten presente que el propio helper ya genera el atributo value</strong>, así que realmente el enlace con knockout no tiene mucho sentido, si tan solo vas a mostrar datos.
                              </p>
                              
                              <p>
                                Mi recomendación es que te olvides de los helpers de MVC si vas a mostrar datos usando knockout. Si de todos modos quieres usarlos debes hacer que dichos helpers generen el atributo data-bind para mostrar los datos usando knockout. P.ej. el siguiente código generaria un textbox con el atributo data-bind=&rdquo;value: Name&rdquo; y de solo lectura:
                              </p>
                              
                              <div style="overflow: auto; cursor: text; font-size: 8pt; height: 35px; font-family: 'Courier New', courier, monospace; direction: ltr; text-align: left; margin: 20px 0px 10px; line-height: 12pt; max-height: 200px; width: 97.5%; background-color: #f4f4f4; border: silver 1px solid; padding: 4px;" id="codeSnippetWrapper">
                                <div style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;" id="codeSnippet">
                                  <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum1">   1:</span> @Html.TextBoxFor(x =<span style="color: #0000ff">&gt;</span> x.Name, new { data_bind = "value: Name", @readonly = true })</pre>
                                  
                                  <p>
                                    <!--CRLF--></div> </div> 
                                    
                                    <blockquote>
                                      <p>
                                        (Fíjate en que como data-bind no es un nombre válido para una propiedad del objeto anónimo en C# se usa data_bind (ASP.NET MVC transforma el guión bajo en un guión al generar el código), y el uso de @readonly en lugar de readonly (que es palabra clave reservada de C#)).
                                      </p>
                                    </blockquote>
                                    
                                    <p>
                                      No obstante... crees que tiene realmente sentido hacer esto? Ya tienes tus datos en el viewmodel de knockout, es mucho mejor usar los mecanismos de knockout para mostrarlos!
                                    </p>
                                    
                                    <p>
                                      <strong><span style="text-decoration: underline;">Escenario 2: Edición</span></strong>
                                    </p>
                                    
                                    <p>
                                      Empecemos por hacer que nuestro controlador devuelva una sola cerveza y vamos a intentar editarla usando knockout. Ni corto ni perzoso he creado la acción nueva (Edit):
                                    </p>
                                    
                                    <div style="overflow: auto; cursor: text; font-size: 8pt; font-family: 'Courier New', courier, monospace; direction: ltr; text-align: left; margin: 20px 0px 10px; line-height: 12pt; max-height: 200px; width: 97.5%; background-color: #f4f4f4; border: silver 1px solid; padding: 4px;" id="codeSnippetWrapper">
                                      <div style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;" id="codeSnippet">
                                        <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum1">   1:</span> <span style="color: #0000ff">public</span> ActionResult Edit()</pre>
                                        
                                        <p>
                                          <!--CRLF-->
                                        </p>
                                        
                                        <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum2">   2:</span> {</pre>
                                        
                                        <p>
                                          <!--CRLF-->
                                        </p>
                                        
                                        <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum3">   3:</span>     var beer = <span style="color: #0000ff">new</span> Beer() {Name = <span style="color: #006080">"Mezquita"</span>, Ibu = 50};</pre>
                                        
                                        <p>
                                          <!--CRLF-->
                                        </p>
                                        
                                        <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum4">   4:</span>     <span style="color: #0000ff">return</span> View(beer);</pre>
                                        
                                        <p>
                                          <!--CRLF-->
                                        </p>
                                        
                                        <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum5">   5:</span> }</pre>
                                        
                                        <p>
                                          <!--CRLF--></div> </div> 
                                          
                                          <p>
                                            Bien, ahora vamos a crear una vista de edición, pero usando knockout, para ello usando el esquema anterior, obtenemos el viewmodel de knockout a partir del viewmodel de ASP.NET MVC:
                                          </p>
                                          
                                          <div style="overflow: auto; cursor: text; font-size: 8pt; font-family: 'Courier New', courier, monospace; direction: ltr; text-align: left; margin: 20px 0px 10px; line-height: 12pt; max-height: 200px; width: 97.5%; background-color: #f4f4f4; border: silver 1px solid; padding: 4px;" id="codeSnippetWrapper">
                                            <div style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;" id="codeSnippet">
                                              <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum1">   1:</span> <span style="color: #cc6633">@using</span> System.Web.Script.Serialization</pre>
                                              
                                              <p>
                                                <!--CRLF-->
                                              </p>
                                              
                                              <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum2">   2:</span> <span style="color: #cc6633">@model</span> MvcApplication1.Models.Beer</pre>
                                              
                                              <p>
                                                <!--CRLF-->
                                              </p>
                                              
                                              <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum3">   3:</span> @{</pre>
                                              
                                              <p>
                                                <!--CRLF-->
                                              </p>
                                              
                                              <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum4">   4:</span>     ViewBag.Title = <span style="color: #006080">"Index"</span>;</pre>
                                              
                                              <p>
                                                <!--CRLF-->
                                              </p>
                                              
                                              <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum5">   5:</span>     <span style="color: #0000ff">var</span> ser = <span style="color: #0000ff">new</span> JavaScriptSerializer();</pre>
                                              
                                              <p>
                                                <!--CRLF-->
                                              </p>
                                              
                                              <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum6">   6:</span>     <span style="color: #0000ff">var</span> jscode = Model != <span style="color: #0000ff">null</span> ? ser.Serialize(Model) : <span style="color: #0000ff">string</span>.Empty;</pre>
                                              
                                              <p>
                                                <!--CRLF-->
                                              </p>
                                              
                                              <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum7">   7:</span> }</pre>
                                              
                                              <p>
                                                <!--CRLF-->
                                              </p>
                                              
                                              <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum8">   8:</span>&nbsp; </pre>
                                              
                                              <p>
                                                <!--CRLF-->
                                              </p>
                                              
                                              <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum9">   9:</span> &lt;script type=<span style="color: #006080">"text/javascript"</span>&gt;</pre>
                                              
                                              <p>
                                                <!--CRLF-->
                                              </p>
                                              
                                              <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum10">  10:</span>     $(document).ready(<span style="color: #0000ff">function</span> () {</pre>
                                              
                                              <p>
                                                <!--CRLF-->
                                              </p>
                                              
                                              <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum11">  11:</span>         <span style="color: #0000ff">var</span> vm = JSON.parse(<span style="color: #006080">'@Html.Raw(jscode)'</span>);</pre>
                                              
                                              <p>
                                                <!--CRLF-->
                                              </p>
                                              
                                              <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum12">  12:</span>         ko.applyBindings(vm);</pre>
                                              
                                              <p>
                                                <!--CRLF-->
                                              </p>
                                              
                                              <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum13">  13:</span>     });</pre>
                                              
                                              <p>
                                                <!--CRLF-->
                                              </p>
                                              
                                              <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum14">  14:</span>&nbsp; </pre>
                                              
                                              <p>
                                                <!--CRLF-->
                                              </p>
                                              
                                              <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum15">  15:</span> &lt;/script&gt;</pre>
                                              
                                              <p>
                                                <!--CRLF--></div> </div> 
                                                
                                                <p>
                                                  Bien, ahora viene el siguiente punto. Intentemos usar los helpers de ASP.NET MVC para crear los controles de edición:
                                                </p>
                                                
                                                <div style="overflow: auto; cursor: text; font-size: 8pt; font-family: 'Courier New', courier, monospace; direction: ltr; text-align: left; margin: 20px 0px 10px; line-height: 12pt; max-height: 200px; width: 97.5%; background-color: #f4f4f4; border: silver 1px solid; padding: 4px;" id="codeSnippetWrapper">
                                                  <div style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;" id="codeSnippet">
                                                    <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum1">   1:</span> @Html.LabelFor(x=&gt;x.Name)</pre>
                                                    
                                                    <p>
                                                      <!--CRLF-->
                                                    </p>
                                                    
                                                    <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum2">   2:</span> @Html.TextBoxFor(x=&gt;x.Name, <span style="color: #0000ff">new</span> {data_bind=<span style="color: #006080">"value: Name"</span>})</pre>
                                                    
                                                    <p>
                                                      <!--CRLF-->
                                                    </p>
                                                    
                                                    <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum3">   3:</span> &lt;br /&gt;</pre>
                                                    
                                                    <p>
                                                      <!--CRLF-->
                                                    </p>
                                                    
                                                    <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum4">   4:</span> @Html.LabelFor(x=&gt;x.Ibu)</pre>
                                                    
                                                    <p>
                                                      <!--CRLF-->
                                                    </p>
                                                    
                                                    <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum5">   5:</span> @Html.TextBoxFor(x=&gt;x.Ibu, <span style="color: #0000ff">new</span> {data_bind=<span style="color: #006080">"value: Ibu"</span>})</pre>
                                                    
                                                    <p>
                                                      <!--CRLF--></div> </div> 
                                                      
                                                      <p>
                                                        Fijaos ya en el &ldquo;primer choque&rdquo;. A pesar de usar las versiones &ldquo;strong typed&rdquo; de los editores, tengo que especificar de nuevo el nombre de la propiedad como valor de data_bind. Eso es proclive a errores, ya que puedo hacer TextBoxFor(x=>x.Name) y en el data_bind poner otro nombre de propiedad. Por supuesto esto se podria arreglar con un helper propio.
                                                      </p>
                                                      
                                                      <p>
                                                        Al margen de este detalle, parece que todo funciona. Incluso si envio el formulario me llegan los datos modificados:
                                                      </p>
                                                      
                                                      <p>
                                                        <a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_7315D5C8.png"><img height="92" width="504" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_49A19488.png" alt="image" border="0" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" title="image" /></a>
                                                      </p>
                                                      
                                                      <p>
                                                        Pero... reflexionemos. Hemos usado knockout para algo? Pues no. De hecho puedes comprobarlo comentando la línea ko.applyBindings y verás que todo sigue funcionando! Eso es porque los helpers generan el atributo value de los controles que ya establece su valor inicial. Luego una vez se envía el formulario entra en acción el ModelBinder que rellena el viewmodel de ASP.NET MVC a partir de los datos del POST. Knockout no hace nada ahí.
                                                      </p>
                                                      
                                                      <p>
                                                        Así pues si nos limitamos a enviar (submit) los datos introducidos en unos campos de texto que están dentro de un formulario, knockout no pinta nada. El binding de knockout es en cliente. Knockout lo usaríamos si &ldquo;antes&rdquo; de enviar los datos queremos hacer algos con ellos <em>en cliente</em>. Así pues vamos a probar de hacer algo con los datos en cliente.
                                                      </p>
                                                      
                                                      <p>
                                                        P.ej. mostrarlos (con un alert). Para ello asignamos un evento javascript en el submit del formulario:
                                                      </p>
                                                      
                                                      <div style="overflow: auto; cursor: text; font-size: 8pt; font-family: 'Courier New', courier, monospace; direction: ltr; text-align: left; margin: 20px 0px 10px; line-height: 12pt; max-height: 200px; width: 97.5%; background-color: #f4f4f4; border: silver 1px solid; padding: 4px;" id="codeSnippetWrapper">
                                                        <div style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;" id="codeSnippet">
                                                          <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum1">   1:</span> $(<span style="color: #006080">"form"</span>).submit(<span style="color: #0000ff">function</span>(evt) {</pre>
                                                          
                                                          <p>
                                                            <!--CRLF-->
                                                          </p>
                                                          
                                                          <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum2">   2:</span>     alert(vm.Name + <span style="color: #006080">" "</span> + vm.Ibu);</pre>
                                                          
                                                          <p>
                                                            <!--CRLF-->
                                                          </p>
                                                          
                                                          <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum3">   3:</span> });</pre>
                                                          
                                                          <p>
                                                            <!--CRLF--></div> </div> 
                                                            
                                                            <p>
                                                              Y lo probamos... ¿Creéis que funcionará? La respuesta en la imagen siguiente:
                                                            </p>
                                                            
                                                            <p>
                                                              <a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_54617C8E.png"><img height="165" width="504" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_041F250E.png" alt="image" border="0" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" title="image" /></a>
                                                            </p>
                                                            
                                                            <p>
                                                              Como era de esperar, funciona porque konockout modifica el viewmodel a partir de los datos de los controles. Recuerda que luego cuando enviamos el formulario, enviamos los datos que están en los controles (no usamos el viewmodel de knockout para nada). &iexcl;Pero ahora sabemos que los tenemos sincronizados!
                                                            </p>
                                                            
                                                            <p>
                                                              Ahora hagamos una cosilla... En el textbox de Ibu (que es un int) introduzcamos algo que no sea numérico, p. ej. &ldquo;Pepe&rdquo;. Y eso es lo que obtenemos de vuelta:
                                                            </p>
                                                            
                                                            <p>
                                                              <a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_6C87779A.png"><img height="191" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_35AD2354.png" alt="image" border="0" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-top-width: 0px" title="image" /></a>
                                                            </p>
                                                            
                                                            <p>
                                                              ¿Qué os parece? ¿Os gusta? <strong>A mi no.</strong> ¿Por que no me gusta esto?
                                                            </p>
                                                            
                                                            <p>
                                                              Pues muy fácil: <em>he perdido el valor incorrecto que había entrado</em>. Y es que ese es un comportamiento de los helpers de asp.net mvc: si entro algún valor incorrecto <em>se preserva</em>. Si queréis hacer la prueba basta con comentar la línea ko.applyBindings y lo veréis:
                                                            </p>
                                                            
                                                            <p>
                                                              <a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_2C53EED1.png"><img height="144" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_329AC55F.png" alt="image" border="0" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-top-width: 0px" title="image" /></a>
                                                            </p>
                                                            
                                                            <p>
                                                              Este comportamiento es por diseño y es propio de los helpers de edición. ¿Como es que al usar knockout perdemos el valor &ldquo;pepe&rdquo; incorrecto y se nos sustituye por un 0? Pues muy sencillo:
                                                            </p>
                                                            
                                                            <ol>
                                                              <li>
                                                                El helper genera el código de forma correcta, con el valor del atributo value a &ldquo;pepe&rdquo;.
                                                              </li>
                                                              <li>
                                                                El viewmodel de ASP.NET MVC pasado a la vista tiene un 0 en la propiedad Ibu (no hay manera de convertir &ldquo;pepe&rdquo; a int, así que el ModelBinder no hace nada (y el valor por defecto de un int es 0) y deja un error en el ModelState).
                                                              </li>
                                                              <li>
                                                                Al crear el viewmodel de knockout lo creamos a partir del viewmodel de ASP.NET MVC que tiene un 0 en la propiedad Ibu.
                                                              </li>
                                                              <li>
                                                                Al llamar a ko.applyBindings() modificamos el valor inicial del control (que era &ldquo;pepe&rdquo;) por el valor del viewmodel de knockout (que es 0).
                                                              </li>
                                                            </ol>
                                                            
                                                            <p>
                                                              Por lo tanto &ldquo;perdemos&rdquo; esta capacidad de los helpers de mantener el dato incorrecto entrado (si este no era asignable al viewmodel de ASP.NET MVC).
                                                            </p>
                                                            
                                                            <p>
                                                              Si has leído los posts anteriores de esta serie, probablemente habrás levantado una ceja cuando creábamos el viewmodel de knockout a partir de la cadena json obtenida de serializar el viewmodel de ASP.NET MVC. Por qué? Bueno... modifiquemos la vista para que el titulo sea así:
                                                            </p>
                                                            
                                                            <div style="overflow: auto; cursor: text; font-size: 8pt; height: 36px; font-family: 'Courier New', courier, monospace; direction: ltr; text-align: left; margin: 20px 0px 10px; line-height: 12pt; max-height: 200px; width: 97.5%; background-color: #f4f4f4; border: silver 1px solid; padding: 4px;" id="codeSnippetWrapper">
                                                              <div style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;" id="codeSnippet">
                                                                <pre style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: white; border-style: none; padding: 0px;"><span style="color: #606060" id="lnum1">   1:</span> &lt;legend&gt;Editando: &lt;span data-bind=<span style="color: #006080">"text: Name"</span>&gt;&lt;/span&gt;&lt;/legend&gt;</pre>
                                                                
                                                                <p>
                                                                  <!--CRLF--></div> </div> 
                                                                  
                                                                  <p>
                                                                    Ahora la idea es que al modificar el textbox que contiene el nombre de la cerveza, se modifique el título, pero eso no ocurre:
                                                                  </p>
                                                                  
                                                                  <p>
                                                                    <a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_62586DDE.png"><img height="145" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_4479E9DD.png" alt="image" border="0" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; border-left: 0px; display: inline; padding-right: 0px" title="image" /></a>
                                                                  </p>
                                                                  
                                                                  <p>
                                                                    He modificado el textbox pero en el título sigue apareciendo el nombre anterior. La razón? Pues muy sencillo, el viewmodel de knockout es un objeto javascript plano. <strong>No tiene definido ningún observable</strong>.
                                                                  </p>
                                                                  
                                                                  <p>
                                                                    Es posible crear observables de forma relativamente sencilla, a partir del viewmodel de ASP.NET MVC? La respuesta es que sí, pero para ello debemos usar un plugin de knockout (knockout mappings). No lo veremos en este post (sí en alguno futuro), quedaros con la idea de que <em>se puede hacer de forma &ldquo;relativamente&rdquo; sencilla</em>.
                                                                  </p>
                                                                  
                                                                  <p>
                                                                    Resumiendo: en ediciones simples (de un solo elemento simultaneo) podemos usar los helpers de asp.net mvc, aunque con las salvedades vistas en este punto. Tu mismo debes decidir si te compensa o no usarlos.
                                                                  </p>
                                                                  
                                                                  <p>
                                                                    <strong>Mi opinión:</strong> Los helpers de ASP.NET MVC existen para dar solución a un problema concreto. La verdad es que knockout, realmente, da solución al mismo problema pero lo hace desde una aproximación radicalmente distinta. Intentar aprovechar los helpers de ASP.NET MVC junto con knockout es posible, pero <strong>no es algo que yo haría</strong>. Recuerda que los helpers están para ayudar, en ningún caso estamos obligados a usarlos!
                                                                  </p>
                                                                  
                                                                  <p>
                                                                    Si no te sientes a gusto creando &ldquo;a mano&rdquo; controles HTML (&iexcl;deberías sentirte a gusto con ello!) y quieres una &ldquo;aproximación tipo helpers MVC&rdquo; para knockout pues la solución pasa por implementarte tus propios helpers...
                                                                  </p>
                                                                  
                                                                  <p>
                                                                    Bueno... dejemos de divagar por hoy!
                                                                  </p>
                                                                  
                                                                  <p>
                                                                    Un saludo!!!!
                                                                  </p>

 [1]: /blogs/etomas/archive/tags/knockout/default.aspx