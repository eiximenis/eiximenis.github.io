---
title: Novedades de Unity 2.0
description: Novedades de Unity 2.0
author: eiximenis

date: 2010-06-02T15:02:56+00:00
geeks_url: /?p=1515
geeks_visits:
  - 2264
geeks_ms_views:
  - 1087
categories:
  - Uncategorized

---
<a href="http://unity.codeplex.com/" target="_blank" rel="noopener noreferrer">Unity</a>, el <a href="http://martinfowler.com/articles/injection.html" target="_blank" rel="noopener noreferrer">contenedor IoC</a> de Microsoft, hace algunas semanas que tiene nueva versión: la 2.0. Y viene con algunas novedades interesantes respecto a la versión anterior, que os comento brevemente 🙂

**Por fin… un único assembly!**

Vale que Unity era poco más que un _wrapper_ sobre <a href="http://objectbuilder.codeplex.com/" target="_blank" rel="noopener noreferrer">ObjectBuilder2</a> pero tampoco era necesario que nos lo recordaran continuamente… ahora ya no tendremos que añadir la referencia a ObjectBuilder2 además de la referencia a Unity cada vez… Ahora existe un solo assembly: Microsoft.Practices.Unity.dll. ¿Un detalle sin apenas importancia? Pues probablemente 🙂

**Resoluciones deferidas**

