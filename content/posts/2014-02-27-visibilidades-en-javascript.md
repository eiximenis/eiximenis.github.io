---
title: Visibilidades en JavaScript
author: eiximenis

date: 2014-02-27T10:19:00+00:00
geeks_url: /?p=1662
geeks_visits:
  - 1127
geeks_ms_views:
  - 1117
categories:
  - Uncategorized

---
Una de las cosas que se argumentan en contra de JavaScript cuando se habla de orientación a objetos es que no soporta la visibilidad de métodos o propiedades. Es decir, todo es público por defecto.

Mucha gente hoy en día cuando programa en JavaScript adopta alguna convención tipo &ldquo;lo que empiece por guión bajo es privado y no debe ser invocado&rdquo;. Como chapuza para ir tirando, pues bueno, pero en JavaScript hay maneras de simular una visibilidad privada y de que realmente el creador de un objeto no pueda invocar algunos métodos. En este post veremos un método rápido y sencillo. **Por supuesto no es el único ni tiene porque ser el mejor...**

Empecemos por la declaración de una función constructora que me permite crear objetos de tipo Foo:

<div style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px" class="wlWriterEditableSmartContent" id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:5e5ec790-dcf1-4f32-b287-388a124a78a9">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px;" start="1">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> Foo </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">count </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b5cea8"></span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">inc </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">_addToCount</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#b5cea8">1</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">_addToCount </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">a</span><span style="background:#1e1e1e;color:#b4b4b4">)</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">count </span><span style="background:#1e1e1e;color:#b4b4b4">+=</span><span style="background:#1e1e1e;color:#dcdcdc"> a</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> foo </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> Foo</span><span style="background:#1e1e1e;color:#b4b4b4">();</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">foo</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">count</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">foo</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">inc</span><span style="background:#1e1e1e;color:#b4b4b4">();</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#57a64a">// Esto debera ser privado</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">foo</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">_addToCount</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#b5cea8">100</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">foo</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">count</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#57a64a">// count no debera poder accederse</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">foo</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">count </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b5cea8">10</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">foo</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">count</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Estoy usando la convención de que los métodos privados empiezan por un guión bajo. Pero es esto: una convención. Para el lenguaje no hay diferencia. De hecho si ejecuto este código el resultado es el siguiente:

[<img height="85" width="460" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_62F5F4F9.png" alt="image" border="0" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" title="image" />][1]

El desarrollador que crea un objeto Foo puede acceder tanto a inc, como a addToCount como a count. Como podemos solucionar eso?

La solución pasa por no devolver a quien crea el objeto Foo entero si no un &ldquo;subobjeto&rdquo; que tan solo contenga las funciones publicas:

<div style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px" class="wlWriterEditableSmartContent" id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:74852a5a-422b-497b-ae42-d6944f7be92f">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px;" start="1">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> Foo </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">count </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b5cea8"></span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">inc </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">_addToCount</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#b5cea8">1</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">_addToCount </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">a</span><span style="background:#1e1e1e;color:#b4b4b4">)</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">count </span><span style="background:#1e1e1e;color:#b4b4b4">+=</span><span style="background:#1e1e1e;color:#dcdcdc"> a</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">inc </span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">inc</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> foo </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> Foo</span><span style="background:#1e1e1e;color:#b4b4b4">();</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">foo</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Si ejecuto este código parece que vamos por el buen camino:

[<img height="58" width="455" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_2972E502.png" alt="image" border="0" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" title="image" />][2]

Ahora el objeto foo contiene tan solo el método inc. Pero, que ocurre si ¿lo ejecutamos? Pues eso:

[<img height="105" width="504" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_64C64AC0.png" alt="image" border="0" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" title="image" />][3]

JavaScript se queja que el método _addToCount no está definido! Que es lo que ha ocurrido? Lo ocurrido tiene que ver con el contexto de JavaScript o <a target="_blank" href="/blogs/etomas/archive/2013/10/29/javascript-191-qu-233-es-exactamente-this.aspx" rel="noopener noreferrer">el valor de this</a>. El método inc que invocamos es el método inc del objeto anónimo que devolvemos al final de la función constructora de Foo. Dentro de este método el valor de this es el valor del objeto anónimo que, por supuesto, no tiene definido _addToCount. Parece que estamos en un callejón sin salida, verdad?

