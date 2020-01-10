---
title: Trasteando con PURE…

author: eiximenis

date: 2010-03-09T17:35:00+00:00
geeks_url: /?p=1500
geeks_visits:
  - 1815
geeks_ms_views:
  - 1096
categories:
  - Uncategorized

---
Estos días he empezado a trastear con <a target="_blank" href="http://beebole.com/pure/demos/" rel="noopener noreferrer">PURE</a> (cuyas siglas significan PURE Unobstrusive Rendering Engine). El objetivo de PURE es proporcionar un mecanismo para transformar datos en <a target="_blank" href="http://es.wikipedia.org/wiki/JSON" rel="noopener noreferrer">JSON</a> a HTML. Cada vez más existen multitud de servicios que devuelven datos en formato JSON, y cada vez más es normal consumir estos servicios desde aplicaciones web, via javascript. Si el resultado final es mostrar los datos debemos realizar una conversión _a mano_ y generar usando javascript el HTML que deseemos. Esto es lento, tedioso y pesado.

<!--more-->

PURE viene a ayudarnos en este punto: básicamente coje datos en JSON y usando una plantilla HTML, genera código HTML que luego &ldquo;incrusta&rdquo; en alguna parte del documento final... Además se integra con jQuery (y otras librerías javascript). Lo poco que he visto de PURE me ha encantado, así que quiero compartirlo con vosotros 🙂

**1. Creación del proyecto web**

Usando <a target="_blank" href="http://www.microsoft.com/downloads/details.aspx?FamilyID=7aba081a-19b9-44c4-a247-3882c8f749e3&displaylang=en" rel="noopener noreferrer">ASP.NET MVC 2 RC2</a> (ya lo teneis todos instalado, no?? ;-)) creamos un proyecto ASP.NET MVC vacío (ASP.NET MVC 2 Empty Web Application) y nos ponemos manos a la obra!

El primer paso es <a target="_blank" href="http://beebole.com/pure/" rel="noopener noreferrer">descargarnos PURE desde su página web</a>. Tendremos un zip bastante grandote (algo más de 200 KBs), pero de todos los archivos que tiene, sólo nos interesa el pure.js dentro de la carpeta lib. Copiamos este archivo dentro de la carpeta scripts de nuestra aplicación MVC.

