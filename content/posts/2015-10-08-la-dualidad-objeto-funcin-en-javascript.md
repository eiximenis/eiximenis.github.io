---
title: La dualidad objeto-función en JavaScript
description: La dualidad objeto-función en JavaScript
author: eiximenis

date: 2015-10-08T21:28:00+00:00
geeks_url: /?p=1705
geeks_visits:
  - 885
geeks_ms_views:
  - 1413
categories:
  - javascript

---
No sé si <a href="https://es.wikipedia.org/wiki/Brendan_Eich" target="_blank" rel="noopener noreferrer">Brendan Eich</a> es un amante de la física cuántica, pero al menos viendo algunas de las características, así lo parece. No solo tenemos el _principio de incertidumbre de this_ si no que también tenemos el hecho de que en JavaScript un objeto puede comportarse como una función y viceversa, es decir una dualidad objeto-función.

Tomemos jQuery p. ej. Por defecto nos define el archiconocido $, con el cual podemos hacer algo como _$(“#mydiv”)_, es decir, usarlo como una función, pero también podemos hacer _$.getJSON(“…”)_, es decir, usarlo como un objeto. A ver como podemos conseguir esa dualidad es a lo que vamos a dedicar este post 🙂

**La dualidad ya existente**

La verdad… es que JavaScript ya viene con esa dualidad de serie… **Todas** las funciones de JavaScript se comportan a su vez como objetos (de hecho, en JavaScript las funciones son un tipo específico de objeto). O si no, observa el siguiente código:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:2908f769-89a4-4c14-9d0e-25641ab8c23c" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> f = </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> () {</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">console.log(</span><span style="background:#ffffff;color:#a31515">"i am a function, or an object?"</span><span style="background:#ffffff;color:#000000">);</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">console.log(</span><span style="background:#ffffff;color:#0000ff">typeof</span><span style="background:#ffffff;color:#000000"> (f));</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">console.log(f.toString());</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

El primer _console.log_, imprime “function”, dejando bien claro que _f_ es una función. Pero luego observa que **se está llamando al método toString de la función f**. De la función en sí, no del valor devuelto por dicha función (lo que, ya puestos, generaría un error, pues _f_ devuelve _undefined_). Así pues, **las funciones tienen propiedades y métodos**. Puede parecer chocante, pero en JavaScript los funciones, son también, objetos.

En JavaScript cada objeto tiene asociado _su prototipo_ que es _otro objeto_ (y dado que el prototipo es un objeto tiene asociado su propio prototipo, lo que genera una cadena de prototipos, que termina en un objeto llamado Object.prototype y que su prototipo es _null_).

Existe una propiedad, llamada \_\_proto\_\_, que nos permite acceder al prototipo de un objeto en concreto. Cuando creamos un objeto, usando _object notation_ dicha propiedad se asigna a Object.prototype (como puedes comprobar en esa captura de pantalla de las herramientas de desarrollador de _Chrome_):

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_1C6351B8.png" width="644" height="79" />][1]

Las funciones, como objeto que son, tienen todas su propio prototipo que es… Function.prototype, que es por cierto el objeto que define las funciones _call_, _apply_ y _bind_, que podemos llamar sobre cualquier función.

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_3A44666A.png" width="644" height="70" />][2]

(Por si lo estás suponiendo, sí: el prototipo de _Function.prototype_ es _Object.prototype_).

Entender bien la cadena de prototipado es fundamental, pues en ella se basa todo el sistema de herencia de JavaScript.

Ahora que ya vemos que las funciones en realidad son objetos, **vamos a ver como podemos hacer como hace jQuery y colocar métodos propios en una función que definamos**.

No nos sirve agregar métodos a Function.prototype: si lo hacemos **esos métodos van a estar disponibles para todas las funciones** y nosotros queremos que dichos métodos estén disponibles solo para una función en concreta (al igual que hace jQuery, que agrega métodos específicos pero solo a la función $).

**Creando nuestra propia dualidad**

Por extraño que parezca crear nuestra propia dualidad es muy sencillo, basta con el siguiente código:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:b89c2146-f5c9-43b2-83a1-499f2f669d96" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> f = </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> (name) {</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#a31515">"Hello "</span><span style="background:#ffffff;color:#000000"> + name;</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">f.foo = </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> () {</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#a31515">"foo Utility method"</span><span style="background:#ffffff;color:#000000">;</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

> Una vez tenemos esto podemos invocar a la función f, pero también a la función foo de la función f:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:684fd1f0-1340-4589-9da9-350748173826" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier
, Monospace; font-size: 10pt">
    </p> 
    
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> x = f(</span><span style="background:#ffffff;color:#a31515">"edu"</span><span style="background:#ffffff;color:#000000">);</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> x2 = f.foo();</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Ya tenemos a nuestra variable f, comportándose dualmente como una función y un objeto a la vez.

Por supuesto podemos refinar un poco esa técnica:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:bd7251aa-98c9-471d-b5d9-b6a4c915b6cb" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> f = </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> (name) {</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#a31515">"Hello "</span><span style="background:#ffffff;color:#000000"> + name;</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> fn = {</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">foo: </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> () {</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000">console.log(</span><span style="background:#ffffff;color:#a31515">"Utility method"</span><span style="background:#ffffff;color:#000000">);</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">Object.assign(f, fn);</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Este código es equivalente al anterior, pero usa Object.assign (método de ES6) para copiar las propiedades contenidas en fn (el método foo) dentro del objeto (función) f.

Usando ES6 es muy fácil crearnos una función factoría que nos permita establecer un objeto con un conjunto de operaciones comunes a una función. Para ello podemos establecer este objeto como prototipo de la función (a su vez este objeto prototipo de la función debe “heredar” de Function.prototype para no perder las funciones que este provee).

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:49efefe4-23fe-4ef4-8376-b48d1a26fb46" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> factory = </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> (p, f) {</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> fp = Object.create(Function.prototype);</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">Object.assign(fp, p);</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">Object.setPrototypeOf(f, fp);</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> f;</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

De este modo si tenemos un objeto cualquiera y queremos que todas sus operaciones sean heredadas por una función:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:c21d6a03-2ef0-4735-8f93-a87552ce017b" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> f = </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> () { </span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">this</span><span style="background:#ffffff;color:#000000">; };</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">factory({</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">dump: </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> () {</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000">console.log(</span><span style="background:#ffffff;color:#a31515">'dump...'</span><span style="background:#ffffff;color:#000000">);</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">}, f);</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">f.dump();</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Ahora, la función f, además de los métodos definidos en Function.prototype (tales como _apply_ o _call_) tiene también todos los métodos definidos en el objeto pasado como primer parámetro de _factory_, es decir el método _dump_.

Fácil y sencillo, ¿no?

**Por cierto, ya puestos a divertirnos un poco. Cual es la salida del siguiente código si lo ejecutas en la consola JS del navegador?**

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:f7f0fe40-16d5-45cf-999d-eaf5a9eca97d" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> f = </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> () { </span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">this</span><span style="background:#ffffff;color:#000000">; };</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> obj =<br /> {</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">foo: f</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">};</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">factory({</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">unbind: </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> () {</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">this</span><span style="background:#ffffff;color:#000000">.bind(</span><span style="background:#ffffff;color:#0000ff">this</span><span style="background:#ffffff;color:#000000">);</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">}, f);</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">console.log(</span><span style="background:#ffffff;color:#a31515">"f()"</span><span style="background:#ffffff;color:#000000">, f());</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">console.log(</span><span style="background:#ffffff;color:#a31515">"f.bind({a:10})()"</span><span style="background:#ffffff;color:#000000">, f.bind({ a: 10 })());</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">console.log(</span><span style="background:#ffffff;color:#a31515">"f.unbind()()"</span><span style="background:#ffffff;color:#000000">, f.unbind()());</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">console.log(</span><span style="background:#ffffff;color:#a31515">"obj.foo()"</span><span style="background:#ffffff;color:#000000">, obj.foo());</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">console.log(</span><span style="background:#ffffff;color:#a31515">"obj.foo.unbind()()"</span><span style="background:#ffffff;color:#000000">, obj.foo.unbind()());</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Piénsalo un poco con calma… **si aciertas los valores de los 5 console.log eres de los que dominan el principio de incertidumbre de this** 😉

Saludos!

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_338ECC36.png
 [2]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_49E4717B.png