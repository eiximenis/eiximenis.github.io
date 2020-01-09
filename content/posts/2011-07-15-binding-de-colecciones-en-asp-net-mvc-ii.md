---
title: Binding de colecciones en ASP.NET MVC (ii)
description: Binding de colecciones en ASP.NET MVC (ii)
author: eiximenis

date: 2011-07-15T17:00:00+00:00
geeks_url: /?p=1572
geeks_visits:
  - 6796
geeks_ms_views:
  - 2598
categories:
  - Uncategorized

---
Bueno... En el <a target="_blank" href="/blogs/etomas/archive/2011/07/09/binding-de-colecciones-en-asp-net-mvc.aspx" rel="noopener noreferrer">post anterior</a> vimos como el DefaultModelBinder esperaba los nombres de los campos para poder realizar el enlace entre los datos de la request y un parámetro de tipo colección en el controlador.

Pero vimos que había un pequeño detalle. Supongamos el siguiente método del controlador:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">[HttpPost]<br /><span style="color: #0000ff">public</span> ActionResult Index(IEnumerable&lt;<span style="color: #0000ff">int</span>&gt; results)<br />{<br />    <span style="color: #0000ff">return</span> View();<br />}</pre>
  
  <p>
    </div> 
    
    <p>
      El método recibe una colección de enteros. Vamos a crearnos una vista de prueba:
    </p>
    
    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">@using (Html.BeginForm())<br />{<br />    for (int i = 0; i <span style="color: #0000ff">&lt;</span> 10; i++)<br />    {<br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">text</span><span style="color: #0000ff">&gt;</span>Pregunta @i:<span style="color: #0000ff">&lt;/</span><span style="color: #800000">text</span><span style="color: #0000ff">&gt;</span><br />        @Html.RadioButton("[" + i + "]", 1);<br />        @Html.RadioButton("[" + i + "]", 2);<br />        @Html.RadioButton("[" + i + "]", 3);                            <br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">p</span> <span style="color: #0000ff">/&gt;</span><br />    }<br /><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="submit"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="enviar!"</span> <span style="color: #0000ff">/&gt;</span><br />}</pre>
      
      <p>
        </div> 
        
        <p>
          Mostramos simplemente 30 (10*3) radiobuttons. Esto nos mostrará 10 filas de radiobuttons. Las radiobuttons de cada fila se llaman igual &ldquo;[i]&rdquo;, siendo i el índice de la fila, que es lo que espera el <em>DefaultModelBinder</em>.
        </p>
        
        <p>
          Ahora fíjemonos que pasa si el usuario selecciona tan solo ALGUNAS de las radiobuttons:
        </p>
        
        <p>
          <a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_551C9655.png"><img height="244" width="127" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_3E2D9F0C.png" alt="image" border="0" title="image" style="background-image: none; border-bottom: 0px; border-left: 0px; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" /></a>
        </p>
        
        <p>
          Lo que recibimos en el controlador es:
        </p>
        
        <p>
          <a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_667CD160.png"><img height="55" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_39F3A17A.png" alt="image" border="0" title="image" style="background-image: none; border-bottom: 0px; border-left: 0px; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" /></a>
        </p>
        
        <p>
          Tan sólo recibimos las radiobuttons marcadas <em>hasta</em> la primera que el usuario no ha marcado. A partir de este punto el <em>DefaultModelBinder</em> deja de enlazar! Por eso recibimos los valores de [0] y [1] ya que [2] es el primer valor que el usuario no informa.
        </p>
        
        <p>
          <strong>Como enlaza colecciones el DefaultModelBinder</strong>
        </p>
        
        <p>
          Bien... os animáis a explorar un poco el DefaultModelBinder? Dejadme que os muestre que pasa, a grandes rasgos, cuando se enlaza una colección... Si no te interesan tanto los detalles de como funciona el DefaultModelBinder puedes saltar al siguiente apartado 😉
        </p>
        
        <p>
          Así que, qué hace el DefaultModelBinder cuando debe enlazar el parámetro <em>results</em>? Simplificando, lo primero es mirar el tipo de este parámetro (IEnumerable<int>) y llamar al método <em>CreateModel</em> que debe devolver un objeto compatible con este tipo. La implementación por defecto devuelve List<T> si el tipo del modelo es IEnumerable<T>.
        </p>
        
        <p>
          Una vez tiene el objeto (una List<int> vacía en nuestro caso) empieza a rellenarla. Esto se hace dentro de un método llamado <em>BindComplexModel</em> que entre otras cosas mira si el modelo es de tipo IDictionary<K,V>, un array o un IEnumerable<T>. Esos tipos tienen tratamientos &ldquo;especiales&rdquo;. Si no es ningún de estos tipos se asume que estamos enlazando un objeto.
        </p>
        
        <p>
          Si estamos enlazando un IEnumerable<T> se llama a otro método de nombre <em>UpdateCollection</em> que es extremadamente simple. Hace dos cosas sólamente:
        </p>
        
        <ol>
          <li>
            Llama a un método <em>GetIndexes</em> para que devuelva que indices debe enlazar
          </li>
          <li>
            Por cada índice busca un valor en la request de nombre &ldquo;[idx]&rdquo; y lo intenta enlazar (llamando a BindModel de nuevo).
          </li>
        </ol>
        
        <p>
          Centrémonos en este primer punto, el método GetIndexes. Lo &ldquo;casi&rdquo; único que hace es lo siguiente:
        </p>
        
        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #008000">// just use a simple zero-based system</span><br />stopOnIndexNotFound = <span style="color: #0000ff">true</span>;<br />indexes = GetZeroBasedIndexes();</pre>
          
          <p>
            </div> 
            
            <p>
              Pone <em>stopOnIdexNotFound</em> a true y llama a GetZeroBasedIndexes(). Y que es GetZeroBasedIndexes()? Pues lo siguiente:
            </p>
            
            <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
              <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">private</span> <span style="color: #0000ff">static</span> IEnumerable&lt;<span style="color: #0000ff">string</span>&gt; GetZeroBasedIndexes() {<br />    <span style="color: #0000ff">for</span> (<span style="color: #0000ff">int</span> i = 0; ; i++) {<br />        <span style="color: #0000ff">yield</span> <span style="color: #0000ff">return</span> i.ToString(CultureInfo.InvariantCulture);<br />    }<br />}</pre>
              
              <p>
                </div> 
                
                <p>
                  Un método que devuelve una colección <em>infinita</em> (entre comillas porque a Int32.MaxValue <em>petaría</em>).
                </p>
                
                <p>
                  Bien, ya tenemos los indices que vamos a mirar en la request: todos desde [0] hasta [Int32.MaxValue-1]
                </p>
                
                <p>
                  Ahora volvemos al código de <em>UpdateCollection. </em>Así es como recorre el bucle de índices:
                </p>
                
                <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">foreach</span> (<span style="color: #0000ff">string</span> currentIndex <span style="color: #0000ff">in</span> indexes) {<br />    <span style="color: #0000ff">string</span> subIndexKey = CreateSubIndexName(bindingContext.ModelName, currentIndex);<br />    <span style="color: #0000ff">if</span> (!bindingContext.ValueProvider.ContainsPrefix(subIndexKey)) {<br />        <span style="color: #0000ff">if</span> (stopOnIndexNotFound) {<br />            <span style="color: #008000">// we ran out of elements to pull</span><br />            <span style="color: #0000ff">break</span>;<br />        }<br />        <span style="color: #0000ff">else</span> {<br />            <span style="color: #0000ff">continue</span>;<br />        }<br />    }<br />    <span style="color: #008000">// codigo para enlazar el elemento y añadirlo (.Add) a la colección</span><br />}</pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Básicamente, en aquel punto donde en la request (recordad que el DefaultModelBinder accede a la request siempre a través de la propiedad <em>ValueProvider</em>) no se encuentre el parámetro correspondiente al índice (en nuestro caso <em>[idx]) </em>dejará de enlazar (el break sale del foreach) y devuelve todo lo enlazado hasta entonces.
                    </p>
                    
                    <p>
                      Bueno... hemos visto como enlaza el DefaultModelBinder una colección y que realmente una vez no haya el parámetro de índice requerido en la request se para de enlazar. Pero... no os he enseñado todo el código, me he dejado una pequeña parte.
                    </p>
                    
                    <p>
                      Recordáis que antes he dicho que el método GetIndexes() lo &ldquo;casi&rdquo; único que hacía era llamar a GetZeroBasedIndexes()? Pues bien <em>antes</em> de hacer esto hace otra cosa... Antes busca si existe un campo en la request llamado &ldquo;index&rdquo;.
                    </p>
                    
                    <p>
                      Este valor si existe, debe contener un string[] con todos aquellos índices que el DefaultModelBinder debe buscar en la request. Pemitidme ahora que os enseñe el código <strong>completo</strong> del método GetIndexes():
                    </p>
                    
                    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">string</span> indexKey = CreateSubPropertyName(bindingContext.ModelName, <span style="color: #006080">"index"</span>);<br />ValueProviderResult vpResult = bindingContext.ValueProvider.GetValue(indexKey);<br /><span style="color: #0000ff">if</span> (vpResult != <span style="color: #0000ff">null</span>) {<br />    <span style="color: #0000ff">string</span>[] indexesArray = vpResult.ConvertTo(<span style="color: #0000ff">typeof</span>(<span style="color: #0000ff">string</span>[])) <span style="color: #0000ff">as</span> <span style="color: #0000ff">string</span>[];<br />    <span style="color: #0000ff">if</span> (indexesArray != <span style="color: #0000ff">null</span>) {<br />        stopOnIndexNotFound = <span style="color: #0000ff">false</span>;<br />        indexes = indexesArray;<br />        <span style="color: #0000ff">return</span>;<br />    }<br />}<br /><span style="color: #008000">// just use a simple zero-based system</span><br />stopOnIndexNotFound = <span style="color: #0000ff">true</span>;<br />indexes = GetZeroBasedIndexes();</pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          No os perdáis en los detalles. Básicamente lo que hace es:
                        </p>
                        
                        <ol>
                          <li>
                            Si existe un valor de request llamado &ldquo;index&rdquo; este valor debe contener un string[] que contendrá los índices a buscar. En este caso la variable <em>stopOnIndexNotFound</em> se pone a false, por lo que el método <em>UpdateCollection</em> <strong>no se parará cuando no encuentre un valor del array</strong>. Simplemente saltará al siguiente
                          </li>
                          <li>
                            Si dicho valor no existe, hace lo que habíamos visto: pone la variable <em>stopOnIndexNotFound</em> a true y devuelve la colección de índices <em>infinita</em> empezando por 0.
                          </li>
                        </ol>
                        
                        <p>
                          <strong>El valor de request &ldquo;index&rdquo;</strong>
                        </p>
                        
                        <p>
                          Así pues la solución consiste en añadir un campo en la request (en nuestro caso en el formulario) cuyo valor sea un string[] con los nombres de todos los campos índice 🙂
                        </p>
                        
                        <p>
                          ¿Y como se envia un string[] desde HTML? Pues muy sencillo, enviando N veces un campo con el MISMO name. Fijaos como nos queda la vista ahora:
                        </p>
                        
                        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">@using (Html.BeginForm())<br />{<br />    for (int i = 0; i <span style="color: #0000ff">&lt;</span> 10; i++)<br />    {<br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">text</span><span style="color: #0000ff">&gt;</span>Pregunta @i:<span style="color: #0000ff">&lt;/</span><span style="color: #800000">text</span><span style="color: #0000ff">&gt;</span><br />        @Html.RadioButton("[" + i + "]", 1);<br />        @Html.RadioButton("[" + i + "]", 2);<br />        @Html.RadioButton("[" + i + "]", 3);                            <br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="hidden"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="index"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="@i"</span> <span style="color: #0000ff">/&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">p</span> <span style="color: #0000ff">/&gt;</span><br />    }<br />    <br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="submit"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="enviar!"</span> <span style="color: #0000ff">/&gt;</span><br />}</pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              Fijaos en el <input type=&rdquo;hidden&rdquo;> con name index que está dentro del for. En el HTML generado habrá 10 hiddens todos con el atributo &ldquo;name&rdquo; con el mismo valor &ldquo;index&rdquo; y cada uno con un valor distinto (de 0 a 9). Esto, a nivel del DefaultModelBinder, se recibe como un string[].
                            </p>
                            
                            <p>
                              Bueno... y que ocurre ahora, si mando exactamente lo mismo que la vez anterior? Pues esto es lo que recibimos en el controlador:
                            </p>
                            
                            <p>
                              <a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_3842D5A6.png"><img height="94" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_0FC3F392.png" alt="image" border="0" title="image" style="background-image: none; border-bottom: 0px; border-left: 0px; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" /></a>
                            </p>
                            
                            <p>
                              Fijaos, que ahora recibimos 7 valores, que se corresponden a las 7 filas con alguna radiobutton marcada.
                            </p>
                            
                            <p>
                              Vale, vale, vale... ya os oigo decir: &ldquo;Sí, todo esto está muy bien, pero tampoco me sirve de nada. Aquí había 10 preguntas (0-9) y el usuario ha marcado sólo 7. Tengo las 7 respuestas ok, pero los índices son incorrectos!&rdquo;. Efectivamente, vemos recibo una coleccción de 7 ints (las 7 respuestas) pero no se cuales han sido las que se han quedado en blanco! Yo había dejado sin marcar la #2, la #6 y la #8. Como puedo saber esto?
                            </p>
                            
                            <p>
                              La respuesta es que tranquilos, que sólo hemos mirado <em>en un lado</em>, la respuesta completa la tenemos en otro. Efectivamente, el DefaultModelBinder nos ha creado una colección <em>con los 7 valores entrados por el usuario. </em>Pero puedo saber exactamente a que posición se corresponde cada valor? Pues sí, gracias a <em>ModelState</em>:
                            </p>
                            
                            <p>
                              <a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_06F3EB46.png"><img height="372" width="644" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_59262280.png" alt="image" border="0" title="image" style="background-image: none; border-bottom: 0px; border-left: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" /></a>
                            </p>
                            
                            <p>
                              Fijaos en el valor de ModelState.Keys. Lo véis? Eso son las claves (los nombres) de los valores de la request. Exactamente! Con esto podemos hacer el mapeo:
                            </p>
                            
                            <ol>
                              <li>
                                results[0] es el valor de la request &#8220;[0]&rdquo;
                              </li>
                              <li>
                                results[1] es el valor de la request &ldquo;[1]&rdquo;
                              </li>
                              <li>
                                results[2] es el valor de la request &ldquo;[3]&rdquo; <&mdash;No hay ModelState.Keys con valor &ldquo;[2]&rdquo;, lo que significa que la fila #2 se había dejado sin ninguna radio marcada.
                              </li>
                            </ol>
                            
                            <p>
                              Lo veis? 😉
                            </p>
                            
                            <p>
                              Por supuesto, todo esto se complica si nuestro controlador recibe varios parámetros o bien recibe una colección que está como propiedad de un objeto, pero no se complica demasiado, no os creais. En un siguiente post lo veremos para dejarlo claro 🙂
                            </p>
                            
                            <p>
                              Y finalmente es posible que digas: &ldquo;Pues, perdón pero eso no me gusta nada!, No podría el ModelBinder devolverme un array con las posiciones rellenadas y con los índices correctos?&rdquo;
                            </p>
                            
                            <p>
                              Bueno... pues poder, se puede pero ya cuesta un poco más de trabajo. pero tranquilos que veremos como... pero de momento, basta por hoy, no? 😀
                            </p>
                            
                            <p>
                              Un saludo a todos!
                            </p>