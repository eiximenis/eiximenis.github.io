---
title: Rendering de vistas parciales en Razor y MVC3

author: eiximenis

date: 2011-02-25T08:53:57+00:00
geeks_url: /?p=1559
geeks_visits:
  - 12076
geeks_ms_views:
  - 4015
categories:
  - Uncategorized

---
Buenas! Una de las dudas que he visto que se van repitiendo por ahí tiene que ver con **como renderizar vistas parciales en MVC3 usando Razor**.

<!--more-->

En MVC2 y anteriores (o en MVC3 usando el ViewEngine de WebForms) la forma de renderizar una vista parcial era sencilla:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">&lt;% Html.RenderPartial(<span style="color: #006080">"VistaParcial"</span>, modelo); %&gt;</pre>
  
  <p>
    </div> 
    
    <p>
      Mucha gente traduce eso a Razor y usa lo siguiente para renderizar una vista parcial:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">@Html.RenderPartial(<span style="color: #006080">"VistaParcial"</span>)</pre>
      
      <p>
        </div> 
        
        <p>
          Y se obtiene un error, quizá un poco críptico, que dice lo siguiente: <em>CS1502: The best overloaded method match for &#8216;System.Web.WebPages.WebPageExecutingBase.Write(System.Web.WebPages.HelperResult)&#8217; has some invalid arguments</em>
        </p>
        
        <p>
          El error <strong>no</strong> está en que Html.RenderPartial no pueda usarse con Razor, el error está en la sintaxis que estamos usando. Cuando en Razor usamos la @ para indicar el inicio de código de servidor, la expresión que viene a continuación <strong>debe devolver un valor</strong>, que será incluído en la respuesta a enviar al navegador. La excepción a esa norma es cuando lo que sigue a la @ es una palabra clave reservada de Razor (como @model) o una palabra clave reservada del lenguaje que estemos usando (como @foreach). Esos casos especiales Razor los sabe tratar y actúa en consecuencia. Pero en el resto de casos <strong>siempre, siempre, siempre la expresión debe devolver un valor</strong>.
        </p>
        
        <p>
          Hablando en términos del engine de WebForms, el código Razor:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">@Html.RenderPartial(<span style="color: #006080">"VistaParcial"</span>, modelo)</pre>
          
          <p>
            </div> 
            
            <p>
              Se corresponde a:
            </p>
            
            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">&lt;%: Html.RenderPartial(<span style="color: #006080">"VistaParcial"</span>,modelo) %&gt;</pre>
              
              <p>
                </div> 
                
                <p>
                  Que es erróneo (y da el error <em>CS1502: The best overloaded method match for &#8216;System.Web.HttpUtility.HtmlEncode(string)&#8217; has some invalid arguments</em>).
                </p>
                
                <p>
                  Entonces… como usar Html.RenderPartial en Razor? Fácil: usando llaves para indicarle al motor de Razor que eso es un código que debe ejecutar, en lugar de un valor que debe incrustar en la respuesta:
                </p>
                
                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">@{ Html.RenderPartial(<span style="color: #006080">"VistaParcial"</span>); }</pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Así pues: Html.RenderPartial <em>puede</em> usarse en Razor sin ningún problema… como el resto de Helpers que conozcáis. Si el método lo usábais con <% … %> en Razor es @{ … }, mientras que si usábais <%: … %> en Razor es simplemente @…
                    </p>
                    
                    <p>
                      <strong>Otras maneras de incrustar vistas parciales </strong>
                    </p>
                    
                    <p>
                      De todas formas hay un par de métodos más para incrustar vistas parciales.
                    </p>
                    
                    <p>
                      El primer método es <strong>Html.Partial()</strong> un método de extensión adicional. Para llamarlo se usan los mismos parámetros que Html.RenderPartial. La diferencia es que Html.Partial devuelve una IHtmlString con los contenidos de la vista renderizada. Por lo tanto, para incrustar una vista usando Html.Partial() usamos:
                    </p>
                    
                    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #008000">// Razor</span><br />@Html.Partial(<span style="color: #006080">"VistaParcial"</span>)<br /><span style="color: #008000">// Webforms viewengine</span><br />&lt;%: Html.Partial(<span style="color: #006080">"VistaParcial"</span>) %&gt;<br /></pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          El segundo método es propio de Razor, ya que está definido dentro del framework que se conoce como “WebPages” y es usar el método RenderPage, definido en la clase WebPageBase de la cual heredan las vistas Razor.
                        </p>
                        
                        <p>
                          Dicho método acepta dos parámetros:
                        </p>
                        
                        <ol>
                          <li>
                            La localización de la vista. Ojo! No el nombre, sinó su localización (incluyendo directorios)
                          </li>
                          <li>
                            Parámetros a pasar a la vista (params object[]).
                          </li>
                        </ol>
                        
                        <p>
                          P.ej. para renderizar la vista parcial usaríamos:
                        </p>
                        
                        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">@RenderPage(<span style="color: #006080">"~/Views/Home/VistaParcial.cshtml"</span>)</pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              Fijaos en que se debe usar el nombre del archivo de la vista a incluir (incluyendo extensión .cshtml).
                            </p>
                            
                            <p>
                              Si se pasan parámetros a la vista parcial, estos <strong>no</strong> están disponibles usando la propiedad Model en la vista, sinó que debe usarse la propiedad <em>PageData</em>. P.ej. podríamos pasar una cadena y un entero a la vista:
                            </p>
                            
                            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">@RenderPage(<span style="color: #006080">"~/Views/Home/VistaParcial.cshtml"</span>, <span style="color: #006080">"Parametro 1"</span>, 10)</pre>
                              
                              <p>
                                </div> 
                                
                                <p>
                                  Y mostrarlos desde la vista con el uso de PageData:
                                </p>
                                
                                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">@foreach (var item in PageData)<br />{<br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span>@item.Value<span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br />}</pre>
                                  
                                  <p>
                                    </div> 
                                    
                                    <p>
                                      Mi opinión sobre el método RenderPage (en MVC): Sobra totalmente, y espero que nunca, nunca, nunca lo uséis. Porque? Pues porque RenderPage <strong>rompe</strong> la encapsulación del framework (en lugar de especificar un <em>nombre</em> de vista debéis especificar un <em>fichero</em>). Es evidente que existe para dar soporte a WebPages pero WebPages y MVC se parecen sólo porque usan Razor como sintaxis, pero en concepción son dos cosas totalmente distintas… Aunque por razones (supongo que técnicas) Razor depende de WebPages y esa dependencia se arrastra a MVC3, cosa que personalmente no me gusta demasiado. Pero es lo hay… 😉
                                    </p>
                                    
                                    <p>
                                      <strong>Conclusiones</strong>
                                    </p>
                                    
                                    <ol>
                                      <li>
                                        Html.RenderPartial funciona correctamente en Razor, al igual que el resto de métodos de <em>siempre</em>. Sólo debemos tener cuidado en usar la sintaxis correcta.
                                      </li>
                                      <li>
                                        Html.Partial es un método adicional para renderizar vistas parciales. La diferencia con Html.RenderPartial() es que este último escribe en la response directamente el contenido de la vista, mientras que Partial() lo devuelve dentro de una cadena. No tengo claras las implicaciones de rendimiento que puede tener empezar a crear mutltiud de cadenas que serán eliminadas por el GC casi de inmediato.
                                      </li>
                                      <li>
                                        RenderPage es el método de WebPages para renderizar vistas parciales. Desde el punto de vista de MVC es un método que sobra totalmente y que rompe la encapsulación del framework.
                                      </li>
                                    </ol>
                                    
                                    <p>
                                      Espero que os sea útil!
                                    </p>
                                    
                                    <p>
                                      Un saludo!
                                    </p>