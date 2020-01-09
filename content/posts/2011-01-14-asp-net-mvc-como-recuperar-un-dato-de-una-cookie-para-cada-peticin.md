---
title: 'ASP.NET MVC: Como recuperar un dato de una cookie para cada petición…'
description: 'ASP.NET MVC: Como recuperar un dato de una cookie para cada petición…'
author: eiximenis

date: 2011-01-14T13:40:08+00:00
geeks_url: /?p=1551
geeks_visits:
  - 3429
geeks_ms_views:
  - 2963
categories:
  - Uncategorized

---
EEhhhmm… bueno, no se me ocurre un título mejor. Este post nace gracias a un tweet de <a href="http://twitter.com/lluisfranco" target="_blank" rel="noopener noreferrer">Lluis Franco</a>. En el tweet Lluís preguntaba <a href="http://twitter.com/lluisfranco/status/25846781329801216" target="_blank" rel="noopener noreferrer">dónde guardar la cultura de una aplicación MVC</a> si no se podía poner en la URL. Después de varios tweets comentando algunas cosillas yo he respondido diciendo que veía <a href="http://twitter.com/eiximenis/status/25849807952154624" target="_blank" rel="noopener noreferrer">dos opciones: o en una cookie o en la base de datos</a>. Una de las cosas que más me gustan de HTTP es que es simple: no hay muchas maneras de pasar estado entre cliente y servidor 😉

En este post vamos a ver como podemos solucionar fácilmente el problema asumiendo que se guarda la cultura del usuario en una cookie.

De mi tweet, la parte importante es la segunda: independizar a los controladores de donde esta la cultura de la aplicación. Ya lo he comentado en varios posts: evitad acceder desde los controladores a objetos que dependen de HTTP: sesión, aplicación, cache y… cookies.

En un post anterior ya comenté <a href="http://geeks.ms/blogs/etomas/archive/2010/07/09/asp-net-mvc-q-amp-a-191-c-243-mo-se-usan-las-cookies.aspx" target="_blank" rel="noopener noreferrer">como usar un value provider para hacer binding de datos de una cookie a un controlador</a>. Esa es una buena solución si los datos de la cookie se usan en algunas pocas acciones de un controlador. Pero ese no es nuestro caso ahora: ahora queremos que la cultura se establezca siempre, para **todas** las acciones de **todos** los controladores.

La solución en este caso pasa por un _Route Handler_ nuevo. Los route handlers son los objetos que se encargan de procesar las rutas (crear los controladores y cederles el control). Son pues objetos _de bajo nivel_. Cuando la tabla de rutas enruta una URL para ser procesada por una ruta concreta, se usa el RouteHandler asociado a dicha ruta para crear toda la infrastructura que MVC necesita para procesar la petición.

