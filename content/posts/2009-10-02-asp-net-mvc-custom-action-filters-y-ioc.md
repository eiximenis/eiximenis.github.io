---
title: ASP.NET MVC Custom Action Filters y IoC
description: ASP.NET MVC Custom Action Filters y IoC
author: eiximenis

date: 2009-10-02T14:48:12+00:00
geeks_url: /?p=1470
geeks_visits:
  - 2773
geeks_ms_views:
  - 1170
categories:
  - Uncategorized

---
Que es una buena práctica usar un contenedor IoC hoy en día es algo que está más que aceptado… la gente que montó ASP.NET MVC lo tiene muy claro y por eso ha creado un framework, que aunque no usa ningún contenedor IoC por defecto, se puede _extender_ para usar uno… P.ej. si quieres que tus controladores sean creados a través de un contenedor IoC (y _deberías querrerlo_) solo debes crearte una factoría de controladores e indicar a ASP.NET MVC que use esta factoría en lugar de la que viene por defecto.

El siguiente código muestra una factoría de controladores que usa Unity:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #008000">/// &lt;summary&gt;</span><br /><span style="color: #008000">/// ControllerFactory que utilitza Unity per crear els controladors.</span><br /><span style="color: #008000">/// &lt;/summary&gt;</span><br /><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> UnityControllerFactory : DefaultControllerFactory<br />{<br />    IUnityContainer container;<br /><br />    <span style="color: #0000ff">public</span> UnityControllerFactory(IUnityContainer container)<br />    {<br />        <span style="color: #0000ff">this</span>.container = container;<br />    }<br /><br />    <span style="color: #0000ff">protected</span> <span style="color: #0000ff">override</span> IController GetControllerInstance (Type controllerType)<br />    {<br />        <span style="color: #0000ff">if</span> (controllerType == <span style="color: #0000ff">null</span>)<br />            <span style="color: #0000ff">throw</span> <span style="color: #0000ff">new</span> ArgumentNullException(<span style="color: #006080">"controllerType"</span>);<br /><br />        <span style="color: #0000ff">if</span> (!<span style="color: #0000ff">typeof</span>(IController).IsAssignableFrom(controllerType))<br />            <span style="color: #0000ff">throw</span> <span style="color: #0000ff">new</span> ArgumentException(<span style="color: #0000ff">string</span>.Format(<br />                <span style="color: #006080">"El tipus {0} NO és un controller."</span>, controllerType.Name),<br />                <span style="color: #006080">"controllerType"</span>);<br /><br />        <span style="color: #0000ff">return</span> <span style="color: #0000ff">this</span>.container.Resolve(controllerType) <span style="color: #0000ff">as</span> IController;<br />    }<br />}</pre>
  
  <p>
    </div> 
    
    <p>
      Indicar a ASP.NET MVC que use dicha factoría es tan fácil como meter lo siguiente en algún punto del Application_Start del Global.asax:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">IUnityContainer container = <span style="color: #0000ff">new</span> UnityContainer();<br />IControllerFactory factory = <span style="color: #0000ff">new</span> UnityControllerFactory(container);<br />ControllerBuilder.Current.SetControllerFactory(factory);</pre>
      
      <p>
        </div> 
        
        <p>
          Listos! Ahora nuestros controladores serán creados por Unity…
        </p>
        
        <p>
          Lamentablemente incluso un framework tan extensible como ASP.NET MVC a veces no es tan extensible como nos gustaría… Una de las capacidades más interesantes de ASP.NET MVC son los llamados filtros. Un filtro es una forma de inyectar código que se ejecuta <em>antes y/o después</em> de que se ejecute el código de una acción de un controlador. El propio framework viene con varios filtros, como p.ej. <em>Authorize</em> que impide ejecutar una acción de un controlador si el usuario no está autenticado. El uso de filtros es uno de los mecanismos más interesantes de ASP.NET MVC y permite la aplicación de técnicas de la programación orientada a aspectos.
        </p>
        
        <p>
          Seguro que si pensáis un poco os vienen a la cabeza muchas posibles ideas: P.ej. hacer un logging de todas las acciones, habilitar políticas de caching, gestionar los errores redirigiendo a una o varias páginas de error… Muchas de estas ideas (y más) ya vienen implementadas en el framework, pero lo bueno es que podemos crearnos <strong>nuestros</strong> propios filtros para crear nuestras propias tareas.
        </p>
        
        <p>
          Los filtros se aplican a las acciones mediante atributos, p.ej. en el caso del filtro para autorización, decoraríamos las acciones:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">[Authorize(Roles=<span style="color: #006080">"Admin"</span>)]<br /><span style="color: #0000ff">public</span> ActionResult Create()<br />{<br />    <span style="color: #0000ff">return</span> View();<br />}</pre>
          
          <p>
            </div> 
            
            <p>
              En este caso la acción sólo se ejecutará si el usuario está autenticado y además tiene el rol “Admin”.
            </p>
            
            <p>
              Crear un filtro propio no es especialmente complejo, basta derivar de FilterAttribute y adicionalmente implementar uno o varios de los siguientes interfaces:
            </p>
            
            <ul>
              <li>
                IAuthorizationFilter: Si debemos autorizar el uso o no de la acción en función de determinados parámetros
              </li>
              <li>
                IActionFilter: Para ejecutar lógica de negocio antes y después de la propia acción (el caso más común)
              </li>
              <li>
                IResultFilter: Para ejecutar lógica de negocio antes y después de la ejecución del resultado de la acción.
              </li>
              <li>
                IExcepcionFilter: Para gestionar errores.
              </li>
            </ul>
            
            <p>
              Se pueden implementar uno o varios de esos interfaces. La clase FilterAttribute deriva de Attribute por lo que para aplicar nuestro propio filtro, basta con decorar las acciones.
            </p>
            
            <p>
              Bueno… a lo que íbamos que me pierdo: si usáis un contenedor IoC para crear vuestros controladores seguramente desearéis usar el mismo contenedor IoC para instanciar vuestros filtros… Desgraciadamente esto no es tan directo como “crear una factoría de filtros” y registrarla, así que requiere un poco de más de trabajo. Os cuento dos técnicas para usar un contenedor IoC (en mi caso Unity) para vuestros filtros propios.
            </p>
            
            <p>
              <strong>Técnica 1: Tener el contenedor como variable de aplicación</strong>
            </p>
            
            <p>
              Esta es la más sencilla de todas: en global.asax, guardais el contenedor IoC en alguna variable de aplicación y luego en el filtro la recuperáis (por suerte tenemos acceso al HttpContext). En mi caso estoy implementando un IAuthorizationFilter, y en el código del método OnAuthorization tengo lo siguiente:
            </p>
            
            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> OnAuthorization(AuthorizationContext filterContext)<br />{<br />    IUnityContainer container =  filterContext.HttpContext.Application[<span style="color: #006080">"container"</span>] <span style="color: #0000ff">as</span> IUnityContainer;<br />    IGlobalSettings settings =  container.Resolve&lt;IGlobalSettings&gt;();<br />    <span style="color: #008000">// Código...</span><br />}</pre>
              
              <p>
                </div> 
                
                <p>
                  En este caso IGlobalSettings es el objeto registrado a Unity. Podemos acceder a Unity, porque lo tengo guardado en una variable de aplicación (en global.asax es donde lo guardo), y puedo acceder a las variables de aplicación a través del filterContext, que recibo como parámetro.
                </p>
                
                <p>
                  Todas las interfaces de los filtros reciben el filterContext <strong>excepto</strong> IActionFilter que recibe un ControllerContext, pero desde el ControllerContext también se puede acceder a las variables de aplicación.
                </p>
                
                <p>
                  <strong>Técnica 2: Usar un Custom ActionInvoker</strong>
                </p>
                
                <p>
                  Esta técnica consiste en extender el framework de ASP.NET MVC por uno de sus muchos puntos, y nos vamos a crear un custom ActionInvoker. El ActionInvoker es el encargado de “invocar” las acciones de los controladores, y por tanto es donde se crean los filtros asociadas a dichas acciones. Luego podemos indicar a los controladores que se creen que usen nuestro propio ActionInvoker.
                </p>
                
                <p>
                  En este caso nuestra propoa ActionInvoker lo que debe hacer es inyectar los valores a los filtros. Nunca podremos crear los filtros usando Unity ya que los filtros son atributos, y por tanto son “creados automáticamente” por el CLR. El código de nuestra ActionInvoker puede ser tal y como sigue:
                </p>
                
                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> UnityActionInvoker : ControllerActionInvoker<br />{<br />    <span style="color: #0000ff">private</span> IUnityContainer container;<br />    <span style="color: #0000ff">public</span> UnityActionInvoker(IUnityContainer container)<br />    {<br />        <span style="color: #0000ff">this</span>.container = container;<br />    }<br />    <span style="color: #0000ff">protected</span> <span style="color: #0000ff">override</span> FilterInfo GetFilters(ControllerContext controllerContext, ActionDescriptor actionDescriptor)<br />    {<br />        var filters = <span style="color: #0000ff">base</span>.GetFilters(controllerContext, actionDescriptor);<br />        <span style="color: #0000ff">foreach</span> (var filter <span style="color: #0000ff">in</span> filters.ActionFilters)<br />        {<br />            <span style="color: #0000ff">if</span> (!(filter <span style="color: #0000ff">is</span> IController)) container.BuildUp(filter.GetType(), filter);<br />        }<br />        <span style="color: #0000ff">foreach</span> (var filter <span style="color: #0000ff">in</span> filters.AuthorizationFilters)<br />        {<br />            <span style="color: #0000ff">if</span> (!(filter <span style="color: #0000ff">is</span> IController)) container.BuildUp(filter.GetType(), filter);<br />        }<br />        <span style="color: #0000ff">foreach</span> (var filter <span style="color: #0000ff">in</span> filters.ExceptionFilters)<br />        {<br />            <span style="color: #0000ff">if</span> (!(filter <span style="color: #0000ff">is</span> IController)) container.BuildUp(filter.GetType(), filter);<br />        }<br />        <span style="color: #0000ff">foreach</span> (var filter <span style="color: #0000ff">in</span> filters.ResultFilters)<br />        {<br />            <span style="color: #0000ff">if</span> (!(filter <span style="color: #0000ff">is</span> IController)) container.BuildUp(filter.GetType(), filter);<br />        }<br />        <span style="color: #0000ff">return</span> filters;<br />    }<br />}</pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      El código es bastante simple: el método que nos interesa se llama “GetFilters”. Este método es el encargado de obtener los filtros asociados a la acción de un controlador. Los filtros se devuelven en un objeto FilterInfo, que básicamente son cuatro colecciones para los distintos tipos de filtros que tenemos. Nuestro código se limita a recorrer dichas colecciones y por cada filtro ejecutar el método <em>BuildUp</em> de Unity que inyecta las dependencias. En este caso obviamente no podemos utilizar inyección de dependencias en el constructor (puesto que el objeto ya está creado), pero si podemos utilizar inyección de propiedades.
                    </p>
                    
                    <p>
                      El código de mi filtro ahora queda tal y como se ve:
                    </p>
                    
                    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">[Dependency]<br /><span style="color: #0000ff">public</span> IGlobalSettings Settings { get; set; }<br /><br /><span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> OnAuthorization(AuthorizationContext filterContext)<br />{<br />    <span style="color: #008000">// Código...</span><br />}</pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          La propiedad Settings es la que Unity inyectará a mi filtro cuando se llame al método BuildUp.
                        </p>
                        
                        <p>
                          Los más observadores habréis visto que en mi Custom ActionInvoker, cuando recorro el FilterInfo miro que el filtro no sea de tipo IController: los controladores <strong>son</strong> filtros de ASP.NET MVC, y además la clase Controller implementa los cuatro interfaces posibles de filtro (por lo que aparecerá UNA vez en cada colección de FilterInfo).
                        </p>
                        
                        <p>
                          Finalmente, nos queda indicar a cada controlador que use nuestra clase UnityActionInvoker. Para ello basta asignar la propiedad ActionInvoker de cada controlador. Esta propiedad es de la clase Controller (no del interfaz IController). Lo podemos hacer de tres maneras:
                        </p>
                        
                        <ol>
                          <li>
                            En el constructor de cada controlador (no muy recomendable por tedioso)
                          </li>
                          <li>
                            Derivar todos nuestros controladores de una clase base y hacerlo en el constructor de la clase base
                          </li>
                          <li>
                            Hacerlo en nuestra factoría de controladores (en el método GetControllerInstance):
                          </li>
                        </ol>
                        
                        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">protected</span> <span style="color: #0000ff">override</span> IController GetControllerInstance (Type controllerType)<br />{<br />    <span style="color: #008000">// Código anterior...</span><br />    controller.ActionInvoker = container.Resolve&lt;UnityActionInvoker&gt;();<br />    <span style="color: #0000ff">return</span> controller;<br />}</pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              Y listos! Ahora sí: nuestros filtros ya pueden usar propiedades inyectadas por Unity! 🙂
                            </p>
                            
                            <p>
                              <strong>Y ya por curiosidad…</strong>
                            </p>
                            
                            <p>
                              Igual alguno de vosotros se pregunta como era mi filtro: en mi caso estoy desarrollando una aplicación web que es (o será cuando los dioses quieran) un juego. Algunas de las acciones sólo pueden ser ejecutadas cuando el juego está en marcha (con independencia de si el usuario está autenticado o no). Podría poner un if() en cada una de las acciones pero… cuando veais que debeis poner un if() idéntico en muchas acciones, considerad el uso de un filtro.
                            </p>
                            
                            <p>
                              En mi caso el filtro lo que hace es redirigir a una página de error en caso de que el juego no esté en marcha y la acción requiera que el juego si que lo esté (o viceversa, ciertas acciones de administración sólo se deben poder realizar con el juego <em>parado</em>).
                            </p>
                            
                            <p>
                              El código del filtro es tal y como sigue:
                            </p>
                            
                            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> GameStartedAttribute : FilterAttribute, IAuthorizationFilter<br />{<br />    <span style="color: #0000ff">private</span> <span style="color: #0000ff">readonly</span> <span style="color: #0000ff">bool</span> started;<br /><br />    <span style="color: #0000ff">public</span> GameStartedAttribute()<br />    {<br />        <span style="color: #0000ff">this</span>.started = <span style="color: #0000ff">false</span>;<br />    }<br /><br />    <span style="color: #0000ff">public</span> GameStartedAttribute(<span style="color: #0000ff">bool</span> started)<br />    {<br />        <span style="color: #0000ff">this</span>.started = started;<br />    }<br /><br />    [Dependency]<br />    <span style="color: #0000ff">public</span> IGlobalSettings Settings { get; set; }<br /><br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> OnAuthorization(AuthorizationContext filterContext)<br />    {<br />        IUnityContainer container =  filterContext.HttpContext.Application[<span style="color: #006080">"container"</span>] <span style="color: #0000ff">as</span> IUnityContainer;<br />        IGlobalSettings settings =  container.Resolve&lt;IGlobalSettings&gt;();<br />        <span style="color: #0000ff">if</span> (settings.GameStarted != <span style="color: #0000ff">this</span>.started)<br />        {<br />            filterContext.Result = <span style="color: #0000ff">new</span> ViewResult()<br />            {<br />                ViewName = settings.GameStarted ? <span style="color: #006080">"GameStarted"</span> : <span style="color: #006080">"GameNotStarted"</span><br />            };<br />        }<br />    }<br />}</pre>
                              
                              <p>
                                </div> 
                                
                                <p>
                                  En este caso es un IAuthorizationFilter, ya que en cierta manera estamos autorizando peticiones (acciones) no en función del usuario, sinó en función del juego. En el método OnAuthorization se redirige al usuario a las dos páginas de error si el estado del juego no es el pedido, y en caso contrario no se hace nada (y se ejecuta la acción).
                                </p>
                                
                                <p>
                                  La forma de aplicar el filtro es decorando las acciones con el atributo [GameStarted]:
                                </p>
                                
                                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #008000">// El juego debe estar en marcha para ejecutar esta acción. Si el juego está</span><br /><span style="color: #008000">// parado el usuario será redirigido a una página explicativa.</span><br />[GameStarted(<span style="color: #0000ff">true</span>)]<br /><span style="color: #0000ff">public</span> ActionResult Index()<br />{<br />    <span style="color: #0000ff">return</span> View();<br />}</pre>
                                  
                                  <p>
                                    </div> 
                                    
                                    <p>
                                      Como siempre espero que el post os haya resultado útil para daros cuenta de lo extensible que resulta ser ASP.NET MVC 😉
                                    </p>
                                    
                                    <p>
                                      Un saludo a todos!!!!!
                                    </p>