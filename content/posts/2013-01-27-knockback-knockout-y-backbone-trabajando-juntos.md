---
title: 'Knockback: Knockout y Backbone trabajando juntos'
description: 'Knockback: Knockout y Backbone trabajando juntos'
author: eiximenis

date: 2013-01-27T13:51:14+00:00
geeks_url: /?p=1628
geeks_visits:
  - 2804
geeks_ms_views:
  - 1382
categories:
  - Uncategorized

---
Este pasado sábado 26 de febrero, tuvimos el Four Sessions de <a href="https://twitter.com/techdencias" target="_blank" rel="noopener noreferrer">Techdencias</a>, en el que me lo pasé genial participando junto a <a href="https://twitter.com/marc_rubino" target="_blank" rel="noopener noreferrer">Marc Rubiño</a> en un duelo entre <a href="http://knockoutjs.com/" target="_blank" rel="noopener noreferrer">Knockout</a> y <a href="http://backbonejs.org/" target="_blank" rel="noopener noreferrer">Backbone</a> para ver que librería era “mejor” para construir aplicaciones web. Al final la gente votó y la verdad és que ganó Marc por KO… 😛

Como digo estuvo divertido, aunque comparar Backbone con Knockout es como comparar peras con guisantes ya que, realmente, poco tenen que ver y además son muy complementarias. Backbone se centra más en dotarnos de herramientas para poder arquitecturar nuestra aplicación javascript. Mientras que Knockout tiene un sistema fenomenal de data-binding entre el DOM y el viewmodel pero nada más que eso (que no es poco): sigue siendo tarea nuestra organizar bien nuestra aplicación javascript y de ello Knockout se mantiene al margen.

La consecuencia de estos dos enfoques es que el código javascript que tiene que colocarse en Backone es siempre muy superior al de Knockout. No sólo eso, si no que Backbone por su propia concepción requiere unas habilidades de javascript superiores a las que requiere Knockout para empezar a hacer cosas. Una vez superados estos dos escollos, Backbone ofrece más facilidades que Knockout no sólo para organizar nuestro código de cliente, si no también para validaciones de modelos y creación de <a href="http://en.wikipedia.org/wiki/Single-page_application" target="_blank" rel="noopener noreferrer">aplicaciones SPA</a>.

**Dos modelos distintos pero similares**

Knockout apuesta por un modelo MVVM, muy semejante al que se encuentra en las tecnologías basadas en XAML: los enlaces se aplican declarativamente en el DOM y en javascript basta con crear un viewmodel que contenga los datos que son enlazados automáticamente (y de forma bidireccional). Knockout ofrece soporte para varios tipos de bindings y la capacidad de definirnos nuestros propios para tareas más avanzadas.

