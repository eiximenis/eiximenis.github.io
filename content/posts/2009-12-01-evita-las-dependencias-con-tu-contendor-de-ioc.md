---
title: Evita las dependencias con tu contendor de IoC
author: eiximenis

date: 2009-12-01T16:16:21+00:00
geeks_url: /?p=1480
geeks_visits:
  - 2460
geeks_ms_views:
  - 953
categories:
  - Uncategorized

---
Usar un contenedor de IoC es una práctica más que recomendable, pero al hacerlo es muy fácil caer en el anti-patrón de _dependencia con el contenedor_. Ese patrón se manifesta de varias formas sútiles, y aunque hay algunos casos en que pueda ser aceptable, en la gran mayoría indica una mala práctica que debemos revisar.

¿Que tiene de malo este código?

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #008000">// IS1 y IS2 son dos interfaces cualesquiera</span><br /><span style="color: #008000">// S1 y S2 son dos clases que implementan dichas interfaces</span><br /><span style="color: #0000ff">class</span> A<br />{<br />    IS1 s1;<br />    IS2 s2;<br />    <span style="color: #0000ff">public</span> A(IUnityContainer container)<br />    {<br />        <span style="color: #0000ff">this</span>.s1 = container.Resolve&lt;IS1&gt;();<br />        <span style="color: #0000ff">this</span>.s2 = container.Resolve&lt;IS2&gt;();<br />    }<br />}<br /><br /><span style="color: #0000ff">class</span> Program<br />{<br />    <span style="color: #0000ff">static</span> <span style="color: #0000ff">void</span> Main(<span style="color: #0000ff">string</span>[] args)<br />    {<br />        IUnityContainer ctr = <span style="color: #0000ff">new</span> UnityContainer();<br />        ctr.RegisterType&lt;IS1, S1&gt;();<br />        ctr.RegisterType&lt;IS2, S2&gt;();<br />        A a = ctr.Resolve&lt;A&gt;();<br />    }<br />}</pre>
  
  <p>
    </div> 
    
    <p>
      El código funciona correctamente, pero ¡ojo! Tenemos una dependencia directa de la clase A hacia <em>IUnityContainer</em>. <em>Realmente</em> la clase A depende de IUnityContainer o bien depende de IS1 y IS2? La realidad es que las dependencias de la clase A son IS1 e IS2.
    </p>
    
    <p>
      <strong>1. La visión filosófica del asunto</strong>
    </p>
    
    <p>
      Si yo leo el constructor de la clase A y veo que pone:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> A(IUnityContainer container)<br />{<br />    <span style="color: #008000">// Código...</span><br />}</pre>
      
      <p>
        </div> 
        
        <p>
          Debo leer <em>todo</em> el código del constructor para ver las dependencias <em>reales</em> de la clase A. Por otro lado si el constructor fuese:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> A(IS1 s1, IS2 s2)<br />{<br />    <span style="color: #008000">// Código</span><br />}</pre>
          
          <p>
            </div> 
            
            <p>
              Ahora queda mucho más claro que las dependencias <em>reales</em> de la clase A son IS1 y IS2.
            </p>
            
            <p>
              En resumen: <strong>evitad </strong>en lo máximo de lo posible pasar el propio contenedor como parámetro de los constructores. En su lugar pasad las dependencias reales y dejad que el contenedor las inyecte.
            </p>
            
            <p>
              Y obviamente <strong>evitad (casi) siempre</strong> un código como:
            </p>
            
            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">class</span> A<br />{<br />    IUnityContainer container;<br />    <span style="color: #0000ff">public</span> A(IUnityContainer container)<br />    {<br />        <span style="color: #0000ff">this</span>.container = container;<br />    }<br />    <span style="color: #008000">// Código</span><br />}<br /></pre>
              
              <p>
                </div> 
                
                <p>
                  ¡Ahí estamos todavía más vendidos! Para averiguar las dependencias reales de la clase A, ahora debemos mirar <em>todo el código</em> de la clase A, puesto que en cualquier sitio alguien puede hacer un <em>resolve </em>(antes de que alguien salte por las paredes que eche un vistazo al punto 4 del post, por favor :p).
                </p>
                
                <p>
                  La visión “filosófica” me indica que si la clase A debería depender solo de IS1 e IS2 no es posible que me aparezca una dependencia sobre IUnityContainer. Necesita la clase A a IUnityContainer para hacer su trabajo? No, verdad? Pues eso.
                </p>
                
                <p>
                  Incluso aunque tengas claro, clarísimo que nunca vas a abandonar Unity (él no lo haría! :p) este código <em>no huele nada bien</em>.
                </p>
                
                <p>
                  <strong>2. La visión práctica</strong>
                </p>
                
                <p>
                  Algún dia quizá te canses e Unity y te decidas a usar por ejemplo, Windsor Container… Este cambio debería ser un cambio sencillo: un cambio en el bootstrapper de tu aplicación, y donde instanciabas Unity ahora instancias Windsor, lo configuras y listo!
                </p>
                
                <p>
                  Listo? Listo sólo si tus clases <strong>no</strong> dependen de IUnityContainer, porque en caso contrario… bueno, puedes tener un buen problemilla 😉
                </p>
                
                <p>
                  <strong>3. Algunos detalles...</strong>
                </p>
                
                <p>
                  Antes he comentado que este anti-patrón puede aparecer de formas realmente sutiles:
                </p>
                
                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">class</span> A<br />{<br />    <span style="color: #0000ff">public</span> A() <br />    {<br />        <span style="color: #008000">//...</span><br />    }<br />    [Dependency()]<br />    <span style="color: #0000ff">public</span> IS1 S1 { get; set; }<br />    <span style="color: #008000">// Más código</span><br />}<br /></pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      ¿Es correcto este código? Mejor que el código anterior donde recibíamos IUnityContainer como parámetro si que es, porque no tenemos ninguna dependencia directa contra Unity… Pero realmente <em>si</em> que estamos dependiendo de Unity: Sólo Unity entenderá el atributo [Dependency()] para inyectarnos la propiedad S1. Así que a la práctica estamos como en el caso anterior.
                    </p>
                    
                    <p>
                      Ahora bien, la diferencia fundamental es que, en mi opinión, que la clase A reciba IUnityContainer como parámetro rebela un mal diseño, mientras que en este caso la dependencia nos aparece porque <strong>no</strong> existe ningún mecanismo estándar para especificar que queremos que una propiedad sea inyectada (por lo que cada contenedor usa su propio mecanismo).
                    </p>
                    
                    <p>
                      Si tienes claro, clarísimo que nunca abandonarás Unity, entonces <strong>no</strong> hay problema alguno en este código. Por otro lado si no quieres atarte al contenedor entonces este código no te sirve (echa un vistazo a mi post <a href="http://geeks.ms/blogs/etomas/archive/2009/02/02/unity-s-237-gracias-pero-no-me-abraces-demasiado.aspx">Unity? Sí gracias, pero no me abraces demasiado</a> para ver más detalles al respecto).
                    </p>
                    
                    <p>
                      <strong>4. Ya, pero yo uso el patrón Service Locator</strong>
                    </p>
                    
                    <p>
                      Todo lo que hemos hablado hasta ahora afecta sobre todo en aquellos casos en que usábamos inyección de dependencias, pero existe otro patrón íntimamente relacionado: el <em>service locator</em>. En este patrón tenemos un objeto (el propio <em>service locator</em>) que se encarga de devolvernos referencias a servicios.
                    </p>
                    
                    <p>
                      Es común que el propio contenedor de IoC se use como service locator, porque ofrece soporte directo para ello. Sin embargo <strong>no es una buena opción</strong>… porque conduce inevitablemente a situaciones como las que hemos visto, en concreto a situaciones como esta:
                    </p>
                    
                    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">class</span> A<br />{<br />    <span style="color: #0000ff">private</span> IUnityContainer serviceLocator;<br />    <span style="color: #0000ff">public</span> A(IUnityContainer serviceLocator) <br />    {<br />        <span style="color: #0000ff">this</span>.serviceLocator = serviceLocator;<br />    }<br />    <span style="color: #0000ff">void</span> foo()<br />    {<br />        <span style="color: #008000">// obtengo los servicios...</span><br />        var logSvc = serviceLocator.Resolve&lt;ILogService&gt;();<br />        var locSvc = serviceLocator.Resolve&lt;ILocalizationService&gt;();<br />        <span style="color: #008000">// hago cosas con mis servicios</span><br />    }<br />}</pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          Este código es prácticamente igual al que os decía que debéis evitar a toda costa. Podríamos pasar ILogService e ILocalizationService en el constructor, pero ahora imaginad que tenemos muchos servicios y nuestras clases los usan todos (en un proyecto en el que estoy trabajando manejamos decenas de servicios, y además es común que las clases usen muchos de los servicios sólo en un método).
                        </p>
                        
                        <p>
                          El error aquí, está en usar <em>el propio contenedor</em> como Service Locator: lo hacemos porque es rápido y cómodo ya que el contenedor nos ofrece soporte para ello, pero a cambio nos estamos atando al contenedor… Nosotros no queremos una dependencia contra IUnityContainer, sinó una dependencia contra el <em>service locator</em>. Y qué es el service locator? Pues <em>algo distinto</em> al propio contenedor. Por ejemplo, eso:
                        </p>
                        
                        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">interface</span> IServiceLocator<br />{<br />    T GetService&lt;T&gt;() <span style="color: #0000ff">where</span> T : <span style="color: #0000ff">class</span>;<br />}<br /><span style="color: #0000ff">class</span> ServiceLocator : IServiceLocator<br />{<br />    <span style="color: #0000ff">private</span> IUnityContainer container;<br />    <span style="color: #0000ff">public</span> ServiceLocator(IUnityContainer container)<br />    {<br />        <span style="color: #0000ff">this</span>.container = container;<br />    }<br /><br />    <span style="color: #0000ff">public</span> T GetService&lt;T&gt;() <span style="color: #0000ff">where</span> T : <span style="color: #0000ff">class</span><br />    {<br />        <span style="color: #0000ff">return</span> <span style="color: #0000ff">this</span>.container.Resolve&lt;T&gt;();<br />    }<br />}</pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              La clase ServiceLocator si que depende de Unity (ahí si que es inevitable la dependencia). Ahora la clase A la podemos reescribir como:
                            </p>
                            
                            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">class</span> A<br />{<br />    <span style="color: #0000ff">private</span> IServiceLocator serviceLocator;<br />    <span style="color: #0000ff">public</span> A(IServiceLocator serviceLocator) <br />    {<br />        <span style="color: #0000ff">this</span>.serviceLocator = serviceLocator;<br />    }<br />    <span style="color: #0000ff">void</span> foo()<br />    {<br />        <span style="color: #008000">// obtengo los servicios...</span><br />        var logSvc = serviceLocator.GetService&lt;ILogService&gt;();<br />        var locSvc = serviceLocator.GetService&lt;ILocalizationService&gt;();<br />        <span style="color: #008000">// hago cosas con mis servicios</span><br />    }<br />}</pre>
                              
                              <p>
                                </div> 
                                
                                <p>
                                  <p>
                                    Y la clase A ya no depende de IUnityContainer: lo hace de IServiceLocator, lo que es aceptable y totalmente lógico.
                                  </p>
                                  
                                  <p>
                                    Además, tener nuestra propia implementación del <em>service locator</em> nos permite adaptarlo a nuestras necesidades (p. ej. ¿qué hacer si nos piden un servicio que no está registrado?).
                                  </p>
                                  
                                  <p>
                                    Así pues, usar el patrón <em>service locator</em> no es excusa para tener nuestro código lleno de dependencias contra el contenedor de IoC.
                                  </p>
                                  
                                  <p>
                                    ¿Opiniones? 😉
                                  </p>
                                  
                                  <p>
                                    Un saludo a todos!
                                  </p>
                                </p>