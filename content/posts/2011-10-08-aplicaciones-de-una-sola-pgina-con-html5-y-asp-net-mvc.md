---
title: Aplicaciones "de una sola página” con HTML5 y ASP.NET MVC
description: Aplicaciones "de una sola página” con HTML5 y ASP.NET MVC
author: eiximenis

date: 2011-10-08T11:29:00+00:00
geeks_url: /?p=1579
geeks_visits:
  - 12446
geeks_ms_views:
  - 1865
categories:
  - Uncategorized

---
Muy buenas!

Cada vez más nos encontramos con aplicaciones web que funcionan &ldquo;en una sola página&rdquo;, es decir que se carga la página inicial y luego todas las nuevas peticiones son via AJAX. Esas aplicaciones funcionan perfectamente hasta que el usuario le daba a atr&agrave;s o a F5 para refrescar la página: en este momento se pierde el estado de la navegación.

Hasta ahora no había una manera estándard y sencilla para lidiar con esto, pero HTML5 ya está aquí y incluye una nueva API de historial que nos va ayudar con estos casos. Aunque hay más, voy a centrarme en este post en dos elementos de dicha API:

  * El evento popstate. Este evento de window se lanza _cuando se navega a una dirección del historial_. Bueno, sí, la definición es un poco ambigua pero básicamente traducido es que se lanza _cuando se pulsa el botón de atrás en el navegador_. 
  * El método pushState del objeto history. Este método permite _meter una entrada en el historial_. Esa entrada consta de un objeto con datos arbitrarios y una url. Cuando llamemos a pushState la barra de direcciones se modificará para mostrar la nueva url, pero el navegador **no navegará hacia allí**. En su lugar habrá añadido una entrada ficticia en el historial con los datos que nosotros le hayamos indicado. Cuando se pulse atrás en el navegador, se lanzará el evento popstate y en él podremos recuperar esos datos y simular lo que sea que tenga que simularse para darle la sensación al usuario de que el botón de atrás funciona. 

Sí, parece un poco lioso, pero en este post veremos como realizar una aplicación ASP.NET MVC que:

  1. Muestre una lista de productos y enlaces a los detalles 
  2. Al pulsar en un detalle se muestre una página del producto 
  3. Todo el refresco sea via ajax 
  4. Al pulsar F5 el usuario _se queda donde está._ Es decir, si estaba viendo los detalles del producto 2 continuará viéndolos. 
  5. El funcionamiento de back y forward será el esperado. 
  6. En la barra de direcciones se mostrará la URL real que se está visitando, aunque esta se haya cargado via ajax. 

En resumen, nuestra aplicación se va a comportar exactamente como se espera de una aplicación que no es ajax salvo que... usará ajax con todas las ventajas (menos refresco y mayor velocidad) que conlleva. Y lo mejor... no nos costará mucho hacerlo. &iexcl;Viva HTML5!

**1. La estrategia general**

Tenemos que definir la estrategia general: por un lado mientras naveguemos por la aplicación todo serán peticiones ajax, pero si el usuario le da a F5, entonces ya no tendremos una petición ajax. En su lugar tendremos una petición estándard que tendremos que gestionar. Por lo tanto nuestros controladores deberán estar capacitados para servir la misma información les venga la petición por Ajax, o les venga de forma tradicional.

Para solventar esto, nos basta con meter todo el contenido en una vista _parcial_ y devolver la vista parcial cuando la petición sea via Ajax. Cuando la petición sea tradicional entonces devolveremos una vista normal que lo único que hará será renderizar la vista parcial.

