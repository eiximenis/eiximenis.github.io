---
title: ASP.NET MVC3 – Filtros que no son atributos

author: eiximenis

date: 2010-07-29T12:03:39+00:00
geeks_url: /?p=1529
geeks_visits:
  - 1675
geeks_ms_views:
  - 914
categories:
  - Uncategorized

---
Antes que nada una nota: Este post está basado en la _preview 1_ de ASP.NET MVC3. Todo lo que comento puede cambiar en futuras previews de MVC3.

<!--more-->

En el post de ayer [comentaba las novedades de ASP.NET MVC3 preview1][1]__ y una de las mejoras más interesantes es el soporte de inyección de dependencias para los filtros. Eso se consigue con el uso de una nueva interfaz _IFilterProvider_ y esa interfaz nos trae de regalo, una posibilidad adicional: Tener filtros que no sean atributos.

Veamos un ejemplo y como podríamos usar este nueva capacidad en MVC3.

**1. Que queremos?**

Queremos poder usar el concepto de filtros de MVC, pero sin usar atributos. Es decir **sin** tener que decorar los controladores o sus acciones con atributos tipo [Authorization].

Vamos a proporcionar una mini api de configuración, para que pueda configurar los filtros, usando una api fluent:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">FluentFilterProvider ffp = <span style="color: #0000ff">new</span> FluentFilterProvider();<br />ffp.AddForController&lt;HomeController, MyCustomFilter&gt;().ForAction(<span style="color: #006080">"Index"</span>);<br />ffp.AddForController&lt;HomeController, MyOtherCustomFilter&gt;();</pre>
  
  <p>
    </div> 
    
    <p>
      Aquí configuro dos filtros para el controlador <em>Home</em>: Uno para la acción “Index” (MyCustomFilter) y otro para todas sus acciones (MyOtherCustomFilter).
    </p>
    
    <p>
      <strong>2. Creación del esqueleto de lo que necesitamos</strong>
    </p>
    
    <p>
      El primer paso es crearnos nuestro propio proveedor de filtros, es decir nuestra clase que implemente la interfaz <em>IFilterProvider</em>. Nada más sencillo que esto:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> FluentFilterProvider : IFilterProvider<br />{<br />    <span style="color: #0000ff">public</span> IEnumerable&lt;Filter&gt; GetFilters(ControllerContext controllerContext, ActionDescriptor actionDescriptor)<br />    {<br />        <span style="color: #0000ff">return</span> <span style="color: #0000ff">null</span>;<br />    }<br />}</pre>
      
      <p>
        </div> 
        
        <p>
          <strong>3. Creación de la interfaz de configuración fluent</strong>
        </p>
        
        <p>
          La idea es que nuestro proveedor de filtros tenga una colección de <em>IFilterConfigItem</em> y cada <em>IFilterConfigItem</em> tenga la información para saber cuando instanciar el filtro (es decir a que controlador y acción afecta y cual es la clase que implementa el filtro).
        </p>
        
        <p>
          Como no sabemos si eso puede complicarse más, definimos una clase <em>FluentFilterProviderConfig</em> que mantenga esta colección:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">class</span> FluentFilterProviderConfig<br />{<br />    <span style="color: #0000ff">private</span> List&lt;FilterConfigItem&gt; _items;<br /><br />    <span style="color: #0000ff">public</span> FluentFilterProviderConfig()<br />    {<br />        _items = <span style="color: #0000ff">new</span> List&lt;FilterConfigItem&gt;();<br />    }<br /><br />    <span style="color: #0000ff">internal</span> ICollection&lt;FilterConfigItem&gt; Items<br />    {<br />        get { <span style="color: #0000ff">return</span> _items; }<br />    }<br />}</pre>
          
          <p>
            </div> 
            
            <p>
              La colección es de objetos de la clase <em>FilterConfigItem</em>, esta clase es la que implementa <em>IFilterConfigItem</em>. Veamos primero como está definido la interfaz <em>IFilterConfigItem</em>:
            </p>
            
            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">interface</span> IFilterConfigItem<br />{<br />    <span style="color: #0000ff">void</span> ForAction(<span style="color: #0000ff">string</span> actionName);<br />}</pre>
              
              <p>
                </div> 
                
                <p>
                  Un solitario método <em>ForAction</em> que nos permite indicar que dicho filtro se aplica a una acción en concreto.
                </p>
                
                <p>
                  Y ahora veamos la clase <em>FilterConfigItem</em>:
                </p>
                
                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">class</span> FilterConfigItem : IFilterConfigItem<br />{<br />    <span style="color: #0000ff">private</span> Type _filterType;<br />    <span style="color: #0000ff">private</span> Type _controllerType;<br />    <span style="color: #0000ff">private</span> <span style="color: #0000ff">string</span> _actionName;<br /><br />    <span style="color: #0000ff">internal</span> FilterConfigItem(Type tf, Type controllerType)<br />    {<br />        _filterType = tf;<br />        _controllerType = controllerType;<br />        _actionName = <span style="color: #0000ff">null</span>;<br />    }<br /><br />    <span style="color: #0000ff">internal</span> Type ControllerType { get { <span style="color: #0000ff">return</span> _controllerType; } }<br />    <span style="color: #0000ff">internal</span> Type FilterType { get { <span style="color: #0000ff">return</span> _filterType; } }<br />    <span style="color: #0000ff">internal</span> <span style="color: #0000ff">string</span> ActionName { get { <span style="color: #0000ff">return</span> _actionName; } }<br /><br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> ForAction(<span style="color: #0000ff">string</span> actionName)<br />    {<br />        _actionName = actionName;<br />    }<br />}</pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Dicha clase guarda toda la información requerida:
                    </p>
                    
                    <ul>
                      <li>
                        El tipo de la clase que implementa el filtro (_filterType)
                      </li>
                      <li>
                        El tipo del controlador al que se aplica el filtro (_controllerType)
                      </li>
                      <li>
                        El nombre de la acción a la que se aplica el filtro (_actionName). Este valor es opcional (si vale null el filtro se aplicará a todas las acciones).
                      </li>
                    </ul>
                    
                    <p>
                      Tenemos también propiedades para acceder a esta información, pero fijaos que son internal y no forman parte de la interfaz. Esto es porque esas propiedades sólo son para la implementación de la API, no para su uso público.
                    </p>
                    
                    <p>
                      Finalmente en la clase <em>FluentFilterProvider</em> vamos a guardar un objeto con la configuración y vamos a proporcionar el punto de entrada a la API fluent: el método <em>AddForController</em>:
                    </p>
                    
                    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> FluentFilterProvider : IFilterProvider<br />{<br />    <span style="color: #0000ff">private</span> FluentFilterProviderConfig _cfg;<br /><br />    <span style="color: #0000ff">public</span> FluentFilterProvider()<br />    {<br />        _cfg = <span style="color: #0000ff">new</span> FluentFilterProviderConfig();<br />    }<br /><br />    <span style="color: #0000ff">public</span> IEnumerable&lt;Filter&gt; GetFilters(ControllerContext controllerContext, ActionDescriptor actionDescriptor)<br />    {<br />        <span style="color: #008000">// Ya veremos que metemos aquí...</span><br />    }<br /><br />    <span style="color: #0000ff">public</span> IFilterConfigItem AddForController&lt;TC, TF&gt;()<br />        <span style="color: #0000ff">where</span> TC : IController<br />        <span style="color: #0000ff">where</span> TF : <span style="color: #0000ff">class</span><br />    {<br />        FilterConfigItem fci = <span style="color: #0000ff">new</span> FilterConfigItem(<span style="color: #0000ff">typeof</span>(TF), <span style="color: #0000ff">typeof</span>(TC));<br />        _cfg.Items.Add(fci);<br />        <span style="color: #0000ff">return</span> fci;<br />    }<br />}</pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          Fijaos que el método <em>AddForController </em>crea un objeto FilterConfigItem, lo añade a la colección y lo devuelve. Esta devolución es lo que permite encadenar las llamadas (característica básica de una interfaz fluent).
                        </p>
                        
                        <p>
                          <strong>4. Creación del filter provider</strong>
                        </p>
                        
                        <p>
                          Bien, sólo nos queda terminar de implementar la clase <em>FluentFilterProvider</em> con la implementación del método GetFilters. El código es muy sencillo:
                        </p>
                        
                        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> IEnumerable&lt;Filter&gt; GetFilters(ControllerContext controllerContext, ActionDescriptor actionDescriptor)<br />{<br />    var items = _cfg.Items.Where(x =&gt; x.ControllerType == controllerContext.Controller.GetType()<br />        && (x.ActionName == <span style="color: #0000ff">null</span> || x.ActionName.Equals(actionDescriptor.ActionName)));<br />    <span style="color: #0000ff">foreach</span> (var item <span style="color: #0000ff">in</span> items)<br />    {<br />        var sl = MvcServiceLocator.Current;<br />        var filterInstance = sl.GetInstance(item.FilterType);<br />        <span style="color: #0000ff">yield</span> <span style="color: #0000ff">return</span> <span style="color: #0000ff">new</span> Filter(filterInstance, FilterScope.Action);<br />    }<br />}</pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              Nos recorremos la colección de <em>FilterConfigItem</em>s y por cada uno miramos si aplica al controlador pedido y a la acción indicada (los parámetros controllerContext y actionDescriptor nos dan información sobre el controlador y la acción para la cual debemos encontrar los filtros).
                            </p>
                            
                            <p>
                              Y por cada elemento <em>FilterConfigItem</em> encontrado:
                            </p>
                            
                            <ol>
                              <li>
                                Creamos la instancia del objeto que implementa el filtro. Fijaos que lo hacemos usando MvcServiceLocator.Current: eso permite aplicar inyección de dependencias.
                              </li>
                              <li>
                                Creamos un objeto <em>Filter </em>(el objeto que espera el framework), lo rellenamos con los datos y lo devolvemos.
                              </li>
                            </ol>
                            
                            <p>
                              Y listos! Nuestro filter provider está listo para usar… Sólo nos queda crearlo, configurarlo y registrarlo en MVC:
                            </p>
                            
                            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">FluentFilterProvider ffp = <span style="color: #0000ff">new</span> FluentFilterProvider();<br />ffp.AddForController&lt;HomeController, MyCustomFilter&gt;().ForAction(<span style="color: #006080">"Index"</span>);<br />ffp.AddForController&lt;HomeController, MyOtherCustomFilter&gt;();<br />FilterProviders.Providers.Add(ffp);</pre>
                              
                              <p>
                                </div> 
                                
                                <p>
                                  Recordad que las clases MyCustomFilter y MyOtherCustomFilter son las clases que implementan los filtros (son las que implementan IActionFilter, IAuthorizationFilter, IResultFilter o IExceptionFilter).
                                </p>
                                
                                <p>
                                  No se que os parece a vosotros, pero a mi me parece una pasada!!! 🙂
                                </p>
                                
                                <p>
                                  Un saludo!!!!
                                </p>

 [1]: http://geeks.ms/blogs/etomas/archive/2010/07/28/asp-net-mvc-3-preview-1.aspx