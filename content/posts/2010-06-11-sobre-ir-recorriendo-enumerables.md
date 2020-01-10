---
title: Sobre ir recorriendo enumerables…

author: eiximenis

date: 2010-06-11T15:28:01+00:00
geeks_url: /?p=1517
geeks_visits:
  - 4228
geeks_ms_views:
  - 1887
categories:
  - Uncategorized

---
El otro día, Oren Eini (aka <a href="http://ayende.com" target="_blank" rel="noopener noreferrer">Ayende</a>) escribió en su blog un post, en respuesta a otro post escrito por Phil Haack (aka <a href="http://haacked.com" target="_blank" rel="noopener noreferrer">Haacked</a>). En su post Phil mostraba un método extensor para comprobar si un IEnumerable<T> era null o estaba vacío (y sí, Phil usa <a href="http://geeks.ms/blogs/jmaguilar/archive/2010/05/20/191-esa-enumeraci-243-n-est-225-vac-237-a.aspx" target="_blank" rel="noopener noreferrer">Any() en lugar de Count() para comprobar si la enumeración está vacía</a>):

<!--more-->

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">bool</span> IsNullOrEmpty&lt;T&gt;(<span style="color: #0000ff">this</span> IEnumerable&lt;T&gt; items) {<br />    <span style="color: #0000ff">return</span> items == <span style="color: #0000ff">null</span> || !items.Any();<br />}</pre>
  
  <p>
    </div> 
    
    <p>
      Aquí tenéis el post de Phil: <a href="http://haacked.com/archive/2010/06/10/checking-for-empty-enumerations.aspx">Checking For Empty Enumerations</a>
    </p>
    
    <p>
      Y este es el de Oren: <a href="http://ayende.com/Blog/archive/2010/06/10/checking-for-empty-enumerations.aspx">Checking For Empty Enumerations</a>
    </p>
    
    <p>
      Oren plantea una cuestión muy interesante al respecto de los enumerables y es la <strong>posibilidad de que haya enumeraciones que sólo se puedan recorrer una sóla vez</strong>. En este caso el método de Phil fallaría, puesto que al llamar a .Any() para validar si hay un elemento este elemento sería leído (y por lo tanto se perdería) por lo que cuando después recorriesemos el enumerable no obtendríamos el primer elemento.
    </p>
    
    <p>
      Pero el tema es… ¿pueden existir enumerables de un sólo recorrido? Pues poder, pueden pero <strong>mi opinión personal es que no son enumerables válidos.</strong>
    </p>
    
    <p>
      Vayamos por partes… Que es un enumerable? Pues algo tan simple como esto:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">interface</span> IEnumerable&lt;<span style="color: #0000ff">out</span> T&gt; : IEnumerable<br />{<br />    IEnumerator&lt;T&gt; GetEnumerator();<br />}</pre>
      
      <p>
        </div> 
        
        <blockquote>
          <p>
            <strong>Nota:</strong> Si no os suena eso de “out” es una novedad de C# 4 que nos permite especificar covarianza en el tipo genérico. No afecta a lo que estamos discutiendo en este post. Tenéis más info en <a href="http://www.matthidinger.com/archive/2010/04/07/introduction-to-c-4-covariance-with-the-is-keyword.aspx" target="_blank" rel="noopener noreferrer">este clarificador post de Matt Hiddinger</a>.
          </p>
        </blockquote>
        
        <p>
          Bueno… resumiendo lo único que podemos hacer con un enumerable es… obtener un enumerador. Y que es un enumerador? Pues eso:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">interface</span> IEnumerator&lt;<span style="color: #0000ff">out</span> T&gt; : IDisposable, IEnumerator<br />{<br />    T Current { get; }<br />    <span style="color: #008000">// Esto se hereda de IEnumerator</span><br />    <span style="color: #0000ff">object</span> Current { get; }<br />    <span style="color: #0000ff">bool</span> MoveNext();<br />    <span style="color: #0000ff">void</span> Reset();<br />    <span style="color: #008000">// Esto se hereda de IDisposable</span><br />    <span style="color: #0000ff">void</span> Dispose();<br />}</pre>
          
          <p>
            </div> 
            
            <p>
              Un enumerador es lo que se <em>recorre</em>: Tenemos una propiedad (Current) que nos permite obtener el elemento actual así como un método MoveNext() que debe moverse al siguiente elemento (devolviendo true si este movimiento ha sido válido). Hay otro método adicional Reset() que debe posicionar el enumerador <em>antes</em> del primer elemento, <a href="http://msdn.microsoft.com/en-us/library/system.collections.ienumerator.reset.aspx" target="_blank" rel="noopener noreferrer">aunque en la propia MSDN se indica que no es un método de obligada implementación</a>. Así pues, ciertamente no podemos asumir que un IEnumerator se pueda recorrer más de una vez. Así que no lo hagáis: asumid que los IEnumerator sólo pueden recorrerse una vez.
            </p>
            
            <p>
              Pero que un IEnumerator sólo pueda recorrerse una vez no implica que no pueda obtener dos, tres o los que quiera IEnumerator a partir del mismo IEnumerable: puedo llamar a GetEnumerator() tantas veces como quiera.
            </p>
            
            <p>
              Y, ahí está el quid de la cuestión del post de Oren: el método Any() <strong>crea</strong> un IEnumerator y luego el foreach <strong>crea otro</strong> IEnumerator. Así pues en este código:
            </p>
            
            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">void</span> foo(IEnumerable&lt;T&gt; els)<br />{<br />   <span style="color: #0000ff">if</span> (els.Any()) {<br />       <span style="color: #0000ff">foreach</span> (var el <span style="color: #0000ff">in</span> els) { ... }<br />   }<br />}</pre>
              
              <p>
                </div> 
                
                <p>
                  Se crean dos IEnumerator: uno cuando se llama a Any() y otro cuando se usa el foreach. <strong>Y ambos IEnumerators nos permiten recorrer el IEnumerable des del principio</strong>: por eso <strong>no</strong> perdemos ningún elemento (al contrario de lo que afirma Oren en su post).
                </p>
                
                <p>
                  <strong>Conclusión</strong>
                </p>
                
                <p>
                  Antes he dicho que pueden existir IEnumerables que solo se puedan recorrer una sola vez, pero que en <strong>mi opinión no son correctos</strong>. Cuando digo que pueden existir me refiero a que se pueden crear, cuando digo que (en mi opinión) no son correctos me refiero a que según la msdn (<a title="http://msdn.microsoft.com/en-us/library/system.collections.ienumerable.getenumerator.aspx" href="http://msdn.microsoft.com/en-us/library/system.collections.ienumerable.getenumerator.aspx">http://msdn.microsoft.com/en-us/library/system.collections.ienumerable.getenumerator.aspx</a>) la llamada a GetEnumerator debe:
                </p>
                
                <ul>
                  <li>
                    Devolver un enumerador (IEnumerator) que itere sobre la colección (Returns an enumerator that iterates through a collection).
                  </li>
                  <li>
                    Inicialmente el enumerador debe estar posicionado antes del primer elemento (Initially, the enumerator is positioned before the first element in the collection).
                  </li>
                </ul>
                
                <p>
                  Por lo tanto de aquí yo interpreto que <strong>cada vez</strong> que llame a GetEnumerator obtendré un enumerador <strong>posicionado antes del primer elemento</strong>, y dado que en ningún momento se me avisa que un IEnumerable pueda admitir una SOLA llamada a GetEnumerator(), entiendo que puedo obtener tantos enumeradores como quiera y que cada llamada me devolverá un IEnumerator posicionado antes del primer elemento.
                </p>
                
                <p>
                  Así que podéis usar el método de Phil para comprobar si un IEnumerable está vacío sin miedo a perder nunca el primer elemento!
                </p>
                
                <p>
                  Un saludo!
                </p>