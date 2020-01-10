---
title: Mi modesta comparativa entre PURE y jQuery-tmpl

author: eiximenis

date: 2010-03-20T10:58:00+00:00
geeks_url: /?p=1502
geeks_visits:
  - 2017
geeks_ms_views:
  - 900
categories:
  - Uncategorized

---
Hola! En mis dos últimos posts he estado hablando un poco de <a target="_blank" href="http://beebole.com/pure/" rel="noopener noreferrer">PURE</a>, una herramienta para generar código HTML a partir de una plantilla y un objeto Json. Hace poco Microsoft ha presentado <a target="_blank" href="http://github.com/nje/jquery-tmpl" rel="noopener noreferrer">jquery-tmpl</a>, <a target="_blank" href="http://wiki.github.com/nje/jquery/jquery-templates-proposal" rel="noopener noreferrer">su propuesta (todavía abierta y en fase de discusión)</a> para realizar exactamente lo mismo: generar HTML a partir de plantillas y json. Más detalles los podeis encontrar en <a target="_blank" href="http://stephenwalther.com/blog/archive/2010/03/16/microsoft-jquery-and-templating.aspx?utm_source=feedburner&utm_medium=feed&utm_campaign=Feed%3A+StephenWalther+(Stephen+Walther+on+ASP.NET+MVC)" rel="noopener noreferrer">este post de Stephen Walher</a>.

<!--more-->

Ni soy (ni me considero) un experto ni en PURE ni en javascript, pero me he permitido realizar una pequeña comparativa entre PURE y la propuesta de microsoft para templates, para ver en que se parecen y en que se diferencian. Esta es _mi_ comparativa, sin duda limitada por mi propia falta de conocimientos, pero que comparto con vosotros por si os parece de interés.

Al final del post encontraréis en enlace con el código fuente del proyecto, que en este caso es un lector de los feeds de geeks.ms, implementado en ASP.NET MVC. Básicamente hay dos vistas: una que se genera usando PURE y otra usando jquery-tmpl.

No voy a comentar nada de la parte &ldquo;servidor&rdquo; del proyecto, puesto que teneis el códgo fuente en el zip (y obviamente si alguien tiene alguna duda puede contactar conmigo). Lo que quiero comentar es el código de generación usando PURE y jquery-tmpl.

**1. Generación usando PURE**

