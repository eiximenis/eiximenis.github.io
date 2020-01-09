---
title: 'How to: Obtener controles de un formulario con generics II (Linq al rescate).'
description: 'How to: Obtener controles de un formulario con generics II (Linq al rescate).'
author: eiximenis

date: 2010-11-08T11:28:41+00:00
geeks_url: /?p=1539
geeks_visits:
  - 2268
geeks_ms_views:
  - 1401
categories:
  - Uncategorized

---
Ayer Lluis escribía este gran post: _[How to: Obtener controles de un formulario con generics][1]_. Como bien dice es una pregunta… recurrente en todos los sitios 🙂

Lo bueno de eso del desarrollo es que para todo hay varias soluciones, así que aquí os propongo otra, pero usando Linq. Personalmente me encanta Linq, supongo que es porqué siempre me han fascinado los lenguajes funcionales…

Antes que nada tenemos que solucionar un temilla: Linq funciona sobre IEnumerable<T> pero la propiedad Controls de un Control devuelve un objeto de tipo ControlCollection (otra de esas n-mil clases que no tienen sentido alguno y que existen sólo porque no teníamos generics en la versión 1 del framework). Así que el primer paso es obtener un IEnumerable<Control> a partir de una ControlCollection. Con un método extensor eso es trivial:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> IEnumerable&lt;Control&gt; AsEnumerable (<span style="color: #0000ff">this</span> Control.ControlCollection @<span style="color: #0000ff">this</span>)<br />{<br />    <span style="color: #0000ff">foreach</span> (var control <span style="color: #0000ff">in</span> @<span style="color: #0000ff">this</span>) <span style="color: #0000ff">yield</span> <span style="color: #0000ff">return</span> (Control)control;<br />}</pre>
  
  <p>
    </div> 
    
    <p>
      Ale… listos, con eso <em>transformamos</em> la CollectionControl en un IEnumerable<Control> y tenemos acceso a todo el potencial de Linq… Y como queda el método para obtener todos los controles de un tipo? Pues así:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> IEnumerable&lt;T&gt; GetAllControls&lt;T&gt;(<span style="color: #0000ff">this</span> Control @<span style="color: #0000ff">this</span>) <span style="color: #0000ff">where</span> T : Control<br />{<br />    <span style="color: #0000ff">return</span> @<span style="color: #0000ff">this</span>.Controls.AsEnumerable().Where(x =&gt; x.GetType() == <span style="color: #0000ff">typeof</span>(T)).<br />        Select(y=&gt;(T)y).<br />        Union(@<span style="color: #0000ff">this</span>.Controls.AsEnumerable().SelectMany(x =&gt; GetAllControls&lt;T&gt;(x)).<br />        Select(y=&gt;(T)y));<br />}</pre>
      
      <p>
        </div> 
        
        <p>
          ¡No me diréis que no es precioso: no hay bucles, no hay ifs… Os he dicho que me encanta Linq? 😉
        </p>
        
        <p>
          Hay una <em>pequeña</em> diferencia entre la versión de Lluís y esta mía: la versión de Lluís usa una List<Control> en la que añade todas las referencias (copia las referencias a la lista. Ojo: las referencias, no los controles). Esa versión que usa Linq, no copia las referencias en ningún sitio, sinó que <em>simplemente</em> itera sobre la coleccion original: no crea listas internas, ni nada… Ese es el poder de Linq!
        </p>
        
        <p>
          Es cierto que al usar IEnumerable<T> como valor de retorno, perdemos el método ForEach() (puesto que IEnumerable<T> no lo tiene y Linq no lo proporciona), pero hacer un método ForEach es trivial:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">void</span> ForEach&lt;T&gt;(<span style="color: #0000ff">this</span> IEnumerable&lt;T&gt; @<span style="color: #0000ff">this</span>, Action&lt;T&gt; action)<br />{<br />    <span style="color: #0000ff">foreach</span> (T t <span style="color: #0000ff">in</span> @<span style="color: #0000ff">this</span>)<br />    {<br />        action(t);<br />    }<br />}</pre>
          
          <p>
            </div> 
            
            <p>
              De todos modos, si os preguntáis <strong>porque Linq no ofrece un método ForEach</strong> quizá deberíais leeros este post de Octavio: <a href="http://geeks.ms/blogs/ohernandez/archive/2008/09/13/observaciones-con-respecto-al-m-233-todo-extensor-foreach-lt-t-gt.aspx">Observaciones con respecto a un método extensor ForEach<T></a>.
            </p>
            
            <p>
              Un&#160; saludo a todos y gracias a Lluís por darme “la excusa” de hacer otro post! 😉
            </p>

 [1]: http://geeks.ms/blogs/lfranco/archive/2010/11/05/how-to-obtener-controles-de-un-formulario-con-generics.aspx