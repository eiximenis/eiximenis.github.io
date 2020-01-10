---
title: 'ASP.NET MVC3: Razor Templates'

author: eiximenis

date: 2011-01-25T16:38:19+00:00
geeks_url: /?p=1554
geeks_visits:
  - 6342
geeks_ms_views:
  - 2028
categories:
  - Uncategorized

---
Muy buenas!

En este post quiero comentaros una característica de Razor que yo considero que es una **<u>auténtica pasada:</u>** los **templates**.

<!--more-->

Básicamente **el meollo de todo está en la posibilidad de guardar el resultado de un parseo de Razor en un Func<T, HelperResult>** siendo T el tipo del modelo que renderiza el template.

Veámoslo con código:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">@{<br />    Func<span style="color: #0000ff">&lt;</span><span style="color: #800000">string</span>, <span style="color: #ff0000">HelperResult</span><span style="color: #0000ff">&gt;</span> h =<br />        @<span style="color: #0000ff">&lt;</span><span style="color: #800000">h2</span><span style="color: #0000ff">&gt;</span>Esto es un template al que se le han pasado los datos: @item<span style="color: #0000ff">&lt;/</span><span style="color: #800000">h2</span><span style="color: #0000ff">&gt;</span><br />    ;<br />}<br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">p</span><span style="color: #0000ff">&gt;</span><br />     Renderizamos el template: @h("datos")<br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">p</span><span style="color: #0000ff">&gt;</span></pre>
  
  <p>
    </div> 
    
    <p>
      Fijaos como nos guardamos en una Func<string, HelperResult> el resultado de renderizar un template razor (en este caso el template <h2>…</h2>). Fijaos en tres detalles:
    </p>
    
    <ol>
      <li>
        Dado que la variable h está declarada como Func<string, HelperResult> el tipo del modelo en este template es “string”
      </li>
      <li>
        Para acceder al modelo que se pasa al template se usa @item
      </li>
      <li>
        Al final del template Razor ponemos un ; (eso es una declaración C# y como tal <em>debe</em> terminar en punto y coma).
      </li>
    </ol>
    
    <p>
      Luego más adelante renderizamos el template, con el código: @h("datos")
    </p>
    
    <h2>
      <strong>Eh!! Eso no es lo mismo que @helper?</strong>
    </h2>
    
    <p>
      Si conoces @helper te puede parecer que esto no es muy novedoso. Con @helper podemos crear helpers inline de la siguiente manera:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">@helper h2(string s) {<span style="color: #0000ff">&lt;</span><span style="color: #800000">h2</span><span style="color: #0000ff">&gt;</span>Esto es un helper: @s<span style="color: #0000ff">&lt;/</span><span style="color: #800000">h2</span><span style="color: #0000ff">&gt;</span>}</pre>
      
      <p>
        </div> 
        
        <p>
          Y lo podíamos invocar con:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">@h2("ufo")</pre>
          
          <p>
            </div> 
            
            <p>
              Por lo que parece que viene a ser lo mismo. Y es que, en el fondo, ambas construcciones devuelven un HelperResult.
            </p>
            
            <blockquote>
              <p>
                <strong>Nota: </strong>Si quieres más información sobre @helper, léete el post que hizo el maestro hace algún tiempecillo: <a title="http://www.variablenotfound.com/2010/11/y-mas-sobre-helpers-en-razor.html" href="http://www.variablenotfound.com/2010/11/y-mas-sobre-helpers-en-razor.html">http://www.variablenotfound.com/2010/11/y-mas-sobre-helpers-en-razor.html</a>
              </p>
            </blockquote>
            
            <p>
              Lo interesante no es que ambas construccione se parezcan, lo que quiero recalcar es que…
            </p>
            
            <p>
              <strong>… Los templates son Func<T, HelperResult>!</strong>
            </p>
            
            <p>
              Lo pongo así en negrita porque eso es importante, y a la vez me sirve de título. Si los templates Razor son Func<T, HelperResult>… <em>cualquier método que reciba un Fun<T, HelperResult> puede recibir un template Razor</em>…
            </p>
            
            <p>
              … Incluídos los helpers!
            </p>
            
            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">@helper Repeater(<span style="color: #0000ff">int</span> count,Func&lt;dynamic, HelperResult&gt; template, <span style="color: #0000ff">string</span> data) {<br />    &lt;ul&gt;<br />    @<span style="color: #0000ff">for</span> (<span style="color: #0000ff">int</span> idx=0; idx&lt;count; idx++)<br />    {<br />        &lt;li&gt;@template(<span style="color: #0000ff">new</span> { Text = data, Index = idx })&lt;/li&gt;<br />    }<br />    &lt;/ul&gt;   <br />}<br /><br />@Repeater(10, @&lt;span&gt;@item.Text (item: #@item.Index)&lt;/span&gt;, <span style="color: #006080">"Ufo"</span>)</pre>
              
              <p>
                </div> 
                
                <p>
                  En este código:
                </p>
                
                <ol>
                  <li>
                    Declaramos un helper, llamado Repeater que acepta tres parámetros:
                  </li>
                  <ol>
                    <li>
                      Un entero
                    </li>
                    <li>
                      Un Func<dynamic, HelperResult> que <em>por lo tanto podrá ser un template Razor</em>
                    </li>
                    <li>
                      Una cadena
                    </li>
                  </ol>
                  
                  <li>
                    El helper se limita a crear una lista y luego:
                  </li>
                  <ol>
                    <li>
                      Crea tantos <li> como indica el paràmetro count y en cada id
                    </li>
                    <ol>
                      <li>
                        Evalúa el segundo parámetro <em>(que es el Func) y le pasa como parámetro un objeto anonimo con dos propiedades (Text e Index), donde Text es el tercer parámetro que recibe (y index el valor de la iteración actual</em>).
                      </li>
                    </ol>
                  </ol>
                  
                  <li>
                    Cuando invocamos al helper, le pasamos como primer parámetro el número de repeticiones, como segundo parámetro <strong>un template de Razor</strong>. Fijaos que dentro de dicho template:
                  </li>
                  <ol>
                    <li>
                      Accedemos a las propiedades @item.Text y @item.Index. Eso podemos hacerlo porque hemos declarado el tipo del modelo de dicho template como <em>dynamic</em> y por eso nos compila (y nos funciona porque el helper cuando invoca el template crea esas propiedades en el objeto anónimo).
                    </li>
                  </ol>
                </ol>
                
                <p>
                  El código HTML generado por dicha llamada al helper Repeater es:
                </p>
                
                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">&lt;</span><span style="color: #800000">ul</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">li</span><span style="color: #0000ff">&gt;</span> <span style="color: #0000ff">&lt;</span><span style="color: #800000">span</span><span style="color: #0000ff">&gt;</span>Ufo (item: #0)<span style="color: #0000ff">&lt;/</span><span style="color: #800000">span</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">li</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">li</span><span style="color: #0000ff">&gt;</span> <span style="color: #0000ff">&lt;</span><span style="color: #800000">span</span><span style="color: #0000ff">&gt;</span>Ufo (item: #1)<span style="color: #0000ff">&lt;/</span><span style="color: #800000">span</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">li</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">li</span><span style="color: #0000ff">&gt;</span> <span style="color: #0000ff">&lt;</span><span style="color: #800000">span</span><span style="color: #0000ff">&gt;</span>Ufo (item: #2)<span style="color: #0000ff">&lt;/</span><span style="color: #800000">span</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">li</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">li</span><span style="color: #0000ff">&gt;</span> <span style="color: #0000ff">&lt;</span><span style="color: #800000">span</span><span style="color: #0000ff">&gt;</span>Ufo (item: #3)<span style="color: #0000ff">&lt;/</span><span style="color: #800000">span</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">li</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">li</span><span style="color: #0000ff">&gt;</span> <span style="color: #0000ff">&lt;</span><span style="color: #800000">span</span><span style="color: #0000ff">&gt;</span>Ufo (item: #4)<span style="color: #0000ff">&lt;/</span><span style="color: #800000">span</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">li</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">li</span><span style="color: #0000ff">&gt;</span> <span style="color: #0000ff">&lt;</span><span style="color: #800000">span</span><span style="color: #0000ff">&gt;</span>Ufo (item: #5)<span style="color: #0000ff">&lt;/</span><span style="color: #800000">span</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">li</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">li</span><span style="color: #0000ff">&gt;</span> <span style="color: #0000ff">&lt;</span><span style="color: #800000">span</span><span style="color: #0000ff">&gt;</span>Ufo (item: #6)<span style="color: #0000ff">&lt;/</span><span style="color: #800000">span</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">li</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">li</span><span style="color: #0000ff">&gt;</span> <span style="color: #0000ff">&lt;</span><span style="color: #800000">span</span><span style="color: #0000ff">&gt;</span>Ufo (item: #7)<span style="color: #0000ff">&lt;/</span><span style="color: #800000">span</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">li</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">li</span><span style="color: #0000ff">&gt;</span> <span style="color: #0000ff">&lt;</span><span style="color: #800000">span</span><span style="color: #0000ff">&gt;</span>Ufo (item: #8)<span style="color: #0000ff">&lt;/</span><span style="color: #800000">span</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">li</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">li</span><span style="color: #0000ff">&gt;</span> <span style="color: #0000ff">&lt;</span><span style="color: #800000">span</span><span style="color: #0000ff">&gt;</span>Ufo (item: #9)<span style="color: #0000ff">&lt;/</span><span style="color: #800000">span</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">li</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">ul</span><span style="color: #0000ff">&gt;</span>   </pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Espero que el post os haya resultado interesante!!! 😉
                    </p>
                    
                    <p>
                      Saludos!
                    </p>