Esta funcionalidad nos permite resolver tipos con Unity sin tener una dependencia contra Unity. P.ej. imagina que tenemos Unity configurado de la siguiente manera:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">IUnityContainer uc = <span style="color: #0000ff">new</span> UnityContainer();<br />uc.RegisterType&lt;IA, A&gt;();</pre>
  
  <p>
    </div> 
    
    <p>
      Y tenemos un método que necesita construir un objeto IA a través de Unity… Antes debíamos pasarle el contenedor Unity:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">static</span> <span style="color: #0000ff">void</span> OldFoo(IUnityContainer ctr)<br />{<br />    <span style="color: #008000">// ... Código antes de resolver IA ...</span><br />    IA a = ctr.Resolve&lt;IA&gt;();<br />}</pre>
      
      <p>
        </div> 
        
        <p>
          Ahora, con Unity 2.0 podemos obtener un <em>resolvedor</em> (perdonad por la palabreja) que no es nada más que un delegate que cuando sea invocado llamará a Resolve de Unity. Esto nos permite que la función OldFoo <em>no tenga dependencia alguna contra Unity</em>:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">static</span> <span style="color: #0000ff">void</span> NewFoo(Func&lt;IA&gt; resolver)<br />{<br />    <span style="color: #008000">// ... Código antes de resolver IA ...</span><br />    IA a = resolver();<br />}</pre>
          
          <p>
            </div> 
            
            <p>
              Y la llamada a NewFoo quedaría de la forma:
            </p>
            
            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">var resolver = uc.Resolve&lt;Func&lt;IA&gt;&gt;();<br />NewFoo(resolver);</pre>
              
              <p>
                </div> 
                
                <p>
                  De esta manera el método NewFoo no necesita para nada el contenedor Unity 🙂
                </p>
                
                <p>
                  <strong>Paso de parámetros en Resolve</strong>
                </p>
                
                <p>
                  Esto permite pasar parámetros a un objeto cuando se llama a su método Resolve.
                </p>
                
                <p>
                  Supon que tenemos lo siguiente:
                </p>
                
                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">interface</span> IB { }<br /><br /><span style="color: #0000ff">class</span> B : IB<br />{<br />    <span style="color: #0000ff">public</span> B(<span style="color: #0000ff">int</span> <span style="color: #0000ff">value</span>) { }<br />}</pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Y tenemos un mapping declarado en Unity:
                    </p>
                    
                    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">IUnityContainer uc = <span style="color: #0000ff">new</span> UnityContainer();<br />uc.RegisterType&lt;IB, B&gt;();</pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          Como es de esperar la siguiente llamada falla:
                        </p>
                        
                        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">IB b = uc.Resolve&lt;IB&gt;();<br /><span style="color: #008000">// Exception is: InvalidOperationException - The type Int32 cannot be constructed.</span></pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              Unity se queja, porque cuando va al constructor de la clase B se encuentra que le debe pasar un parámetro de tipo Int32 (int). Pero como no tiene ningún mapeo de int, se queja.
                            </p>
                            
                            <p>
                              Para solucionar este caso podemos usar los <em>ResolverOverride</em>. P.ej. para pasarle el valor 3 al parámetro “value” basta con usar:
                            </p>
                            
                            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">IB b = uc.Resolve&lt;IB&gt;(<span style="color: #0000ff">new</span> ParameterOverride(<span style="color: #006080">"value"</span>, 3));</pre>
                              
                              <p>
                                </div> 
                                
                                <p>
                                  Pero no solo nos sirven para especificar valores de parámetros cuyos tipos no estén registrados en el contenedor, también podemos indicarle valor a un parámetro <em>incluso</em> en el caso que Unity pudiese proveer un valor para el parámetro:
                                </p>
                                
                                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">interface</span> IA { }<br /><span style="color: #0000ff">class</span> A : IA { }<br /><span style="color: #0000ff">interface</span> IB { IA A { get; } }<br /><span style="color: #0000ff">class</span> B : IB<br />{<br />    <span style="color: #0000ff">public</span> IA A { get; <span style="color: #0000ff">private</span> set; }<br />    <span style="color: #0000ff">public</span> B(IA a)<br />    {<br />        A = a;<br />    }<br />}<br /><br /><span style="color: #0000ff">class</span> Program<br />{<br />    <span style="color: #0000ff">static</span> <span style="color: #0000ff">void</span> Main(<span style="color: #0000ff">string</span>[] args)<br />    {<br />        IUnityContainer uc = <span style="color: #0000ff">new</span> UnityContainer();<br />        uc.RegisterType&lt;IA, A&gt;(<span style="color: #0000ff">new</span> ContainerControlledLifetimeManager());<br />        uc.RegisterType&lt;IB, B&gt;();<br />        IA a1 = uc.Resolve&lt;IA&gt;();<br />        IA a2 = uc.Resolve&lt;IA&gt;();<br />        IB b = uc.Resolve&lt;IB&gt;();<br />        IB b2 = uc.Resolve&lt;IB&gt;(<span style="color: #0000ff">new</span> ParameterOverride(<span style="color: #006080">"a"</span>, <span style="color: #0000ff">new</span> A()));<br />    }<br />}<br /></pre>
                                  
                                  <p>
                                    </div> 
                                    
                                    <p>
                                      Dado el siguiente código tenemos que:
                                    </p>
                                    
                                    <ol>
                                      <li>
                                        a1 == a2 es <em>true</em> puesto que IA está registrado como singleton, por lo que todas las llamadas a Resolve<IA> devuelven el mismo valor
                                      </li>
                                      <li>
                                        a1 == b.A es <em>true</em> por la misma razón de antes: Cuando Unity debe proporcionar un valor IA al constructor de la clase B, utiliza el método Resolve por lo que devuelve el singleton.
                                      </li>
                                      <li>
                                        a1 == b2.A es <em>false</em> porque aquí aunque Unity puede proporcionar un valor para el parámetro, el valor especificado en el <em>ParameterOverride</em> tiene preferencia.
                                      </li>
                                    </ol>
                                  </p></p> 
                                  
                                  <p>
                                    <strong>Por fin!!! Podemos saber que hay registrado en el contenedor</strong>
                                  </p>
                                  
                                  <p>
                                    Se han añadido métodos para saber todos los registros de mapeos del contenedor y para saber si un mapeo en concreto existe:
                                  </p>
                                  
                                  <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">IUnityContainer uc = <span style="color: #0000ff">new</span> UnityContainer();<br />uc.RegisterType&lt;IA, A&gt;(<span style="color: #0000ff">new</span> ContainerControlledLifetimeManager());<br />uc.RegisterType&lt;IB, B&gt;();<br /><span style="color: #0000ff">bool</span> isTrue = uc.IsRegistered&lt;IA&gt;();<br /><span style="color: #0000ff">bool</span> isFalse = uc.IsRegistered&lt;IC&gt;();<br /><span style="color: #0000ff">int</span> numRegs = uc.Registrations.Count();</pre>
                                    
                                    <p>
                                      </div> 
                                      
                                      <p>
                                        Por cierto, que dado el siguiente código… cuanto vale numRegs?
                                      </p>
                                      
                                      <p>
                                        Sí habeis dicho <strong>2</strong>, habéis fallado… recordad que Unity se <em>registra</em> a si mismo, por lo que realmente numRegs vale 3 (el propio registro de Unity, el de IA y el de IB).
                                      </p>
                                      
                                      <p>
                                        <strong>InjectionFactory</strong>
                                      </p>
                                      
                                      <p>
                                        InjectionFactory es un mecanismo que permite indicarle a Unity un método (una Func<IUnityContainer, object>, usualmente una lambda expresion) a usar cada vez que deba resolver un objeto especificado. Permite que Unity use factorías nuestras.
                                      </p>
                                      
                                      <p>
                                        Imagina que tenemos una factoría para crear objetos IA:
                                      </p>
                                      
                                      <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                        <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">interface</span> IFactory<br />{<br />    IA GetNewInstance();<br />}<br /><span style="color: #0000ff">class</span> Factory : IFactory<br />{<br />    <span style="color: #0000ff">public</span> IA GetNewInstance()<br />    {<br />        <span style="color: #008000">// Hace lo que tenga que hacer nuestra factoria... </span><br />        <span style="color: #0000ff">return</span> <span style="color: #0000ff">new</span> A();<br />    }<br />}</pre>
                                        
                                        <p>
                                          </div> 
                                          
                                          <p>
                                            Y ahora deseamos usar Unity para la creación de objetos IA. Pues podemos realizar el siguiente Register:
                                          </p>
                                          
                                          <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                            <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">IUnityContainer uc = <span style="color: #0000ff">new</span> UnityContainer();<br />uc.RegisterType&lt;IFactory, Factory&gt;(<span style="color: #0000ff">new</span> ContainerControlledLifetimeManager());<br />uc.RegisterType&lt;IA&gt;(<span style="color: #0000ff">new</span> InjectionFactory<br />    (x =&gt; x.Resolve&lt;IFactory&gt;().GetNewInstance()));</pre>
                                            
                                            <p>
                                              </div> 
                                              
                                              <p>
                                                En la última línea le estamos indicando a Unity que cada vez que alguien haga un Resolve<IA> ejecute el delegate que le indicamos (en este caso que obtenga una IFactory y llame al método GetNewInstance).
                                              </p>
                                              
                                              <p>
                                                Esto si lo combinamos con lo de las resoluciones diferidas es realmente interesante.
                                              </p>
                                            </p>
                                            
                                            <p>
                                              Y estas serían las novedades, a mi juicio, más interesantes de Unity 2 (que no son todas… si las queréis saber todas, <a href="http://unity.codeplex.com/wikipage?title=Unity2ChangeLog&referringTitle=Unity2#changes" target="_blank" rel="noopener noreferrer">las tenéis aquí</a>!).
                                            </p>