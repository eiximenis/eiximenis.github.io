---
title: '[C# Básico] Delegates'
author: eiximenis

date: 2010-07-21T14:56:00+00:00
geeks_url: /?p=1527
geeks_visits:
  - 23411
geeks_ms_views:
  - 5421
categories:
  - Uncategorized

---
Hola a todos! Este es el tercer post de esa &ldquo;serie&rdquo; de C# Básico. En [el primero vimos las interfaces][1] y en [segundo intenté responder a la pregunta de que es la herencia][2].

Hoy quiero hablaros de un tema sobre el que bastante gente tiene dificultades y sobre el que hay bastante confusión, pero que en el fondo es mucho más simple de lo que parece! Sí, me estoy refiriendo a los delegates (o delegados).

**1. La necesidad de los delegates**

Cuando queremos enviar valores a un método que son desconocidos para este, lo que hacemos es usar parámetros. Vamos, a nadie le sorprende el código:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">int</span> Sumar (<span style="color: #0000ff">int</span> a, <span style="color: #0000ff">int</span> b)<br />{<br />   <span style="color: #0000ff">return</span> a + b;<br />}</pre>
  
  <p>
    </div> 
    
    <p>
      El método Sumar recibe dos parámetros y hace con ellos lo que sea (en este caso sumarlos). El uso de parámetros es vital porque sino deberíamos declarar un método por cada combinación posible de valores... En nuestro caso deberíamos hacer un método Sumar1y1 que sumase 1+1, otro Sumar1y2 que sumase 1+2...&nbsp; ya se ve que eso por un lado es imposible y por otra un poco (bastante) estúp&igrave;do. El uso de parámetros es algo que todos aprendemos más bien pronto al desarrollar y con los que todos nos sentimos cómodos.
    </p>
    
    <p>
      Un parámetro tiene básicamente dos cosas:
    </p>
    
    <ul>
      <li>
        Un nombre, que se usa dentro del método para referirse a él
      </li>
      <li>
        Y un tipo que indica que <em>clase</em> de valores puede tener el parámetro.
      </li>
    </ul>
    
    <p>
      El tipo es lo que me importa ahora: Si el parámetro es de tipo <em>int </em>puede contener cualquier entero, si es un <em>double</em> puede contener cualquier valor en coma flotante y si es <em>FooClass</em> puede contener cualquier objeto de la clase <em>FooClass</em>. Nada nuevo bajo el sol.
    </p>
    
    <p>
      Bien, ahora imaginad que teneis una lista de cadenas y queréis ordenarla. Para ello os creais un método tal que:
    </p>
    
    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">void</span> SortList(List&lt;<span style="color: #0000ff">string</span>&gt; lst)<br />{<br />   <span style="color: #008000">// ...</span><br />}</pre>
      
      <p>
        </div> 
        
        <p>
          El método recibe un parámetro que es la lista a ordenar. Da igual que algoritmo de ordenación implementéis, en algún momento deberéis hacer algo como:
        </p>
        
        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">void</span> SortList(List&lt;<span style="color: #0000ff">string</span>&gt; lst)<br />{<br />   <span style="color: #008000">// En str1 y str2 tenemos dos elementos que debemos comparar</span><br />   <span style="color: #0000ff">if</span> (FirstStringGoesBefore(str1, str2)) { ... }<br />}</pre>
          
          <p>
            </div> 
            
            <p>
              El método <em>FirstStringGoesBefore</em> es un método helper vuestro que indica si una cadena va antes que otra <span style="text-decoration: underline;">según vuestro criterio de ordenación</span>. P.ej. podría ser:
            </p>
            
            <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
              <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">bool</span> FirstString (<span style="color: #0000ff">string</span> first, <span style="color: #0000ff">string</span> second)<br />{<br />   <span style="color: #0000ff">return</span> first.Length &lt;= second.Length;<br />}</pre>
              
              <p>
                </div> 
                
                <p>
                  Esto ordenaria las cadenas según su número de carácteres.
                </p>
                
                <p>
                  Vale... ¿observas un pequeño problema? ¿Cuál?
                </p>
                
                <p>
                  &iexcl;Exacto! Nuesta función SortList tiene el criterio de ordenación fijado, siempre ordena las cadenas según su número de caracteres! El algoritmo para ordenar cadenas (implementado en SortList) siempre es igual sea cual sea el criterio. Pero el criterio está en el método FirstStringGoesBefore() que decide, dadas dos cadenas, cual va antes que cual. No se a vosotros pero a mi, criterios de ordenación de cadenas me salen muchísimos: por número de carácteres, alfabéticamente, según representación numérica (es decir &ldquo;01&rdquo; antes que &ldquo;112&rdquo; porque 1 es menor que 112), etc, etc
                </p>
                
                <p>
                  La pregunta es <strong>debo hacer un método SortList para cada criterio? Es decir debo hacer un SortListByLength, otro SortListByAlphabetic,...</strong>
                </p>
                
                <p>
                  Dicho de otro modo: <strong>no le podríamos pasar al método SortList el criterio a utilizar como un parámetro? </strong>Ya, pero resulta que el criterio de ordenación resulta que es un método, no un valor (como int) o un objeto. Pues bien si comprendes la necesidad de pasar métodos como parámetros, comprendes la necesidad de los delegates. Y es que <strong>un delegate no es nada más que la posibilidad de pasar un método como parámetro</strong>.
                </p>
                
                <p>
                  <strong>2. Declración de un delegate</strong>
                </p>
                
                <p>
                  Cuando declaramos un parámetro, el <em>tipo</em> de ese parámetro determina que valores son válidos. Si el parámetro és int el valor 2 es correcto pero el valor 3.14159 no, ya que cae fuera de los valores válidos para int. Si el parámetro es <em>FooClass</em> entonces cualquier objeto <em>FooClass</em> es válido, pero un objeto de cualquier otra clase (p.ej. string) pues no.
                </p>
                
                <p>
                  Los delegates sirven para poder pasar métodos como parámetros y también hay <em>varios tipos de delegates</em>. Tiene lógica: no es lo mismo un método que espera dos cadenas y devuelve un bool que otra que espera dos ints y no devuelve nada. El primero sería un criterio de ordenación válido, la segunda no. Si yo creo el método SortList y admito un delegate como parámetro para el criterio de ordenación, es lógico que quiera asegurarme que sólo funciones que aceptan dos strings y devuelven bool són válidas. Total: yo voy a usar ese delegate pasándole dos cadenas y esperando un bool.
                </p>
                
                <p>
                  Por ello, cuando se define un delegate debe definirse que métodos acepta ese delegate: o sea los parámetros y el valor de retorno que los métodos deben tener.
                </p>
                
                <p>
                  Así no es de extrañar que la definición de un delegate se parezca sospechosamente a la definición de un método:
                </p>
                
                <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">delegate</span> <span style="color: #0000ff">bool</span> CriterioOrdenacion(<span style="color: #0000ff">string</span> str1, <span style="color: #0000ff">string</span> str2);</pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Fijaos en el uso de la palabra clave delegate, que indica que lo que viene a continuación es la declaración de un delegate (y no de un método). El ejemplo declara un delegate llamado <em>CriterioOrdenacion</em> que permite pasar como parámetro métodos que acepten dos cadenas y que devuelva bool.
                    </p>
                    
                    <blockquote>
                      <p>
                        <strong>Nota:</strong> Que las dos cadenas se llamen str1 y str2 es totalmente superficial y no tiene ninguna importancia. Si yo creo un método que acepte dos cadenas y devuelva un bool podré pasarlo como parámetro usando este delegate aunque las cadenas no se llamen str1 y str2. Personalmente creo que el hecho de que en la declaración de un delegate los parámetros aparezcan nombrados sólo crea confusión.
                      </p>
                    </blockquote>
                    
                    <p>
                      <strong>3. Uso de un delegate</strong>
                    </p>
                    
                    <p>
                      Una vez tengo definido el delegate ya puedo usarlo. Cuando defino un delegate creo un nuevo tipo de datos. Así pues la el método SortList quedaría como:
                    </p>
                    
                    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">void</span> SortList(List&lt;<span style="color: #0000ff">string</span>&gt; lst, CriterioOrdenacion criterio)<br />{<br />    <span style="color: #008000">// ...</span><br />    <span style="color: #0000ff">if</span> (criterio(first, second)) {...}<br />}</pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          Fijaos en dos cosas:
                        </p>
                        
                        <ol>
                          <li>
                            El parámetro <em>criterio</em> cuyo tipo es CriterioOrdenacion que es el delegate que habíamos definido antes.
                          </li>
                          <li>
                            La &ldquo;invocación&rdquo; al delegate. Para invocar a un delegate se invoca como si fuese un método. Es decir criterio(first, second) llamará realmente al método que hayamos recibido como parámetro. Nosotros no sabemos cual es ese método, pero gracias al delegate <ol>
                              <li>
                                Sabemos que recibe dos cadenas y devuelve un bool
                              </li>
                              <li>
                                Podemos invocarlo
                              </li>
                            </ol>
                          </li>
                        </ol>
                        
                        <p>
                          Bien, ahora nos falta ver como podemos llamar al método SortList pasándole un criterio de ordenación en concreto. Para ello hemos de ver como instanciar un delegate.
                        </p>
                        
                        <p>
                          Imaginad que tengo un método llamado CriterioOrdenacionSegunLength que queremos usar como un criterio de ordenación:
                        </p>
                        
                        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">bool</span> CriterioOrdenacionSegunLength(<span style="color: #0000ff">string</span> s1, <span style="color: #0000ff">string</span> s2)<br />{<br />    <span style="color: #0000ff">return</span> s1.Length &lt;= s2.Length;<br />}</pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              Y ahora quiero invocar al método SortList usando el método CriterioOrdenacionSegunLength como criterio de ordenación. Para ello defino una variable de tipo <em>CriterioOrdenacion</em> (el delegate) y la instancío con el método:
                            </p>
                            
                            <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                              <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">CriterioOrdenacion miCriterio = <br /><span style="color: #0000ff">new</span> CriterioOrdenacion(CriterioOrdenacionSegunLength);</pre>
                              
                              <p>
                                </div> 
                                
                                <p>
                                  Para instanciar un delegate se usa <em>new</em> al igual que para instanciar una clase, y se pasa <strong>el nombre del método</strong> con el que se instancia este delegate.
                                </p>
                                
                                <p>
                                  Y ahora ya puedo llamar a SortList:
                                </p>
                                
                                <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                                  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">SortList(lst, miCriterio);</pre>
                                  
                                  <p>
                                    </div> 
                                    
                                    <p>
                                      Y listos! El método SortList usará ahora el criterio de ordenación del delegate <em>miCriterio</em> que es el método <em>CriterioOrdenacionSegunLength</em>.
                                    </p>
                                    
                                    <p>
                                      <strong>4. Delegates y eventos</strong>
                                    </p>
                                    
                                    <p>
                                      Mucha gente confunde delegates y eventos, aunque no es exactamente lo mismo. Ya hemos visto un delegate, y lo que conocemos como &ldquo;evento&rdquo; se basa, ciertamente, en delegates. Lo que pasa es que podemos ver un evento como <em>una lista de delegates</em> (lo que en .NET llamamos un <a href="http://msdn.microsoft.com/en-us/library/system.multicastdelegate.aspx">MulticastDelegate</a>). Cuando en C# estamos haciendo:
                                    </p>
                                    
                                    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                                      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">btn.Click += <span style="color: #0000ff">new</span> EventHandler(btn_Click);</pre>
                                      
                                      <p>
                                        </div> 
                                        
                                        <p>
                                          Estáis creando un delegate de tipo EventHandler y lo añadís a &ldquo;la lista&rdquo; de delegates del evento. Cuando el botón lanza el evento, lo que hace es invocar uno a uno todos los delegates de la lista, y así es como se llama al método <em>btn_Click</em>. Si mirais como está definido EventHandler vereis que su definición es:
                                        </p>
                                        
                                        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                                          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">delegate</span> <span style="color: #0000ff">void</span> EventHandler(<span style="color: #0000ff">object</span> sender, EventArgs e);</pre>
                                          
                                          <p>
                                            </div> 
                                            
                                            <p>
                                              De ahí los famosos dos parámetros que tenemos en (casi) todas nuestras funciones gestoras de eventos!
                                            </p>
                                            
                                            <p>
                                              <strong>5. Delegates y threads</strong>
                                            </p>
                                            
                                            <p>
                                              Hay gente que también confunde delegates con threads. Delegates ya hemos visto que son, y los threads no son nada más que ejecutar un código en <em>segundo plano</em>. La confusión viene porque para crear un thread debemos decirle cual es el código a ejecutar en segundo plano. Es decir que método ejecutar en segundo plano. Es decir, debemos pasarle un delegate.
                                            </p>
                                            
                                            <p>
                                              <strong>6. Para ver más...</strong>
                                            </p>
                                            
                                            <p>
                                              Me he dejado cosas en tintero, porque sino este post, honestamente, no se terminaría nunca... Si os interesa profundizar sobre el tema, sabed que me he dejado adrede (os dejo algunos links):
                                            </p>
                                            
                                            <ol>
                                              <li>
                                                <a href="http://www.switchonthecode.com/tutorials/csharp-tutorial-the-built-in-generic-delegate-declarations">Delegates y genéricos</a>
                                              </li>
                                              <li>
                                                <a href="http://www.codedigest.com/Articles/CSHARP/5_Delegates_and_Anonymous_methods_in_C_.aspx">Delegates y métodos anónimos</a>
                                              </li>
                                              <li>
                                                <a href="http://blogs.msdn.com/b/ericwhite/archive/2006/10/03/lambda-expressions.aspx">Lambda expressions</a>
                                              </li>
                                              <li>
                                                <a href="http://msdn.microsoft.com/en-us/library/ms228976.aspx">Delegates y reflection</a>
                                              </li>
                                              <li>
                                                <a href="http://msdn.microsoft.com/en-us/library/ms173174(VS.80).aspx">Reglas de covarianza y contravarianza de los delegates</a>
                                              </li>
                                            </ol>
                                            
                                            <p>
                                              Si alguien está interesado en profundizar sobre alguno de esos temas, que me lo diga y veremos que podemos hacer al respecto! 😉
                                            </p>
                                            
                                            <p>
                                              Espero que este post os haya ayudado a tener más claro el concepto de un delegate!
                                            </p>
                                            
                                            <p>
                                              Un saludo! 😉
                                            </p>

 [1]: /blogs/etomas/archive/2010/07/07/c-b-225-sico-interfaces.aspx
 [2]: /blogs/etomas/archive/2010/07/14/c-b-225-sico-191-que-es-la-herencia.aspx