---
title: Nombres de algunos controladores distintos en ASP.NET MVC (ii)
author: eiximenis

date: 2010-02-19T15:39:13+00:00
geeks_url: /?p=1498
geeks_visits:
  - 1221
geeks_ms_views:
  - 833
categories:
  - Uncategorized

---
Bueno… Este post es la continuación / aclaración del <a href="http://geeks.ms/blogs/etomas/archive/2010/02/19/nombres-de-algunos-controladores-distintos-en-asp-net-mvc.aspx" target="_blank" rel="noopener noreferrer">post anterior</a>. En el post anterior configuramos la tabla de rutas junto con un RouteHandler propio y vimos que realmente las llamadas se enrutaban al controlador que tocaba: /api/Foo me enrutaba al controlador WarFoo y /Foo me enrutaba al controlador Foo.

Lo que _no comenté_ es lo que _deja_ de funcionar… 🙂

Supongo que teneis la tabla de rutas de la siguiente manera:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">routes.MapRoute(<span style="color: #006080">"Default"</span>,<span style="color: #006080">"{controller}/{action}/{id}"</span>, <br />    <span style="color: #0000ff">new</span> { controller = <span style="color: #006080">"Home"</span>, action = <span style="color: #006080">"Index"</span>, id = <span style="color: #006080">""</span> });<br />routes.Add(<span style="color: #0000ff">new</span> Route(<span style="color: #006080">"api/{controller}/{action}/{id}"</span>,<br />        <span style="color: #0000ff">new</span> RouteValueDictionary(<br />            <span style="color: #0000ff">new</span> { controller = <span style="color: #006080">"Home"</span>, action = <span style="color: #006080">"Index"</span>, id = <span style="color: #006080">""</span> }),    <br />        <span style="color: #0000ff">new</span> WarRouteHandler())<br />);</pre>
  
  <p>
    </div> 
    
    <p>
      Suponemos también que tenemos el controlador Foo (al que queremos acceder vía (/Foo) y WarFoo al cual queremos acceder via /api/Foo). Si uso Html.ActionLink para generar los enlaces obtengo lo siguiente:
    </p>
    
    <table border="0" cellspacing="0" cellpadding="2" width="515">
      <tr>
        <td valign="top" width="222">
          <strong>Llamada</strong>
        </td>
        
        <td valign="top" width="129">
          <strong>URL obtenida</strong>
        </td>
        
        <td valign="top" width="162">
          <strong>URL que </strong><em><strong>querria</strong> yo</em>
        </td>
      </tr>
      
      <tr>
        <td valign="top" width="222">
          Html.ActionLink("Texto", "XX","Foo")
        </td>
        
        <td valign="top" width="129">
          /Foo/XX
        </td>
        
        <td valign="top" width="162">
          /Foo/XX
        </td>
      </tr>
      
      <tr>
        <td valign="top" width="222">
          Html.ActionLink("Texto", "XX", "WarFoo")
        </td>
        
        <td valign="top" width="129">
          /WarFoo/XX
        </td>
        
        <td valign="top" width="162">
          /api/Foo/XX
        </td>
      </tr>
    </table>
    
    <p>
      Vamos a ver como podemos empezar a arreglar este desaguisado… Debemos recordar que <em>el orden</em> de las rutas en la tabla de rutas afecta: toda URL se empieza a comprobar en cada ruta, y se usa la primera que encaje. Dado que no hay nada que impida en la ruta <em>Default</em> que haya un controlador llamado WarFoo se usa esa ruta, y por eso obtenemos /WarFoo/XX como URL final.
    </p>
    
    <p>
      Uno puede pensar que la solució es invertir el orden de las rutas en la tabla de rutas… Si lo haceis el reultado no es mucho mejor:
    </p>
    
    <table border="0" cellspacing="0" cellpadding="2" width="515">
      <tr>
        <td valign="top" width="222">
          <strong>Llamada</strong>
        </td>
        
        <td valign="top" width="129">
          <strong>URL obtenida</strong>
        </td>
        
        <td valign="top" width="162">
          <strong>URL que </strong><em><strong>querria</strong> yo</em>
        </td>
      </tr>
      
      <tr>
        <td valign="top" width="222">
          Html.ActionLink("Texto", "XX","Foo")
        </td>
        
        <td valign="top" width="129">
          /api/Foo/XX
        </td>
        
        <td valign="top" width="162">
          /Foo/XX
        </td>
      </tr>
      
      <tr>
        <td valign="top" width="222">
          Html.ActionLink("Texto", "XX", "WarFoo")
        </td>
        
        <td valign="top" width="129">
          /api/WarFoo/XX
        </td>
        
        <td valign="top" width="162">
          /api/Foo/XX
        </td>
      </tr>
    </table>
    
    <p>
      Que nos ocurre? Lo mismo de antes, salvo que ahora la primera ruta que se evalúa es la que tiene las URLs que empiezan por api. Pero esta ruta tampoco impone ninguna restricción sobre los controladores, así <em>cualquier</em> nombre de controlador también encaja en esta ruta, y es por eso que obtenemos siempre URLs que empiezan por api.
    </p>
    
    <p>
      Cuando tenemos dos URLs que ambas aceptan cualquier controlador y acción, es dificil que ActionLink pueda distinguir entre una u otra, así que generalmente nos usará la primera definida para generar los enlaces. Dado que por norma general <strong>no</strong> queremos poner /api/ en todas las URLs podemos dejar la ruta “Default” como la primera. Ahora entra en acción RouteLink: podemos usar RouteLink para generar las URLs que deben empezar con /api y ActionLink para las que no. P.ej. puedo usar la siguiente llamada RouteLink, para onbtener la url /api/Foo/XX:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">Html.RouteLink(<span style="color: #006080">"Texto"</span>, <span style="color: #006080">"api"</span>, <span style="color: #0000ff">new</span> RouteValueDictionary(<br />  <span style="color: #0000ff">new</span> { controller=<span style="color: #006080">"Foo"</span>, action=<span style="color: #006080">"XX"</span>})) </pre>
      
      <p>
        </div> 
        
        <p>
          Aquí estoy generando un link a la ruta “api” para generar el enlace. Debemos modificar la tabla de rutas para que la ruta que genera las urls con /api/ se llame api. Esto es tan simple como poner el nombre de la ruta como primer parámetro del método Add:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">routes.Add(<span style="color: #006080">"api"</span>, <span style="color: #0000ff">new</span> Route(<span style="color: #006080">"api/{controller}/{action}/{id}"</span>,<br />    <span style="color: #0000ff">new</span> RouteValueDictionary(<span style="color: #0000ff">new</span> { controller = <span style="color: #006080">"Home"</span>, action = <span style="color: #006080">"Index"</span>, id = <span style="color: #006080">""</span> }),<br />    <span style="color: #0000ff">new</span> WarRouteHandler())<br />);</pre>
          
          <p>
            </div> 
            
            <p>
              y todo solucionado no??? 🙂
            </p>
            
            <p>
              Pues no… todavía nos queda un fleco… un fleco que también <em>pase por alto</em> en el post anterior.
            </p>
            
            <p>
              Los enlaces estan bien generados, uno apunta a /Foo/XX y el otro a /api/Foo/XX. El primero funciona bien pero el segundo da un error… y eso?
            </p>
            
            <p>
              Pensemos de nuevo en como ASP.NET MVC evalúa las rutas: por orden. Y la pregunta es la ruta /api/Foo/XX se puede procesar con la ruta {controller}/{action}/{id} (la <em>Default</em>)? Pues sí, suponiendo que controller es “api”, action es “Foo” e Id es “XX”. Es decir la ruta /api/Foo/XX me intenta invocar la acción Foo del controlador Api, pasándole XX como Id.
            </p>
            
            <p>
              ¿Cual es la solución entonces? Pues añadir una restricción a la ruta (<em>Default</em>) que <strong>impida </strong>que el nombre de los controladores sea “api”. De este modo si {controller} debe tomar el valor api para satisfacer la ruta, como tenemos la restricción la ruta no será satisfecha y ASP.NET MVC intentará usar la siguiente. Las restricciones se añaden como un nuevo parámetro en MapRoute:
            </p>
            
            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">routes.MapRoute(<span style="color: #006080">"Default"</span>, <span style="color: #006080">"{controller}/{action}/{id}"</span>,<br />    <span style="color: #0000ff">new</span> { controller = <span style="color: #006080">"Home"</span>, action = <span style="color: #006080">"Index"</span>, id = <span style="color: #006080">""</span> },<br />    <span style="color: #0000ff">new</span> { controller = <span style="color: #006080">"restriccion"</span> });<br /><br /></pre>
              
              <p>
                </div> 
                
                <p>
                  He añadido una restricción que afecta al parámetro <em>controller</em>. Y como se interpreta esta restricción. Pues bien, si es una cadena se interpreta <strong>como una expresión regular</strong> que debe cumplirse. Si la restricción no se puede (o no sabemos :p) expresar como una expresión regular podemos parar un objeto que implemente <a href="http://msdn.microsoft.com/es-es/library/system.web.routing.irouteconstraint.aspx" target="_blank" rel="noopener noreferrer">IRouteConstraint</a>. Dado que yo soy muy torpe con las expresiones regulares, me he creado una clase que me comprueba que el valor <strong>no sea igual a</strong> un determinado valor:
                </p>
                
                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> NotEqualConstraint : IRouteConstraint<br /> {<br />     <span style="color: #0000ff">private</span> <span style="color: #0000ff">string</span> _match = String.Empty;<br />     <span style="color: #0000ff">public</span> NotEqualConstraint(<span style="color: #0000ff">string</span> match)<br />     {<br />         _match = match;<br />     }<br /><br />     <span style="color: #0000ff">public</span> <span style="color: #0000ff">bool</span> Match(HttpContextBase httpContext, Route route, <span style="color: #0000ff">string</span> parameterName, RouteValueDictionary values, RouteDirection routeDirection)<br />     {<br />         <span style="color: #0000ff">return</span> String.Compare(values[parameterName].ToString(), _match, <span style="color: #0000ff">true</span>) != 0;<br />     }<br /> }</pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Finalmente coloco la restricción en la ruta:
                    </p>
                    
                    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">routes.MapRoute(<span style="color: #006080">"Default"</span>, <span style="color: #006080">"{controller}/{action}/{id}"</span>,<br />    <span style="color: #0000ff">new</span> { controller = <span style="color: #006080">"Home"</span>, action = <span style="color: #006080">"Index"</span>, id = <span style="color: #006080">""</span> },<br />    <span style="color: #0000ff">new</span> { controller = <span style="color: #0000ff">new</span> NotEqualConstraint(<span style="color: #006080">"api"</span>) });</pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          Ahora sí que sí. Los enlaces /api/Foo/XX <strong>no pueden ser procesados</strong> por la ruta Default, ya que no se cumple mi restricción (controller vale api), y entonces son procesados por la siguiente ruta (que es lo que queríamos). Ahora pues la url /Map/XX es procesada por la primera ruta y la URL /api/Map/XX es procesada por la segunda y enrutada al controlador WarMap.
                        </p>
                        
                        <p>
                          Espero que estos dos posts os hayan ayudado a ver la potencia del sistema de rutas de ASP.NET MVC!
                        </p>
                        
                        <p>
                          Un saludo a todos!
                        </p>