Aquí es cuando entra en escena la función bind: bind es un función que se llama sobre una función. El resultado de aplicar bind a una función es otra función pero atada permanentemente al contexto que se pasa como parámetro a bind. Dicho de otra manera cuando devolvemos el objeto anónimo, tenemos que modificar el contexto del método inc para que sea el objeto Foo entero. Así modificamos el return para que quede como:

<div style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px" class="wlWriterEditableSmartContent" id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:6790cc1a-d4c9-4eba-906d-f8535de2f665">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol style="background: #1e1e1e; margin: 0 0 0 2em; padding: 0 0 0 5px;" start="1">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">inc</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">inc</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">bind</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Cuando se ejecuta este return el valor the this es el objeto Foo entero así que lo que estamos devolviendo es un objeto anónimo, con una función inc (que realmente es this.inc es decir la función inc del objeto Foo entero), pero que está bindeada a this (el objeto Foo entero), es decir que cuando se ejecute este método inc del objeto anónimo el valor de this no será el objeto anónimo si no el propio objeto Foo.

Con esto hemos terminado! Ahora cuando llamamos a new Foo(), lo que obtenemos es un objeto solo con el método inc. Cuando invocamos inc todo funciona ahora correctamente. Y ya no podemos invocar el método privado _addToCount ni acceder a la propiedad count.

Esto es tan solo un mecanismo, hay varias maneras distintas de hacer lo mismo pero todas se basan en este mismo principio.

Saludos!

PD: Os dejo el código de un método, que he llamado _publicInterface. Dicho método lo que hace es, a partir de un objeto, crear otro objeto que contenga tan solo aquellas funciones que NO empiezan por guión bajo:

<div style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px" class="wlWriterEditableSmartContent" id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:29169227-04dd-4d66-a291-d0395ca32232">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px;" start="1">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">_publicInterface </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> keys </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> Object</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">keys</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> protocol </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> {}</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">for</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> idx </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b5cea8"></span><span style="background:#1e1e1e;color:#b4b4b4">;</span><span style="background:#1e1e1e;color:#dcdcdc"> idx </span><span style="background:#1e1e1e;color:#b4b4b4"><</span><span style="background:#1e1e1e;color:#dcdcdc"> keys</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">length</span><span style="background:#1e1e1e;color:#b4b4b4">;</span><span style="background:#1e1e1e;color:#dcdcdc"> idx</span><span style="background:#1e1e1e;color:#b4b4b4">++)</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> key </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> keys</span><span style="background:#1e1e1e;color:#b4b4b4">[</span><span style="background:#1e1e1e;color:#dcdcdc">idx</span><span style="background:#1e1e1e;color:#b4b4b4">];</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">key</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">charAt</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#b5cea8"></span><span style="background:#1e1e1e;color:#b4b4b4">)</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">!==</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">&#8216;_&#8217;</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">&&</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">typeof</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">[</span><span style="background:#1e1e1e;color:#dcdcdc">key</span><span style="background:#1e1e1e;color:#b4b4b4">])</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">===</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">&#8220;function&#8221;</span><span style="background:#1e1e1e;color:#b4b4b4">)</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">protocol</span><span style="background:#1e1e1e;color:#b4b4b4">[</span><span style="background:#1e1e1e;color:#dcdcdc">key</span><span style="background:#1e1e1e;color:#b4b4b4">]</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">[</span><span style="background:#1e1e1e;color:#dcdcdc">key</span><span style="background:#1e1e1e;color:#b4b4b4">].</span><span style="background:#1e1e1e;color:#dcdcdc">bind</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> protocol</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Así podéis definir en vuestros objetos funciones públicas y privadas (que empiecen por guión bajo) y en el return final hacer: return this._publicInterface();

 [1]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_0BFE0345.png
 [2]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_50AD2E37.png
 [3]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_3025EE85.png