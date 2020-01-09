---
title: 'ASP.NET MVC: Como recuperar un dato de una cookie para cada petición… Una alternativa ¿igual?'
description: 'ASP.NET MVC: Como recuperar un dato de una cookie para cada petición… Una alternativa ¿igual?'
author: eiximenis

date: 2011-01-19T11:04:53+00:00
geeks_url: /?p=1553
geeks_visits:
  - 2189
geeks_ms_views:
  - 2707
categories:
  - Uncategorized

---
Muy buenas! Hace algunos días escribí el post <a href="http://geeks.ms/blogs/etomas/archive/2011/01/14/asp-net-mvc-como-recuperar-un-dato-de-una-cookie-para-cada-petici-243-n.aspx" target="_blank" rel="noopener noreferrer">ASP.NET MVC: Como recuperar datos de una cookie en cada petición</a>, donde mostraba el uso de un _route handler_ propio para recuperar los datos de una cookie y colocarlos en el Route Data. En el ejemplo era una cookie de cultura de la aplicación, pero se puede aplicar a lo que queráis.

Lo que más me gusta de ASP.NET MVC es que muy expandible, que muchas cosas pueden hacerse de más de una forma. Pues bien, una de las novedades más interesantes de MVC3 (al margen de Razor) son los action filters **globales**.

En este post os propongo una **solución alternativa** (aunque ya veremos que tiene una _ligerísima_ diferencia) al mismo problema. La diferencia es que **no se debe alterar la tabla de rutas para nada**. Y dicha solución pasa por usar un action filter global.

Una de las cosas que en MVC nos debe quedar claro es que _cuando repitamos muchas veces un mismo código_ _de un controlador_ debemos considerar de ponerlo en un Action Filter. El “problema” está que los action filters deben aplicarse controlador a controlador (o acción a acción). Si tenemos un filtro que debe aplicarse a todos los controladores podemos considerar crear una clase base que lo tenga y heredar todos los controladores de ella…

… o al menos eso era así _antes_ de MVC3.

Con MVC3 y los filtros globales podemos aplicar un filtro _a todas las acciones de todos los controladores_. Y todo ello con **una sola línea en global.asax.** Es brutal!

**El filtro global…**

Lo bueno es que los filtros globales **se implementan igual** que los filtros no globales clásicos que teníamos en MVC2. En este caso la implementación es super sencilla:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> CookieCultureFilterAttribute : ActionFilterAttribute<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">override</span> <span style="color: #0000ff">void</span> OnActionExecuting(ActionExecutingContext filterContext)<br />    {<br />        var cultureCookieVal = GetCultureFromCookie(filterContext.HttpContext.Request.Cookies);<br />        var culture = <span style="color: #0000ff">new</span> CultureInfo(cultureCookieVal);<br />        filterContext.RouteData.Values.Add(<span style="color: #006080">"culture"</span>, cultureCookieVal);<br />        Thread.CurrentThread.CurrentCulture = culture;<br />        Thread.CurrentThread.CurrentUICulture = culture;<br />    }<br /><br />    <span style="color: #0000ff">private</span> <span style="color: #0000ff">string</span> GetCultureFromCookie(HttpCookieCollection cookies)<br />    {<br />        var retValue = <span style="color: #006080">"ca-ES"</span>;<br />        <span style="color: #0000ff">if</span> (cookies.AllKeys.Contains(<span style="color: #006080">"userculture"</span>))<br />        {<br />            retValue = cookies[<span style="color: #006080">"userculture"</span>].Value;<br />        }<br />        <span style="color: #0000ff">return</span> retValue;<br />    }<br />}</pre>
  
  <p>
    </div> 
    
    <p>
      El código es trivial: derivo de ActionFilterAttribute (clase que ya existía) y redefino el método <em>OnActionExecuting</em> que se ejecuta <em>antes</em> de ejecutar la acción del controlador. En este método tengo el código para leer la cookie de cultura (exactamente lo mismo que tenía antes en el Route Handler).
    </p>
    
    <p>
      <strong>Activar el filtro global</strong>
    </p>
    
    <p>
      Os dije que era una sóla línea en global.asax, verdad? Pues concretamente esta:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">GlobalFilters.Filters.Add(<span style="color: #0000ff">new</span> CookieCultureFilterAttribute());</pre>
      
      <p>
        </div> 
        
        <p>
          Otra opción es colocar esa línea dentro de la función <em>RegisterGlobalFilters</em> que crea el VS2010, aunque entonces no es necesario usar la clase GlobalFilters (usad en su lugar el parámetro <em>filters</em>):
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">filters.Add(<span style="color: #0000ff">new</span> CookieCultureFilterAttribute());</pre>
          
          <p>
            </div> 
            
            <p>
              <strong>Y ya está</strong>. Nada más. Antes de cada acción de cualquier controlador se ejecutará el código de nuestro filtro global. <strong>No es necesario</strong> modificar la tabla de rutas para añadir nuestro route handler.
            </p>
            
            <p>
              <strong>¿Puedo usar el filtro en una aplicación MVC2?</strong>
            </p>
            
            <p>
              Si, si que puedes, pero entonces debes:
            </p>
            
            <ol>
              <li>
                Aplicarlo en cada controlador (usando [CookieCultureFilter] <em>antes</em> de cada acción o cada controlador que quieras que use la cookie).
              </li>
              <li>
                Derivar todos tus controladores de un controlador base que tenga [CookieCultureFilter] aplicado.
              </li>
            </ol>
            
            <p>
              <strong>Y lo más importante… ¿Ambas soluciones son equivalentes?</strong>
            </p>
            
            <p>
              Pues NO. Ambas soluciones no son equivalentes… En el caso del post anterior, si recordáis, si declaraba un parámetro <em>culture</em> en una acción, recibía el valor de la cookie, ya que el route handler me añadía este parámetro en el RouteData. Pues bien, <strong>eso</strong> dejará de funcionar. Es decir, en este caso la acción:
            </p>
            
            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> ActionResult Foo(<span style="color: #0000ff">string</span> culture)<br />{<br />}</pre>
              
              <p>
                </div> 
                
                <p>
                  que en el post anterior recibía el valor de la cookie en <em>culture</em>, con esa nueva aproximación siempre recibirá <em>null</em>.
                </p>
                
                <p>
                  ¿Y por que si mi filtro también añade en el RouteData el valor de la cookie? Pues muy sencillo: el Model Binder (que se encarga de hacer binding a los parámetros de las acciones) se ejecuta <strong>antes</strong> que el filtro. Simplificando, el flujo de ejecuciones sería:
                </p>
                
                <ol>
                  <li>
                    Route Handler
                  </li>
                  <li>
                    Model Binder
                  </li>
                  <li>
                    Action Filters
                  </li>
                  <li>
                    Acción del controlador
                  </li>
                </ol>
                
                <p>
                  En este caso, estamos añadiendo un valor en el RouteData <em>después</em> de que el Model Binder haya actuado. Por eso el parámetro no tendrá el valor. Eso no quita que desde el controlador lo podáis consultar (tenéis acceso al RouteData).
                </p>
                
                <p>
                  Y con esto termino… espero que el post os haya ayudado un poco más a entender como funciona MVC y ver distintas alternativas de cómo hacer las cosas!
                </p>