Recordad que la tabla de rutas se define en Global.asax y que por defecto tiene el siguiente código:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">void</span> RegisterRoutes(RouteCollection routes)<br />{<br />    routes.IgnoreRoute(<span style="color: #006080">"{resource}.axd/{*pathInfo}"</span>);<br />    routes.MapRoute(<br />        <span style="color: #006080">"Default"</span>, <span style="color: #008000">// Route name</span><br />        <span style="color: #006080">"{controller}/{action}/{id}"</span>, <span style="color: #008000">// URL with parameters</span><br />        <span style="color: #0000ff">new</span> { controller = <span style="color: #006080">"Home"</span>, action = <span style="color: #006080">"Index"</span>, id = UrlParameter.Optional } <span style="color: #008000">// Parameter defaults</span><br />    );<br />}</pre>
  
  <p>
    </div> 
    
    <p>
      Aquí no estamos especificando ningún route handler, por lo que se usará el que tiene MVC por defecto… Pero como (casi) todo en MVC lo podemos cambiar 🙂
    </p>
    
    <p>
      En lugar de usar el método MapRoute (que por si alguien no lo sabe es un método de extensión) podemos crear un objeto Route y añadirlo directamente a la tabla de rutas. El constructor de Route tiene un parámetro que es de tipo <a href="http://msdn.microsoft.com/es-es/library/system.web.routing.iroutehandler.aspx" target="_blank" rel="noopener noreferrer">IRouteHandler</a> y que es el route handler para esta ruta. Así que puedo transformar la tabla de rutas anterior en esta:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">void</span> RegisterRoutes(RouteCollection routes)<br />{<br />    routes.IgnoreRoute(<span style="color: #006080">"{resource}.axd/{*pathInfo}"</span>);<br /><br />    routes.Add(<span style="color: #006080">"Default"</span>, <span style="color: #0000ff">new</span> Route(<span style="color: #006080">"{controller}/{action}/{id}"</span>, <br />        <span style="color: #0000ff">new</span> CultureRouteHandler())<br />                {<br />                    Defaults = <span style="color: #0000ff">new</span> RouteValueDictionary(<br />                        <span style="color: #0000ff">new</span><br />                            {<br />                                controller = <span style="color: #006080">"Home"</span>, action = <span style="color: #006080">"Index"</span>, id = UrlParameter.Optional<br />                            })<br />                });<br />}<br /></pre>
      
      <p>
        </div> 
        
        <p>
          Ambas son equivalentes, salvo que esta usará un objeto de tipo <em>CultureRouteHandler</em> para procesar las peticiones.
        </p>
        
        <p>
          Ahora vamos a ver como es el CultureRouteHandler:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> CultureRouteHandler : MvcRouteHandler<br />{<br />    <span style="color: #0000ff">protected</span> <span style="color: #0000ff">override</span> IHttpHandler GetHttpHandler(System.Web.Routing.RequestContext requestContext)<br />    {<br />        var cultureCookieVal = GetCultureFromCookie(requestContext.HttpContext.Request.Cookies);<br />        var culture = <span style="color: #0000ff">new</span> CultureInfo(cultureCookieVal);<br />        requestContext.RouteData.Values.Add(<span style="color: #006080">"culture"</span>, cultureCookieVal);<br />        Thread.CurrentThread.CurrentCulture = culture;<br />        Thread.CurrentThread.CurrentUICulture = culture;<br />        <span style="color: #0000ff">return</span> <span style="color: #0000ff">base</span>.GetHttpHandler(requestContext);<br />    }<br /><br />    <span style="color: #0000ff">private</span> <span style="color: #0000ff">string</span> GetCultureFromCookie(HttpCookieCollection cookies)<br />    {<br />        var retValue = <span style="color: #006080">"ca-ES"</span>;<br />        <span style="color: #0000ff">if</span> (cookies.AllKeys.Contains(<span style="color: #006080">"userculture"</span>))<br />        {<br />            retValue = cookies[<span style="color: #006080">"userculture"</span>].Value;<br />        }<br />        <span style="color: #0000ff">return</span> retValue;<br />    }<br />}</pre>
          
          <p>
            </div> 
            
            <p>
              En este caso derivo de MvcRouteHandler (como casi siempre en MVC es mucho más sencillo derivar de alguna clase base que implementar la interfaz entera), y en el método <em>GetHttpHandler</em> lo que hago es llamar al método de la clase base pero <strong>antes</strong>:
            </p>
            
            <ol>
              <li>
                Recupero el valor de la cookie de cultura
              </li>
              <li>
                Guardo este valor en el route data con el nombre culture (por si alguien quiere consultarlo)
              </li>
              <li>
                Creo un CultureInfo a partir de los datos de la cookie y establezco la cultura del thread actual a este valor: así cualquier mecanismo que tenga de “localización” debería funcionar igualmente.
              </li>
            </ol>
            
            <p>
              Finalmente para probar el tema me he creado un pequeño controlador:
            </p>
            
            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> HomeController : Controller<br />{<br />    [OutputCache(NoStore = <span style="color: #0000ff">true</span>, Location = OutputCacheLocation.None)]<br />    <span style="color: #0000ff">public</span> ActionResult Index()<br />    {<br />        <span style="color: #0000ff">return</span> View();<br />    }<br /><br />    <span style="color: #0000ff">public</span> ActionResult SetCookie(<span style="color: #0000ff">string</span> id)<br />    {<br />        <span style="color: #0000ff">if</span> (!<span style="color: #0000ff">string</span>.IsNullOrEmpty(id))<br />        {<br />            <span style="color: #0000ff">this</span>.ControllerContext.HttpContext.Response.Cookies.Add(<span style="color: #0000ff">new</span> HttpCookie(<span style="color: #006080">"userculture"</span>, id));<br />        }<br />        <span style="color: #0000ff">return</span> RedirectToAction(<span style="color: #006080">"Index"</span>);<br />    }<br />}</pre>
              
              <p>
                </div> 
                
                <p>
                  La acción /Home/Index simplemente retorna una vista. La acción /Home/SetCookie/id establece la cookie de cultura (se supone que el id es válido, algo así como /Home/SetCookie/es-ES p.ej.).
                </p>
                
                <p>
                  La vista que devuelve /Home/Index simplemente muestra la cultura actual:
                </p>
                
                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">&lt;</span><span style="color: #800000">p</span><span style="color: #0000ff">&gt;</span><br />    Cultura actual: @System.Threading.Thread.CurrentThread.CurrentUICulture.ToString();<br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">p</span><span style="color: #0000ff">&gt;</span></pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      <strong>Bonus track:</strong> Y si quiero que algún controlador <em>reciba</em> la cultura actual como parámetro de alguna de sus acciones?
                    </p>
                    
                    <p>
                      Bien, recordad que hemos hecho que el route handler guardase el valor de cultura en los route values. MVC tiene un value provider que permite realizar bindings desde los route values hacia los controladores. Guardábamos el valor con el nombre “culture” así que nos basta con:
                    </p>
                    
                    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> ActionResult Foo(<span style="color: #0000ff">string</span> culture)<br />{<br />    <span style="color: #008000">// Código...</span><br />}</pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          El parámetro <em>culture</em> tiene el valor de la cultura.
                        </p>
                        
                        <p>
                          Si quieres saber exactamente <a href="http://geeks.ms/blogs/etomas/archive/2010/07/02/asp-net-mvc-q-amp-a-191-como-reciben-par-225-metros-los-controladores.aspx" target="_blank" rel="noopener noreferrer">cómo reciben los datos los controladores, hace algún tiempecillo escribí un post al respecto</a>.
                        </p>
                        
                        <p>
                          De esa manera conseguimos lo que yo decía en mi tweet: agnostizar los controladores de <em>dónde</em> se guarda la cultura!
                        </p>
                        
                        <p>
                          Un saludo!
                        </p>