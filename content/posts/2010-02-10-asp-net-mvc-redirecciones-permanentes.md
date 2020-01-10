---
title: 'ASP.NET MVC: Redirecciones permanentes'

author: eiximenis

date: 2010-02-10T09:54:00+00:00
geeks_url: /?p=1495
geeks_visits:
  - 4010
geeks_ms_views:
  - 2396
categories:
  - Uncategorized

---
Buenas! Los que estéis al tanto de las novedades de ASP.NET 4, sabreis que una de ella es <a target="_blank" href="http://msdn.microsoft.com/en-us/library/system.web.httpresponse.redirectpermanent(VS.100).aspx" rel="noopener noreferrer">Response.RedirectPermanent</a> (de la cual Ibon habla <a target="_blank" href="/blogs/ilanda/archive/2010/01/26/m-233-todo-response-redirectpermanent.aspx" rel="noopener noreferrer">un poco en este post</a>). La diferencia con respecto a Response.Redirect es que esta emite un código <a target="_blank" href="http://en.wikipedia.org/wiki/HTTP_302" rel="noopener noreferrer">HTTP 302 (Found)</a> mientras que RedirectPermanent emite un código <a target="_blank" href="http://en.wikipedia.org/wiki/HTTP_301" rel="noopener noreferrer">HTTP 301 (Moved Permanently)</a>.

<!--more-->

A efectos del usuario final el resultado es exactamente el mismo: cuando el navegador recibe un HTTP 301 o bien un HTTP 302, realiza _otra_ petición a la URL especificada en el header &ldquo;Location&rdquo;, con lo cual se consigue el objetivo final: que el usuario sea redirigido a otra página.

¿Entonces? Bueno, como vivimos en un mundo dominado por las máquinas y por el software, ahora resulta que no nos basta con contentar al usuario: también debemos contentar a... Google y similares. Aunque para el usuario final un HTTP 302 y un HTTP 301 sean lo mismo (una redirección) los buscadores los tratan de forma muy distinta. Cuando Google realiza una petición a una URL digamos A, y recibe un código HTTP 302 que le redirecciona a B, básicamente Google _sigue manteniendo en su índice_ la página A, puesto que un código HTTP 302 significa, en el fondo, que la redirección es _temporal_. Por otro lado, cuando Google recibe un HTTP 301, _elimina_ la página A de su índice y en su lugar guarda el resultado de la redirección...

... en el fondo lo importante de este rollo es que **a efectos de SEO a Google no le gustan los HTTP 302**. En este post del blog de Matt Cutts hay <a target="_blank" href="http://www.mattcutts.com/blog/seo-advice-discussing-302-redirects/" rel="noopener noreferrer">más información sobre 302 vs 301</a>.

En ASP.NET MVC tenemos varias maneras de realizar redirecciones, la más común de la cual es llamar al método _RedirectToAction_ que devuelve un objeto _RedirectToRouteResult_ configurado para redirigir al usuario a la acción especificada del controlador indicado... El tema está en que RedirectToRouteResult utiliza el código HTTP 302 para realizar la redirección (en el fondo termina llamando a Response.Redirect).

