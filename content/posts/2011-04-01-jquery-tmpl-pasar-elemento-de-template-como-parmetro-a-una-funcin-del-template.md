---
title: '[jQuery tmpl] Pasar elemento de template como parámetro a una función del template'

author: eiximenis

date: 2011-04-01T09:42:00+00:00
geeks_url: /?p=1564
geeks_visits:
  - 2606
geeks_ms_views:
  - 1207
categories:
  - Uncategorized

---
Bueno... Vaya título me ha salido, eh? 😛 A ver, realmente este post es para evitar que alguien pierda el mismo tiempo que he pedido yo, para una chorrada...

<!--more-->

En fin, al tajo. No sé si conocéis [jQuery templates][1]. Para los que no, que sepáis que es un plugin de jQuery para convertir objetos json en html. No es la única manera de hacerlo, [hace tiempo escribí sobre PURE][2] ([http://beebole.com/pure/][3]) otra herramienta para hacer lo mismo, y que os animo a que al menos le echéis un vistazo. Poco después apareció la alfa de jquery-tmpl (y siguiendo con el autobombo escribí una [pequeña comparativa][4], que, todo debe reconocerse, hoy ha quedado un poco desfasada). Poco después se anunció que jQuery-tmpl pasaba a ser considerado plugin oficial de jQuery y se pasó a llamar &ldquo;_jQuery templates_&rdquo;. Actualmente está en beta, pero ya es extremadamente estable. Luis Ruiz Pavón escribió un [artículo introductorio a jQuery templates][5] que os recomiendo que le echéis un vistazo (aunque la sintaxis actual sea un poco diferente a la de ese artículo, es lo que tiene escribir sobre versiones alfa y demás).

En jQuery templates se usan básicamente &ldquo;tres&#8221; cosas:

  1. Una definición de template, que se suele incorporar dentro de un tag <script> con un type inválido para que el navegador lo ignore (no lo incorpore al DOM).
  2. Un contenedor donde se incrustará el DOM generado.
  3. Un objeto json inicio de los datos.

La sintaxis para definir el template es el punto fuerte de jQuery templates:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">&lt;script type=<span style="color: #006080">"text/javascript"</span>&gt;<br />    $(document).ready(<span style="color: #0000ff">function</span> () {<br />        <span style="color: #0000ff">var</span> data = { nombre: <span style="color: #006080">'edu'</span>, twitter: <span style="color: #006080">'eiximenis'</span> };<br /><br />        $(<span style="color: #006080">"#template"</span>).tmpl(data).appendTo(<span style="color: #006080">"#placeholder"</span>);<br />    });<br />&lt;/script&gt;<br /><br />&lt;script id=<span style="color: #006080">"template"</span> type=<span style="color: #006080">"text/x-jquery-tmpl"</span>&gt;<br />    &lt;div style=<span style="color: #006080">"background: green"</span>&gt;Usuario ${nombre} - Twitter: &lt;a href=<span style="color: #006080">"http://twitter.com/${twitter}"</span>&gt;${twitter}&lt;/a&gt;<br />&lt;/script&gt;<br /><br />&lt;div id=<span style="color: #006080">"placeholder"</span>&gt;&lt;/div&gt;</pre>
  
  <p>
    </div> 
    
    <p>
      Este código, en tiempo de ejcución genera el DOM siguiente:
    </p>
    
    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">style</span><span style="color: #0000ff">="background: green;"</span><span style="color: #0000ff">&gt;</span>Usuario edu - Twitter: <span style="color: #0000ff">&lt;</span><span style="color: #800000">a</span> <span style="color: #ff0000">href</span><span style="color: #0000ff">="http://twitter.com/eiximenis"</span><span style="color: #0000ff">&gt;</span>eiximenis<span style="color: #0000ff">&lt;/</span><span style="color: #800000">a</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span></pre>
      
      <p>
        </div> 
        
        <p>
          Hay varios tags para controlar el template. Para este post vamos a ver uno, que es {{each}} que permite repetir parte del template por cada elemento del array:
        </p>
        
        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">&lt;script type=<span style="color: #006080">"text/javascript"</span>&gt;<br />    $(document).ready(<span style="color: #0000ff">function</span> () {<br />        <span style="color: #0000ff">var</span> data = {<br />            titulo: <span style="color: #006080">'Twitters en geeks'</span>,<br />            users: [{ nombre: <span style="color: #006080">'edu'</span>, twitter: <span style="color: #006080">'eiximenis'</span> },<br />                    { nombre: <span style="color: #006080">'jorge'</span>, twitter: <span style="color: #006080">'j0rgeSerran0'</span> },<br />                    { nombre: <span style="color: #006080">'javi'</span>, twitter: <span style="color: #006080">'jtorrecilla'</span> },<br />                    ]   <br />        };<br /><br />        $(<span style="color: #006080">"#template"</span>).tmpl(data).appendTo(<span style="color: #006080">"#placeholder"</span>);<br />    });<br />&lt;/script&gt;<br /><br />&lt;script id=<span style="color: #006080">"template"</span> type=<span style="color: #006080">"text/x-jquery-tmpl"</span>&gt;<br />    <br />    &lt;div style=<span style="color: #006080">"background: #EEEEEE"</span>&gt;<br />        &lt;h4&gt;${titulo}&lt;/h4&gt;<br />        {{each users}}<br />            Usuario ${$value.nombre} - Twitter: &lt;a href=<span style="color: #006080">"http://twitter.com/${twitter}"</span>&gt;${$value.twitter}&lt;/a&gt;&lt;br /&gt;<br />        {{/each}}<br />    &lt;/div&gt;<br />&lt;/script&gt;<br /><br />&lt;div id=<span style="color: #006080">"placeholder"</span>&gt;&lt;/div&gt;</pre>
          
          <p>
            </div> 
            
            <p>
              Con {{each}} iteramos sobre los elementos del array &ldquo;users&rdquo;. Dentro del template el valor $value me permite referenciar el valor del array para el que se está renderizando el template (p.ej. ${$value.nombre} me permite acceder al nombre del elemento que se está renderizando). Si el nombre de $value no nos gusta, lo podemos indicar como parámetro de each:
            </p>
            
            <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
              <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">&lt;script id=<span style="color: #006080">"template"</span> type=<span style="color: #006080">"text/x-jquery-tmpl"</span>&gt;<br />    &lt;div style=<span style="color: #006080">"background: #EEEEEE"</span>&gt;<br />        &lt;h4&gt;${titulo}&lt;/h4&gt;<br />        {{each(idx, user) users}}<br />            &lt;strong&gt;${idx}:&lt;/strong&gt;Usuario ${user.nombre} - Twitter: &lt;a href=<span style="color: #006080">"http://twitter.com/${twitter}"</span>&gt;${user.twitter}&lt;/a&gt;&lt;br /&gt;<br />        {{/each}}<br />    &lt;/div&gt;<br />&lt;/script&gt;</pre>
              
              <p>
                </div> 
                
                <p>
                  En este código podemos usar ${idx} para acceder al índice del elemento y ${user} para acceder a cada elemento que se está renderizando.
                </p>
                
                <p>
                  <strong>Bien, vayamos ahora al tema del post...</strong>
                </p>
                
                <p>
                  Una cosilla interesante es que a la llamada a tmpl() se le puede pasar un segundo parámetro, con <em>datos</em> adicionales globables que pueden usarse en el template. Esos parámetros pueden ser, entre otras cosas, funciones. Para acceder a los elementos &ldquo;globales&rdquo; se usa la variable de template $item. Imaginad que tenemos esto ahora:
                </p>
                
                <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">&lt;script type=<span style="color: #006080">"text/javascript"</span>&gt;<br />    $(document).ready(<span style="color: #0000ff">function</span> () {<br />        <span style="color: #0000ff">var</span> data = {<br />            titulo: <span style="color: #006080">'Twitters en geeks'</span>,<br />            users: [{ nombre: <span style="color: #006080">'edu'</span>, twitter: <span style="color: #006080">'eiximenis'</span> },<br />                    { nombre: <span style="color: #006080">'jorge'</span>, twitter: <span style="color: #006080">'j0rgeSerran0'</span> },<br />                    { nombre: <span style="color: #006080">'javi'</span>, twitter: <span style="color: #006080">'jtorrecilla'</span> },<br />                    { nombre: <span style="color: #006080">'alguien'</span> }<br />                    ]<br />        };<br /><br />        $(<span style="color: #006080">"#template"</span>).tmpl(data,<br />            { getUrl: <span style="color: #0000ff">function</span> (name) { <span style="color: #0000ff">return</span> name != undefined ? <span style="color: #006080">"http://twitter.com/"</span> + name : <span style="color: #006080">'#'</span>; } }).appendTo(<span style="color: #006080">"#placeholder"</span>);<br />    });<br />&lt;/script&gt;<br /></pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Como segundo parámetro a tmpl() le pasamos un objeto con una función getUrl que dado una cadena me construirá la URL de twitter asociada. Si el la cadena es undefined, me generará una URL que no haga nada (#). Ahora toca llamar a esta función desde el template:
                    </p>
                    
                    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">&lt;script id=<span style="color: #006080">"template"</span> type=<span style="color: #006080">"text/x-jquery-tmpl"</span>&gt;<br />    &lt;div style=<span style="color: #006080">"background: #EEEEEE"</span>&gt;<br />        &lt;h4&gt;${titulo}&lt;/h4&gt;<br />        {{each(idx, user) users}}<br />            &lt;strong&gt;${idx}:&lt;/strong&gt;Usuario ${user.nombre} - <br />            Twitter: &lt;a href=<span style="color: #006080">"${$item.getUrl(${user.twitter})}"</span>&gt;${user.twitter}&lt;/a&gt;&lt;br /&gt;<br />        {{/each}}<br />    &lt;/div&gt;<br />&lt;/script&gt;</pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          Fijaos que llamamos a getUrl usando $item.getUrl() (porque getUrl está definido en el ámbito <em>global</em> del template). Y como parámetro le pasamos el valor de la propiedad twitter del elemento que estamos renderizando. Esto, <span style="text-decoration: underline;">yo suponía</span> que era ${user.twitter}. Pero <strong>eso no funciona</strong>. No aparece el template y en su lugar aparece un error de javascript en jquery-tmpl.js. Eso es muy común: errores en el template generan errores de javascript dentro de jquery-tmpl.js.
                        </p>
                        
                        <p>
                          Después de varios intentos descubrí que pasaba: <strong>Cuando se pasan parámetros a una función definida dentro del ámbito global del template esos parámetros de pasan sin ${}.</strong> Es decir debemos usar:
                        </p>
                        
                        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">&lt;script id=<span style="color: #006080">"template"</span> type=<span style="color: #006080">"text/x-jquery-tmpl"</span>&gt;<br />    &lt;div style=<span style="color: #006080">"background: #EEEEEE"</span>&gt;<br />        &lt;h4&gt;${titulo}&lt;/h4&gt;<br />        {{each(idx, user) users}}<br />            &lt;strong&gt;${idx}:&lt;/strong&gt;Usuario ${user.nombre} - <br />            Twitter: &lt;a href=<span style="color: #006080">"${$item.getUrl(user.twitter)}"</span>&gt;${user.twitter}&lt;/a&gt;&lt;br /&gt;<br />        {{/each}}<br />    &lt;/div&gt;<br />&lt;/script&gt;</pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              Fijaos en la llamada a getUrl, ahora simplemente la pasamos getUrl(user.twitter). Y eso funciona correctamente!
                            </p>
                            
                            <p>
                              Así pues:
                            </p>
                            
                            <ul>
                              <li>
                                ${$item.getUrl(${user.name})} &ndash;> NO FUNCIONA
                              </li>
                              <li>
                                ${item.getUrl(user.name)} &ndash;> FUNCIONA CORRECTAMENTE
                              </li>
                            </ul>
                            
                            <p>
                              En fin... cosillas que descubre uno 😉
                            </p>
                            
                            <p>
                              Saludos!
                            </p>

 [1]: http://api.jquery.com/category/plugins/templates/
 [2]: /blogs/etomas/archive/2010/03/09/trasteando-con-pure.aspx
 [3]: http://beebole.com/pure/ "http://beebole.com/pure/"
 [4]: /blogs/etomas/archive/2010/03/20/mi-modesta-comparativa-entre-pure-y-jquery-tmpl.aspx
 [5]: /blogs/lruiz/archive/2010/05/11/asp-net-jquery-templates.aspx