La generación usando PURE no tiene mucho secreto. La plantilla está definida de la siguiente manera:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="pure"</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">h1</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;/</span><span style="color: #800000">h1</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">h2</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;/</span><span style="color: #800000">h2</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">span</span> <span style="color: #ff0000">class</span><span style="color: #0000ff">="t1"</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="generator"</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">span</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">p</span> <span style="color: #ff0000">class</span><span style="color: #0000ff">="feed"</span><span style="color: #0000ff">&gt;</span><br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">a</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">a</span><span style="color: #0000ff">&gt;</span><br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">br</span> <span style="color: #0000ff">/&gt;</span><br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">span</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">span</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;/</span><span style="color: #800000">p</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br /></pre>
  
  <p>
    </div> 
    
    <p>
      <strong>Toda</strong> la vista está generada usando PURE, puesto que el objeto JSON tiene toda la información necesaria.
    </p>
    
    <p>
      El código para generar el template, también es sencillo:
    </p>
    
    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">var</span> directive = {<br />    <span style="color: #006080">'h1'</span>: <span style="color: #006080">'Channel.Title'</span>,<br />    <span style="color: #006080">'h2'</span>: <span style="color: #006080">'Channel.Description'</span>,<br />    <span style="color: #006080">'span#generator'</span>: <span style="color: #006080">'Powered by: #{Channel.Generator}'</span>,<br />    <span style="color: #006080">'p.feed'</span>: {<br />        <span style="color: #006080">'feed&lt;-Channel.Items'</span>: {<br />            <span style="color: #006080">'a'</span>: <span style="color: #006080">'feed.Title'</span>,<br />            <span style="color: #006080">'a@href'</span>: <span style="color: #006080">'feed.Link'</span>,<br />            <span style="color: #006080">'span'</span>: <span style="color: #0000ff">function</span>(ctx) {<br />                <span style="color: #0000ff">return</span> stripped = ctx.item.Description.replace(/(&lt;([^&gt;]+)&gt;)/ig, <span style="color: #006080">""</span>).<br />                    substring(0, 350) + <span style="color: #006080">"..."</span>;<br />            }<br />        }<br />    }<br />};<br /><br />$(<span style="color: #006080">"#pure"</span>).render(data, directive);<br /></pre>
      
      <p>
        </div> 
        
        <p>
          Como casi siempre, se declara una directiva que controlará el renderizado y finalmente se llama al método render.
        </p>
        
        <p>
          Si echamos un vistazo a la directiva vemos una cosa que no había mostrado en los posts anteriores, y es la posibilidad de llamar a métodos javascript: en este caso en el span se colocará el resultado que devuelva el método anónimo declarado en la directiva. Cuando se llama a una función javascript desde un bucle, PURE le pasa un objeto que tiene, entre otras, la propiedad <em>item</em> que es el objeto correspondiente a la iteración que se está renderizando. El método es sencillo: lo que hace es eliminar el código HTML del feed y truncarlo a 350 carácteres.
        </p>
        
        <p>
          <strong>2. Generación usando jquery-tmpl</strong>
        </p>
        
        <p>
          Antes que me olvide: jquery-tmpl <strong>requiere jquery 1.4.2</strong>. Con la versión 1.4.1 (que es la que usa el generador de proyectos de ASP.NET MVC 2 RC2) no funciona.
        </p>
        
        <p>
          A diferencia de PURE, jquery-tmpl opta por tener los templates dentro de un tag <script> cuyo type sea text/html. Este no es un type <em>correcto</em> para un tag <script> por lo que es ignorado por el naveagador. Eso hace que, a diferencia de PURE, los elementos que forman el template <strong>no</strong> pertenecen al DOM del documento, puesto que son ignorados. Podréis ver la diferencia ejecutando el proyecto y consultando primero los datos con PURE y luego con jquery-tmpl. En el primer caso vereis como el template, a pesar de ser elementos vacíos, es visible ya que tienen estilos de colores y borders. Esto tampoco es que represente más problema: generalmente usando PURE el template está oculto inicialmente y se <em>muestra</em> cuando ha terminado la generación (p.ej. usando el método toggle() de jQuery). Yo no le hecho adrede, para que veáis el efecto.
        </p>
        
        <p>
          La <strong>gran</strong> diferencia de jquery-tmpl respecto PURE, es que no existe el concepto de directiva: el propio template contiene la directiva <em>embebida</em> en su interior. Para ello Microsoft ha recurrido a una sintaxis que nos resulte lo más familiar posible. La definición del template queda así:
        </p>
        
        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">&lt;script id=<span style="color: #006080">"template"</span> type=<span style="color: #006080">"text/html"</span>&gt;<br />    &lt;h1&gt;{%= Channel.Title %}&lt;/h1&gt;<br />    &lt;h2&gt;{%= Channel.Description %}&lt;/h2&gt;<br />           &lt;span <span style="color: #0000ff">class</span>=<span style="color: #006080">"t1"</span> id=<span style="color: #006080">"generator"</span>&gt;{%= Channel.Generator %}&lt;/span&gt;<br />        {% <span style="color: #0000ff">for</span> (<span style="color: #0000ff">var</span> i=0, l = Channel.Items.length; i&lt;l; i++) { %}<br />            &lt;p <span style="color: #0000ff">class</span>=<span style="color: #006080">"feed"</span>&gt;<br />                &lt;a href=<span style="color: #006080">"{%= Channel.Items[i].Link %}"</span>&gt;<br />                    {%= Channel.Items[i].Title %}<br />                &lt;/a&gt;<br />                &lt;br /&gt;<br />                &lt;span&gt;{%= strip(Channel.Items[i].Description) %}&lt;/span&gt;&lt;/p&gt;<br />        {% } %}  <br />&lt;/script&gt;<br /></pre>
          
          <p>
            </div> 
            
            <p>
              Se puede observar el uso de la sintaxis {% y %}, que recuerda a la <% y %> usada en páginas aspx para el código de servidor. De esta manera <em>{%= expr %} </em>se traduce por el valor de <em>expr</em> donde expr se evalúa en el contexto del objeto json. Así {%= Channel.Description %} se traduce por el valor de la propiedad Description, de la propiedad Channel del objeto json.
            </p>
            
            <p>
              También vemos el uso de {% y %} para iterar: Las etiquetas {% y %} nos permiten colocar código javasacript en nuestro template, en este caso un bucle for para iterar sobre todos los elementos de la colección Channel.Items.
            </p>
            
            <p>
              Finalmente fijaos en la llamada a la función strip dentro del <span>. La función strip sirve para eliminar el código html del feed y truncarlo a 350 carácteres, y está definida dentro de un tag <script> en la propia página:
            </p>
            
            <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
              <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">function</span> strip(str) {<br />    <span style="color: #0000ff">return</span> str.replace(/(&lt;([^&gt;]+)&gt;)/ig, <span style="color: #006080">""</span>).substring(0, 350) + <span style="color: #006080">"..."</span>;<br />}<br /></pre>
              
              <p>
                </div> 
                
                <p>
                  Hemos visto como definimos el template... y como lo aplicamos? Pues muy fácil, primeramente necesitamos el contenedor (o sea, el sitio donde se va a colocar el código HTML generado):
                </p>
                
                <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">&lt;div id=<span style="color: #006080">"mstemplate"</span>&gt;<br />&lt;/div&gt;<br /></pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Y una vez tenemos los datos en json, seleccionamos el template (usando un selector de jquery) y llamamos a su método render, pasándole los datos json y finalmente llamamos a appendTo para que el resultado se coloque dentro del contenedor que indiquemos.
                    </p>
                    
                    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">$(<span style="color: #006080">"#template"</span>).render(data).appendTo(<span style="color: #006080">"#mstemplate"</span>);</pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          <strong>3. Algunas conclusiones mías</strong>
                        </p>
                        
                        <p>
                          La principal diferencia es que en PURE el template es un conjunto de objetos DOM conocidos por el navegador, mientras que en jquery-tmpl, el template es ignorado por el navegador ya que está dentro de un <script> cuyo tipo es &ldquo;text/html&rdquo;. En el <a target="_blank" href="http://wiki.github.com/nje/jquery/jquery-templates-proposal#discussion" rel="noopener noreferrer">apartado de discusiones del documento de propuesta de microsoft</a>, se comenta el hecho que los templates no son objetos DOM para evitar efectos colaterales. Esto es cierto si se adopta la filosofía de que el template tenga la directiva embebida. Si mi template tiene algo como:
                        </p>
                        
                        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">&lt;</span><span style="color: #800000">img</span> <span style="color: #ff0000">src</span><span style="color: #0000ff">="{%= ImageUrl %}"</span> <span style="color: #0000ff">/&gt;</span></pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              Si el template fuese un objeto DOM reconocido por el navegador, éste intentaría cargar la imagen &ldquo;{%= ImagenUrl %}&rdquo;, la primera vez. Esto es algo que NO podemos evitar haciendo que el template sea invisible. Es por ello que Microsoft opta por poner el template dentro de un tag <script> cuyo type sea &ldquo;text/html&rdquo; y de esta manera sea ignorado por el navegador.
                            </p>
                            
                            <p>
                              En PURE no hay este problema, ya que la directiva está separada y además PURE puede crear atributos que no estén en el template si la directiva así lo indica. De este modo, el template para el caso anterior en PURE queda como:
                            </p>
                            
                            <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                              <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">&lt;</span><span style="color: #800000">img</span> <span style="color: #0000ff">/&gt;</span></pre>
                              
                              <p>
                                </div> 
                                
                                <p>
                                  Y la directiva asociada es la que crea el tag src dentro del tag <img>:
                                </p>
                                
                                <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                                  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">var</span> directive = { <span style="color: #006080">'img@src'</span> : <span style="color: #006080">'ImageUrl'</span>};</pre>
                                  
                                  <p>
                                    </div> 
                                    
                                    <p>
                                      De este modo el template puede ser un objeto DOM sin efectos colaterales.
                                    </p>
                                    
                                    <p>
                                      No sé si el hecho de que los templates sean objetos DOM reales o no, tiene mucha importancia. Lo que si me hizo notar <a target="_blank" href="http://twitter.com/tchvil" rel="noopener noreferrer">@tchvil</a> es que con PURE la página sigue siendo xhtml, mientras que con jquery-tmpl no, por el uso de la sintaxis {% ... %} y que eso puede tener su importancia en según que casos.
                                    </p>
                                    
                                    <p>
                                      <strong>4. ¿Es necesario un estándard de templates en jquery?</strong>
                                    </p>
                                    
                                    <p>
                                      Si no he entendido mal, la intención de microsoft con jquery-tmpl (que recuerdo está en fase de definición) es que se convierta en el método estándard de templates en jquery. Yo no se si es necesario que haya un mecanismo estándard de templates en jquery. Pienso que es bueno que el core sea lo más pequeño posible y que se use el mecanismo de plugins para ir enchufando las distintas funcionalidades. De este modo cualquiera podrá usar el mecanismo de templates que más le convenga en cada momento. Aunque en el documento se justifica la inclusión de un mecanismo estándard de templates en el core de jquery para que así si alguien quiere desarrollar un plug-in para jquery pueda usar templates sabiendo que estarán soportados...
                                    </p>
                                    
                                    <p>
                                      Lo que sí me ha parecido ir leyendo en algunos posts, es que Microsoft va a ir abandonando su Ajax Library (me refiero a la parte de cliente, no a los controles ajax de asp.net) para ir centrando esfuerzos en jquery. Si realmente es así me parece una decisión excelente y que apoyo plenamente.
                                    </p>
                                    
                                    <p>
                                      Como comenté al principio <a target="_blank" href="http://cid-6521c259e9b1bec6.skydrive.live.com/self.aspx/BurbujasNet/ZipsPosts/TemplatingDemo.zip" rel="noopener noreferrer">aquí tenéis el código del proyecto para que hagáis con él lo que queráis</a>! (Está en mi skydrive).
                                    </p>
                                    
                                    <p>
                                      Un saludo!
                                    </p>