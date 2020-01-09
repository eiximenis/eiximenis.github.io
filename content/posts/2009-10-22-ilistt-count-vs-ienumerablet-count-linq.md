---
title: 'IList<T>.Count vs IEnumerable<T>.Count() (LINQ)'
author: eiximenis

date: 2009-10-22T17:22:52+00:00
geeks_url: /?p=1475
geeks_visits:
  - 4250
geeks_ms_views:
  - 1312
categories:
  - Uncategorized

---
Te has preguntado alguna vez la diferencia de rendimiento que pueda haber entre el método extensor Count() proporcionado por LINQ y la propiedad Count de la interfaz IList<T>.

Es decir dado el siguiente código:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">List&lt;<span style="color: #0000ff">int</span>&gt; lst = <span style="color: #0000ff">new</span> List&lt;<span style="color: #0000ff">int</span>&gt;();<br /><span style="color: #008000">// Añadimos ints a la lista...</span><br /><span style="color: #008000">// Qué es más rápido?</span><br />var count = lst.Count;<br />var count2 = ((IEnumerable&lt;<span style="color: #0000ff">int</span>&gt;)lst).Count();</pre>
  
  <p>
    </div> 
    
    <p>
      A veces hacemos <em>suposiciones</em> sobre como funciona LINQ to objects. Uno puede pensar que el método Count() de LINQ está definido como:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">int</span> Count&lt;T&gt;(<span style="color: #0000ff">this</span> IEnumerable&lt;T&gt; @<span style="color: #0000ff">this</span>)<br />{<br />    <span style="color: #0000ff">int</span> count = 0;<br />    <span style="color: #0000ff">foreach</span> (var x <span style="color: #0000ff">in</span> @<span style="color: #0000ff">this</span>) count++;<br />    <span style="color: #0000ff">return</span> count;<br />}</pre>
      
      <p>
        </div> 
        
        <p>
          Hay gente que basándose en estas suposiciornes intenta evitar el uso de Count() cuando sabe que la colección real es una List<T> p.ej. Desgraciadamente esto les lleva a no poder hacer métodos genéricos con IEnumerable<T> (empiezan a trabajar con IList<T>). A veces comentan que usarían mucho más LINQ to Objects, pero que trabajan habitualmente con listas, y que no pueden permitirse el sobrecoste de recorrer toda la lista simplemente para contar los elementos, cuando la clase List<T> ya tiene una propiedad para ello…
        </p>
        
        <p>
          … están totalmente equivocados.
        </p>
        
        <p>
          LINQ to Objects está <em>optimizado</em>, no es un proveedor tan tonto como algunos piensan… Así realmente si el objeto sobre el que usamos Count() implementa ICollection o ICollection<T>, LINQ usará la propiedad Count directamente, sin recorrer los elementos.
        </p>
        
        <p>
          Para que veais que es cierto he realizado un pequeño test:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">class</span> Program<br />{<br />    <span style="color: #0000ff">static</span> <span style="color: #0000ff">void</span> Main(<span style="color: #0000ff">string</span>[] args)<br />    {<br />        List&lt;<span style="color: #0000ff">int</span>&gt; list = <span style="color: #0000ff">new</span> List&lt;<span style="color: #0000ff">int</span>&gt;();<br />        <span style="color: #0000ff">for</span> (<span style="color: #0000ff">int</span> i = 0; i &lt; 10000000; i++)<br />        {<br />            list.Add(i);<br />        }<br /><br />        Stopwatch sw = <span style="color: #0000ff">new</span> Stopwatch();<br />        sw.Start();<br />        CountList(list);<br />        sw.Stop();<br />        Console.WriteLine(<span style="color: #006080">"List.Count:"</span> + sw.ElapsedMilliseconds);<br />        sw.Reset();<br />        sw.Start();<br />        CountLinq(list);<br />        sw.Stop();<br />        Console.WriteLine(<span style="color: #006080">"LINQ.Count():"</span> + sw.ElapsedMilliseconds);<br />        sw.Reset();<br />        sw.Start();<br />        CountLoop(list);<br />        sw.Stop();<br />        Console.WriteLine(<span style="color: #006080">"foreach count"</span> + sw.ElapsedMilliseconds);<br />        sw.Reset();<br />        Console.ReadLine();<br />    }<br /><br />    <span style="color: #0000ff">static</span> <span style="color: #0000ff">void</span> CountList (IList&lt;<span style="color: #0000ff">int</span>&gt; list)<br />    {<br />        <span style="color: #0000ff">for</span> (<span style="color: #0000ff">int</span> i=0; i&lt; 100; i++)<br />        {<br />            var a = list.Count;<br />        }<br />    }<br /><br />    <span style="color: #0000ff">static</span> <span style="color: #0000ff">void</span> CountLinq(IEnumerable&lt;<span style="color: #0000ff">int</span>&gt; list)<br />    {<br />        <span style="color: #0000ff">for</span> (<span style="color: #0000ff">int</span> i = 0; i &lt; 100; i++)<br />        {<br />            var a = list.Count();<br />        }<br />    }<br /><br />    <span style="color: #0000ff">static</span> <span style="color: #0000ff">void</span> CountLoop(IEnumerable&lt;<span style="color: #0000ff">int</span>&gt; list)<br />    {<br />        <span style="color: #0000ff">for</span> (<span style="color: #0000ff">int</span> i = 0; i &lt; 100; i++)<br />        {<br />            var a = list.Count2();<br />        }<br />    }<br />}</pre>
          
          <p>
            </div> 
            
            <p>
              El test cuenta 100 veces una lista con 10 millones de elementos, y cuenta lo que se tarda usando la propiedad Count de la lista, el método Count() de LINQ y el método Count2, que es un método extensor que recorre la lista (es exactamente el mismo método que he puesto antes).
            </p>
            
            <p>
              Los resultados no dejan lugar a dudas:
            </p>
            
            <ol>
              <li>
                Usando la propiedad Count, se tarda menos de un ms en contar 100 veces la lista.
              </li>
              <li>
                Usando el método Count() de LINQ se tarda igualmente menos de un ms en contar la lista 100 veces.
              </li>
              <li>
                Usando el método extensor Count2 se tarda más de 9 segundos en contar la lista 100 veces…
              </li>
            </ol>
            
            <p>
              Si en lugar de 100 veces la contamos diez millones de veces, los resultados son:
            </p>
            
            <ol>
              <li>
                30 ms usando la propiedad Count
              </li>
              <li>
                247 ms usando el método Count() de LINQ
              </li>
              <li>
                Ni idea usando el método extensor Count2… pero vamos si para 100 veces ha tardado 9 segundos… para diez millones… no quiero ni pensarlo!
              </li>
            </ol>
            
            <p>
              Los tiempos han sido medidos con la aplicación en <em>Release</em>.
            </p>
            
            <p>
              La conclusión es clara: no tengáis miedo a LINQ, que MS no ha hecho algo tan cutre como un triste foreach!! 😉
            </p>
            
            <p>
              Saludos!
            </p>
            
            <p>
              PD: <a href="http://blogs.msdn.com/csharpfaq/archive/2009/01/26/does-the-linq-to-objects-provider-have-built-in-performance-optimization.aspx">En este post del blog del equipo de C# cuentan esta y otras optimizaciones más de LINQ to Objects</a>… lectura imprescindible! 🙂
            </p>