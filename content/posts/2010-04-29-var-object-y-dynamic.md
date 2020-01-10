---
title: Var, object y dynamic

author: eiximenis

date: 2010-04-29T08:59:15+00:00
geeks_url: /?p=1505
geeks_visits:
  - 4086
geeks_ms_views:
  - 4542
categories:
  - Uncategorized

---
Hola a todos! El otro día me preguntaban sobre las diferencias entre usar var, object y dynamic, y por lo que he podido observar no todo el mundo tiene claro que diferencias hay en cada caso, de ahí que me haya decidido escribir este post.

<!--more-->

**1. Inferencia de tipos (var)**

Para ver el uso de _var_ lo mejor es un ejemplo:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">var i = 10;         <span style="color: #008000">// Ok</span><br /><span style="color: #0000ff">int</span> i2 = i + 1;     <span style="color: #008000">// Ok</span><br />i = <span style="color: #006080">"20"</span>;           <span style="color: #008000">// error CS0029: Cannot implicitly convert type 'string' to 'int'</span><br /><span style="color: #0000ff">string</span> s = i;       <span style="color: #008000">// error CS0029: Cannot implicitly convert type 'int' to 'string'</span><br />var j;              <span style="color: #008000">// error CS0818: Implicitly-typed local variables must be initialized</span><br />var k = <span style="color: #0000ff">null</span>;       <span style="color: #008000">// error CS0815: Cannot assign &lt;null&gt; to an implicitly-typed local variable</span><br /></pre>
  
  <p>
    </div> 
    
    <p>
      La palabra clave <em>var</em> declara una variable cuyo tipo es inferido a partir de la expresión que se le asigna. Así de sencillo. Quizá hay gente que asume que var declara una variable <em>dinámica</em> debido a la influencia de javascript. Pero el var de C# no tiene nada que ver con el var de javascript. P.ej. en C++ se usa la palabra clave <a href="http://msdn.microsoft.com/en-us/library/dd293667.aspx" target="_blank" rel="noopener noreferrer">auto</a> en lugar de var (de todos modos si buscas información sobre la palabra clave auto en C++ vigila <a href="http://msdn.microsoft.com/en-us/library/6k3ybftz(VS.80).aspx" target="_blank" rel="noopener noreferrer">ya que antes tenía otro significado</a> (que casi nadie utilizaba)).
    </p>
    
    <p>
      Si analizamos el código anterior vemos que la línea <em>var i=10</em> declara una variable <em>i</em> cuyo tipo se infiere de la expresión que se le asigna. Dado que la expresión 10 se resuelve a tipo <em>int</em>, la variable i es de tipo <em>int</em>. Podemos ver que podemos asignar <em>i</em> a otra variable de tipo int, pero no podemos asignar i a una variable de tipo string, ni tampoco asignar un string a i (la variable i <strong>no</strong> es de tipo dinámico y por lo tanto no puede cambiar de tipo).&#160; También podemos observar que no podemos declarar una variable con var <strong>sin</strong> asignarle expresión (lógico puesto que entonces el compilador no sabe el tipo de dicha variable) y que tampoco la podemos declarar asignándole <em>null</em> (lógico porque null no tiene tipo).
    </p>
    
    <p>
      Si os preguntáis para que existe <em>var</em>, no es (sólo) para complacer a los perezosos sino para dar soporte a los <strong>tipos anónimos</strong>.
    </p>
    
    <p>
      <strong>2. Dynamic vs object</strong>
    </p>
    
    <p>
      Visual Studio 2010 viene con la nueva palabra clave <em>dynamic</em> que <strong>ahora sí</strong> nos permite declarar una variable de tipo dinámico:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">dynamic i = 10;         <span style="color: #008000">// Ok</span><br /><span style="color: #0000ff">int</span> i2 = i + 1;         <span style="color: #008000">// Ok</span><br />i = <span style="color: #006080">"20"</span>;               <span style="color: #008000">// Ok</span><br /><span style="color: #0000ff">string</span> s = i;           <span style="color: #008000">// Ok</span><br />dynamic j;              <span style="color: #008000">// Ok</span><br />dynamic k = <span style="color: #0000ff">null</span>;       <span style="color: #008000">// Ok</span><br /></pre>
      
      <p>
        </div> 
        
        <p>
          Ahora nuestra variable <em>i</em> es de tipo dinámico y es por ello que le podemos asignar un int (como en la primera línea) o bien una cadena (como en la tercera) y del mismo modo podemos asignar la variable <em>i</em> a una cadena. Ojo! Que podamos asignar la variable <em>i</em> a una cadena no significa que sea válido hacerlo: en tiempo de ejecución se realiza la transformación y puede ser que nos de un error. P.ej. el siguiente código compila pero (obviamente) da una excepción en ejecución:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">dynamic i = 10;<br />Guid guid = i;</pre>
          
          <p>
            </div> 
            
            <p>
              Si lo probamos vemos que sí, que compila, pero en ejecución nos lanza la excepción <em>RuntimeBinderException</em> con el mensaje <em>Cannot implicitly convert type &#8216;int&#8217; to &#8216;System.Guid&#8217;</em>.
            </p>
            
            <p>
              Veamos ahora más temas interesantes sobre <em>dynamic</em>. Por ejemplo, que creeis que imprime por pantalla el siguiente código:
            </p>
            
            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">dynamic i = 10;<br />Console.WriteLine(i.GetType().FullName);<br />i = <span style="color: #006080">"20"</span>;<br />Console.WriteLine(i.GetType().FullName);</pre>
              
              <p>
                </div> 
                
                <p>
                  Pues esto:
                </p>
                
                <p>
                  <font size="2" face="Courier">System.Int32<br /> <br />System.String</font>
                </p>
                
                <p>
                  Es decir, vemos que aunque la variable i se haya declarado como dynamic, cuando se ejecuta el método GetType() se ejecuta sobre el objeto <em>real</em> contenido por i.
                </p>
                
                <p>
                  Alguien puede decir que si en el código anterior cambiamos <em>dynamic</em> por <em>object</em> el resultado es idéntico… Entonces… ¿donde está la diferencia? ¿Cuando debo usar dynamic?
                </p>
                
                <p>
                  Bien, simplificando podemos asumir lo siguiente: En tiempo de ejecución las variables dynamic se traducen a <em>object</em> (el CLR no entiende de <em>dynamic</em>). Pero cuando usamos dynamic el compilador desactiva toda comprobación de tipos, cosa que no ocurre cuando usamos object. Compara los dos códigos:
                </p>
                
                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #008000">// Compila</span><br />dynamic d = <span style="color: #006080">"eiximenis"</span>;<br /><span style="color: #0000ff">string</span> str = d.ToUpper();<br /><span style="color: #008000">// NO compila</span><br /><span style="color: #0000ff">object</span> d2 = <span style="color: #006080">"eiximenis"</span>;        <span style="color: #008000">// Ok</span><br /><span style="color: #0000ff">string</span> str2 = d2.ToUpper();     <span style="color: #008000">// error CS1061: 'object' does not contain a definition for 'ToUpper' and no extension method 'ToUpper' accepting a first argument of type 'object' could be found (are you missing a using directive or an assembly reference?)</span><br /></pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      El primer código compila mientras que el segundo <strong>no</strong>, puesto que aunque la variable d2 contiene un objeto de tipo string, la referencia es de tipo object y object no contiene ningún método ToUpper. Mientras que en el caso de dynamic el compilador <em>asume</em> que sabemos lo que estamos haciendo, así que desactiva la comprobación de tipos y listos… Por supuesto si en tiempo de ejecución el objeto referido por la variable dinámica no contiene el método especificado… excepción al canto.
                    </p>
                    
                    <p>
                      O sea que dynamic no es más que un <em>truco</em> que nos proporciona el compilador: el CLR no sabe nada de dynamic, es el compilador de C# quien <em>hace toda la magia</em>. ¿Quieres otro ejemplo de ello? Aquí lo tienes:
                    </p>
                    
                    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">List&lt;dynamic&gt; lst = <span style="color: #0000ff">new</span> List&lt;dynamic&gt;();<br />List&lt;<span style="color: #0000ff">object</span>&gt; lst2 = <span style="color: #0000ff">new</span> List&lt;<span style="color: #0000ff">object</span>&gt;();<br /><span style="color: #0000ff">bool</span> b = lst.GetType() == lst2.GetType();<br /><span style="color: #008000">// b vale true</span><br /></pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          La variable b vale <em>true</em> porque en tiempo de ejecución, tanto lst como lst2 son una List<object>, dado que dynamic se traduce en tiempo de ejecución por object.
                        </p>
                        
                        <p>
                          <strong>3. DLR</strong>
                        </p>
                        
                        <p>
                          Bueno… hemos dicho que cuando usamos dynamic, el compilador lo que hace es básicamente declarar la variable como object y suspender su comprobación de tipos… pero que más hace? Es decir, como traduce:
                        </p>
                        
                        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">d.foo();    <span style="color: #008000">// d es dynamic</span><br /></pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              Siendo d una variable declarada como dynamic.
                            </p>
                            
                            <p>
                              Lo que <em>podría</em> hacer el compilador es simplemente “no traducirlo por nada”, es decir generar el mismo código (IL) como si d fuese una variable tradicional. P.ej. dado el siguiente código C#:
                            </p>
                            
                            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">int</span> i = 0;<br />i.ToString();<br /></pre>
                              
                              <p>
                                </div> 
                                
                                <p>
                                  El compilador lo traduce en el siguiente código IL (se puede ver con ildasm):
                                </p>
                                
                                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #008000">// int i=0;</span><br />ldc.i4.0        <span style="color: #008000">// Cargamos el valor 0 a la pila</span><br />stloc.0         <span style="color: #008000">// Sacamos el top de la pila y lo guardamos en la var #0 (i)</span><br /><span style="color: #008000">// i.ToString();</span><br />ldloca.s   i    <span style="color: #008000">// Ponemos la dirección de la variable #0 (i) en la pila</span><br /><span style="color: #008000">// Llamamos al método ToString. El valor the 'this' se obtiene del top de la pila</span><br />call       instance <span style="color: #0000ff">string</span> [mscorlib]System.Int32::ToString()</pre>
                                  
                                  <p>
                                    </div> 
                                    
                                    <p>
                                      Una opción que tendría el compilador si i estuviese declarada como <em>dynamic</em> en lugar de int seria generar el mismo IL, es decir una llamada tradicional a call. Si en tiempo de ejecución el método indicado no se encuentra en la clase, el CLR da un error.
                                    </p>
                                    
                                    <p>
                                      Otra opción que tiene el compilador es usar <em>Reflection</em>, es decir traducir la llamada d.foo(); a un código <em>parecido</em> a:
                                    </p>
                                    
                                    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #008000">// código original es d.foo();</span><br />var mi = d.GetType().GetMethods().FirstOrDefault(x =&gt; x.Name.Equals(<span style="color: #006080">"foo"</span>));<br /><span style="color: #0000ff">object</span> retval = mi.Invoke(d, <span style="color: #0000ff">null</span>);</pre>
                                      
                                      <p>
                                        </div> 
                                        
                                        <p>
                                          Este segundo método es más elegante puesto que el compilador podría añadir código propio para gestionar los errores (p.ej. comprobar si mi es null). De hecho el primer método (no traducir nada y generar un IL parecido al que hemos visto) sería muy bestia ya que estamos confiando en la seguridad del CLR y no es esa su función.
                                        </p>
                                        
                                        <p>
                                          Bueno… supongo que si te imaginas que si te estoy metiendo ese rollo es para decirte que el compilador no usa ninguna de esas dos opciones. En su lugar utiliza llamadas al <a href="http://msdn.microsoft.com/en-us/library/dd233052.aspx" target="_blank" rel="noopener noreferrer">DLR</a>. ¿Y que es el DLR? Pues un conjunto de servicios (construídos encima del CLR) para añadir soporte a lenguajes dinámicos en .NET.
                                        </p>
                                        
                                        <p>
                                          Te puedes preguntar porque necesitamos el DLR y no podemos usar simplemente Reflection. Bien, aunque con Reflection podemos simular llamadas dinámicas, los lenguajes dinámicos permiten <em>más</em> cosas, como p.ej. añadir en tiempo de ejecución métodos a clases o objetos ya existentes. Hacer esto con Reflection es imposible, puesto que Reflection nos permite invocar cualquier miembro de una clase, pero dicho miembro debe estar definido en la clase cuando esta se crea (no se pueden añadir miembros en tiempo de ejecución).
                                        </p>
                                        
                                        <p>
                                          Así pues dado que tenemos al DLR que nos ofrece soporte para tipos dinámicos, el compilador de C# usa llamadas al DLR cuando debe resolver llamadas a miembros de objetos contenidas en variables declaradas como <em>dynamic</em>. Así pues podemos ver que una referencia dynamic se traduce en tiempo de ejecución (gracias al compilador) en una referencia object pero que usará el DLR para acceder a sus miembros.
                                        </p>
                                        
                                        <p>
                                          <strong>4. ExpandoObject</strong>
                                        </p>
                                        
                                        <p>
                                          Vamos a ver un poco el poder del DLR en acción. Y un ejemplo sencillo y rápido es la clase <em>ExpandoObject</em>.
                                        </p>
                                        
                                        <p>
                                          La clase <em>ExpandoObject</em> representa un objeto <strong>al que en tiempo de ejecución se le pueden añadir o quitar propiedades y/o métodos</strong>. Fíjate en el siguiente código:
                                        </p>
                                        
                                        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">static</span> <span style="color: #0000ff">void</span> Main(<span style="color: #0000ff">string</span>[] args)<br />{<br />    dynamic eo = <span style="color: #0000ff">new</span> ExpandoObject();<br />    eo.MiPropiedad = 10;<br />    eo.MiOtraPropiedad = <span style="color: #006080">"Cadena"</span>;<br />    Dump(eo);<br />    Console.ReadLine();<br />}<br /><br /><span style="color: #0000ff">static</span> <span style="color: #0000ff">void</span> Dump(dynamic d)<br />{<br />    Console.WriteLine(<span style="color: #006080">"MiPropiedad:"</span> + d.MiPropiedad);<br />    Console.WriteLine(<span style="color: #006080">"MiOtraPropiedad:"</span> + d.MiOtraPropiedad);<br />}</pre>
                                          
                                          <p>
                                            </div> 
                                            
                                            <p>
                                              Creamos un ExpandoObject y luego <em>creamos</em> las propiedades MiPropiedad y MiOtraPropiedad. Crear una propiedad en un ExpandoObject es tan simple como asignarle un valor (ojo! la propiedad sólo se crea cuando se asigna un valor a ella, no cuando se consulta). Luego en el método Dump consultamos dichas propiedades y obtenemos sus valores.
                                            </p>
                                            
                                            <p>
                                              Aquí el uso de dynamic es obligatorio: No podemos declarar la variable eo como <em>ExpandoObject</em> ya que entonces no podemos “añadir propiedades”. Al declarar la variable como dynamic, hacemos que el código compile (el compilador no comprueba que existan las propiedades) <strong>y que se use el DLR para llamar a las propiedades MiPropiedad y MiOtraPropiedad</strong>. La clase ExpandoObject se integra con el DLR (a través de la interfaz <a href="http://msdn.microsoft.com/en-us/library/dd630200(v=VS.100).aspx" target="_blank" rel="noopener noreferrer">IDynamicMetaObjectProvider</a>) y eso es lo que permite que se añadan esas propiedades al objeto en cuestión.
                                            </p>
                                            
                                            <p>
                                              Resumiendo pues, hemos visto que <em>var</em> simplemente sirve para declarar variables cuyo tipo se infiere de la expresión que se les asigna (necesario para poder asignar un objeto anónimo a una variable) mientras que dynamic es el mecanismo que tenemos en C# para declarar una variable, para la cual el compilador suspenda la comprobación de tipos por un lado y que genere código para usar el DLR por otro.
                                            </p>
                                            
                                            <p>
                                              Saludos!
                                            </p>