El siguiente código muestra un textbox y una label que se va modificando a medida que vas modificando el contenido del textbox:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: gray"><!</span><span style="color: #569cd6">DOCTYPE</span> <span style="color: #9cdcfe">html</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray"><</span><span style="color: #569cd6">html</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">head</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">script</span> <span style="color: #9cdcfe">src</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"http://knockoutjs.com/downloads/knockout-2.2.1.js"</span><span style="color: gray">></</span><span style="color: #569cd6">script</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"></</span><span style="color: #569cd6">head</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">body</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">input</span> <span style="color: #9cdcfe">type</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"text"</span> <span style="color: #9cdcfe">data-bind</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"value: name, valueUpdate: &#8216;afterkeydown&#8217;"</span> <span style="color: gray">/></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">br</span> <span style="color: gray">/></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">label</span> <span style="color: #9cdcfe">data-bind</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"text: greeting"</span> <span style="color: gray">/></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"></</span><span style="color: #569cd6">body</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">script</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #b4b4b4">(</span><span style="color: #569cd6">function</span> <span style="color: #b4b4b4">()</span> <span style="color: #b4b4b4">{</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">MyViewModel</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">function</span> <span style="color: #b4b4b4">(</span><span style="color: white">aName</span><span style="color: #b4b4b4">)</span> <span style="color: #b4b4b4">{</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">self</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">this</span><span style="color: #b4b4b4">;</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">self</span><span style="color: #b4b4b4">.</span><span style="color: white">name</span> <span style="color: #b4b4b4">=</span> <span style="color: white">ko</span><span style="color: #b4b4b4">.</span><span style="color: white">observable</span><span style="color: #b4b4b4">(</span><span style="color: white">aName</span><span style="color: #b4b4b4">);</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">self</span><span style="color: #b4b4b4">.</span><span style="color: white">greeting</span> <span style="color: #b4b4b4">=</span> <span style="color: white">ko</span><span style="color: #b4b4b4">.</span><span style="color: white">computed</span><span style="color: #b4b4b4">(</span><span style="color: #569cd6">function</span> <span style="color: #b4b4b4">()</span> <span style="color: #b4b4b4">{</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: #d69d85">"Hola "</span> <span style="color: #b4b4b4">+</span> <span style="color: white">self</span><span style="color: #b4b4b4">.</span><span style="color: white">name</span><span style="color: #b4b4b4">();</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #b4b4b4">});</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #b4b4b4">}</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">vm</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">new</span> <span style="color: white">code</span><span style="color: #b4b4b4">(</span><span s
tyle="color: #d69d85">"eiximenis"</span><span style="color: #b4b4b4">);</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">ko</span><span style="color: #b4b4b4">.</span><span style="color: white">applyBindings</span><span style="color: #b4b4b4">(</span><span style="color: white">vm</span><span style="color: #b4b4b4">);</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #b4b4b4">})();</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"></</span><span style="color: #569cd6">script</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray"></</span><span style="color: #569cd6">html</span><span style="color: gray">></span>
  </p></p>
</div>

Podemos ver el uso del atributo data-bind para declarar los enlaces declarativamente y el código javascript que se limit a crear el viewmodel con las dos propiedades usadas (name y greeting).

