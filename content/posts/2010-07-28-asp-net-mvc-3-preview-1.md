---
title: ASP.NET MVC 3 – Preview 1
author: eiximenis

date: 2010-07-28T19:03:00+00:00
geeks_url: /?p=1528
geeks_visits:
  - 6348
geeks_ms_views:
  - 1829
categories:
  - Uncategorized

---
Ayer saltaba la noticia en [el blog de scottgu][1]: la [preview 1 de ASP.NET MVC3 ya está disponible para descargar][2]. En fin, podríamos discutir largo y tendido sobre la política de actualizaciones a lo bestia de las APIs que está realizando microsoft desde hace algún tiempo, pero como cada uno tendría su opinión, mejor vamos a ver las novedades que trae esa _preview 1_. Antes que nada, podéis instalarla sin miedo: se instala side by side con MVC2 y además los proyectos que ya teníais no se ven afectados.

Una vez instalado MVC3, aparecen tres nuevos tipos de proyectos en VS2010: _ASP.NET MVC 3 Web Application (ASPX)_, _ASP.NET MVC 3 Web Application (Razor)_ y _ASP.NET MVC 3 Emtpy Web Application_. No hay soporte para VS2008 puesto que MVC3 usa el .NET Framework 4. Vamos a ver las novedades que tiene esta preview1 de MVC3 🙂

**1. Razor View Engine**

El nuevo _Razor_ viene incluído en MVC3. Razor no es nada más que _otro_ ViewEngine para MVC. Ya se ha hablado largo y tendido de Razor porque se incluye también en [WebMatrix][3]. Si bien en WebMatrix, Razor venía acompañado del concepto de [ASP.NET WebPage][4], en ASP.NET MVC dicho concepto carece de sentido y Razor es _simplemente_ otro [ViewEngine][5] más.

Razor ofrece una sintaxis _alternativa_ a la sintaxis clásica de <% %> tan típica de ASP.NET MVC, pero **no ofrece nada nuevo** que no ofrezcan el ViewEngine de .aspx u otros como [Nhaml][6] o [Spark][7]. 

Mi &ldquo;decepción&rdquo; ha sido que de hecho, tanto si usamos el ViewEngine de .aspx como si usamos Razor, las clases de las vistas derivan de [System.Web.Mvc.ViewPage<>][8] (yo tenía la esperanza de que Razor ofreciera un framework mucho más ligero, puesto que ViewPage deriva de [System.Web.UI.Page][9] que tiene muchas propiedades y métodos que tienen sentido en WebForms pero sólo añaden ruído en MVC).

