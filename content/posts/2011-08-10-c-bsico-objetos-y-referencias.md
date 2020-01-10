---
title: 'C# Básico: Objetos y referencias'

author: eiximenis

date: 2011-08-10T18:22:57+00:00
geeks_url: /?p=1574
geeks_visits:
  - 7803
geeks_ms_views:
  - 4439
categories:
  - Uncategorized

---
La verdad es que ahora hacía bastantes meses que no publicaba nada de la serie “C# básico”. En esta serie pongo posts sobre temas básicos del lenguaje. No es un libro por fascículos, ni un tutorial al uso puesto que los posts no tienen orden en concreto y _nacen_ a partir de inquietudes que observo (mayoritariamente en los foros, pero también por correos que recibo). Todos <a href="http://geeks.ms/blogs/etomas/archive/tags/c_2300_+basico/default.aspx" target="_blank" rel="noopener noreferrer">los posts de esta serie los podéis ver aquí</a>.

<!--more-->

En el post de hoy quiero hablar de la diferencia entre objetos y referencias ya que observo que no siempre está clara. Gente que entiende los conceptos básicos de _<a href="http://geeks.ms/blogs/etomas/archive/2010/07/14/c-b-225-sico-191-que-es-la-herencia.aspx" target="_blank" rel="noopener noreferrer">herencia</a>_ parece liarse en este punto. Muchas veces es un tema pasado rápidamente en muchos libros y tutoriales. Y es que, la verdad, es un tema muy sencillo… 😉 

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">MiClase miObjeto = <span style="color: #0000ff">new</span> MiClase();</pre>
  
  <p>
    </div> 
    
    <p>
      ¿Qué hace este código? En muchos sitios leerás que lo que hace es crear un objeto de la clase <em>MiClase</em>. Eso es cierto, pero describe lo que hace lo que hay <em>a la derecha</em> del símbolo de asignación. Qué hace el código que está a la <em>izquierda</em>? Pues lo que hace es <em>declarar una referencia de tipo MiClase</em>. Otra palabra que se usa muchas veces en lugar de referencia es <em>variable</em> aunque no son técnicamente lo mismo (hay variables que no son referencias y las referencias pueden asignarse a otros elementos que no llamamos usualmente variables como p.ej. los parámetros a una función).
    </p>
    
    <p>
      Las referencias contienen objetos. Yo prefiero decir que las referencias <em>apuntan a</em> objetos (aunque esta palabra parece como “maldita”, sin duda por culpa de los punteros) para que quede claro que <strong>un mismo objeto puede estar contenido en (apuntado por) más de una referencia</strong>.
    </p>
    
    <p>
      <strong>El tipo de una referencia</strong>
    </p>
    
    <p>
      Todas las refencias tienen un tipo. Este tipo <strong>es único e inmutable durante toda la vida de la referencia.</strong> El tipo de una referencia determina que objetos puede contener dicha referencia. En concreto:
    </p>
    
    <ol>
      <li>
        Objetos del mismo tipo. Es decir, una referencia de tipo MiClase puede contener objetos de la clase MiClase.
      </li>
      <li>
        Objetos de una clase derivada de la clase del tipo de la referencia. Si la referencia es de tipo MiClase puede contener objetos de cualquier clase derivada de MiClase.
      </li>
      <li>
        Objetos de cualquier clase que implemente el tipo de la referencia. Eso aplica sólo si el tipo de la referencia es una interfaz. En este caso la referencia puede contener un objeto de cualquier clase que implemente dicha interfaz.
      </li>
    </ol>
    
    <p>
      Todas las clases en .NET <strong>derivan de Object</strong>. Por lo tanto, según el punto (2) una referencia de tipo Object, puede contener cualquier objeto de cualquier clase:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">Object objeto = <span style="color: #0000ff">new</span> CualquierClase();</pre>
      
      <p>
        </div> 
        
        <p>
          ¿Condiciona alguna cosa más el tipo de la referencia? Pues sí: el tipo de la referencia condiciona <em>como vemos al objeto contenido en dicha referencia.</em> Es decir, la referencia <em>es como un disfraz</em> para el objeto. Le permite “ocultar su tipo real” y mostrarse como el “tipo de la referencia”.
        </p>
        
        <p>
          P.ej. dado el siguiente código:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">class</span> MiClase<br />{<br />  <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Foo() {}<br />}<br /><br /><span style="color: #0000ff">class</span> MiClaseDerivada : MiClase<br />{<br />   <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Bar() {}<br />}<br /><br />MiClase c1 = <span style="color: #0000ff">new</span> MiClaseDerivada();<br />MiClaseDerivada c2 = <span style="color: #0000ff">new</span> MiClaseDerivada();</pre>
          
          <p>
            </div> 
            
            <p>
              Podemos ver como <em>MiClase</em> define un método (Foo) y <em>MiClaseDerivada</em> que deriva de MiClase añade el método Bar. Luego c1 es una referencia de tipo MiClase que contiene un objeto de MiClaseDerivada (puede según el punto 2 anterior). Y c2 es una referencia de tipo MiClaseDerivada que contiene un objeto de MiClaseDerivada (posible según el punto 1 anterior). Entonces tenemos que:
            </p>
            
            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">c1.Foo();   <span style="color: #008000">// Ok.</span><br />c1.Bar();   <span style="color: #008000">// No compila.</span><br />c2.Foo();   <span style="color: #008000">// Ok.</span><br />c2.Bar();   <span style="color: #008000">// Ok.</span><br /></pre>
              
              <p>
                </div> 
                
                <p>
                  La llamada c1.Bar() <strong>no compila</strong>. ¿Por que? Pues simplemente porque la referencia es de tipo MiClase. Y MiClase no tiene ningún método Bar. Da igual que el objeto contenido por dicha referencia sea de tipo MiClaseDerivada, que sí que tiene el método Bar. El <strong>compilador no se fija en los tipos de los objetos. Se fija en los tipos de las referencias</strong>.
                </p>
                
                <p>
                  <strong>Objetos compartidos</strong>
                </p>
                
                <p>
                  Como hemos dicho antes un mismo objeto puede estar contenido por más de una referencia:
                </p>
                
                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">MiClaseDerivada c1 = <span style="color: #0000ff">new</span> MiClaseDerivada();<br />MiClase c2 = c1;<br /></pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      En este punto <strong>tenemos dos referencias. Pero un sólo objeto</strong>. Es decir, c1 y c2 contienen el mismo objeto, que es un objeto de tipo MiClaseDerivada. Si accedo al objeto a través de c1 lo veo como un objeto de tipo MiClaseDerivada (ya que este es el tipo de c1). Por otro lado si accedo al objeto a través de c2 lo veo como un objeto de tipo MiClase (al ser este el tipo de c2). Por lo tanto c1.Bar() es correcto y c2.Bar() no compila.
                    </p>
                    
                    <p>
                      Pero insisto: son el <strong>mismo</strong> objeto. Observad el siguiente código:
                    </p>
                    
                    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">class</span> Program<br /> {<br />     <span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">void</span> Main()<br />     {<br />         MiClaseDerivada c1 = <span style="color: #0000ff">new</span> MiClaseDerivada();<br />         MiClase c2 = c1;<br />         c1.Incrementar();<br />         c2.Incrementar();<br />         Console.WriteLine(<span style="color: #006080">"El valor de c1 es:"</span> + c1.Valor);<br />         Console.WriteLine(<span style="color: #006080">"El valor de c2 es:"</span> + c2.Valor);<br />     }<br /> }<br /><br /><br /> <span style="color: #0000ff">class</span> MiClase<br /> {<br />     <span style="color: #0000ff">private</span> <span style="color: #0000ff">int</span> valor;<br /><br />     <span style="color: #0000ff">public</span> <span style="color: #0000ff">int</span> Valor { get { <span style="color: #0000ff">return</span> <span style="color: #0000ff">this</span>.valor; } }<br />     <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> Incrementar()<br />     {<br />         valor++;<br />     }<br /> }<br /><br /> <span style="color: #0000ff">class</span>  MiClaseDerivada : MiClase<br /> {<br />     <span style="color: #008000">// Código</span><br /> }</pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          ¿Cual es la salida por pantalla de dicho código? Pensadlo con detenimiento. Pues&#160; la siguiente:
                        </p>
                        
                        <p>
                          <font face="Courier New">El valor de c1 es:2<br /> <br />El valor de c2 es:2</font>
                        </p>
                        
                        <p>
                          Eso es debido porque c1 y c2 contienen el mismo objeto. Por lo tanto inicialmente tenemos que el valor de dicho objeto es 0. Al llamar a c1.Incrementar() el valor pasa a ser 1. Y al llamar a c2.Incrementar(), el valor pasa a ser 2, ya que el objeto que contiene c2 <strong>es el mismo</strong> que el objeto que contiene c1.
                        </p>
                        
                        <p>
                          Así pues recordadlo siempre: <strong>Asignar una referencia a otra NO crea un nuevo objeto. Simplemente hace que la referencia contenida a la izquierda de la asignación contenga EL MISMO objeto que la referencia situada a la derecha</strong>.
                        </p>
                        
                        <p>
                          <strong>Comparando objetos y referencias.</strong>
                        </p>
                        
                        <p>
                          De nuevo la forma más fácil es verlo con un código de ejemplo:
                        </p>
                        
                        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">class</span> Program<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">void</span> Main()<br />    {<br />        Persona p1 = <span style="color: #0000ff">new</span> Persona();<br />        p1.Nombre = <span style="color: #006080">"Pepito"</span>;<br />        p1.Edad = 20;<br />        Persona p2 = p1;<br />        Persona p3 = <span style="color: #0000ff">new</span> Persona();<br />        p3.Nombre = <span style="color: #006080">"Pepito"</span>;<br />        p3.Edad = 20;<br />        <span style="color: #0000ff">bool</span> b = p2 == p1;<br />        <span style="color: #0000ff">bool</span> b2 = p3 == p2;<br />    }<br />}<br /><br /><span style="color: #0000ff">class</span> Persona<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Nombre { get; set; }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">int</span> Edad { get; set; }<br />}<br /></pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              ¿Cual es el valor de b y b2?
                            </p>
                            
                            <ul>
                              <li>
                                b vale <em>true</em> porque p1 y p2 contienen el mismo objeto
                              </li>
                              <li>
                                b2 vale <em>false</em> porque p3 y p2 contienen objetos distintos. Da igual que los dos objetos sean del mismo tipo y sean idénticos. En este caso son dos <em>Personas</em> idénticas: mismo nombre y edad. Pero el operador == compara referencias, no objetos.
                              </li>
                            </ul>
                            
                            <p>
                              Así pues recuerda: El operador == al comparar referencias devuelve <em>true</em> sólo si las dos referencias contienen el mismo objeto. En caso contrario devuelve<em> false</em> (aunque las dos referencias apunten a dos objetos idénticos).
                            </p>
                            
                            <blockquote>
                              <p>
                                <strong>Nota:</strong> Este comportamiento del operador == puede <strong>modificarse</strong> para que compare el valor de los objetos en lugar de indicar si las dos referencias contienen el mismo objeto. P.ej. la clase string tienen modificado dicho operador para comparar el <em>valor</em> de las cadenas. Esto queda fuera del alcance de este post.
                              </p>
                            </blockquote>
                            
                            <p>
                              La comparación de objetos (es decir, determinar si dos objetos son idénticos pese a ser dos objetos distintos) es algo que por norma general depende de la clase. P.ej. dos Personas serán iguales si tienen el mismo nombre y edad. Dos cadenas serán iguales si contienen los mismos carácteres. Depende de cada clase <em>determinar que significa que dos objetos son iguales</em>. Para estandarizar un poco la comparación de objetos, en .NET tenemos el método Equals. Dicho método está definido en la clase Object y por lo tanto, por herencia, existe en todas las clases. Si quiero indicarle al framework como comparar dos objetos de tipo Persona puedo añadir a la clase Persona el siguiente código:
                            </p>
                            
                            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">override</span> <span style="color: #0000ff">bool</span> Equals(<span style="color: #0000ff">object</span> obj)<br />{<br />    <span style="color: #0000ff">if</span> (obj <span style="color: #0000ff">is</span> Persona)<br />    {<br />        Persona otro = (Persona) obj;<br />        <span style="color: #0000ff">return</span> otro.Edad == Edad &&<br />               otro.Nombre == Nombre;<br />    }<br />    <span style="color: #0000ff">return</span> <span style="color: #0000ff">false</span>;<br />}<br /></pre>
                            </div>
                            
                            <p>
                              Y para comparar los objetos, debo llamar a Equals en lugar del operador ==
                            </p>
                            
                            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">bool</span> b2 = p3.Equals(p2);</pre>
                              
                              <p>
                                </div> 
                                
                                <p>
                                  <strong>Conversiones (castings)</strong>
                                </p>
                                
                                <p>
                                  En el código del método Equals anterior hay el siguiente código:
                                </p>
                                
                                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">Persona otro = (Persona)obj;</pre>
                                  
                                  <p>
                                    </div> 
                                    
                                    <p>
                                      El código (Persona) es lo que se llama casting. El casting lo que hace es <strong>cambiar el tipo de una referencia</strong>. Es decir en el caso anterior obj era una referencia de tipo object (los parámetros también pueden ser referencias). Recordad que las referencias de tipo object pueden contener cualquier objeto. Pero yo quiero acceder a Nombre y Edad que son campos definidos en la clase Persona y por ello necesito una referencia de tipo Persona que me contenga el mismo objeto que la referencia obj.
                                    </p>
                                    
                                    <p>
                                      Si directamente probáramos:
                                    </p>
                                    
                                    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">Persona otro = obj;</pre>
                                      
                                      <p>
                                        </div> 
                                        
                                        <p>
                                          Dicho código no compila. ¿Porque? Pues porque otro es una referencia de tipo Persona y por lo tanto solo puede contener:
                                        </p>
                                        
                                        <ol>
                                          <li>
                                            Un objeto de tipo Persona
                                          </li>
                                          <li>
                                            Un objeto de cualquier clase que derive de Persona
                                          </li>
                                        </ol>
                                        
                                        <p>
                                          Pero obj es una referencia de tipo object y puede contener un objeto de tipo object o bien un objeto de cualquier clase que derive de object… es decir, de cualquier clase. Imaginad, entonces:
                                        </p>
                                        
                                        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">object</span> obj = <span style="color: #0000ff">new</span> Perro();<br />Persona otro = obj;</pre>
                                          
                                          <p>
                                            </div> 
                                            
                                            <p>
                                              Es evidente que el objeto contenido por obj es un perro y no una persona. Si el código de la segunda línea compilase estaríamos viendo un perro como una persona y bueno… se supone que no se puede, no? Por eso, como el compilador no puede garantizar que el objeto (recordad que el compilador no se fija en objetos) contenido por la referencia obj sea de un tipo válido para la referencia otro, se cura en salud y no nos deja compilar el código.
                                            </p>
                                            
                                            <p>
                                              Pero… tu no eres el compilador y tu sí te fijas en los objetos. ¿Qué pasa en aquellos casos en que <strong>tu sabes</strong> que el objeto contenido por la referencia obj es de un tipo válido para la referencia Persona? Pues que debes decírselo al compilador. ¿Cómo? Usando el casting:
                                            </p>
                                            
                                            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">Persona otro = (Persona)obj;</pre>
                                              
                                              <p>
                                                </div> 
                                                
                                                <p>
                                                  Aquí le estás diciendo al compilador: <strong>Quiero que la referencia otro contenga el mismo objeto que la referencia obj y tranquilo, no te quejes porque yo te digo que el objeto es de tipo Persona</strong>. Con el casting el compilador te cree y te deja hacer la asignación.
                                                </p>
                                                
                                                <p>
                                                  Eh… que te crea el compilador no significa que te crea el CLR. El CLR no se fía ni de su madre, así que si tu haces:
                                                </p>
                                                
                                                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">object</span> perro = <span style="color: #0000ff">new</span> Perro();<br />Persona persona = (Persona)perro;<br /></pre>
                                                  
                                                  <p>
                                                    </div> 
                                                    
                                                    <p>
                                                      El compilador no se quejará, pero cuando ejecutes, vas a recibir una hermosa InvalidCastException. El CLR sí que se fija en los objetos, como tu 🙂
                                                    </p>
                                                    
                                                    <p>
                                                      Ah! Y aunque el compilador no se fije en objetos… no lo insultes, eh? No intentes algo como:
                                                    </p>
                                                    
                                                    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                                      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">Perro perro = <span style="color: #0000ff">new</span> Perro();<br />Persona persona = (Persona)perro;<br /></pre>
                                                      
                                                      <p>
                                                        </div> 
                                                        
                                                        <p>
                                                          Eso no compila. La razón es porque no es necesario fijarse en los objetos para ver que una referencia de tipo Persona <strong>nunca podrá contener el mismo objeto</strong> que una referencia de tipo Perro: Persona y Perro no tienen nada en común. El compilador puede no fijarse en los objetos, pero no es tonto!
                                                        </p>
                                                        
                                                        <p>
                                                          Un saludo!
                                                        </p>