Backbone por su parte apuesta por un modelo más parecido a MVC, con sus modelos y sus vistas claramente diferenciados. Para crear la misma aplicación con Backbone se requiere bastante más código:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: gray"><!</span><span style="color: #569cd6">DOCTYPE</span> <span style="color: #9cdcfe">html</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray"><</span><span style="color: #569cd6">html</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">head</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">script</span> <span style="color: #9cdcfe">src</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"http://code.jquery.com/jquery.min.js"</span><span style="color: gray">></</span><span style="color: #569cd6">script</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">script</span> <span style="color: #9cdcfe">src</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"http://underscorejs.org/underscore-min.js"</span><span style="color: gray">></</span><span style="color: #569cd6">script</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">script</span> <span style="color: #9cdcfe">src</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"http://backbonejs.org/backbone-min.js"</span><span style="color: gray">></</span><span style="color: #569cd6">script</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">script</span> <span style="color: #9cdcfe">type</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"text/template"</span> <span style="color: #9cdcfe">id</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"viewTemplate"</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <input type="text" value="<%- name %>" />
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <br />
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <label><%- greeting %></label>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: gray"></</span><span style="color: #569cd6">script</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"></</span><span style="color: #569cd6">head</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">body</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"></</span><span style="color: #569cd6">body</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">script</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #b4b4b4">(</span><span style="color: #569cd6">function</span> <span style="color: #b4b4b4">()</span> <span style="color: #b4b4b4">{</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">MyModel</span> <span style="color: #b4b4b4">=</span> <span style="color: white">Backbone</span><span style="color: #b4b4b4">.</span><span style="color: white">Model</span><span style="color: #b4b4b4">.</span><span style="color: white">extend</span><span style="color: #b4b4b4">({</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">defaults</span><span style="color: #b4b4b4">:</span> <span style="color: #b4b4b4">{</span> <span style="color: white">name</span><span style="color: #b4b4b4">:</span> <span style="color: #d69d85">&#8216;eiximenis&#8217;</span> <span style="color: #b4b4b4">},</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #b4b4b4">});</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">MyView</span> <span style="color: #b4b4b4">=</span> <span style="color: white">Backbone</span><span style="color: #b4b4b4">.</span><span style="color: white">View</span><span style="color: #b4b4b4">.</span><span style="color: white">extend</span><span style="color: #b4b4b4">({</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">initialize</span><span style="color: #b4b4b4">:</span> <span style="color: #569cd6">function</span> <span style="color: #b4b4b4">()</span> <span style="color: #b4b4b4">{</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">self</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">this</span><span style="color: #b4b4b4">;</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">this</span><span style="color: #b4b4b4">.</span><span style="color: white">model</span><span style="color: #b4b4b4">.</span><span style="color: white">on</span><span style="color: #b4b4b4">(</span><span style="color: #d69d85">&#8216;change:name&#8217;</span><span style="color: #b4b4b4">,</span> <span style="color: #569cd6">function</span> <span style="color: #b4b4b4">()</span> <span style="color: #b4b4b4">{</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">self</span><span style="color: #b4b4b4">.</span><span style="color: white">updateLabel</span><span style="color: #b4b4b4">();</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160<br /> ;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #b4b4b4">});</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #b4b4b4">},</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">events</span><span style="color: #b4b4b4">:</span> <span style="color: #b4b4b4">{</span> <span style="color: #d69d85">&#8216;keyup input&#8217;</span><span style="color: #b4b4b4">:</span> <span style="color: #d69d85">&#8216;changeName&#8217;</span> <span style="color: #b4b4b4">},</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">changeName</span><span style="color: #b4b4b4">:</span> <span style="color: #569cd6">function</span> <span style="color: #b4b4b4">(</span><span style="color: white">evt</span><span style="color: #b4b4b4">)</span> <span style="color: #b4b4b4">{</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">this</span><span style="color: #b4b4b4">.</span><span style="color: white">model</span><span style="color: #b4b4b4">.</span><span style="color: white">set</span><span style="color: #b4b4b4">(</span><span style="color: #d69d85">&#8216;name&#8217;</span><span style="color: #b4b4b4">,</span> <span style="color: white">$</span><span style="color: #b4b4b4">(</span><span style="color: white">evt</span><span style="color: #b4b4b4">.</span><span style="color: white">target</span><span style="color: #b4b4b4">).</span><span style="color: white">val</span><span style="color: #b4b4b4">());</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #b4b4b4">},</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">updateLabel</span><span style="color: #b4b4b4">:</span> <span style="color: #569cd6">function</span> <span style="color: #b4b4b4">()</span> <span style="color: #b4b4b4">{</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">this</span><span style="color: #b4b4b4">.</span><span style="color: white">$</span><span style="color: #b4b4b4">(</span><span style="color: #d69d85">"label"</span><span style="color: #b4b4b4">).</span><span style="color: white">text</span><span style="color: #b4b4b4">(</span><span style="color: #569cd6">this</span><span style="color: #b4b4b4">.</span><span style="color: white">greeting</span><span style="color: #b4b4b4">());</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #b4b4b4">},</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">greeting</span><span style="color: #b4b4b4">:</span> <span style="color: #569cd6">function</span> <span style="color: #b4b4b4">()</span> <span style="color: #b4b4b4">{</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: #d69d85">"Hola "</span> <span style="color: #b4b4b4">+</span> <span style="color: #569cd6">this</span><span style="color: #b4b4b4">.</span><span style="color: white">model</span><span style="color: #b4b4b4">.</span><span style="color: white">get</span><span style="color: #b4b4b4">(</span><span style="color: #d69d85">&#8216;name&#8217;</span><span style="color: #b4b4b4">);</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #b4b4b4">},</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">render</span><span style="color: #b4b4b4">:</span> <span style="color: #569cd6">function</span> <span style="color: #b4b4b4">()</span> <span style="color: #b4b4b4">{</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">this</span><span style="color: #b4b4b4">.</span><span style="color: white">$el</span><span style="color: #b4b4b4">.</span><span style="color: white">empty</span><span style="color: #b4b4b4">();</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">data</span> <span style="color: #b4b4b4">=</span> <span style="color: #b4b4b4">{</span> <span style="color: white">name</span><span style="color: #b4b4b4">:</span> <span style="color: #569cd6">this</span><span style="color: #b4b4b4">.</span><span style="color: white">model</span><span style="color: #b4b4b4">.</span><span style="color: white">get</span><span style="color: #b4b4b4">(</span><span style="color: #d69d85">&#8216;name&#8217;</span><span style="color: #b4b4b4">),</span> <span style="color: white">greeting</span><span style="color: #b4b4b4">:</span> <span style="color: #569cd6">this</span><span style="color: #b4b4b4">.</span><span style="color: white">greeting</span><span style="color: #b4b4b4">()</span> <span style="color: #b4b4b4">};</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">this</span><span style="color: #b4b4b4">.</span><span style="color: white">$el</span><span style="color: #b4b4b4">.</span><span style="color: white">html</span><span style="color: #b4b4b4">(</span><span style="color: white">_</span><span style="color: #b4b4b4">.</span><span style="color: white">template</span><span style="color: #b4b4b4">(</span><span style="color: white">$</span><span style="color: #b4b4b4">(</span><span style="color: #d69d85">"#viewTemplate"</span><span style="color: #b4b4b4">).</span><span style="color: white">html</span><span style="color: #b4b4b4">(),</span> <span style="color: white">data</span><span style="color: #b4b4b4">));</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: #569cd6">this</span><span style="color: #b4b4b4">;</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #b4b4b4">}</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #b4b4b4">});</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">$</span><span style="color: #b4b4b4">(</span><span style="color: #569cd6">function</span> <span style="color: #b4b4b4">()</span> <span style="color: #b4b4b4">{</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">model</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">new</span> <span style="color: white">MyModel</span><span style="color: #b4b4b4">();</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">view</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6"><br /> new</span> <span style="color: white">MyView</span><span style="color: #b4b4b4">({</span> <span style="color: white">model</span><span style="color: #b4b4b4">:</span> <span style="color: white">model</span><span style="color: #b4b4b4">,</span> <span style="color: white">el</span><span style="color: #b4b4b4">:</span> <span style="color: #d69d85">&#8216;body&#8217;</span> <span style="color: #b4b4b4">});</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">view</span><span style="color: #b4b4b4">.</span><span style="color: white">render</span><span style="color: #b4b4b4">();</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #b4b4b4">});</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #b4b4b4">})();</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"></</span><span style="color: #569cd6">script</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray"></</span><span style="color: #569cd6">html</span><span style="color: gray">></span>
  </p></p>