Imaginad que tengo en el controlador _Home_ la acción _Index_ que me redirecciona a la acción _Destination_, que es la que me muestra una vista:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> ActionResult Index()<br />{<br />    <span style="color: #0000ff">return</span> RedirectToAction(<span style="color: #006080">"Destination"</span>);<br />}<br /><br /><span style="color: #0000ff">public</span> ActionResult Destination()<br />{<br />    <span style="color: #0000ff">return</span> View();<br />}</pre>
  
  <p>
    </div> 
    
    <p>
      Si abro la url /Home/Index, evidentemento soy redirigido a Home/Destination, pero si observo las peticiones del navegador (en mi caso uso firefox + firebug):
    </p>
    
    <p>
      <a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_036867E3.png"><img height="147" width="501" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_7CD4E852.png" alt="image" border="0" title="image" style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" /></a>
    </p>
    
    <p>
      Observad como hay dos peticiones: la primera devuelve el código HTTP 302 y el campo Location con la URL a la que el navegador debe redirigirse. La segunda petición devuelve un HTTP 200 (código para indicar que todo ha ido OK y que mandamos la respuesta al navegador).
    </p>
    
    <p>
      Ahora el quid de la cuestión: como podemos realizar redirecciones permanentes en ASP.NET MVC? Pues hasta donde <strong>yo sé</strong>, no es posible directamente, pero por suerte, dado que el framework es bastante extensible no nos va a costar nada añadir dicha posibilidad... vamos a ello!
    </p>
    
    <p>
      <strong>1. Creación de un ActionResult propio</strong>
    </p>
    
    <p>
      El primer paso es crearnos un ActionResult propio... dado que el <em>RedirectToRouteResult</em> no nos sirve, ya que termina usando Response.Redirect, vamos a crearnos uno de propio, que he llamado (en un alarde de originalidad) <em>PermanentRedirectToRouteResult</em>.
    </p>
    
    <p>
      Esta clase tendrá un constructor con cuatro parámetros: acción, controlador, el nombre de la ruta a usar y los valores de la ruta. De hecho sólo el primero (acción) es imprescindible. El resto son &ldquo;avanzados&rdquo; y sirven para redirigirnos a acciones de <em>otros</em> controladores o bien para pasar parámetros. La definición inicial de la clase es simple:
    </p>
    
    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> PermanentRedirectToRouteResult : ActionResult<br />{<br />    <span style="color: #0000ff">private</span> RouteCollection _routes;<br />    <br />    <span style="color: #0000ff">public</span> PermanentRedirectToRouteResult(<span style="color: #0000ff">string</span> action, <span style="color: #0000ff">string</span> controller, <span style="color: #0000ff">string</span> routeName, RouteValueDictionary routeValues)<br />    {<br />        Action = action;<br />        Controller = controller;<br />        RouteName = routeName ?? String.Empty;<br />        RouteValues = routeValues ?? <span style="color: #0000ff">new</span> RouteValueDictionary();<br />    }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> RouteName {<br />        get;<br />        <span style="color: #0000ff">private</span> set;<br />    }<br />    <span style="color: #0000ff">public</span> RouteValueDictionary RouteValues {<br />        get;<br />        <span style="color: #0000ff">private</span> set;<br />    }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Action { get; <span style="color: #0000ff">private</span> set; }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Controller { get; <span style="color: #0000ff">private</span> set; }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">override</span> <span style="color: #0000ff">void</span> ExecuteResult(ControllerContext context)<br />    {<br />    }<br />}</pre>
      
      <p>
        </div> 
        
        <p>
          Guay! Solo nos falta rellenar el método ExecuteResult para hacer lo que queramos (en este caso una redirecciónmediante HTTP 301). Para ello, primero calculamos la URL de redirección (usando la clase UrlHelper) y luego establecemos el código HTTP en 301 y el campo Location de la Response:
        </p>
        
        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">override</span> <span style="color: #0000ff">void</span> ExecuteResult(ControllerContext context)<br />{<br />    <span style="color: #0000ff">string</span> url = UrlHelper.GenerateUrl(RouteName, Action, Controller,<br />        <span style="color: #0000ff">new</span> RouteValueDictionary(), RouteTable.Routes, context.RequestContext, <span style="color: #0000ff">false</span>);<br />    context.HttpContext.Response.StatusCode = 301;<br />    context.HttpContext.Response.RedirectLocation = url;<br />}</pre>
          
          <p>
            </div> 
            
            <p>
              No uso Response.RedirectPermanent porque si no mi código sólo seria compatible con ASP.NET 4 (es decir con VS2010)... de esta manera mi código sigue siendo compatible con VS2008 🙂
            </p>
            
            <p>
              <strong>2. Un par de métodos de extensión...</strong>
            </p>
            
            <p>
              Este punto simplemente es para que podamos hacer algo parecido a lo que hacemos con RedirectToAction: llamar a a un método que es quien crea el objeto RedirectToRouteResult. En mi caso, he creado métodos de extensión sobre la clase Controller:
            </p>
            
            <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
              <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">class</span> ControllerExtensions<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> PermanentRedirectToRouteResult PermanentRedirectToAction(<span style="color: #0000ff">this</span> Controller @<span style="color: #0000ff">this</span>,<br />        <span style="color: #0000ff">string</span> action)<br />    {<br />        <span style="color: #0000ff">return</span> PermanentRedirectToAction(@<span style="color: #0000ff">this</span>, action, <span style="color: #0000ff">null</span>);<br />    }<br /><br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> PermanentRedirectToRouteResult PermanentRedirectToAction(<span style="color: #0000ff">this</span> Controller @<span style="color: #0000ff">this</span>,<br />        <span style="color: #0000ff">string</span> action, <span style="color: #0000ff">string</span> controller)<br />    {<br />        <span style="color: #0000ff">return</span> PermanentRedirectToAction(@<span style="color: #0000ff">this</span>, action, controller, <span style="color: #0000ff">null</span>, <span style="color: #0000ff">null</span>);<br />    }<br /><br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> PermanentRedirectToRouteResult PermanentRedirectToAction(<span style="color: #0000ff">this</span> Controller @<span style="color: #0000ff">this</span>,<br />        <span style="color: #0000ff">string</span> action, <span style="color: #0000ff">string</span> controller, <span style="color: #0000ff">string</span> routeName, RouteValueDictionary values)<br />    {<br />        <span style="color: #0000ff">return</span> <span style="color: #0000ff">new</span> PermanentRedirectToRouteResult(action, controller, routeName, values);<br />    }<br />}</pre>
              
              <p>
                </div> 
                
                <p>
                  Mola! Ahora ya podemos hacer nuestras redirecciones permanentes de una forma muy similar a como hacemos las normales. P.ej. puedo añadir la siguiente acción a mi controlador:
                </p>
                
                <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> ActionResult IndexPermanent()<br />{<br />    <span style="color: #0000ff">return</span> <span style="color: #0000ff">this</span>.PermanentRedirectToAction(<span style="color: #006080">"Destination"</span>);<br />}</pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Y si ahora abro la URL /Home/IndexPermanent, observo que he sido redirigido a Home/Destination, pero ahora con el código HTTP 301:
                    </p>
                    
                    <p>
                      <a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_4683C043.png"><img height="101" width="353" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_32F660A2.png" alt="image" border="0" title="image" style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" /></a>
                    </p>
                    
                    <p>
                      Y listos! Ya lo tenemos... ¿veis que fácil? 😀
                    </p>
                    
                    <p>
                      PD: He dejado <a target="_blank" href="http://cid-6521c259e9b1bec6.skydrive.live.com/self.aspx/BurbujasNet/ZipsPosts/MvcRedirectPermanent.zip" rel="noopener noreferrer">un zip con todo el código</a> (en mi skydrive).
                    </p>