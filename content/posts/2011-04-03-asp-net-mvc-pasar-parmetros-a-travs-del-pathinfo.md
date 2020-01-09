---
title: '[ASP.NET MVC] Pasar parámetros a través del PathInfo'
description: '[ASP.NET MVC] Pasar parámetros a través del PathInfo'
author: eiximenis

date: 2011-04-03T21:06:48+00:00
geeks_url: /?p=1565
geeks_visits:
  - 4711
geeks_ms_views:
  - 2094
categories:
  - Uncategorized

---
¡Muy buenas! Bueno, el título del post no queda demasiado claro, pero a ver si consigo explicar un poco la idea. 😉

Los que habéis usado ASP.NET MVC estáis muy acostumbradas a las URLs del estilo /controlador/accion/id, es decir algo como:

  * /Home/Index/10
  * /Articles/View/Eiximenis
  * /Blog/View/10293

Sabemos que gracias a la tabla de rutas podemos pasar tantos parámetros como queramos, y así podríamos tener URLs del tipo:

  * /Articles/View/Eiximenis/MVC/2011

Que podría devolverme los articulos de “Eiximenis” con el tag “MVC” y del año 2011. 

El único punto a tener presente es que _el orden de los parámetros importa_, es decir no es lo mismo /Articles/View/Eiximenis/MVC/2011 que /Articles/View/2011/MVC/Eiximenis. En el primer caso buscamos los artículos de Eiximenis sobre MVC en el 2011 y en el segundo caso buscaríamos los artículos del _blogger_ 2011, sobre MVC en el año de Eiximenis. Y sin duda [Fra Francesc Eiximenis][1], fue un gran escritor, pero que yo sepa todavía no se le ha dedicado un año (algo totalmente injusto, por supuesto :p).

En este artículo quiero enseñaros una manera para que podáis gestionar URLs del tipo:

  * /Articles/View/Author/Eiximenis/Tag/MVC/Year/2011
  * /Articles/View/Tag/MVC/Year/2011/Author/Eiximenis

Y que ambas URLs sean tratadas de forma idéntica. En este caso estaríamos pasando tres parámetros: Author, Tag y Year.

Para conseguir este efecto nos bastan dos acciones muy simples: definir un route handler nuevo y una entrada a la tabla de rutas.

El route handler lo único que debe hacer es recoger la información de la URL y parsearla en “tokens” (usando el ‘/’ como separador). Y por cada par de tokens añadir una entrada en los valores de ruta (route values). El código es muy simple:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> UrlRouteHandler : MvcRouteHandler<br />{<br />    <span style="color: #0000ff">protected</span> <span style="color: #0000ff">override</span> IHttpHandler GetHttpHandler(RequestContext requestContext)<br />    {<br />        <span style="color: #0000ff">var</span> path = requestContext.RouteData.Values[<span style="color: #006080">"pathInfo"</span>];<br />        <span style="color: #0000ff">if</span> (path != <span style="color: #0000ff">null</span>)<br />        {<br />            <span style="color: #0000ff">var</span> tokens = path.ToString().Split(<span style="color: #006080">'/'</span>);<br />            <span style="color: #0000ff">for</span> (<span style="color: #0000ff">var</span> idx =0; idx&lt;tokens.Length; idx+=2)<br />            {<br />                <span style="color: #0000ff">if</span> (idx+1 &lt; tokens.Length)<br />                {<br />                    requestContext.RouteData.Values.Add(tokens[idx], tokens[idx+1]);<br />                }<br />            }<br />        }<br /><br />        <span style="color: #0000ff">return</span> <span style="color: #0000ff">base</span>.GetHttpHandler(requestContext);<br />    } <br />}<br /></pre>
  
  <p>
    </div> 
    
    <p>
      Una pequeña nota es que la cadena que separamos en tokens, no es toda la URL sino “pathInfo” un parámetro de ruta que ya nos vendrá dado. Este parámetro de ruta contendrá todo aquello que no es ni el controlador ni la acción. Es decir en la URL /Articles/View/Author/Eiximenis/Tag/MVC/Year/2011 el valor de pathInfo será Author/Eiximenis/Tag/MVC/Year/2011 (que son justo los parámetros).
    </p>
    
    <p>
      Ahora nos queda añadir la entrada a la tabla de rutas. En mi ejemplo yo he eliminado la entrada “Default” que genera VS2010 y la he sustituído por:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">routes.Add(<span style="color: #006080">"Default"</span>, <span style="color: #0000ff">new</span> Route(url: <span style="color: #006080">"{controller}/{action}/{*pathInfo}"</span>,<br />    routeHandler: <span style="color: #0000ff">new</span> UrlRouteHandler(),<br />    defaults: <span style="color: #0000ff">new</span> RouteValueDictionary(<span style="color: #0000ff">new</span> {controller = <span style="color: #006080">"Home"</span>, action = <span style="color: #006080">"Index"</span>})));<br /></pre>
      
      <p>
        </div> 
        
        <p>
          La clave aquí está en el {*pathInfo}. Aquí le digo al sistema de rutas que coja todo lo que venga después de /{controller}/{action} y me lo añada a un parámetro de ruta llamado pathInfo. Además de eso, en esta ruta le indico que su routeHandler será una instancia de la clase UrlRouteHandler que hemos creado antes.
        </p>
        
        <p>
          Y listos! Una vez los datos están en el route value ya puede entrar en acción el sistema de binding de ASP.NET MVC lo que quiere decir que puedo crear un controlador como el siguiente:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> ArticlesController : Controller<br />{<br />    <span style="color: #0000ff">public</span> ActionResult View(<span style="color: #0000ff">string</span> author, <span style="color: #0000ff">string</span> tag, <span style="color: #0000ff">int</span>? year)<br />    {<br />        dynamic data = <span style="color: #0000ff">new</span> ExpandoObject();<br />        data.Author = author ?? <span style="color: #006080">"Sin formato"</span>;<br />        data.Tag = tag ?? <span style="color: #006080">"Sin confirmación"</span>;<br />        data.Year = year.HasValue ? year.ToString() : <span style="color: #006080">"Sin año"</span>;<br />        <span style="color: #0000ff">return</span> View(data);<br />    }<br />}<br /></pre>
          
          <p>
            </div> 
            
            <p>
              Que recibiría los parámetros de las URLs que hemos visto anteriormente.
            </p>
            
            <p>
              Un saludo a todos!
            </p>

 [1]: http://es.wikipedia.org/wiki/Francesc_Eiximenis