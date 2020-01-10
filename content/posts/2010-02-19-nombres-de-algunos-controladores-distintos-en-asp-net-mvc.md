---
title: Nombres de algunos controladores distintos en ASP.NET MVC

author: eiximenis

date: 2010-02-19T11:07:01+00:00
geeks_url: /?p=1497
geeks_visits:
  - 2168
geeks_ms_views:
  - 1406
categories:
  - Uncategorized

---
Hola! Un post para comentaros como he implementado una cosilla que necesitaba en ASP.NET MVC (v1). En concreto necesitaba mapear las URLs de tipo /api/{controller}/{action} al controlador especificado, pero con la salvedad de que el nombre del controlador empezaba por War. Es decir la URL /api/Foo/Index debía llamar a la acción del controlador WarFoo (en lugar del controlador Foo). 

<!--more-->

En resumen lo que quería era:

/Foo/Index –> Llamar a acción Index del controlador Foo

/api/Foo/Index –> llamar a acción Index del controlador WarFoo

/api/WarFoo/Index –> Llamar a acción Index del controlador WarFoo

La primera de las reglas se cumple fácilmente con la definición estándard de la tabla de rutas de MVC:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">routes.MapRoute(<br />    <span style="color: #006080">"Default"</span>,                           <br />    <span style="color: #006080">"{controller}/{action}/{id}"</span>,  <br />    <span style="color: #0000ff">new</span> { controller = <span style="color: #006080">"Home"</span>, action = <span style="color: #006080">"Index"</span>, id = <span style="color: #006080">""</span> }<br />);</pre>
  
  <p>
    </div> 
    
    <p>
      Para dar soporte a las otras dos reglas me he creado un RouteHandler propio…
    </p>
    
    <p>
      <strong>¿Que son los RouteHandlers?</strong>
    </p>
    
    <p>
      Los RouteHandlers son clases que implementan la interfaz <a href="http://msdn.microsoft.com/es-es/library/system.web.routing.iroutehandler.aspx" target="_blank" rel="noopener noreferrer">IRouteHandler</a> y que son las encargadas de procesar las peticiones que vienen via una ruta (casi todas en el caso de MVC). Generalmente un RouteHandler lo que hace es crear un IHttpHandler que procese la petición enrutada. ASP.NET MVC viene con un RouteHandler por defecto (MvcRouteHandler) que termina creando un IHttpHandler por defecto (MvcHandler) que es el que crea el controlador asociado y le cede el control. En mi caso el comportamiento de MvcHandler ya me va bien: no quiero cambiar la política de creación de controladores ni nada, solo quiero añadir el prefijo “War” al controlador.
    </p>
    
    <p>
      Por suerte, ASP.NET MVC es muy extensible y nos lo pone realmente fácil para añadir un RouteHandler propio: basta con crearlo y vincularlo con una ruta que tengamos en la tabla de rutas.
    </p>
    
    <p>
      Veamos primero como vincularíamos nuestro RouteHandler (que he llamado WarRouteHandler) con una ruta, usando el método Add de la tabla de rutas:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">routes.Add(<span style="color: #0000ff">new</span> Route(<span style="color: #006080">"api/{controller}/{action}/{id}"</span>,<br />    <span style="color: #0000ff">new</span> RouteValueDictionary(<br />        <span style="color: #0000ff">new</span> { controller = <span style="color: #006080">"Home"</span>, action = <span style="color: #006080">"Index"</span>, id = <span style="color: #006080">""</span> }),<br />    <span style="color: #0000ff">new</span> WarRouteHandler())<br />);</pre>
      
      <p>
        </div> 
        
        <p>
          El primer parámetro es la URL de la ruta, el segundo los valores de la ruta (a diferencia del método MapRoute que acepta un objeto anónimo para esto, aquí nosotros debemos crear específicamente el RouteValuesDictionary) y finalmente el RouteHandler a utilizar!
        </p>
        
        <p>
          <strong>Nota: </strong>Para que todo funcione bien, esta segunda ruta debe estar en la table de rutas <strong>antes</strong> que la ruta anterior. <a href="http://geeks.ms/blogs/etomas/archive/2010/02/19/nombres-de-algunos-controladores-distintos-en-asp-net-mvc-ii.aspx" target="_blank" rel="noopener noreferrer">Más detalles en el post continuación de este</a>!
        </p>
        
        <p>
          Ya casi estamos: Sólo nos falta crear el RouteHandler, para ello creamos una clase que implemente IRouteHandler, y en el método GetHttpHandler (que es el único) vamos a añadir “War” al nombre del controlador. Para saber el nombre del controlador nos basta con acceder a los valores de la ruta (RouteData.Values) donde están todos (controlador, acción, id y otros adicionales que hubiesen). El código es simple:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> IHttpHandler GetHttpHandler(RequestContext requestContext)<br />{<br />    <span style="color: #0000ff">string</span> controller = requestContext.RouteData.Values[<span style="color: #006080">"controller"</span>].ToString();<br />    <span style="color: #0000ff">if</span> (!controller.ToLowerInvariant().StartsWith(<span style="color: #006080">"war"</span>))<br />    {<br />        requestContext.RouteData.Values[<span style="color: #006080">"controller"</span>] = <span style="color: #0000ff">string</span>.Format(<span style="color: #006080">"War{0}"</span>, controller);<br />    }<br /><br />    <span style="color: #0000ff">return</span> <span style="color: #0000ff">new</span> MvcHandler(requestContext);<br />}</pre>
          
          <p>
            </div> 
            
            <p>
              Ahora ya tenemos las URLs de tipo /api/{controlador} enrutadas al controlador War{controlador}.
            </p>
            
            <p>
              Un saludo!
            </p>
            
            <p>
              <strong>Nota:</strong> Este post <em>no está completo</em>. Si te interesa el tema <a href="http://geeks.ms/blogs/etomas/archive/2010/02/19/nombres-de-algunos-controladores-distintos-en-asp-net-mvc-ii.aspx" target="_blank" rel="noopener noreferrer">lee la continuación</a>, puesto que no todo es tan fácil!
            </p>