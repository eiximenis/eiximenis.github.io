---
title: Y el combate se decidió por KO (iii)
author: eiximenis

date: 2012-08-04T18:54:59+00:00
geeks_url: /?p=1609
geeks_visits:
  - 1433
geeks_ms_views:
  - 512
categories:
  - Uncategorized

---
Bueno, continuamos aquí nuestra serie explorando las maravillas de Knockout. Todos los posts de esta serie los podéis encontrar en: <http://geeks.ms/blogs/etomas/archive/tags/knockout/default.aspx>

**Serializando viewmodels**

En el post anterior, vimos los observables de knockout y como funcionaban. Vimos como crear un formulario, enlazarlo a un viewmodel que usara observables y como mandar el viewmodel serializado en json hacia un servicio REST.

Ciertamente, el tema de la serialización a JSON de nuestro viewmodel era un poco peliagudo. Dado que los observables son funciones debíamos “crear” un objeto adicional a partir de nuestro viewmodel, invocando a los observables de forma manual, ya que usar JSON.stringify sobre nuestro viewmodel no funcionaba (los observables no se serializaban). El código que usábamos era:

<div id="codeSnippetWrapper" style="overflow: auto; cursor: text; font-size: 8pt; border-top: silver 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; border-right: silver 1px solid; border-bottom: silver 1px solid; padding-bottom: 4px; direction: ltr; text-align: left; padding-top: 4px; padding-left: 4px; margin: 20px 0px 10px; border-left: silver 1px solid; line-height: 12pt; padding-right: 4px; max-height: 200px; width: 97.5%; background-color: #f4f4f4">
  <div id="codeSnippet" style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4">
    <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum1" style="color: #606060">   1:</span> <span style="color: #0000ff">var</span> jsonBeer = JSON.stringify({</pre>
    
    <p>
      <!--CRLF-->
    </p>
    
    <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum2" style="color: #606060">   2:</span>    Name : <span style="color: #0000ff">this</span>.Name(),</pre>
    
    <p>
      <!--CRLF-->
    </p>
    
    <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum3" style="color: #606060">   3:</span>    Ibu : <span style="color: #0000ff">this</span>.Ibu(),</pre>
    
    <p>
      <!--CRLF-->
    </p>
    
    <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum4" style="color: #606060">   4:</span>    Abv : <span style="color: #0000ff">this</span>.Abv()</pre>
    
    <p>
      <!--CRLF-->
    </p>
    
    <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum5" style="color: #606060">   5:</span> });</pre>
    
    <p>
      <!--CRLF--></div> </div> 
      
      <p>
        Tener que crear otro objeto “con la misma” estructura que nuestro viewmodel e ir llamando manualmente a los observables es posible para viewmodels sencillitos como este, pero para viewmodels más grandes es, como mínimo, un peñazo.
      </p>
      
      <p>
        Como ya debes estar pensando, knockout ofrece una solución a ese problema, ya que como comentamos en el post anterior enviar objetos via JSON es bastante común hoy en día. Si vuestro viewmodel contiene observables entonces la mejor manera de serializarlos es usar uno de los métodos siguientes:
      </p>
      
      <ol>
        <li>
          ko.toJS: Convierte el viewmodel a un objeto “plano” javascript (es decir hace justo lo que hemos hecho nosotros a mano, es decir invocar los observables). El objeto devuelto puede serializarse a JSON de la forma que se prefiera.
        </li>
        <li>
          ko.toJSON: Llama a ko.toJS y serializa el objeto usando JSON.stringify.
        </li>
      </ol>
      
      <p>
        Así, en lugar del código anterior podemos usar tranquilamente:
      </p>
      
      <div id="codeSnippetWrapper" style="overflow: auto; cursor: text; font-size: 8pt; border-top: silver 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; border-right: silver 1px solid; border-bottom: silver 1px solid; padding-bottom: 4px; direction: ltr; text-align: left; padding-top: 4px; padding-left: 4px; margin: 20px 0px 10px; border-left: silver 1px solid; line-height: 12pt; padding-right: 4px; max-height: 200px; width: 97.5%; background-color: #f4f4f4">
        <div id="codeSnippet" style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4">
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum1" style="color: #606060">   1:</span> <span style="color: #0000ff">var</span> jsonBeer = ko.toJSON(<span style="color: #0000ff">this</span>);</pre>
          
          <p>
            <!--CRLF--></div> </div> 
            
            <p>
              Para la deserialización (es decir, obtención del objeto viewmodel a partir de los datos JSON), Knockout no trae de serie nada. Esto implica que si nuestro viewmodel usa observables debemos manualmente crear nuestro viewmodel y llamar a los observables para inicializar el valor. Eso también puede ser un problema y para lidiar con ello existe un plugin de knockout. Hablaremos de él más adelante en esta serie de posts.
            </p>
            
            <p>
              <strong>Observables calculados</strong>
            </p>
            
            <p>
              Recuerda que los viewmodels están fuertemente atados a la vista. De hecho son la abstracción del modelo para una vista en particular y su tarea es “facilitar” al máximo el código de la vista.
            </p>
            
            <p>
              Vamos a añadir en el formulario de modificación, el tipo de cerveza (si es una IPA, una stout o una pale ale, p. ej.). A nivel del servicio REST hemos de modificar la clase Beer:
            </p>
            
            <div id="codeSnippetWrapper" style="overflow: auto; cursor: text; font-size: 8pt; border-top: silver 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; border-right: silver 1px solid; border-bottom: silver 1px solid; padding-bottom: 4px; direction: ltr; text-align: left; padding-top: 4px; padding-left: 4px; margin: 20px 0px 10px; border-left: silver 1px solid; line-height: 12pt; padding-right: 4px; max-height: 200px; width: 97.5%; background-color: #f4f4f4">
              <div id="codeSnippet" style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4">
                <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum1" style="color: #606060">   1:</span> <span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> Beer</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum2" style="color: #606060">   2:</span> {</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum3" style="color: #606060">   3:</span>     <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Name { get; set; }</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum4" style="color: #606060">   4:</span>     <span style="color: #0000ff">public</span> <span style="color: #0000ff">decimal</span> Abv { get; set; }</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum5" style="color: #606060">   5:</span>     <span style="color: #0000ff">public</span> <span style="color: #0000ff">int</span> Ibu { get; set; }</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum6" style="color: #606060">   6:</span>     <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Style { get; set; }</pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum7" style="color: #606060">   7:</span> }</pre>
                
                <p>
                  <!--CRLF--></div> </div> 
                  
                  <p>
                    Por el momento esto nos basta. Ahora, modificamos nuestra vista de Edicion (Edit.cshtml) para que tenga un campo adicional más donde entrar el tipo de cerveza:
                  </p>
                  
                  <div id="codeSnippetWrapper" style="overflow: auto; cursor: text; font-size: 8pt; border-top: silver 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; border-right: silver 1px solid; border-bottom: silver 1px solid; padding-bottom: 4px; direction: ltr; text-align: left; padding-top: 4px; padding-left: 4px; margin: 20px 0px 10px; border-left: silver 1px solid; line-height: 12pt; padding-right: 4px; max-height: 200px; width: 97.5%; background-color: #f4f4f4">
                    <div id="codeSnippet" style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4">
                      <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum1" style="color: #606060">   1:</span> <span style="color: #0000ff">&lt;</span><span style="color: #800000">label</span> <span style="color: #ff0000">for</span><span style="color: #0000ff">="Name"</span><span style="color: #0000ff">&gt;</span>Style<span style="color: #0000ff">&lt;/</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span></pre>
                      
                      <p>
                        <!--CRLF-->
                      </p>
                      
                      <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum2" style="color: #606060">   2:</span> <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="Name"</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="Style"</span> <span style="color: #ff0000">data-bind</span><span style="color: #0000ff">="value: Style"</span> <span style="color: #0000ff">/&gt;</span></pre>
                      
                      <p>
                        <!--CRLF-->
                      </p>
                      
                      <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum3" style="color: #606060">   3:</span> <span style="color: #0000ff">&lt;</span><span style="color: #800000">br</span> <span style="color: #0000ff">/&gt;</span></pre>
                      
                      <p>
                        <!--CRLF--></div> </div> 
                        
                        <p>
                          Bien, fijaos en el uso del data-bind para enlazar este <input /> al valor de la propiedad Style del viewmodel.
                        </p>
                        
                        <p>
                          Por supuesto debo modificar en la vista cuando creamos el viewmodel, para inicializar el observable Style, a partir de los datos JSON devueltos por el servicio:
                        </p>
                        
                        <div id="codeSnippetWrapper" style="overflow: auto; cursor: text; font-size: 8pt; border-top: silver 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; border-right: silver 1px solid; border-bottom: silver 1px solid; padding-bottom: 4px; direction: ltr; text-align: left; padding-top: 4px; padding-left: 4px; margin: 20px 0px 10px; border-left: silver 1px solid; line-height: 12pt; padding-right: 4px; max-height: 200px; width: 97.5%; background-color: #f4f4f4">
                          <div id="codeSnippet" style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4">
                            <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum1" style="color: #606060">   1:</span> $.getJSON(uri, <span style="color: #0000ff">function</span>(data) {</pre>
                            
                            <p>
                              <!--CRLF-->
                            </p>
                            
                            <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum2" style="color: #606060">   2:</span>     vm = {</pre>
                            
                            <p>
                              <!--CRLF-->
                            </p>
                            
                            <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum3" style="color: #606060">   3:</span>         Name  : ko.observable(data.Name),</pre>
                            
                            <p>
                              <!--CRLF-->
                            </p>
                            
                            <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum4" style="color: #606060">   4:</span>         Ibu  : ko.observable(data.Ibu),</pre>
                            
                            <p>
                              <!--CRLF-->
                            </p>
                            
                            <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum5" style="color: #606060">   5:</span>         Abv : ko.observable(data.Abv),</pre>
                            
                            <p>
                              <!--CRLF-->
                            </p>
                            
                            <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum6" style="color: #606060">   6:</span>         Style : ko.observable(data.Style),</pre>
                            
                            <p>
                              <!--CRLF-->
                            </p>
                            
                            <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum7" style="color: #606060">   7:</span>         editBeer : <span style="color: #0000ff">function</span>() {</pre>
                            
                            <p>
                              <!--CRLF-->
                            </p>
                            
                            <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum8" style="color: #606060">   8:</span>     // Continua...</pre>
                            
                            <p>
                              <!--CRLF--></div> </div> 
                              
                              <p>
                                Finalmente modifico el método GetById del controlador ApiBeerController para que me informe del valor del campo Style:
                              </p>
                              
                              <div id="codeSnippetWrapper" style="overflow: auto; cursor: text; font-size: 8pt; border-top: silver 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; border-right: silver 1px solid; border-bottom: silver 1px solid; padding-bottom: 4px; direction: ltr; text-align: left; padding-top: 4px; padding-left: 4px; margin: 20px 0px 10px; border-left: silver 1px solid; line-height: 12pt; padding-right: 4px; max-height: 200px; width: 97.5%; background-color: #f4f4f4">
                                <div id="codeSnippet" style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4">
                                  <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum1" style="color: #606060">   1:</span> <span style="color: #0000ff">public</span> Beer GetById(<span style="color: #0000ff">int</span> id)</pre>
                                  
                                  <p>
                                    <!--CRLF-->
                                  </p>
                                  
                                  <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum2" style="color: #606060">   2:</span> {</pre>
                                  
                                  <p>
                                    <!--CRLF-->
                                  </p>
                                  
                                  <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum3" style="color: #606060">   3:</span>     <span style="color: #0000ff">return</span> <span style="color: #0000ff">new</span> Beer { Name = <span style="color: #006080">"Imperial Stout #"</span> + id, Abv = 10.0M, Ibu = 120, Style=<span style="color: #006080">"Imperial Stout"</span>};</pre>
                                  
                                  <p>
                                    <!--CRLF-->
                                  </p>
                                  
                                  <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum4" style="color: #606060">   4:</span> }</pre>
                                  
                                  <p>
                                    <!--CRLF--></div> </div> 
                                    
                                    <p>
                                      Si ahora ejecuto obtengo lo esperado:
                                    </p>
                                    
                                    <p>
                                      <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_762F01BB.png"><img title="image" style="border-top: 0px; border-right: 0px; border-bottom: 0px; border-left: 0px; display: inline" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_22DCDCF7.png" width="331" height="260" /></a>
                                    </p>
                                    
                                    <p>
                                      Hasta ahí no hemos hecho nada nuevo
                                    </p>
                                  </p></p> 
                                  
                                  <p>
                                    Bien, ahora imaginad que queremos presentar en el título el nombre y el tipo de la cerveza, algo como “Heineken (American Lager)”. Una solución es componer esto en la vista. Para ello modifico el <h2>:
                                  </p>
                                  
                                  <div id="codeSnippetWrapper" style="overflow: auto; cursor: text; font-size: 8pt; border-top: silver 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; border-right: silver 1px solid; border-bottom: silver 1px solid; padding-bottom: 4px; direction: ltr; text-align: left; padding-top: 4px; padding-left: 4px; margin: 20px 0px 10px; border-left: silver 1px solid; line-height: 12pt; padding-right: 4px; max-height: 200px; width: 97.5%; background-color: #f4f4f4">
                                    <div id="codeSnippet" style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4">
                                      <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum1" style="color: #606060">   1:</span> &lt;h2 id=<span style="color: #006080">"title"</span>&gt;&lt;/h2&gt;</pre>
                                      
                                      <p>
                                        <!--CRLF--></div> </div> 
                                        
                                        <p>
                                          Y luego en el código javascript, cuando he cargado el viewmodel, justo antes (o después) de la llamada a ko.applyBindings coloco el siguiente código javascript:
                                        </p>
                                        
                                        <div id="codeSnippetWrapper" style="overflow: auto; cursor: text; font-size: 8pt; border-top: silver 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; border-right: silver 1px solid; border-bottom: silver 1px solid; padding-bottom: 4px; direction: ltr; text-align: left; padding-top: 4px; padding-left: 4px; margin: 20px 0px 10px; border-left: silver 1px solid; line-height: 12pt; padding-right: 4px; max-height: 200px; width: 97.5%; background-color: #f4f4f4">
                                          <div id="codeSnippet" style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4">
                                            <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum1" style="color: #606060">   1:</span> $(<span style="color: #006080">"#title"</span>).text(vm.Name() + <span style="color: #006080">"("</span> + vm.Style() + <span style="color: #006080">")"</span>);</pre>
                                            
                                            <p>
                                              <!--CRLF--></div> </div> 
                                              
                                              <p>
                                                El resultado es el esperado (se muestra el nombre y el tipo de cerveza dentro del <h2>. Pero… no os suena eso a un enlace? La función de un viewmodel es facilitar al máximo la creación de la vista, siendo una representación de lo que la vista muestra. No podría tener el viewmodel una propiedad que devolviese esta cadena? Es una propiedad especial cierto, ya que es de solo lectura y su valor está calculado a partir del valor de otras propiedades.
                                              </p>
                                              
                                              <p>
                                                Esto en knockout se conoce como un observable calculado y se crean usando el método ko.computed. Este método recibe un parámetro que es la función que calcula el valor del observable:
                                              </p>
                                              
                                              <div id="codeSnippetWrapper" style="overflow: auto; cursor: text; font-size: 8pt; border-top: silver 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; border-right: silver 1px solid; border-bottom: silver 1px solid; padding-bottom: 4px; direction: ltr; text-align: left; padding-top: 4px; padding-left: 4px; margin: 20px 0px 10px; border-left: silver 1px solid; line-height: 12pt; padding-right: 4px; max-height: 200px; width: 97.5%; background-color: #f4f4f4">
                                                <div id="codeSnippet" style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4">
                                                  <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum1" style="color: #606060">   1:</span> vm.Title = ko.computed(<span style="color: #0000ff">function</span>() {</pre>
                                                  
                                                  <p>
                                                    <!--CRLF-->
                                                  </p>
                                                  
                                                  <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum2" style="color: #606060">   2:</span>     <span style="color: #0000ff">return</span> vm.Name() + <span style="color: #006080">"("</span> + vm.Style() + <span style="color: #006080">")"</span>;</pre>
                                                  
                                                  <p>
                                                    <!--CRLF-->
                                                  </p>
                                                  
                                                  <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum3" style="color: #606060">   3:</span> });</pre>
                                                  
                                                  <p>
                                                    <!--CRLF--></div> </div> 
                                                    
                                                    <p>
                                                      Aquí debo hacer un apunte importante sobre la sintaxis Javascript. Como quizá ya sabéis hay dos maneras en Javascript de inicializar objetos: usando un constructor o bien usando notación literal. Yo he usado en los ejemplos de esta serie he usado la notación literal (mientras que en los ejemplos de la propia web usan más la notación de constructor). Pues bien, los observables calculados son incompatibles con la notación literal. Eso significa que NO puedes declarar observables calculados a la vez que declaras el resto del viewmodel, y los tienes que añadir después. De hecho el código completo de la creación del viewmodel es:
                                                    </p>
                                                    
                                                    <div id="codeSnippetWrapper" style="overflow: auto; cursor: text; font-size: 8pt; border-top: silver 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; border-right: silver 1px solid; border-bottom: silver 1px solid; padding-bottom: 4px; direction: ltr; text-align: left; padding-top: 4px; padding-left: 4px; margin: 20px 0px 10px; border-left: silver 1px solid; line-height: 12pt; padding-right: 4px; max-height: 200px; width: 97.5%; background-color: #f4f4f4">
                                                      <div id="codeSnippet" style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4">
                                                        <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum1" style="color: #606060">   1:</span> vm = {</pre>
                                                        
                                                        <p>
                                                          <!--CRLF-->
                                                        </p>
                                                        
                                                        <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum2" style="color: #606060">   2:</span>     Name  : ko.observable(data.Name),</pre>
                                                        
                                                        <p>
                                                          <!--CRLF-->
                                                        </p>
                                                        
                                                        <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum3" style="color: #606060">   3:</span>     Ibu  : ko.observable(data.Ibu),</pre>
                                                        
                                                        <p>
                                                          <!--CRLF-->
                                                        </p>
                                                        
                                                        <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum4" style="color: #606060">   4:</span>     Abv : ko.observable(data.Abv),</pre>
                                                        
                                                        <p>
                                                          <!--CRLF-->
                                                        </p>
                                                        
                                                        <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum5" style="color: #606060">   5:</span>     Style : ko.observable(data.Style),</pre>
                                                        
                                                        <p>
                                                          <!--CRLF-->
                                                        </p>
                                                        
                                                        <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum6" style="color: #606060">   6:</span>     editBeer : <span style="color: #0000ff">function</span>() {</pre>
                                                        
                                                        <p>
                                                          <!--CRLF-->
                                                        </p>
                                                        
                                                        <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum7" style="color: #606060">   7:</span>        <span style="color: #008000">// Codigo...</span></pre>
                                                        
                                                        <p>
                                                          <!--CRLF-->
                                                        </p>
                                                        
                                                        <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum8" style="color: #606060">   8:</span>     }</pre>
                                                        
                                                        <p>
                                                          <!--CRLF-->
                                                        </p>
                                                        
                                                        <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum9" style="color: #606060">   9:</span> };</pre>
                                                        
                                                        <p>
                                                          <!--CRLF-->
                                                        </p>
                                                        
                                                        <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum10" style="color: #606060">  10:</span> vm.Title = ko.computed(<span style="color: #0000ff">function</span>() {</pre>
                                                        
                                                        <p>
                                                          <!--CRLF-->
                                                        </p>
                                                        
                                                        <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum11" style="color: #606060">  11:</span>     <span style="color: #0000ff">return</span> vm.Name() + <span style="color: #006080">"("</span> + vm.Style() + <span style="color: #006080">")"</span>;</pre>
                                                        
                                                        <p>
                                                          <!--CRLF-->
                                                        </p>
                                                        
                                                        <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum12" style="color: #606060">  12:</span> });</pre>
                                                        
                                                        <p>
                                                          <!--CRLF--></div> </div> 
                                                          
                                                          <p>
                                                            Fijaos como las propiedades Name, Ibu, Abv, Style y editBeer son definidas en notación literal (propiedad : valor), pero el observable calculado lo he añadido luego.
                                                          </p>
                                                          
                                                          <p>
                                                            Aclarado este aspecto “notacional”,&#160; tan solo nos queda enlazar el <h2> con el valor del campo Title de nuestro viewmodel:
                                                          </p>
                                                          
                                                          <div id="codeSnippetWrapper" style="overflow: auto; cursor: text; font-size: 8pt; border-top: silver 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; border-right: silver 1px solid; border-bottom: silver 1px solid; padding-bottom: 4px; direction: ltr; text-align: left; padding-top: 4px; padding-left: 4px; margin: 20px 0px 10px; border-left: silver 1px solid; line-height: 12pt; padding-right: 4px; max-height: 200px; width: 97.5%; background-color: #f4f4f4">
                                                            <div id="codeSnippet" style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4">
                                                              <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum1" style="color: #606060">   1:</span> &lt;h2 id=<span style="color: #006080">"title"</span> data-bind=<span style="color: #006080">"text: Title"</span>&gt;&lt;/h2&gt;</pre>
                                                              
                                                              <p>
                                                                <!--CRLF--></div> </div> 
                                                                
                                                                <p>
                                                                  Y ahora podemos ver como todo funciona correctamente:
                                                                </p>
                                                                
                                                                <p>
                                                                  <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_001BA53A.png"><img title="image" style="border-top: 0px; border-right: 0px; border-bottom: 0px; border-left: 0px; display: inline" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_2F6D1AC4.png" width="244" height="152" /></a>
                                                                </p>
                                                                
                                                                <p>
                                                                  Pero recordad que los observables son “bidireccionales” no? Basta con modificar el textbox de Name o el de Style para que… ¡se cambie el título!
                                                                </p>
                                                                
                                                                <p>
                                                                  <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_61674BFF.png"><img title="image" style="border-top: 0px; border-right: 0px; border-bottom: 0px; border-left: 0px; display: inline" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_49CF9E8C.png" width="244" height="191" /></a>
                                                                </p>
                                                                
                                                                <p>
                                                                  ¿No os parece una pasada?
                                                                </p>
                                                                
                                                                <p>
                                                                  Bueno, lo dejamos aquí por hoy, en futuros posts seguiremos explorando las capacidades de knockout!
                                                                </p>
                                                                
                                                                <p>
                                                                  Saludos!
                                                                </p>