</div>

Aunque pueda parecer que están a las antípodas, en el fondo los modelos de Knockout y Backbone son similares: ambos tienen una clase javascript que se encarga de mantener los datos y ambos tienen una vista que se encarga de representarlos. La diferencia fundamental es que en Knockout la vista es el propio DOM que contiene los atributos data-bind y en Backbone la vista debe crearse y debe tener todo el código necesario para gestionar la comunicación entre el DOM (DOM que crea la propia vista) y el modelo de datos.

**¿Y pueden trabajar juntos?**

La verdad es que para pequeñas aplicaciones no tiene sentido usar Backbone, y probablemente con Knockout tirarás perfectamente. Pero, si nos ponemos en la situación de que queremos usar Backbone… no podríamos aprovecharnos del sistema de enlaces de Knockout y ahorrarnos toda esa parte?

Pues sí, y la respuesta se llama <a href="http://kmalakoff.github.com/knockback/" target="_blank" rel="noopener noreferrer">Knockback</a>.

Usando Knockback nuestro código quedaría más o menos como:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: gray"><!</span><span style="color: #569cd6">DOCTYPE</span> <span style="color: #9cdcfe">html</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray"><</span><span style="color: #569cd6">html</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">head</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">script</span> <span style="color: #9cdcfe">src</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"http://code.jquery.com/jquery.min.js"</span><span style="color: gray">></</span><span style="color: #569cd6">script</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">script</span> <span style="color: #9cdcfe">src</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"http://underscorejs.org/underscore-min.js"</span><span style="color: gray">></</span><span style="color: #569cd6">script</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">script</span> <span style="color: #9cdcfe">src</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"http://backbonejs.org/backbone-min.js"</span><span style="color: gray">></</span><span style="color: #569cd6">script</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">script</span> <span style="color: #9cdcfe">src</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"http://knockoutjs.com/downloads/knockout-2.2.1.js"</span><span style="color: gray">></</span><span style="color: #569cd6">script</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">script</span> <span style="color: #9cdcfe">src</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"https://raw.github.com/kmalakoff/knockback/0.16.8/knockback.min.js"</span><span style="color: gray">> </</span><span style="color: #569cd6">script</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"></</span><span style="color: #569cd6">head</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">body</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">input</span> <span style="color: #9cdcfe">type</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"text"</span> <span style="color: #9cdcfe">data-bind</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"value: name, valueUpdate:&#8217;afterkeydown&#8217;"</span> <span style="color: gray">/></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">br</span> <span style="color: gray">/></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">label</span> <span style="color: #9cdcfe">data-bind</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"text: greeting"</span><span style="color: gray">></</span><span style="color: #569cd6">label</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">input</span> <span style="color: #9cdcfe">type</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"button"</span> <span style="color: #9cdcfe">id</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"ju"</span> <span style="color: #9cdcfe">value</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"!!"</span> <span style="color: gray">/></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"></</span><span style="color: #569cd6">body</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">script</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #b4b4b4">(</span><span style="color: #569cd6">function</span> <span style="color: #b4b4b4">()</span> <span style="color: #b4b4b4">{</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">MyModel</span> <span style="color: #b4b4b4">=</span> <span style="color: white">Backbone</span><span style="color: #b4b4b4">.</span><span style="color: white">Model</span><span style="color: #b4b4b4">.</span><span style="color: white">extend</span><span style="color: #b4b4b4">({</span>
  </p>
  
  <p style="margin: 0px">
    &#16<br /> 0;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">defaults</span><span style="color: #b4b4b4">:</span> <span style="color: #b4b4b4">{</span> <span style="color: white">name</span><span style="color: #b4b4b4">:</span> <span style="color: #d69d85">&#8216;eiximenis&#8217;</span> <span style="color: #b4b4b4">},</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #b4b4b4">});</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">$</span><span style="color: #b4b4b4">(</span><span style="color: #569cd6">function</span> <span style="color: #b4b4b4">()</span> <span style="color: #b4b4b4">{</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">model</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">new</span> <span style="color: white">MyModel</span><span style="color: #b4b4b4">();</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">ViewModel</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">function</span> <span style="color: #b4b4b4">(</span><span style="color: white">model</span><span style="color: #b4b4b4">)</span> <span style="color: #b4b4b4">{</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">self</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">this</span><span style="color: #b4b4b4">;</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">this</span><span style="color: #b4b4b4">.</span><span style="color: white">name</span> <span style="color: #b4b4b4">=</span> <span style="color: white">kb</span><span style="color: #b4b4b4">.</span><span style="color: white">observable</span><span style="color: #b4b4b4">(</span><span style="color: white">model</span><span style="color: #b4b4b4">,</span> <span style="color: #d69d85">&#8216;name&#8217;</span><span style="color: #b4b4b4">);</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">this</span><span style="color: #b4b4b4">.</span><span style="color: white">greeting</span> <span style="color: #b4b4b4">=</span> <span style="color: white">ko</span><span style="color: #b4b4b4">.</span><span style="color: white">computed</span><span style="color: #b4b4b4">(</span><span style="color: #569cd6">function</span> <span style="color: #b4b4b4">()</span> <span style="color: #b4b4b4">{</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: #d69d85">"Hello "</span> <span style="color: #b4b4b4">+</span> <span style="color: white">self</span><span style="color: #b4b4b4">.</span><span style="color: white">name</span><span style="color: #b4b4b4">();</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #b4b4b4">})</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #b4b4b4">}</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">ko</span><span style="color: #b4b4b4">.</span><span style="color: white">applyBindings</span><span style="color: #b4b4b4">(</span><span style="color: white">vm</span><span style="color: #b4b4b4">);</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #b4b4b4">});</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #b4b4b4">})();</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"></</span><span style="color: #569cd6">script</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray"></</span><span style="color: #569cd6">html</span><span style="color: gray">></span>
  </p></p>
