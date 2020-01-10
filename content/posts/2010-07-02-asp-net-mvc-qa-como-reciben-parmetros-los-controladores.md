---
title: 'ASP.NET MVC Q&A: ¿Como reciben parámetros los controladores?'

author: eiximenis

date: 2010-07-02T12:04:06+00:00
geeks_url: /?p=1523
geeks_visits:
  - 14767
geeks_ms_views:
  - 4021
categories:
  - Uncategorized

---
Este es el segundo post de la serie que nace a raíz de las preguntas que se me realizaron en el Webcast que di sobre ASP.NET MVC.

Estaba explicando la tabla de rutas por “defecto” de ASP.NET MVC, indicando que el primer valor era el controlador, el segundo la acción y el tercero un parámetro llamado id:

<!--more-->

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">routes.MapRoute(<br />    <span style="color: #006080">"Default"</span>, <span style="color: #008000">// Nombre de la ruta</span><br />    <span style="color: #006080">"{controller}/{action}/{id}"</span>,  <span style="color: #008000">// Formato de url /Controlador/Accion/Id</span><br />    <span style="color: #0000ff">new</span> { controller = <span style="color: #006080">"Home"</span>, action = <span style="color: #006080">"Index"</span>, id = UrlParameter.Optional } <span style="color: #008000">// Valores x defecto</span><br />);</pre>
  
  <p>
    </div> 
    
    <p>
      Comenté que dada esa tabla de rutas una url /Home/Index/100 se mapeaba a l’acción Index del controlador Home con un parámetro id a valor 100 (parámetro que era opcional).
    </p>
    
    <p>
      En el Webcast no comenté nada más… voy a ampliar el tema ahora, porque es realmente interesante.
    </p>
    
    <p>
      <strong>1. Acciones y tabla de rutas</strong>
    </p>
    
    <p>
      Vamos a aclarar algunos puntos que son fundamentales relativos a las acciones y la tabla de rutas:
    </p>
    
    <ol>
      <li>
        Las acciones se distinguen <strong>por su nombre</strong>, sin tener en cuenta sus parámetros. Eso significa que un controlador <strong>no puede definir dos veces la misma acción con parámetros distintos</strong>, salvo que se responda a verbos http distintos (p.ej. una acción responda a GET y otra a POST).
      </li>
      <li>
        La tabla de rutas mapea una URL a una única acción de un controlador.
      </li>
      <li>
        La tabla de rutas tiene una o más rutas, que se evalúan en orden. Cuando una URL hace <em>matching</em> con una ruta, se termina la evaluación.
      </li>
    </ol>
    
    <p>
      Así el siguiente código <strong>es erróneo</strong>:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> HomeController : Controller<br />{<br />    <span style="color: #0000ff">public</span> ActionResult Index()<br />    {<br />        <span style="color: #0000ff">return</span> View();<br />    }<br />    <span style="color: #0000ff">public</span> ActionResult Index(<span style="color: #0000ff">int</span> id)<br />    {<br />        <span style="color: #0000ff">return</span> View();<br />    }<br />}</pre>
      
      <p>
        </div> 
        
        <p>
          Uno podría pensar que la tabla de rutas enrutaría /Home/Index al método Index() sin parámetros y /Home/Index/100 al método Index() que acepta un entero, pero no. <strong>La tabla de rutas enruta tanto /Home/Index/ como /Home/Index/100 a la acción Index</strong>. Luego el framework cuando va a invocarla se encuentra con dos métodos que la implementan y da un error: <i>The current request for action &#8216;Index&#8217; on controller type &#8216;HomeController&#8217; is ambiguous between the following action methods: System.Web.Mvc.ActionResult Index() on type MvcControllerParams.Controllers.HomeController System.Web.Mvc.ActionResult Index(Int32) on type MvcControllerParams.Controllers.HomeController</i>
        </p>
        
        <p>
          <strong>2. Parámetros opcionales en las rutas</strong>
        </p>
        
        <p>
          ¿Y que ocurriría con el siguiente código?
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> HomeController : Controller<br />{<br />    <span style="color: #0000ff">public</span> ActionResult Index(<span style="color: #0000ff">int</span> id)<br />    {<br />        <span style="color: #0000ff">return</span> View();<br />    }<br />}</pre>
          
          <p>
            </div> 
            
            <p>
              La tabla de rutas enruta /Home/Index/100 a dicha acción y el parámetro id vale 100… pero que ocurre si entramos /Home/Index?
            </p>
            
            <p>
              Pues que da error: El parámetro id está declarado como opcional en la ruta, eso significa que puede no aparecer en la url. Así pues la url /Home/Index es válida y se enruta a la acción Index. Y cuando el framework se encuentra que quiere pasarle un valor al parámetro id, <strong>no puede</strong> porque no hay parámetro id en la url <strong>y int no es nullable</strong>. El mensaje de error que da es:
            </p>
            
            <p>
              <em>The parameters dictionary contains a null entry for parameter &#8216;id&#8217; of non-nullable type &#8216;System.Int32&#8217; for method &#8216;System.Web.Mvc.ActionResult Index(Int32)&#8217; in &#8216;MvcControllerParams.Controllers.HomeController&#8217;. An optional parameter must be a reference type, a nullable type, or be declared as an optional parameter.</em>
            </p>
            
            <p>
              La solución? Pues declarar que el parámetro id sea nullable:
            </p>
            
            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> HomeController : Controller<br />{<br />    <span style="color: #0000ff">public</span> ActionResult Index(<span style="color: #0000ff">int</span>? id)<br />    {<br />        <span style="color: #0000ff">return</span> View();<br />    }<br />}</pre>
              
              <p>
                </div> 
                
                <p>
                  Fíjate en el uso de <strong>int?</strong> (que es la sintaxis que da C# para usar <a href="http://msdn.microsoft.com/en-us/library/system.nullable.aspx">Nullable</a><int>).
                </p>
                
                <p>
                  Otra posible opción seria modificar la tabla de rutas para que en lugar de declarar el parámetro simplemente como opcional, asignarle un valor por defecto:
                </p>
                
                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">routes.MapRoute(<br />    <span style="color: #006080">"Default"</span>, <span style="color: #008000">// Route name</span><br />    <span style="color: #006080">"{controller}/{action}/{id}"</span>, <span style="color: #008000">// URL with parameters</span><br />    <span style="color: #0000ff">new</span> { controller = <span style="color: #006080">"Home"</span>, action = <span style="color: #006080">"Index"</span>, id = 0 } <span style="color: #008000">// Parameter defaults</span><br />);</pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Fíjate que donde antes ponía <em>id = UrlParameter.Optional</em> ahora pone <em>id = 0</em>. La diferencia es sutil pero importante: antes si id no aparecía el framework asumía que no tenía valor, ahora si el parámetro no aparece el framework le asigna un valor por defecto de 0. Así, ahora en el controlador podemos volver a usar int en lugar de Nullable<int> (int? en c#).
                    </p>
                    
                    <p>
                      El “pero” de esta opción es que no puedes distinguir entre /Home/Index y /Home/Index/0. Ambas URLs invocan la acción Index con el valor id=0.
                    </p>
                    
                    <p>
                      Bien… ¿y que ocurrirá si entro la url /Home/Index/eiximenis? Pues que la tabla de rutas mapeará esta url a la acción Index y <strong>cuando el framework vaya a invocarla intentará convertir “eiximenis” a int y dicha conversión devolverá null al no ser posible</strong>. Así:
                    </p>
                    
                    <ul>
                      <li>
                        Si teníamos Index(int id) el framework dará error, indicando que no puede asignar “null” a int.
                      </li>
                      <li>
                        Si teníamos Index (int? id) recibiremos el valor null en el parámetro id. Efectivamente en este caso es lo mismo /Home/Index que /Home/Index/eiximenis
                      </li>
                    </ul>
                    
                    <p>
                      Si en lugar de declarar la acción con un parámetro int la declaro que recibe una cadena:
                    </p>
                    
                    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> ActionResult Index(<span style="color: #0000ff">string</span> id)<br />{<br />    <span style="color: #0000ff">return</span> View();<br />}</pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          Todo es mucho más sencillo:
                        </p>
                        
                        <ul>
                          <li>
                            /Home/Index –> Invoca Index() con id = null
                          </li>
                          <li>
                            /Home/Index/100 –> Invoca Index() con id = “100”
                          </li>
                          <li>
                            /Home/Index/eiximenis –> Invoca Index() con id=”eiximenis”
                          </li>
                        </ul>
                        
                        <p>
                          <strong>3. Acciones sin parámetros opcionales</strong>
                        </p>
                        
                        <p>
                          Vamos a cambiar de nuevo la tabla de rutas para que quede:
                        </p>
                        
                        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">routes.MapRoute(<br />    <span style="color: #006080">"Default"</span>, <span style="color: #008000">// Route name</span><br />    <span style="color: #006080">"{controller}/{action}/{id}"</span>, <span style="color: #008000">// URL with parameters</span><br />    <span style="color: #0000ff">new</span> { controller = <span style="color: #006080">"Home"</span>, action = <span style="color: #006080">"Index"</span> } <span style="color: #008000">// Parameter defaults</span><br />);</pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              Fíjate que hemos eliminado la parte donde ponía <em>id = UrlParameter.Optional</em>. Así ahora el parámetro id es obligatorio en la ruta.
                            </p>
                            
                            <p>
                              La url /Home/Index/eiximenis nos invoca la acción Index() con id=”eiximenis” pero que hace la url /Home/Index? Pues devuelve un 404. ¿Por que? Bien, cuando una URL no satisface ninguna de las entradas de la tabla de rutas, el framework devuelve un 404 para indicar que la URL no se corresponde a ningún recurso. En nuestro caso la tabla de rutas tiene una sola entrada que obliga que las urls tengan la forma /controlador/accion/id. Y dado que id no se ha declarado opcional ni se le ha dado ningún valor por defecto, debe aparecer explícitamente en la URL. Como en nuestro caso no aparece, la URL /Home/Index no cumple esta entrada en la tabla de rutas y por eso recibimos el 404.
                            </p>
                            
                            <p>
                              <strong>4. Rutas con más de un parámetro</strong>
                            </p>
                            
                            <p>
                              La pregunta explícita que me hicieron en el Webcast fue “Y se puede pasar más de un parámetro”? Mi respuesta en aquel momento fue simplemente que sí, añadiendolos a la tabla de rutas:
                            </p>
                            
                            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">routes.MapRoute(<br />    <span style="color: #006080">"Default"</span>, <span style="color: #008000">// Route name</span><br />    <span style="color: #006080">"{controller}/{action}/{id}/{desc}"</span>, <span style="color: #008000">// URL with parameters</span><br />    <span style="color: #0000ff">new</span> { controller = <span style="color: #006080">"Home"</span>, action = <span style="color: #006080">"Index"</span>, <br />        id = UrlParameter.Optional, desc = UrlParameter.Optional } <span style="color: #008000">// Parameter defaults</span><br />);</pre>
                              
                              <p>
                                </div> 
                                
                                <p>
                                  Ahora la ruta tiene dos parámetros (ambos opcionales) id y desc.
                                </p>
                                
                                <p>
                                  En el controlador definimos la acción con los dos parámetros:
                                </p>
                                
                                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> ActionResult Index(<span style="color: #0000ff">string</span> id, <span style="color: #0000ff">string</span> desc)<br />{<br />    <span style="color: #0000ff">return</span> View();<br />}</pre>
                                  
                                  <p>
                                    </div> 
                                    
                                    <p>
                                      Tal y como lo tenemos:
                                    </p>
                                    
                                    <ul>
                                      <li>
                                        /Home/Index –> Invoca a Index con id=null y desc=null
                                      </li>
                                      <li>
                                        /Home/Index/eiximenis –> Invoca a Index con id=”eiximenis” y desc=null
                                      </li>
                                      <li>
                                        /Home/Index/eiximenis/edu –> Invoca a Index con id=”eiximenis” y desc=”edu”
                                      </li>
                                    </ul>
                                    
                                    <p>
                                      De esta manera podemos pasar más de un parámetro a los controladores.
                                    </p>
                                    
                                    <p>
                                      <strong>5. Otros parámetros (querystrings,…)</strong>
                                    </p>
                                    
                                    <p>
                                      Bueno… vamos a liarla un poco más! Tal y como lo tenemos que ocurre con la url <em>/Home/Index/eiximenis?desc=edu</em> ?
                                    </p>
                                    
                                    <p>
                                      Pues veamos: La tabla de rutas <strong>no</strong> usa la querystring, así que para la tabla de rutas es como si hubiesemos entrado /Home/Index/eiximenis, lo que se mapea a la acción Index. Luego cuando el framework va a invocar dicha acción:
                                    </p>
                                    
                                    <ul>
                                      <li>
                                        Pone el valor de id a “eiximenis” (primer valor de la url)
                                      </li>
                                      <li>
                                        El valor de la ruta para desc és null (recordad: la tabla de rutas no entiende de querystring). Entonces se analiza la querystring y el framework ve que el controlador acepta otro parámetro llamado desc y que en la querystring aparece dicho parámetro, así que le asigna valor.
                                      </li>
                                    </ul>
                                    
                                    <p>
                                      Así por lo tanto, la url /Home/Index/eiximenis?desc=edu llama a l’acción Index con el valor de id a “eiximenis” y el valor de desc a “edu”.
                                    </p>
                                    
                                    <p>
                                      Fijaos pues que las urls<em> /Home/Index/eiximenis/edu</em> y <em>/Home/Index/eiximenis?desc=edu</em> son equivalentes, puesto que ambas terminan llamando a la misma acción con los dos parámetros… pero que sean equivalentes no quiere decir que sean indistinguibles. Desde el controlador podemos saber si los parámetros nos vienen via ruta o bien via querystring:
                                    </p>
                                    
                                    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> ActionResult Index(<span style="color: #0000ff">string</span> id, <span style="color: #0000ff">string</span> desc)<br />{<br />    var routedesc = <span style="color: #0000ff">this</span>.RouteData.Values[<span style="color: #006080">"desc"</span>];<br />    <span style="color: #0000ff">return</span> View();<br />}</pre>
                                      
                                      <p>
                                        </div> 
                                        
                                        <p>
                                          El valor de routdesc será “edu” si venimos por la url /Home/Index/eiximenis/edu y null si venimos por la url /Home/Index/eiximenis?desc=edu
                                        </p>
                                        
                                        <p>
                                          <strong>Nota:</strong> Todo lo que hemos dicho sobre la querystring aplica también a parámetros POST.
                                        </p>
                                        
                                        <p>
                                          <strong>6. Bindings</strong>
                                        </p>
                                        
                                        <p>
                                          Y vamos ya con la última… Imagina que tenemos una clase así:
                                        </p>
                                        
                                        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> FooModel<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> id { get; set; }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> desc { get; set; }<br />}</pre>
                                          
                                          <p>
                                            </div> 
                                            
                                            <p>
                                              Y declaramos nuestra acción para que reciba, no dos strings sinó un objeto FooModel:
                                            </p>
                                            
                                            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> ActionResult Index(FooModel m)<br />{<br />    <span style="color: #0000ff">return</span> View();<br />}</pre>
                                              
                                              <p>
                                                </div> 
                                                
                                                <p>
                                                  Ahora en este caso la url:
                                                </p>
                                                
                                                <ul>
                                                  <li>
                                                    /Home/Index –> Invoca a Index() con un objeto FooModel con id=null y desc=null
                                                  </li>
                                                  <li>
                                                    /Home/Index/eiximenis –> Invoca a Index() con un objeto FooModel con id=”eiximenis” y desc=null
                                                  </li>
                                                  <li>
                                                    /Home/Index/eiximenis/edu –> Invoca a Index() con un objeto FooModel con id=”eiximenis” y desc=”edu”
                                                  </li>
                                                  <li>
                                                    /Home/Index/eiximenis?desc=edu –> Invoca a Index() con un objeto FooModel con id=”eiximenis” y desc=”edu”
                                                  </li>
                                                </ul>
                                                
                                                <p>
                                                  Que te parece? El framework es capaz de convertir los parámetros (vengan de la ruta o del querystring) en el objeto que espera la acción, instanciando sus propiedades!
                                                </p>
                                                
                                                <p>
                                                  Y bueno… vamos a dejarlo aquí por el momento 🙂 Espero que ahora tengáis un poco más claro como funciona el “paso de parámetros” a los controladores.
                                                </p>
                                                
                                                <p>
                                                  Un saludo a todos! 😉
                                                </p>
                                                
                                                <p>
                                                  <strong>Una postdata para quien quiera profundizar</strong>
                                                </p>
                                                
                                                <p>
                                                  Si quieres profundizar en este tema, antes que nada ten presente los siguientes puntos:
                                                </p>
                                                
                                                <ul>
                                                  <li>
                                                    A través de la tabla de rutas el framework decide que acción implementa la URL
                                                  </li>
                                                  <li>
                                                    Antes de invocar una acción, los Value Providers entran en acción, procesando los valores de la request. Los valores de la request pueden estar en:
                                                  </li>
                                                  <ul>
                                                    <li>
                                                      RouteData (puestos por la tabla de rutas)
                                                    </li>
                                                    <li>
                                                      QueryString
                                                    </li>
                                                    <li>
                                                      POST
                                                    </li>
                                                    <li>
                                                      Cualquier otro sitio (p.ej. cookies). Se pueden crear value providers propios.
                                                    </li>
                                                  </ul>
                                                  
                                                  <li>
                                                    Antes de invocar la acción, el ModelBinder entra en acción: coge los valores que han procesado los Value Providers y los intenta mapear a los parámetros de la acción.
                                                  </li>
                                                  <li>
                                                    Se invoca la acción del controlador.
                                                  </li>
                                                </ul>
                                                
                                                <p>
                                                  Ahora algunos links:
                                                </p>
                                                
                                                <ul>
                                                  <li>
                                                    <a href="http://weblogs.asp.net/scottgu/archive/2007/12/03/asp-net-mvc-framework-part-2-url-routing.aspx">La tabla de rutas (por Sott Guthrie)</a>. Es sobre MVC1 pero es útil igualmente (aunque recuerda que no hay UrlParameter.Optional)
                                                  </li>
                                                  <li>
                                                    <a href="http://geeks.ms/blogs/etomas/archive/2010/05/07/asp-net-mvc-valueproviders.aspx">Post mío sobre los Value Providers</a>.
                                                  </li>
                                                  <li>
                                                    <a href="http://geeks.ms/blogs/etomas/archive/2010/05/10/asp-net-mvc-el-defaultmodelbinder.aspx">Post mío sobre el DefaultModelBinder</a> (el Model Binder por defecto que usa el framework)
                                                  </li>
                                                  <li>
                                                    Interesantes posts de MehdiGolchin sobre <a href="http://mgolchin.net/posts/20/dive-deep-into-mvc-imodelbinder-part-1">Model binders</a> y <a href="http://mgolchin.net/posts/19/dive-deep-into-mvc-ivalueprovider">value providers</a>.
                                                  </li>
                                                </ul>