Vamos a crear ahora la master page de nuestro proyecto: no vamos a meter mucha cosa en la master page, simplemente vamos a incluir las dos librerias javascript que usaremos: jQuery y PURE. Para ello en View/Shared le damos a _Add New Item_ y seleccionamos _MVC 2 View Master Page_ le damos como nombre Site.master y le añadimos los tags <script> necesarios (dentro del head):

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">&lt;script src=<span style="color: #006080">"../../Scripts/jquery-1.4.1.js"</span>&gt;&lt;/script&gt;<br />&lt;script src=<span style="color: #006080">"../../Scripts/pure.js"</span>&gt;&lt;/script&gt;</pre>
  
  <p>
    </div> 
    
    <p>
      Ahora vamos a crear un controlador que nos devuelva datos en JSON... Vamos a la carpeta Controllers, le damos a <em>Add Controller </em>y le damos un nombre (en mi caso HomeController). Esto nos creará el archivo HomeController.cs, con la clase <em>HomeController </em>con un método Index.
    </p>
    
    <p>
      Vamos a crear ahora una clase cualquiera con datos que vamos a devolver. En la carpeta Models agregamos la clase TwitterUsers:
    </p>
    
    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> TwitterUser<br />{<br />    <span style="color: #0000ff">public</span> string Name { get; set; }<br />    <span style="color: #0000ff">public</span> string Twitter { get; set; }<br />}</pre>
      
      <p>
        </div> 
        
        <p>
          Finalmente en el método Index de nuestro HomeController, creamos una Lista de <em>TwitterUser</em>s y la devolvemos:
        </p>
        
        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> ActionResult Index()<br />{<br />    var data = <span style="color: #0000ff">new</span> List&lt;TwitterUser&gt; {<br />        <span style="color: #0000ff">new</span> TwitterUser() { Name=<span style="color: #006080">"Eduard Tom&agrave;s"</span>, Twitter=<span style="color: #006080">"eiximenis"</span>},<br />        <span style="color: #0000ff">new</span> TwitterUser() { Name=<span style="color: #006080">"José Miguel Torres"</span>, Twitter=<span style="color: #006080">"alegrebandolero"</span>},<br />        <span style="color: #0000ff">new</span> TwitterUser() { Name=<span style="color: #006080">"Gisela Torres"</span>, Twitter=<span style="color: #006080">"0GiS0"</span>},<br />        <span style="color: #0000ff">new</span> TwitterUser() { Name=<span style="color: #006080">"David Salgado"</span>, Twitter=<span style="color: #006080">"davidsb"</span>},<br />        <span style="color: #0000ff">new</span> TwitterUser() { Name=<span style="color: #006080">"Toni Recio"</span>, Twitter=<span style="color: #006080">"stormc23"</span>},<br />    };<br />    <span style="color: #0000ff">return</span> Json(data, JsonRequestBehavior.AllowGet);<br />}</pre>
          
          <p>
            </div> 
            
            <p>
              Fíjaos en el uso de Json para devolver los datos en formato JSON y el parámetro JsonRequestBehavior para permitir devolver datos JSON usando GET (<a target="_blank" href="/blogs/jmaguilar/archive/2010/03/03/cambios-en-el-retorno-de-datos-json-con-mvc-2.aspx" rel="noopener noreferrer">ver el post de José M. Aguilar para más información</a>).
            </p>
            
            <p>
              &nbsp;
            </p>
            
            <p>
              &nbsp;
            </p>
            
            <p>
              Si ponemos el proyecto en marcha y dirigimos Firefox a la URL /Home/Index veremos (gracias Firebug!) nuestros datos en JSON:
            </p>
            
            <p>
              <a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_0289B019.png"><img height="86" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_65FC986B.png" alt="image" border="0" title="image" style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" /></a>
            </p>
            
            <p>
              Es fácil mandar datos en JSON usando ASP.NET MVC, eh?? 😉
            </p>
            
            <p>
              <strong>2. Crear una vista para ver los datos</strong>
            </p>
            
            <p>
              Vamos ahora a crear una vista para ver esos datos usando jQuery y PURE. Para ello primero debemos crear una acción en nuestro controlador Home que nos muestre la vista:
            </p>
            
            <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
              <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> ActionResult List()<br />{<br />    <span style="color: #0000ff">return</span> View();<br />}</pre>
              
              <p>
                </div> 
                
                <p>
                  Una vez hecho añadimos la carpeta Home dentro de Views y creamos una vista (<em>Add View</em>) llamada List.
                </p>
                
                <p>
                  Ahora nos toca añadir el código en la vista para:
                </p>
                
                <ul>
                  <li>
                    Hacer una llamada AJAX a la url /Home/Index para obtener los datos en JSON
                  </li>
                  <li>
                    Usar PURE para mostrarlos
                  </li>
                </ul>
                
                <p>
                  El primer punto es casi trivial gracias a jQuery. Añadimos el siguiente tag <script> justo antes del <h2>:
                </p>
                
                <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">&lt;script type=<span style="color: #006080">"text/javascript"</span>&gt;<br />    $(document).ready(<span style="color: #0000ff">function</span>() {<br />        <span style="color: #0000ff">var</span> url=<span style="color: #006080">"&lt;%= Url.Action("</span>Index<span style="color: #006080">", "</span>Home<span style="color: #006080">") %&gt;"</span>;<br />        $.getJSON(url, process);<br />    });<br />    <br />    <span style="color: #0000ff">function</span> process(data)<br />    {<br />        <span style="color: #008000">// Código para procesar el resultado json</span><br />    }<br />&lt;/script&gt; </pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      El método getJSON de jQuery es quien realiza todo el trabajo: Llama a una url usando AJAX y cuando la llamada devuelve llama a una función de callback (process).
                    </p>
                    
                    <p>
                      &nbsp;
                    </p>
                    
                    <p>
                      Vamos ahora a usar PURE para convertir los datos en JSON a datos en HTML.
                    </p>
                    
                    <p>
                      <strong>3. Usando PURE...</strong>
                    </p>
                    
                    <p>
                      Para usar PURE necesitamos tres cosas:
                    </p>
                    
                    <ol>
                      <li>
                        Unos datos en JSON (ya los tenemos!)
                      </li>
                      <li>
                        Una plantilla HTML
                      </li>
                      <li>
                        Unas reglas de conversión (directivas).
                      </li>
                    </ol>
                    
                    <p>
                      La plantilla HTML es simple y se coloca en la propia página, en el sitio donde queremos que se coloque el HTML generado por pure. En nuestro caso en la vista List:
                    </p>
                    
                    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="puredata"</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">ul</span><span style="color: #0000ff">&gt;</span><br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">li</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">li</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;/</span><span style="color: #800000">ul</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span></pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          El div <em>puredata</em> es nuestra plantilla, en nuestro caso vamos a generar una lista (ul) de elementos (li) a partir de los datos JSON.
                        </p>
                        
                        <p>
                          Ahora biene lo &ldquo;bueno&rdquo;... las reglas de conversión.
                        </p>
                        
                        <p>
                          En PURE las reglas de conversión (directivas les llaman ellos) se especifican <strong>usando variables javascript</strong> que básicamente tienen este formato:
                        </p>
                        
                        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">var</span> directive={<span style="color: #006080">'selector'</span> : <span style="color: #006080">'valor'</span>};<br /></pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              Donde <em>selector</em> es un selector (CSS) para seleccionar un elemento <em>dentro</em> de la plantilla y valor es un valor (propiedad) del elemento json. Nuestro caso es un poco más complejo, ya que queremos mostrar <strong>una lista</strong> de valores. En este caso debemos usar la sintaxis extendida de directivas:
                            </p>
                            
                            <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                              <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">var</span> directive={<br />  <span style="color: #006080">'selector'</span> :  {<br />    <span style="color: #006080">'variable-loop&lt;-coleccion json'</span>: {<br />        directivas-del-loop<br />    }<br />};</pre>
                              
                              <p>
                                </div> 
                                
                                <p>
                                  Escrito así parece un poco lioso, pero veamos un ejemplo de como sería nuestra directiva si lo que queremos es mostrar el nombre de nuestros usuarios de Twitter:
                                </p>
                                
                                <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                                  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">var</span> directive = {<br />    <span style="color: #006080">'li'</span> :{<br />        <span style="color: #006080">'user&lt;-'</span>:{<br />            <span style="color: #006080">'.'</span>: <span style="color: #006080">'user.Name'</span><br />        }<br />    }<br />};</pre>
                                  
                                  <p>
                                    </div> 
                                    
                                    <p>
                                      Si diseccionamos por parte la directiva:
                                    </p>
                                    
                                    <ul>
                                      <li>
                                        user<- Significa que vaya iterando <strong>directamente</strong> sobre los elementos de los datos json (nuestro objeto json ya es por sí un array).
                                      </li>
                                      <li>
                                        El operador punto (.) se refiere al propio elemento que se está generando.
                                      </li>
                                    </ul>
                                    
                                    <p>
                                      Así estamos indicando que <strong>por cada elemento</strong> del array json <strong>genere un tag li</strong> y que coloque como <strong>texto</strong> del <strong>propio tag li</strong> el valor de la propiedad <strong>Name </strong>del elemento actual.
                                    </p>
                                    
                                    <p>
                                      Finalmente sólo nos queda realizar la llamada para que PURE realice la generación del HTML... como PURE se integra con jQuery, eso es tan sencillo como:
                                    </p>
                                    
                                    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                                      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">$(<span style="color: #006080">"#puredata"</span>).render(data, directive);</pre>
                                      
                                      <p>
                                        </div> 
                                        
                                        <p>
                                          Con esto le decimos a PURE que use la plantilla dentro del div cuyo id es &ldquo;puredata&rdquo; y que la aplique a los datos indicados con las reglas que le decimos.
                                        </p>
                                        
                                        <p>
                                          Y el resultado es el que esperamos:
                                        </p>
                                        
                                        <p>
                                          <a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_5839B696.png"><img height="189" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_03BA0F2C.png" alt="image" border="0" title="image" style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" /></a>
                                        </p>
                                        
                                        <p>
                                          Que... impresionante, eh??? 🙂
                                        </p>
                                        
                                        <p>
                                          Otra demo... vamos a generar junto con el nombre, el enlace al twitter de cada persona.
                                        </p>
                                        
                                        <p>
                                          Primero modificamos la plantilla para que quede de la siguiente manera:
                                        </p>
                                        
                                        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                                          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="puredata"</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">ul</span><span style="color: #0000ff">&gt;</span><br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">li</span><span style="color: #0000ff">&gt;&lt;</span><span style="color: #800000">span</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">span</span><span style="color: #0000ff">&gt;</span> <span style="color: #0000ff">&lt;</span><span style="color: #800000">a</span> <span style="color: #ff0000">href</span><span style="color: #0000ff">="http://twitter.com/"</span><span style="color: #0000ff">&gt;</span>Ver su twitter<span style="color: #0000ff">&lt;/</span><span style="color: #800000">a</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">li</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;/</span><span style="color: #800000">ul</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span></pre>
                                          
                                          <p>
                                            </div> 
                                            
                                            <p>
                                              El tag <span> contendrá el nombre y en el atibuto href del tag <a> vamos a añadir su nombre de usuario de twitter... La directiva que debemos usar es:
                                            </p>
                                            
                                            <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                                              <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">var</span> directive = {<br />    <span style="color: #006080">'li'</span> :{<br />        <span style="color: #006080">'user&lt;-'</span>:{<br />            <span style="color: #006080">'span'</span>: <span style="color: #006080">'user.Name'</span>,<br />            <span style="color: #006080">'a@href+'</span> :<span style="color: #006080">'user.Twitter'</span><br />        }<br />    }<br />};</pre>
                                              
                                              <p>
                                                </div> 
                                                
                                                <p>
                                                  Con esta directiva le indicamos a PURE que: <strong>Por cada elemento </strong>del array json:
                                                </p>
                                                
                                                <ol>
                                                  <li>
                                                    Coja el tag <span> dentro del <li> y coloque el valor de la propiedad Name del elemento
                                                  </li>
                                                  <li>
                                                    Coja el tag <a> dentro del <li> coja el valor del atributo href y le concatene (el + del final) el valor de la propiedad Twitter del elemento.
                                                  </li>
                                                </ol>
                                                
                                                <p>
                                                  Y este es el resultado:
                                                </p>
                                                
                                                <p>
                                                  <a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_345CF10A.png"><img height="189" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_33D0727B.png" alt="image" border="0" title="image" style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" /></a>
                                                </p>
                                                
                                                <p>
                                                  Impresionante... verdad?
                                                </p>
                                                
                                                <p>
                                                  Espero que el post os haya servido para ver un poco en que consiste PURE y el enorme potencial que atesora...
                                                </p>
                                                
                                                <p>
                                                  <a target="_blank" href="http://cid-6521c259e9b1bec6.skydrive.live.com/self.aspx/BurbujasNet/ZipsPosts/PureDemo.zip" rel="noopener noreferrer">Os dejo el .zip con el proyecto final</a> (en mi skydrive).
                                                </p>
                                                
                                                <p>
                                                  Un saludo!!!!
                                                </p>