</div>

En el fondo volvemos a un esquema muy parecido al código con knockout, la única diferencia principal es que ahora tenemos un modelo de Backbone y un viewmodel de Knockout. Lo que ya no tenemos (puesto que nos lo proporciona Knockout) es la vista de Backbone.

Evidentemente, si en este ejemplo no tiene mucho sentido aplicar Backbone, tampoco lo tendrá aplicar Knockback, pero vamos a hacer un ejercicio de imaginación y situarnos en un proyecto mucho más complejo.

En este caso, al usar Knockback obtenemos la sencillez de Knockout para los enlaces y además la potencia de Backbone para los modelos. En nuestro ejemplo ViewModel y modelo eran casi idénticos, pero en un ejemplo más grande el ViewModel puede diferir bastante del modelo: Este contiene los datos tal y como van a ser enviados y recibidos desde el servidor y lo gestiona Backbone. Así nos podemos aprovechar de la persistencia automática de Backbone, el enrutamiento en cliente, la ordenación y filtrado de colecciones automática y todo lo que Backbone nos ofrece en los modelos. Por otra parte el Viewmodel contendría los datos preparados para ser enlazados directamente con el DOM. 

Imagina una aplicación que muestre los datos generales de una póliza, y luego tenga varias pestañas con información addicional. Pues bien, toda la información puede estar en un modelo de Backbone, que contendrá todos los datos de la póliza, mientras que tendríamos varios viewmodels de Knockout, uno por cada pestaña, que son los que usaríamos para enlazar con el DOM.

Lo que Knockback nos ofrece es que los viewmodels y los modelos estén sincronizados entre ellos (si modifico algo en el viewmodel el modelo se entera de dicha modifcación).

A modo de resumen podríamos decir que un **Modelo de Backbone:**

  * Es independiente del DOM 
  * Ligado fuertemente a los datos que vienen del servidor 
  * Representa datos complejos, muchos de los cuales pueden no visualizarse 
  * Se modifica a través de la lógica de negocio 

Mientras que un **ViewModel de Knockout**:

  * Está atado a la vista, en el sentido que contiene aquellas propiedades que la vista quiere mostrar 
  * No está atado a los datos del servidor, de hecho probablemente tenga tan solo un subconjunto de dichos datos 
  * Aplicaciones complejas tenderán a tener varios viewmodels de knockout por cada modelo de Backbone 

Por supuesto lo más importante es **usar cada librería para lo que ha sido pensada** y tener siempre bien claro que ventajas e inconvenientes nos puede tener usar cada una de ellas 🙂

Un saludo!

pd: Ah sí… y eso de las librerías de javascript está en constante evolución! **No todo es Backbone o Knockout**! Existen más por ahí afuera y en breve espero poder hablaros de alguna de ellas! 😉