P.ej. Este es el código de la vista Home/Index.cshtml:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">@{<br />    ViewBag.Title = "Indice";<br />}<br /><br />@{Html.RenderPartial("Index_partial");}</pre>
  
  <p>
    </div> 
    
    <p>
      Simplemente renderizar Index_partial que es donde habrá el contenido. Luego nos basta un método en el controlador como el siguiente:
    </p>
    
    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> ActionResult Index()<br /> {<br />     <span style="color: #0000ff">return</span> Request.IsAjaxRequest() ?<br />          (ActionResult)PartialView(<span style="color: #006080">"Index_partial"</span>) :<br />          (ActionResult)View();<br /> }</pre>
      
      <p>
        </div> 
        
        <p>
          Esa estrategia la repetiremos en todas las acciones de los controladores.
        </p>
        
        <p>
          Bien, vayamos ahora a por las vistas. Este es el código de la vista Index_partial:
        </p>
        
        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="source"</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">h2</span><span style="color: #0000ff">&gt;</span>Index<span style="color: #0000ff">&lt;/</span><span style="color: #800000">h2</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">a</span> <span style="color: #ff0000">href</span><span style="color: #0000ff">="@Url.Action("</span><span style="color: #ff0000">List</span><span style="color: #0000ff">", "</span><span style="color: #ff0000">Products</span><span style="color: #0000ff">")"</span> <span style="color: #ff0000">data-ajax</span><span style="color: #0000ff">="true"</span> <span style="color: #0000ff">&gt;</span>Ver productos<span style="color: #0000ff">&lt;/</span><span style="color: #800000">a</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">br</span> <span style="color: #0000ff">/&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span></pre>
          
          <p>
            </div> 
            
            <p>
              Tan sólo un enlace, con el atributo data-ajax=&rdquo;true&rdquo; y un div llamado source. El atributo data-ajax a true es importante porque es el que usaremos para <em>interceptar</em> este enlace y cargarlo via ajax. Por otro lado el div source también es importante porque es <em>donde</em> vamos a poner el contenido que nos venga de la petición ajax: es decir machacaremos todo nuestro contenido con el que nos venga de la petición ajax.
            </p>
            
            <p>
              Con eso, ya tenemos la estrategia montada: Vamos a tener una vista &ldquo;normal&rdquo; y una parcial por cada acción y las vistas parciales serán las que realmente muestren el contenido.
            </p>
            
            <p>
              <strong>2. Interceptar las llamadas a los links</strong>
            </p>
            
            <p>
              Para esto vamos a valernos del atributo data-ajax que hemos creado antes. Este atributo es un atributo que me he inventado yo. En HTML5 te puedes inventar los atributos que te da la gana, siempre que empiecen por data-. Es una forma de estandarizar lo que antes hacíamos con clases CSS o bien inventándonos atributos. Pero a diferencia de usar una clase CSS (que usa algo pensado para aspecto para comportamiento) o inventarse un atributo cualquiera (que hace que el código deje de ser HTML válido) usar un atributo data-* no tiene ninguna contra-indicación. Así funcionan muchas de las características de HTML5: se han cogido muchas cosas que se hacían antes y se ha buscado una manera estándard de hacerlas!
            </p>
            
            <p>
              Bueno, al tajo, vamos a crearnos un archivo myscripts.js al que vamos a meter algunas funciones, empezando por lo siguiente:
            </p>
            
            <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
              <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">function</span> bindLinks() {<br />    $(<span style="color: #006080">"a[data-ajax]"</span>).each(<span style="color: #0000ff">function</span> () {<br />        $(<span style="color: #0000ff">this</span>).click(<span style="color: #0000ff">function</span> (evt) {<br />            evt.preventDefault();<br />            ajaxload($(<span style="color: #0000ff">this</span>).attr(<span style="color: #006080">"href"</span>), <span style="color: #0000ff">true</span>);<br />        });<br />    });<br />}</pre>
              
              <p>
                </div> 
                
                <p>
                  La función bindLinks recorre todos los tags <a> que tengan el atributo data-ajax y les asigna una función gestora al evento click. Esa función gestora lo que hace es evitar que actúe el click estándard (por lo que NO navegaremos a través del link) y luego llama a ajaxload que es un método que veremos a continuación pasándole el valor del atributo href del tag <a> pulsado.
                </p>
                
                <p>
                  Ya tenemos todos los clicks de los enlaces interceptados. Para que esto se ejecute nos basta con asegurar que al cargar la página se llame a bindLinks. Para esto en la página de Layout añadimos:
                </p>
                
                <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">&lt;script type=<span style="color: #006080">"text/javascript"</span>&gt;<br />    $(document).ready(<span style="color: #0000ff">function</span> () {<br />        bindLinks();<br />    });<br />&lt;/script&gt;</pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Finalmente la definición de la función ajaxload es casi trivial, ya que lo único que hace es llamar a load() de jQuery para cargar los datos y meterlos dentro del div <em>source</em> que hemos mencionado antes.
                    </p>
                    
                    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">function</span> ajaxload(url, add) {<br />    $(<span style="color: #006080">"#source"</span>).load(url, <span style="color: #0000ff">function</span> () {<br />        bindLinks();<br />    });<br />}</pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          El único detalle es que luego llama de nuevo a bindLinks() para volver a interceptar los clicks de los nuevos tags <a> que hayan aparecido!
                        </p>
                        
                        <p>
                          <strong>3. Simular la modificación de la barra de direcciones</strong>
                        </p>
                        
                        <p>
                          Vale, tenemos un enlace, que nos debería dirigir a una cierta URL, pongamos /Productos/Ver/10 pero en lugar de seguir el enlace, lo estamos cargando via ajax. Si queremos que el F5 funcione correctamente, debemos &ldquo;modificar&rdquo; la barra de direcciones, para que aparezca la URL &ldquo;real&rdquo;. Esto, que antes no se podía hacer, ahora es posible con la nueva <a target="_blank" href="http://www.w3.org/TR/html5/history.html" rel="noopener noreferrer">History API</a> de HTML5.
                        </p>
                        
                        <p>
                          Cada vez que carguemos un enlace via Ajax añadiremos una entrada en el historial. Para ello usaremos el método pushState del objeto history. A este método se le pasan tres parámetros:
                        </p>
                        
                        <ol>
                          <li>
                            Un objeto de estado. Es un objeto javascript cualquiera.
                          </li>
                          <li>
                            Un título
                          </li>
                          <li>
                            Una URL
                          </li>
                        </ol>
                        
                        <p>
                          El objeto de estado es una de las partes más importantes: cuando el usuario pulse el botón de <em>atrás</em> vamos a recibir el evento popstate y en este evento tendremos el objeto de estado. En este objeto pues nosotros podemos poner toda aquella información necesaria para que luego, en el evento popstate, podamos &ldquo;deshacer&rdquo; los cambios y darle la ilusión al usuario de que el botón de atrás funciona bien.
                        </p>
                        
                        <p>
                          La URL que pasemos es la URL que se mostrará en la barra de direcciones. Pero sólo se mostrará, el navegador <strong>no navegará hacia allí</strong>. De nuevo es para <em>engañar</em> al usuario y hacerle creer que realmente ha ido a otra URL cuando en realidad simplemente hemos modificado el contenido de la página usando javascript (ajax).
                        </p>
                        
                        <p>
                          Para modificar la barra de direcciones nos basta un pequeño añadido a la función <em>ajaxload</em> que teníamos antes:
                        </p>
                        
                        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">function</span> ajaxload(url, add) {<br />    history.pushState({ uri: url }, <span style="color: #006080">''</span>, url);<br />    $(<span style="color: #006080">"#source"</span>).load(url, <span style="color: #0000ff">function</span> () {<br />        bindLinks();<br />    });<br />}</pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              Cada vez que cargamos una vista via Ajax, añadimos una entrada en el historial. El primer parámetro es el objeto de estado. En este caso simplemente colocamos la URL &ldquo;a la que navegamos&rdquo; (simplemente porque no necesitaremos nada más). El tercer parámetro es la URL que mostrará el navegador en la barra de direcciones. Pero recordad: el navegador NO irá a esa URL (nosotros estamos cargando su contenido por ajax).
                            </p>
                            
                            <p>
                              <strong>4. Soporte para back</strong>
                            </p>
                            
                            <p>
                              Al modificar la barra de direcciones hemos dado soporte a F5. Porque al pulsar F5 el navegador refrescará la última entrada del historial que ahora contiene la URL que hemos puesto antes. Y recordad que nosotros antes hemos preparado los controladores para responder por Ajax y para responder a peticiones normales: por lo tanto en este punto tenemos una aplicación que se refresca totalmente por ajax... pero con soporte para F5. ¿No os parece genial?
                            </p>
                            
                            <p>
                              Pues ahora vamos a añadir el soporte para el botón de atrás. Para ello nos vamos a basar en el evento popstate que se lanza cuando el navegador debe navegar a otra entrada del historial (básicamente cuando se pulsa el botón de back). En este evento recibimos el objeto de estado de la posición de historial que se está eliminando.
                            </p>
                            
                            <p>
                              Vamos a modificar la página de Layout para añadir un manejador al evento popstate:
                            </p>
                            
                            <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                              <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">&lt;script type=<span style="color: #006080">"text/javascript"</span>&gt;<br />    $(document).ready(<span style="color: #0000ff">function</span> () {<br />        $(window).bind(<span style="color: #006080">'popstate'</span>, <span style="color: #0000ff">function</span> (evt) {<br />            doPopstate(evt.originalEvent.state);<br />        });<br />        bindLinks();<br />    });<br />&lt;/script&gt;<br /></pre>
                              
                              <p>
                                </div> 
                                
                                <p>
                                  Usamos el método bind() de jQuery para enlazar un manejador al evento popstate del objeto window y llamar a doPopstate, pasándole la propiedad <em>state</em> del evento (que es el objeto de estado).
                                </p>
                                
                                <p>
                                  La función doPopstate es nuestra y tiene el siguiente código:
                                </p>
                                
                                <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                                  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">function</span> doPopstate(data) {<br />    <span style="color: #0000ff">if</span> (data != <span style="color: #0000ff">null</span>) {<br />        ajaxload(data.uri, <span style="color: #0000ff">false</span>);<br />    }<br />}</pre>
                                  
                                  <p>
                                    </div> 
                                    
                                    <p>
                                      Si tenemos objeto de estado (teoricamente debemos tenerlo SIEMPRE, pero Chrome p.ej. al cargar una página por primera vez lanza un popstate sin objeto de estado, mientras que Firefox no lo lanza. Personalmente creo que es comportamiento de Firefox el correcto), nos limitamos a cargar via ajax la URL de este objeto de estado. Dado que están apilados será la URL <em>anterior</em> a la que estamos.
                                    </p>
                                    
                                    <p>
                                      En este punto hemos tenido que añadir un parámetro a ajaxload (el booleano), para evitar que ajaxload nos añada una entrada de historial si estamos yendo hacia atrás. El nuevo código de ajaxload queda así:
                                    </p>
                                    
                                    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                                      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">function</span> ajaxload(url, add) {<br />    <span style="color: #0000ff">if</span> (add) {<br />        history.pushState({ uri: url }, <span style="color: #006080">''</span>, url);<br />    }<br />    $(<span style="color: #006080">"#source"</span>).load(url, <span style="color: #0000ff">function</span> () {<br />        bindLinks();<br />    });<br />}</pre>
                                      
                                      <p>
                                        </div> 
                                        
                                        <p>
                                          Por supuesto también modificamos la llamada a ajaxload dentro del manejador del evento de click de los enlaces en bindLinks(), para pasarle un true. Y con eso... hemos terminado.
                                        </p>
                                        
                                        <p>
                                          Para entender exactamente que pasa he modificado ligeramente ajaxload y doPopstate para que hagan un log de lo que ocurre en un <div> de la página de Layout. Y eso es un poco más o menos lo que ocurre...
                                        </p>
                                        
                                        <p>
                                          <a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_2B89DA1E.png"><img height="142" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_61AB526D.png" alt="image" border="0" title="image" style="background-image: none; border-bottom: 0px; border-left: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" /></a><a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_586F172C.png"><img height="142" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_238211EF.png" alt="image" border="0" title="image" style="background-image: none; border-bottom: 0px; border-left: 0px; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" /></a>
                                        </p>
                                        
                                        <p>
                                          <a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_15CF55E7.png"><img height="148" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_64806B86.png" alt="image" border="0" title="image" style="background-image: none; border-bottom: 0px; border-left: 0px; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" /></a><a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_48FB6983.png"><img height="148" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_1F37EE90.png" alt="image" border="0" title="image" style="background-image: none; border-bottom: 0px; border-left: 0px; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" /></a>
                                        </p>
                                        
                                        <ol>
                                          <li>
                                            Lanzamos la aplicación
                                          </li>
                                          <li>
                                            Pulsamos &ldquo;Ver productos&rdquo;
                                          </li>
                                          <li>
                                            Pulsamos &ldquo;Detalle del producto 2&rdquo;
                                          </li>
                                          <li>
                                            Pulsamos &ldquo;Inicio&rdquo;
                                          </li>
                                        </ol>
                                        
                                        <p>
                                          Fijaos como en todo este tiempo el reloj de &ldquo;Tiempo Actual&rdquo; es siempre el mismo (estamos cargando via Ajax) y como la barra de direcciones va cambiando. También podéis ver el log en la parte inferior. Continuemos...
                                        </p>
                                        
                                        <p>
                                          <a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_5CE4D64C.png"><img height="157" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_005154E5.png" alt="image" border="0" title="image" style="background-image: none; border-bottom: 0px; border-left: 0px; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" /></a><a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_76A8E6AE.png"><img height="169" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_01199502.png" alt="image" border="0" title="image" style="background-image: none; border-bottom: 0px; border-left: 0px; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" /></a>
                                        </p>
                                        
                                        <p>
                                          <a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_6E34EB8A.png"><img height="169" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_58F6C015.png" alt="image" border="0" title="image" style="background-image: none; border-bottom: 0px; border-left: 0px; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" /></a><a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_761C681F.png"><img height="169" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_530BF6AF.png" alt="image" border="0" title="image" style="background-image: none; border-bottom: 0px; border-left: 0px; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" /></a>
                                        </p>
                                        
                                        <ol>
                                          <li>
                                            Pulsamos &ldquo;Ver productos&rdquo;
                                          </li>
                                          <li>
                                            Pulsamos &ldquo;Detalles del producto 1&rdquo;
                                          </li>
                                          <li>
                                            Pulsamos BACK &ndash;> Fijaos en este punto en la aparición del evento popState y como recuperamos el evento del historial anterior.
                                          </li>
                                          <li>
                                            Pulsamos BACK de nuevo &ndash;> Otro evento popState y recuperamos la posición <em>anterior</em>.
                                          </li>
                                        </ol>
                                        
                                        <p>
                                          De nuevo en todo este tiempo el reloj de &ldquo;Tiempo actual&rdquo; se ha movido: todo son refrescos Ajax. Sigamos...
                                        </p>
                                        
                                        <p>
                                          <a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_74A81F80.png"><img height="184" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_06380A4C.png" alt="image" border="0" title="image" style="background-image: none; border-bottom: 0px; border-left: 0px; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" /></a><a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_2A10BBD9.png"><img height="184" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_381F8509.png" alt="image" border="0" title="image" style="background-image: none; border-bottom: 0px; border-left: 0px; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" /></a>
                                        </p>
                                        
                                        <ol>
                                          <li>
                                            Pulsamos &ldquo;Ver Productos&rdquo;
                                          </li>
                                          <li>
                                            Pulsamos F5 &ndash;> En este punto se lanza una petición &ldquo;real&rdquo;. Podeis ver como el reloj de &ldquo;Tiempo actual&rdquo; se ha modificado y el log ha desaparecido (normal se ha cargado toda la Layout de nuevo). Pero nos hemos quedado en la página donde estábamos (el listado de productos).
                                          </li>
                                        </ol>
                                        
                                        <p>
                                          Bueno... os dejo el proyecto en VS2010 para que juguéis con él e investiguéis un poco como funciona el History API de HTML5.
                                        </p>
                                        
                                        <p>
                                          <strong>Importante:</strong> El proyecto lo he probado con:
                                        </p>
                                        
                                        <ol>
                                          <li>
                                            Firefox 7.0 y funciona correctamente
                                          </li>
                                          <li>
                                            Chrome 15.0.874.83 y funciona correctamente
                                          </li>
                                          <li>
                                            <span style="color: #ff0000;">Internet Explorer 9 y <strong>no funciona </strong>(no tiene soporte para History API)</span>
                                          </li>
                                          <li>
                                            Opera 11.5 y funciona correctamente
                                          </li>
                                        </ol>
                                        
                                        <p>
                                          <strong>Editado:</strong> Edito para informar que <a target="_blank" href="http://twitter.com/#!/wasat" rel="noopener noreferrer">@wasat</a>&nbsp;me ha comentado que <strong>Internet Explorer 10 soporta también History API</strong>. Para más info: <a href="http://msdn.microsoft.com/en-us/ie/hh272905.aspx#_HTML5History">http://msdn.microsoft.com/en-us/ie/hh272905.aspx#_HTML5History</a>&nbsp;Gracias Jose&nbsp;por la información!!!!
                                        </p>
                                        
                                        <p>
                                          Os dejo el enlace con el proyecto para que veais como está hecho y podais jugar con él: <a href="https://skydrive.live.com/?cid=6521c259e9b1bec6&sc=documents&uc=1&id=6521C259E9B1BEC6%21167" title="https://skydrive.live.com/?cid=6521c259e9b1bec6&sc=documents&uc=1&id=6521C259E9B1BEC6%21167#">https://skydrive.live.com/?cid=6521c259e9b1bec6&sc=documents&uc=1&id=6521C259E9B1BEC6%21167#</a>
                                        </p>
                                        
                                        <p>
                                          <strong>Nota: </strong>También os dejo el enlace de History.js, que es un plugin de jQuery para soportar History API de forma consistente en todos los navegadores y que es mejor usar antes que meterse a hacerlo a mano: <a href="http://plugins.jquery.com/project/history-js">http://plugins.jquery.com/project/history-js</a> No he mencionado el plugin antes porque el objetivo del post <strong>no </strong>era mostraros el plugin sino la nueva History API de HTML5.
                                        </p>
                                        
                                        <p>
                                          Un saludo!!!!
                                        </p>