P.ej. en Razor para mostrar una lista de elementos (FooItem) que tuviesen dos propiedades llamadas First y Second usaríamos:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">@inherits System.Web.Mvc.WebViewPage<span style="color: #0000ff">&lt;</span><span style="color: #800000">IEnumerable</span><span style="color: #0000ff">&lt;</span><span style="color: #800000">MvcApplication1.Models.FooItem</span><span style="color: #0000ff">&gt;&gt;</span><br /><br />@{<br />    View.Title = "TableView";<br />    LayoutPage = "~/Views/Shared/_Layout.cshtml";<br />}<br /><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">h2</span><span style="color: #0000ff">&gt;</span>TableView<span style="color: #0000ff">&lt;/</span><span style="color: #800000">h2</span><span style="color: #0000ff">&gt;</span><br /><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">ul</span><span style="color: #0000ff">&gt;</span><br />    @foreach (var item in Model)<br />    {<br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">li</span><span style="color: #0000ff">&gt;</span>@item.First - @item.Second<span style="color: #0000ff">&lt;/</span><span style="color: #800000">li</span><span style="color: #0000ff">&gt;</span><br />    }<br />    <span style="color: #0000ff">&lt;/</span><span style="color: #800000">ul</span><span style="color: #0000ff">&gt;</span></pre>
  
  <p>
    </div> 
    
    <p>
      Este es el código<strong> entero</strong> de la vista: como veis es mucho más compacto que su equivalente en .aspx. ¿Cual preferís? Es una cuestión de gustos, pero según comenta Scott en su post, será posible probar templates de Razor de forma individual sin necesidad de tener un servidor web, lo que ayudará a la generación de pruebas... Por lo que parece esto no está previsto para el ViewEngine de .aspx.
    </p>
    
    <p>
      Las vistas en Razor (usando c#) tienen la extensión .cshtml y de momento el soporte en vs2010 de Razor es muy limitado en esta preview1: Ni sintaxis coloreada, ni intellisense... 🙂
    </p>
    
    <p>
      <strong>2. Mejor soporte para DI</strong>
    </p>
    
    <p>
      MVC2 ya tenía un soporte más que decente para dependency injection, pero en MVC3 lo han mejorado: no sólo han simplificado la tarea de usar DI sinó que ahora también se soporta la inyección de dependencias en los filtros.
    </p>
    
    <p>
      El soporte para contenedores IoC se basa en la interfaz <a href="http://commonservicelocator.codeplex.com/">IServiceLocator</a> y como es uno de los puntos más interesantes a mi parecer, dejadme que me extienda un poco 🙂
    </p>
    
    <p>
      Primero, antes que nada, en la preview1, la interfaz IServiceLocator <strong>está definida dentro del propio assembly de System.Web.Mvc.dll</strong>, en lugar de usar el del ensamblado Microsoft.Practices.ServiceLocation.dll. Esto genera algunos problemas y es algo que está previsto que cambie en futuras previews.
    </p>
    
    <p>
      P.ej. si usamos Unity 2.0, para obtener el IServiceLocator nos basta con hacer:
    </p>
    
    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">IUnityContainer iuc = <span style="color: #0000ff">new</span> UnityContainer();<br />Microsoft.Practices.ServiceLocation.IServiceLocator ul = <span style="color: #0000ff">new</span> UnityServiceLocator(iuc);<br /></pre>
      
      <p>
        </div> 
        
        <p>
          La idea es el IServiceLocator que hemos obtenido (ul) lo podamos usar en MVC3, pero por ahora es imposible. La razón es la que comentaba: MVC3 define su propio IServiceLocator en lugar de utilizar el que viene en Microsoft.Practices.ServiceLocation.dll. Así pues, de momento, estamos obligados a implementarnos nuestro &ldquo;propio&rdquo; IServiceLocator:
        </p>
        
        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> MvcUnityServiceLocation : IServiceLocator<br />{<br />    <span style="color: #0000ff">private</span> <span style="color: #0000ff">readonly</span> IUnityContainer _container;<br /><br />    <span style="color: #0000ff">public</span> MvcUnityServiceLocation(IUnityContainer ctr)<br />    {<br />        _container = ctr;<br />    }<br /><br /><br />    <span style="color: #0000ff">public</span> IEnumerable&lt;<span style="color: #0000ff">object</span>&gt; GetAllInstances(Type serviceType)<br />    {<br />        <span style="color: #0000ff">return</span> _container.ResolveAll(serviceType);<br />    }<br /><br />    <span style="color: #0000ff">public</span> IEnumerable&lt;TService&gt; GetAllInstances&lt;TService&gt;()<br />    {<br />        <span style="color: #0000ff">return</span> _container.ResolveAll&lt;TService&gt;();<br />    }<br /><br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">object</span> GetInstance(Type serviceType, <span style="color: #0000ff">string</span> key)<br />    {<br />        <span style="color: #0000ff">return</span> _container.Resolve(serviceType, key);<br />    }<br /><br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">object</span> GetInstance(Type serviceType)<br />    {<br />        <span style="color: #0000ff">return</span> _container.Resolve(serviceType);<br />    }<br /><br />    <span style="color: #0000ff">public</span> TService GetInstance&lt;TService&gt;(<span style="color: #0000ff">string</span> key)<br />    {<br />        <span style="color: #0000ff">return</span> _container.Resolve&lt;TService&gt;(key);<br />    }<br /><br />    <span style="color: #0000ff">public</span> TService GetInstance&lt;TService&gt;()<br />    {<br />        <span style="color: #0000ff">return</span> _container.Resolve&lt;TService&gt;();<br />    }<br /><br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">object</span> GetService(Type serviceType)<br />    {<br />        <span style="color: #0000ff">return</span> _container.Resolve(serviceType);<br />    }<br />}</pre>
          
          <p>
            </div> 
            
            <p>
              La interfaz que estamos implementando NO ES Microsoft.Practices.IServiceLocator sino System.Web.Mvc.IServiceLocator. Como digo eso es algo que se supone cambiará en futuras previews.
            </p>
            
            <p>
              Ahora ya podemos registrar este IServiceLocator, usando la clase MvcServiceLocator (esto se suele hacer en el Application_Start):
            </p>
            
            <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
              <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">IUnityContainer iuc = <span style="color: #0000ff">new</span> UnityContainer();<br />System.Web.Mvc.IServiceLocator ul = <span style="color: #0000ff">new</span> MvcUnityServiceLocation(iuc);<br />MvcServiceLocator.SetCurrent(ul);</pre>
              
              <p>
                </div> 
                
                <p>
                  Bueno... Con eso establecemos cual va a ser el ServiceLocator que usará MVC para instanciar sus objetos. Ahora debemos configurarlo. Una cosa que me he encontrado es que <strong>debemos establecer la factoría de controladores explicitamente</strong> en el contenedor de IoC:
                </p>
                
                <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">iuc.RegisterType&lt;IControllerFactory, DefaultControllerFactory&gt;<br />(<span style="color: #0000ff">new</span> ContainerControlledLifetimeManager());</pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Aquí estoy diciendo a MVC que use DefaultControllerFactory como factoría de controladores, y la forma de hacerlo es simplemente establecer un mapping entre IControllerFactory y DefaultControllerFactory en mi contenedor IoC.
                    </p>
                    
                    <p>
                      Y aquí viene la mejora: DefaultControllerFactory ya está preparada para usar el IServiceLocator, por lo que <strong>ya tenemos DI en los controladores</strong>. Sin hacer nada más (antes uno debía crearse su propia IControllerFactory). Podemos establecer un mapping cualquiera en nuestro contenedor:
                    </p>
                    
                    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">iuc.RegisterType&lt;IFoo, Foo&gt;();</pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          Y ya podemos usar IFoo sin ningún problema en nuestros controladores:
                        </p>
                        
                        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> HomeController : Controller<br />{<br />    <span style="color: #0000ff">public</span> HomeController(IFoo foo)<br />    {<br />        <span style="color: #008000">// ...</span><br />    }<br />}</pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              Cada vez que se cree un <em>HomeController</em> se le inyectará el parámetro IFoo.
                            </p>
                            
                            <p>
                              Antes he comentado que una de las novedades de MVC3 es el soporte de inyección de dependencias para los filtros... Los que conozcais un poco ASP.NET MVC seguramente os estareis preguntando cómo, dado que los filtros son clases que heredan de <a href="http://msdn.microsoft.com/en-us/library/system.attribute.aspx">Attribute</a> y por lo tanto es el propio runtime de .NET quien los crea, sin que el contenedor de IoC pueda intervenir...
                            </p>
                            
                            <p>
                              Para ayudar a la inyección de dependencias en filtros, en MVC3 se han inventado el concepto del proveedor de filtros, representado por la interfaz <em>IFilterProvider</em> y que es el responsable de obtener instancias de todos los filtros que necesite el framework. Nosotros ahora podemos crear nuestra propia clase que implemente IFilterProvider y que aplique la inyección de dependencias. Veamos primero como está definido IFilterProvider:
                            </p>
                            
                            <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                              <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">interface</span> IFilterProvider<br />{<br />    IEnumerable&lt;Filter&gt; GetFilters(ControllerContext controllerContext, <br />    ActionDescriptor actionDescriptor);<br />}</pre>
                              
                              <p>
                                </div> 
                                
                                <p>
                                  Un solo método GetFilters que es el encargado de devolver el conjunto de filtros necesarios cada vez. La clase <em>Filter</em> no es el filtro en sí, sinó una clase de metadatos que contiene una referencia al objeto que implementa el filtro (en nuestro caso el atributo), además de información sobre el orden (que ya existía en MVC2) y el &agrave;mbito del filtro (p.ej. si es un filtro que se aplica a un controlador, a una acción,...). Los filtros se ejecutan unos antes de otros en función de su ámbito y de su orden.
                                </p>
                                
                                <p>
                                  Bien, si queremos implementar filtros usando atributos podemos hacerlo igual que en MVC2 derivando de <em>FilterAttribute</em>:
                                </p>
                                
                                <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                                  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> CustomActionFilterAttribute : ActionFilterAttribute<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">override</span> <span style="color: #0000ff">void</span> OnActionExecuting(ActionExecutingContext filterContext)<br />    {<br />        <span style="color: #0000ff">base</span>.OnActionExecuting(filterContext);<br />    }<br />}<br /></pre>
                                  
                                  <p>
                                    </div> 
                                    
                                    <p>
                                      Nada nuevo aquí, lo nuevo viene ahora: Podemos crear un IFilterProvider que nos inyecte las dependencias a los filtros. Este IFilterProvider, obviamente, no creará los filtros (puesto que son atributos y los crea el runtime de .NET). Entonces que hace? Pues tres cosas:
                                    </p>
                                    
                                    <ol>
                                      <li>
                                        Recoge los objetos que implementan los filtros (creados por el runtime)
                                      </li>
                                      <li>
                                        Por cada objeto le inyecta las dependencias. Para ello necesitamos que el contenedor de DI soporte inyectar dependencias a objetos ya existentes y usar inyección de propiedades.
                                      </li>
                                      <li>
                                        Finalmente crea el objeto <em>Filter</em> que contendrá el objeto con las dependencias creadas.
                                      </li>
                                    </ol>
                                    
                                    <p>
                                      Un ejemplo, usando Unity:
                                    </p>
                                    
                                    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                                      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> UnityFilterAttributeFilterProvider : FilterAttributeFilterProvider<br />{<br />    <span style="color: #0000ff">private</span> IUnityContainer _container;<br /><br />    <span style="color: #0000ff">public</span> UnityFilterAttributeFilterProvider(IUnityContainer container)<br />    {<br />        _container = container;<br />    }<br />    <span style="color: #0000ff">protected</span> <span style="color: #0000ff">override</span> IEnumerable&lt;FilterAttribute&gt; GetControllerAttributes(<br />        ControllerContext controllerContext, ActionDescriptor actionDescriptor)<br />    {<br />        var attributes = <span style="color: #0000ff">base</span>.GetControllerAttributes(controllerContext, actionDescriptor);<br />        <span style="color: #0000ff">foreach</span> (var attribute <span style="color: #0000ff">in</span> attributes)<br />        {<br />            _container.BuildUp(attribute.GetType(), attribute);<br />        }<br />        <span style="color: #0000ff">return</span> attributes;<br />    }<br /><br />    <span style="color: #0000ff">protected</span> <span style="color: #0000ff">override</span> IEnumerable&lt;FilterAttribute&gt; GetActionAttributes(<br />        ControllerContext controllerContext, ActionDescriptor actionDescriptor)<br />    {<br />        var attributes = <span style="color: #0000ff">base</span>.GetActionAttributes(controllerContext, actionDescriptor);<br />        <span style="color: #0000ff">foreach</span> (var attribute <span style="color: #0000ff">in</span> attributes)<br />        {<br />            _container.BuildUp(attribute.GetType(), attribute);<br />        }<br />        <span style="color: #0000ff">return</span> attributes;<br />    }<br />}</pre>
                                      
                                      <p>
                                        </div> 
                                        
                                        <p>
                                          Derivamos de FilterAttributeFilterProvider (que implementa IFilterProvider) y usamos Unity para inyectar las dependencias (BuildUp) a los objetos que implementan los filtros.
                                        </p>
                                        
                                        <p>
                                          Y ahora? Pues como siempre: registrar este FilterAttributeProvider en el framework a través del Applicaton_Start de global.asax:
                                        </p>
                                        
                                        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                                          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">var provider = <span style="color: #0000ff">new</span> UnityFilterAttributeFilterProvider(container);  <br />FilterProviders.Providers.Add(provider);  <br /></pre>
                                          
                                          <p>
                                            </div> 
                                            
                                            <p>
                                              Por defecto hay 3 IFilterProviders instalados en el framework (en la colección <em>Providers</em>):
                                            </p>
                                            
                                            <ol>
                                              <li>
                                                &nbsp; <ol>
                                                  <li>
                                                    Un filter provider que se encarga de devolver los controladores (sí, los controladores son filtros también (desde siempre)).
                                                  </li>
                                                  <li>
                                                    Un filter provider que se encarga de devolver los filtros que son atributos
                                                  </li>
                                                  <li>
                                                    Un filter provider que se encarga de devolver los filtros globales
                                                  </li>
                                                </ol>
                                              </li>
                                            </ol>
                                            
                                            <p>
                                              Dado que nosotros estamos añadiendo otro filter provider que sustituye al que viene por defecto al que se encarga de devolver los filtros que son atributos, no estaría de más eliminar el que viene por defecto antes de registrar el nuestro (aunque funciona si no lo hacemos):
                                            </p>
                                            
                                            <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                                              <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">FilterProviders.Providers.Remove(FilterProviders.Providers.FirstOrDefault(x =&gt; x <span style="color: #0000ff">is</span> FilterAttributeFilterProvider));</pre>
                                              
                                              <p>
                                                </div> 
                                                
                                                <p>
                                                  Y listos! Ahora ya tenemos inyección de dependencias en nuestros filtros! Recordad que debemos usar inyección de propiedades, dado que los filtros son creados por el runtime de .NET. Es decir algo como:
                                                </p>
                                                
                                                <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                                                  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> CustomActionFilterAttribute : ActionFilterAttribute<br />{<br />    <span style="color: #008000">// Esta propiedad la inyecta Unity...</span><br />    [Dependency]<br />    <span style="color: #0000ff">public</span> IFoo Foo { get; set; }<br />    <span style="color: #008000">// ...</span><br />}<br /> </pre>
                                                  
                                                  <p>
                                                    </div> 
                                                    
                                                    <p>
                                                      Que os parece? No es mucho trabajo para el beneficio que obtenemos!
                                                    </p>
                                                    
                                                    <p>
                                                      Otra ventaja del uso de filter providers es poder tener filtros que no sean atributos, algo que no estaba permitido en MVC2: cualquier &ldquo;cosa&rdquo; que el filter provider devuelva será considerado un filtro para el framework.
                                                    </p>
                                                    
                                                    <p>
                                                      <strong>3. Otras mejoras menores</strong>
                                                    </p>
                                                    
                                                    <p>
                                                      Ahora listo brevemente un conjunto de &ldquo;mejoras menores&rdquo; que pese a no suponer un avance brutal sirven para simplificarnos el trabajo o mejorar el código:
                                                    </p>
                                                    
                                                    <ol>
                                                      <li>
                                                        <strong>Soporte para objetos JSON en POST:</strong> MVC3 es capaz de tratar datos que están en POST en formato JSON y usarlos cuando hacemos binding del viewmodel. MVC2 no era capaz, aunque no era muy complejo añadir un custom value provider que diera esa capacidad. De hecho en MvcFutures ya estaba y es lógico que haya entrado en esta versión.
                                                      </li>
                                                      <li>
                                                        <strong>Nueva propiedad ViewModel:</strong> ViewModel es una propiedad declarada como <em>dynamic</em> que permite usar la antigua ViewData de forma tipada. Es decir en lugar de hacer <em>ViewData[&ldquo;Foo&rdquo;] = new Bar();</em> podemos hacer <em>ViewModel.Foo = new Bar();</em> y el resultado es equivalente. Esto mejora la legibilidad y facilita refactorings.
                                                      </li>
                                                      <li>
                                                        <strong>Soporte del interfaaz </strong><a href="http://msdn.microsoft.com/es-es/library/system.componentmodel.dataannotations.ivalidatableobject.aspx"><strong>IValidatableObject</strong></a><strong>:</strong> Esta interfaz (propia de .NET 4) tiene un sólo método (Validate) que determina si un objeto es &ldquo;válido&rdquo;. El model binder por defecto ahora usa ese método si el viewmodel implementa dicha interfaz.
                                                      </li>
                                                      <li>
                                                        <strong>Nuevos ActionResults:</strong> Para devolver un 404 (HttpNotFoundResult), o un código específico de HTTP (HttpStatusCodeResult).
                                                      </li>
                                                      <li>
                                                        <strong>Filtros globales:</strong> Filtros que se aplican a todos los controladores de nuestra aplicación de forma automática.
                                                      </li>
                                                    </ol>
                                                    
                                                    <p>
                                                      Bueno... como podeis ver MVC3 viene con un buen puñado de novedades, muchas de ellas pequeñitas pero sin duda alguna interesantes... larga, larga, larga vida a MVC!!! 🙂
                                                    </p>
                                                    
                                                    <p>
                                                      <strong>Referencias</strong>
                                                    </p>
                                                    
                                                    <p>
                                                      <a href="http://weblogs.asp.net/scottgu/archive/2010/07/27/introducing-asp-net-mvc-3-preview-1.aspx">El post de ScottGu explicando MVC3</a>
                                                    </p>
                                                    
                                                    <p>
                                                      <a href="http://bradwilson.typepad.com/blog/2010/07/service-location-pt4-filters.html">El post de Brad Wilson explicando la DI en filtros</a>
                                                    </p>

 [1]: http://weblogs.asp.net/scottgu
 [2]: http://www.microsoft.com/downloads/details.aspx?FamilyID=cb42f741-8fb1-4f43-a5fa-812096f8d1e8&displaylang=en
 [3]: http://www.microsoft.com/web/webmatrix/
 [4]: http://weblogs.asp.net/mikaelsoderstrom/archive/2010/07/06/introduction-to-asp-net-web-pages.aspx
 [5]: /blogs/gtorres/archive/2010/02/19/view-engine-asp-net-mvc.aspx
 [6]: http://code.google.com/p/nhaml/
 [7]: http://sparkviewengine.com/
 [8]: http://msdn.microsoft.com/en-us/library/system.web.mvc.viewpage.aspx
 [9]: http://msdn.microsoft.com/en-us/library/system.web.ui.page.aspx