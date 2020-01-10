---
title: '[C# Básico] Interfaces'

author: eiximenis

date: 2010-07-07T12:45:26+00:00
geeks_url: /?p=1524
geeks_visits:
  - 26894
geeks_ms_views:
  - 19653
categories:
  - Uncategorized

---
Hola a todos! El otro día recibí un correo que decía lo siguiente:

> _¿Podrías escribir algo sobre el uso de Interfaces? Yo por ahi he leido que es algo recomendado crear interfaces que es como un patrón.. Yo la verdad no las uso en mis proyectos pero me gustaría saber para qué sirven y porque se deberían usar y en qué casos._

<!--more-->

Reconozco que es un correo para reflexionar: muchas veces tendemos a escribir sobre lo _más_: lo más avanzado, lo más novedoso, lo más _cool_… y quizá nos olvidamos de que hay gente que lo que busca son artículos para iniciarse. Es cierto que [Tutoriales de C#][1] los hay a patadas en internet, pero una cosa es un tutorial planteado como un curso y otra un pequeño post sobre algún tema básico concreto.

Por esto me he decidido a hacer este post. Como digo en el título es C# básico, pero si como quien me mandó el correo, tienes dudas sobre como funcionan las interfaces en C#… bienvenido a bordo! 😉

Seguiré la serie [C# Básico] si veo que hay demanda de ella, es decir que si alguien me propone que escriba sobre temas concretos (a nivel de introducción) no tengo ningún problema en hacerlo! 🙂

**1.¿ Que es la interfaz de una clase?**

En teoría de orientación a objetos, la interfaz de una clase es _todo lo que podemos hacer con ella_. A efectos prácticos: todos los métodos, propiedades y variables públicas (aunque no deberían haber nunca variables públicas, debemos usar propiedades en su lugar) de la clase conforman _su interfaz_.

Dada la siguiente clase:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">class</span> Contenedor<br />{<br />   <span style="color: #0000ff">public</span> <span style="color: #0000ff">int</span> Quitar();<br />   <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Meter(<span style="color: #0000ff">int</span> v);<br />   <span style="color: #0000ff">private</span> <span style="color: #0000ff">bool</span> EstaRepetido(<span style="color: #0000ff">int</span> v);<br />}</pre>
  
  <p>
    </div> 
    
    <p>
      Su interfaz está formada por los métodos <em>Quitar</em> y <em>Meter</em>. El método <em>EstaRepetido</em> <strong>no</strong> forma parte de la interfaz de dicha clase, ya que es privado.
    </p>
    
    <p>
      En orientación a objetos decimos que la interfaz de una clase define <strong>el comportamiento</strong> de dicha clase, ya que define que podemos y que no podemos hacer con objetos de dicha clase: dado un objeto de la clase <em>Contenedor</em> yo puedo llamar al método <em>Quitar</em> y al métdo <em>Meter</em> pero no puedo llamar al método <em>EstaRepetido</em>.
    </p>
    
    <p>
      Así pues: <strong>toda clase tiene una interfaz</strong> que define que podemos hacer con los objetos de dicha clase.
    </p>
  </p>
  
  <p>
    <strong>2. Interfaces idénticas no significa clases intercambiables</strong>
  </p>
  
  <p>
    Fíjate en estas dos clases:
  </p>
  
  <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">class</span> Contenedor<br />{<br />   <span style="color: #0000ff">public</span> <span style="color: #0000ff">int</span> Quitar() { ... }<br />   <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Meter (<span style="color: #0000ff">int</span> v) { ... }<br />}<br /><span style="color: #0000ff">class</span> OtroContenedor<br />{<br />   <span style="color: #0000ff">public</span> <span style="color: #0000ff">int</span> Quitar() { ... }<br />   <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Meter (<span style="color: #0000ff">int</span> v) { ... }<br />}<br /></pre>
    
    <p>
      </div> 
      
      <p>
        Que puedes deducir de ellas? Exacto! Su inerfaz es la misma: con ambas clases podemos hacer lo mismo: llamar al método <em>Quitar</em> y al método <em>Meter</em>.
      </p>
      
      <p>
        Ahora imagina que en cualquier otro sitio tienes un método definido tal y como sigue:
      </p>
      
      <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
        <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> foo (Contenedor c)<br />{<br />   <span style="color: #008000">// Hacer cosas con c como p.ej:</span><br />   <span style="color: #0000ff">int</span> i = c.Quitar();<br />   c.Meter(10);<br />}</pre>
        
        <p>
          </div> 
          
          <p>
            El método recibe un <em>Contenedor</em> y opera con él. Ahora dado que las interfaces de <em>Contenedor</em> y <em>OtroContenedor</em> son iguales, uno podría esperar que lo siguiente funcionase:
          </p>
          
          <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
            <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">OtroContenedor oc = <span style="color: #0000ff">new</span> OtroContenedor();<br />foo(oc);</pre>
            
            <p>
              </div> 
              
              <p>
                Pero esto <strong>no va a compilar</strong>. ¿Por que? Pues aunque nosotros somos capaces leyendo el código de comparar la interfaz de ambas clases, el compilador no puede hacer esto. Para el compilador <em>Contenedor</em> y <em>OtroContenedor</em> son <strong>dos clases totalmente distintas sin ninguna relación</strong>. Por lo tanto un método que espera un <em>Contenedor</em> no puede aceptar un objeto de la clase <em>OtroContenedor</em>.
              </p>
              
              <p>
                Quiero recalcar que el hecho de que el compilador no <em>compare las interfaces de las clases</em> no se debe a una imposibilidad técnica ni nada parecido: se debe a que no tiene sentido hacerlo.
              </p>
              
              <p>
                ¿Por que? Pues simplemente porque las interfaces son idénticas por pura casualidad. Supón que fuese legal llamar a foo con un objeto <em>OtroContenedor</em>, ok?
              </p>
              
              <p>
                Entonces podría pasar lo siguiente:
              </p>
              
              <ol>
                <li>
                  Alguien <strong>añade</strong> un método público a la clase <em>Contenedor</em>.
                </li>
                <li>
                  Se modifica el método foo para que llame a dicho método nuevo. Eso es legal porque foo espera un <em>Contenedor</em> como parámetro
                </li>
                <li>
                  La llamada a foo(oc) donde oc es <em>OtroContenedor</em>… como debe comportarse ahora? <em>OtroContenedor</em> no tiene el método nuevo que se añadió a <em>Contenedor</em>!
                </li>
              </ol>
              
              <p>
                Así pues: dos clases con la misma interfaz <strong>no tienen relación alguna entre ellas y por lo tanto no se pueden intercambiar.</strong>
              </p>
              
              <p>
                <strong>3. Implementación de interfaces</strong>
              </p>
              
              <p>
                El ejemplo anterior ejemplifica un caso muy común: tener dos clases que hacen <em>lo mismo</em> pero de diferente manera. P.ej. imagina que <em>Contenedor</em> está implementado usando un array en memoria y <em>OtroContenedor</em> está implementando usando, que sé yo, pongamos un fichero en disco. La funcionalidad (la interfaz) es la misma, lo que varía es <strong>la implementación.</strong> Es por ello que en programación orientada a objetos decimos que las interfaces son funcionalidades (o comportamientos) y las clases representen implementaciones.
              </p>
              
              <p>
                Ahora bien, si dos clases representan <strong>dos implementaciones distintas de la misma funcionalidad</strong>, es muy enojante (y estúpido) que no las podamos intercambiar. Para que dicho intercambio sea posible C# (y en general cualquier lenguaje orientado a objetos) permite explicitar la interfaz, es decir <strong>separar la declaración de la interfaz de su implementación</strong> (de su clase). Para ello usamos la palabra clave <em>interface</em>:
              </p>
              
              <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">interface</span> IContenedor<br />{<br />   <span style="color: #0000ff">int</span> Quitar();<br />   <span style="color: #0000ff">void</span> Meter(<span style="color: #0000ff">int</span> i);<br />}</pre>
                
                <p>
                  </div> 
                  
                  <p>
                    Este código declara una interfaz <em>IContenedor</em> que declara los métodos <em>Quitar y Meter</em>. Fíjate que los métodos no se declaran como public (en una interfaz la visibilidad no tiene sentido, ya que todo es public) y que <strong>no se implementan los métodos</strong>.
                  </p>
                  
                  <p>
                    Las interfaces son un concepto más teórico que real. No se pueden crear interfaces. El siguiente código NO compila:
                  </p>
                  
                  <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">IContenedor c = <span style="color: #0000ff">new</span> IContenedor();<br /><span style="color: #008000">// Error: No se puede crear una interfaz!</span><br /></pre>
                    
                    <p>
                      </div> 
                      
                      <p>
                        Es lógico que NO podamos crear interfaces, ya que si se nos dejara, y luego hacemos c.Quitar()… que método se llamaría si el método Quitar() no está implementado?
                      </p>
                      
                      <p>
                        Aquí es donde volvemos a las clases: podemos indicar explícitamente que una clase <strong>implementa</strong> una interfaz, es decir <strong>proporciona implementación (código) a todos y cada uno de los métodos (y propiedades) declarados en la interfaz:</strong>
                      </p>
                      
                      <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                        <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">class</span> Contenedor : IContenedor<br />{<br />   <span style="color: #0000ff">public</span> <span style="color: #0000ff">int</span> Quitar() { ... }<br />   <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Meter(<span style="color: #0000ff">int</span> i) { ... }<br />}</pre>
                        
                        <p>
                          </div> 
                          
                          <p>
                            La clase <em>Contenedor</em> declara explícitamente que implementa la interfaz <em>IContenedor</em>. Así pues la clase <strong>debe proporcionar implementación para todos los métodos</strong> de la interfaz. El siguiente código p.ej. no compila:
                          </p>
                          
                          <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                            <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">class</span> Contenedor : IContenedor<br />{<br />   <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Meter(<span style="color: #0000ff">int</span> i) { ... }<br />}<br /><span style="color: #008000">// Error: Y el método Quitar()???</span><br /></pre>
                            
                            <p>
                              </div> 
                              
                              <p>
                                Es por esto que en orientación a objetos decimos que las interfaces son <em>contratos</em>, porque si yo creo la clase la interfaz me obliga a implementar ciertos métodos y si yo uso la clase, la interfaz me dice que métodos puedo llamar.
                              </p>
                              
                              <p>
                                Y ahora viene lo bueno: Si dos clases <strong>implementan la misma interfaz</strong> son intercambiables. Dicho de otro modo, en cualquier sitio donde se espere una instancia de la interfaz puede pasarse una instancia de cualquier clase que implemente dicha interfaz.
                              </p>
                              
                              <p>
                                Podríamos declarar nuestro método foo anterior como sigue:
                              </p>
                              
                              <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">void</span> foo(IContenedor c)<br />{<br />   <span style="color: #008000">// Cosas con c...</span><br />   c.Quitar();<br />   c.Meter(10);<br />}</pre>
                                
                                <p>
                                  </div> 
                                  
                                  <p>
                                    Fíjate que la clave es que el parámetro de foo está declarado como <em>IContenedor</em>, no como <em>Contenedor</em> o <em>OtroContenedor</em>, con esto indicamos que el método foo() trabaja con cualquier objeto de cualquier clase que implemente IContenedor.
                                  </p>
                                  
                                  <p>
                                    Y ahora, si supones que tanto Contenedor como OtroContenedor implementan la interfaz IContenedor el siguiente código es válido:
                                  </p>
                                  
                                  <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">Contenedor c = <span style="color: #0000ff">new</span> Contenedor();<br />foo(c);    <span style="color: #008000">// Ok. foo espera IContenedor y Contenedor implementa IContenedor</span><br />OtroContenedor oc = <span style="color: #0000ff">new</span> OtroContenedor();<br />foo(oc); <span style="color: #008000">// Ok. foo espera IContenedor y OtroContenedor implementa IContenedor</span><br /><span style="color: #008000">// Incluso esto es válido:</span><br />IContenedor ic = <span style="color: #0000ff">new</span> Contenedor();<br />IContenedor ic2 = <span style="color: #0000ff">new</span> OtroContenedor();<br /></pre>
                                    
                                    <p>
                                      </div> 
                                      
                                      <p>
                                        <strong>4. ¿Cuando usar interfaces?</strong>
                                      </p>
                                      
                                      <p>
                                        En general siempre que tengas, o preveas que puedes tener más de una clase para hacer lo mismo: usa interfaces. Es mejor pecar de exceso que de defecto en este caso. No te preocupes por penalizaciones de rendimiento en tu aplicación porque no las hay.´
                                      </p>
                                      
                                      <p>
                                        No digo que <strong>toda</strong> clase deba implementar una interfaz obligatoriamente, muchas clases <strong>internas</strong> no lo implementarán, pero en el caso de las clases <strong>públicas (visibles desde el exterior</strong>) deberías pensarlo bien. Además pensar en la interfaz antes que en la clase en sí, es <em>pensar en lo que debe hacerse</em> en lugar de pensar <em>en como debe hacerse</em>. Usar interfaces permite a posteriori cambiar una clase por otra que implemente la misma interfaz y poder integrar la nueva clase de forma mucho más fácil (sólo debemos modificar donde instanciamos los objetos pero el resto de código queda igual).
                                      </p>
                                      
                                      <p>
                                        <strong>5. Segregación de interfaces</strong>
                                      </p>
                                      
                                      <p>
                                        Imagina que tenemos un sistema que debe trabajar con varios vehículos, entre ellos aviones y coches, así que declaramos la siguiente interfaz:
                                      </p>
                                      
                                      <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                        <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">interface</span> IVehiculo<br />{<br />    <span style="color: #0000ff">void</span> Acelerar(<span style="color: #0000ff">int</span> kmh);   <br />    <span style="color: #0000ff">void</span> Frenar();   <br />    <span style="color: #0000ff">void</span> Girar(<span style="color: #0000ff">int</span> angulos);   <br />    <span style="color: #0000ff">void</span> Despegar();   <br />    <span style="color: #0000ff">void</span> Aterrizar();<br />}</pre>
                                        
                                        <p>
                                          </div> 
                                          
                                          <p>
                                            Luego implementamos la clase avión:
                                          </p>
                                          
                                          <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                            <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">class</span> Avion : IVehiculo<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Acelerar(<span style="color: #0000ff">int</span> kmh) { ... }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Frenar() { ... }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Girar (<span style="color: #0000ff">int</span> angulos) { ... }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Despegar() { ... }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Aterrizar() { ... }<br />}</pre>
                                            
                                            <p>
                                              </div> 
                                              
                                              <p>
                                                Y luego vamos a por la clase coche… y aquí surge el problema:
                                              </p>
                                              
                                              <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                                <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">class</span> Coche : IVehiculo<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Acelerar(<span style="color: #0000ff">int</span> kmh) { ... }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Frenar() { ... }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Girar (<span style="color: #0000ff">int</span> angulos) { ... }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Despegar() {<span style="color: #0000ff">throw</span> <span style="color: #0000ff">new</span> NotImplementedException(<span style="color: #006080">"Coches no vuelan"</span>); }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Aterrizar(){<span style="color: #0000ff">throw</span> <span style="color: #0000ff">new</span> NotImplementedException(<span style="color: #006080">"Coches no vuelan"</span>); }<br />}</pre>
                                                
                                                <p>
                                                  </div> 
                                                  
                                                  <p>
                                                    La interfaz IVehiculo tiene demasiados métodos y no define el comportamiento de todos los vehículos, dado que no todos los vehículos despegan y aterrizan. En este caso es mejor dividir la interfaz en dos:
                                                  </p>
                                                  
                                                  <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                                    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">interface</span> IVehiculo<br />{<br />    <span style="color: #0000ff">void</span> Acelerar(<span style="color: #0000ff">int</span> kmh);<br />    <span style="color: #0000ff">void</span> Frenar();<br />    <span style="color: #0000ff">void</span> Girar (<span style="color: #0000ff">int</span> angulos);<br />}<br /><br /><span style="color: #0000ff">interface</span> IVehiculoVolador : IVehiculo<br />{<br />    <span style="color: #0000ff">void</span> Despegar();<br />    <span style="color: #0000ff">void</span> Aterrizar();<br />}</pre>
                                                    
                                                    <p>
                                                      </div> 
                                                      
                                                      <p>
                                                        Fíjate además que <em>IVehiculoVolador</em> deriva de <em>IVehiculo </em>(en orientación a objetos decimos que hay una relación de herencia entre IVehiculoVolador y IVehiculo), <strong>eso significa que una clase que implemente IVehiculoVolador debe implementar también IVehiculo forzosamente</strong>. Por lo tanto podemos afirmar que todos los vehículos voladores son también vehículos.
                                                      </p>
                                                    </p></p> 
                                                    
                                                    <p>
                                                      Ahora si que la clase Coche puede implementar IVehiculo y la clase Avion puede implementar IVehiculoVolador (y por lo tanto también IVehiculo). Si un método foo() recibe un objeto IVehiculoVolador puede usar métodos tanto de IVehiculoVolador como de IVehiculo:
                                                    </p>
                                                    
                                                    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                                      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">void</span> foo (IVehiculoVolador vv)<br />{<br />   vv.Acelerar(10);   <span style="color: #008000">// Ok. Acelerar es de IVehiculo y IVehiculoVolador deriva de IVehiculo</span><br />   vv.Despegar();     <span style="color: #008000">// Ok. Despegar es de IVehiculoVolador</span><br />}</pre>
                                                      
                                                      <p>
                                                        </div> 
                                                        
                                                        <p>
                                                          Al reves no! Si un método foo recibe un IVehiculo <strong>no puede llamar a métodos de IVehiculoVolador</strong>. Lógico: todos los vehículos voladores son vehículos pero al revés no… no todos los vehículos son vehículos voladores!
                                                        </p>
                                                        
                                                        <p>
                                                          Siempre que haya segregación no tiene por que haber herencia de interfaces. Imagina el caso de que además de vehículos debemos tratar con Armas de guerra. Tenemos otra interfaz:
                                                        </p>
                                                        
                                                        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                                          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">interface</span> IArmaDeGuerra<br />{<br />    <span style="color: #0000ff">void</span> Apuntar();<br />    <span style="color: #0000ff">void</span> Disparar();<br />}</pre>
                                                          
                                                          <p>
                                                            </div> 
                                                            
                                                            <p>
                                                              Ahora podrían existir clases que implementen <em>IArmaDeGuerra</em> como p.ej. una torreta defensiva:
                                                            </p>
                                                            
                                                            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                                              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">class</span> TorretaDefensiva : IArmaDeGuerra<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Apuntar() { ... }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Disparar() { ... }<br />}</pre>
                                                              
                                                              <p>
                                                                </div> 
                                                                
                                                                <p>
                                                                  Pero claro… también tenemos vehículos que pueden ser a la vez armas de guerra, p.ej. un tanque! Que hacemos? <strong>Ningún problema: una clase puede implementar más de una interfaz a la vez! </strong>Para ello debe implementar todos los métodos de todas la interfaces:
                                                                </p>
                                                                
                                                                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                                                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">class</span> Tanque : IVehiculo, IArmaDeGuerra<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Acelerar(<span style="color: #0000ff">int</span> kmh) { ... }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Frenar() { ... }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Girar (<span style="color: #0000ff">int</span> angulos) { ... }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Apuntar() { ... }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Disparar() { ... }<br />}</pre>
                                                                  
                                                                  <p>
                                                                    </div> 
                                                                    
                                                                    <p>
                                                                      Ahora si un método foo() recibe un IVehiculo le puedo pasar un Tanque y si otro método foo2 recibe un IArmaDeGuerra también le puedo pasar un Tanque. O dicho de otro modo, los tanques se comportan como vehículos y como armas de guerra a la vez!
                                                                    </p>
                                                                    
                                                                    <p>
                                                                      Así pues, es importante segregar bien nuestras interfaces porque en caso contrario vamos a tener dificultades a la hora de implementarlos. La segregación de interfaces es uno <strong>de los 5 principios SOLID </strong>(concretamente la letra I: <a href="http://dotnetcenter.it/articles/10/SOLID-5-simple-principles-Interface-Segregation-Principle-Part-1.html">interface segregation principle</a>).
                                                                    </p>
                                                                    
                                                                    <p>
                                                                      Bueno… espero que este post os haya ayudado a comprender mejor que son las interfaces y como usarlas!
                                                                    </p>
                                                                    
                                                                    <p>
                                                                      Un saludo a todos!
                                                                    </p>

 [1]: http://www.google.es/search?hl=es&rls=com.microsoft%3Aes&q=Tutorial+C%23&aq=f&aqi=&aql=&oq=&gs_rfai=