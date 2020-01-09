---
title: Variables privadas en clases ES2015
description: Variables privadas en clases ES2015
author: eiximenis

date: 2015-11-02T18:11:02+00:00
geeks_url: /?p=1709
geeks_visits:
  - 552
geeks_ms_views:
  - 1393
categories:
  - javascript

---
Una de las novedades de ES6 es la posibilidad de usar _clases_. Si bien esas clases son realmente _sugar syntax_ sobre el patrón de función constructora, tienen alguna característica que simplifica el uso de dicho patrón y ofrecen una sintaxis que según como se mire puede ser más cómoda. Pero el estándard de ES2015 no tiene ningún mecanismo para indicar que una variable de una clase sea pública o privada. La visibilidad de variables en JavaScript es un clásico, <a href="http://geeks.ms/blogs/etomas/archive/2014/02/27/visibilidades-en-javascript.aspx" target="_blank" rel="noopener noreferrer">sobre el que ya he escrito algún post</a>. Pero las técnicas que usábamos hasta ahora para simular la visibilidad privada no aplican cuando usamos la sintaxis de clases… y debemos usar mecanismos alternativos. En este post veremos dos de ellos.

Supongamos que tenemos una clase ES2015 como la siguiente:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:1be996ad-5bdf-4e3a-9858-a92f90f8e0f9" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#0000ff">class</span><span style="background:#ffffff;color:#000000"> Foo {</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">constructor(name) {</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">this</span><span style="background:#ffffff;color:#000000">._name = name;</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">get name() {</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> _name;</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

El objetivo es que _name sea privado y solo pueda accederse a su valor a través de la propiedad _name_.

**Símbolos al rescate**

La primera solución implica el uso de un nuevo concepto de ES2015: los símbolos. Un símbolo permite definir **un nombre único** desconocido y que tan solo puede ser obtenido a partir de la referencia al símbolo original que ha creado dicho nombre.&#160; El siguiente código explica el uso de símbolos:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:8351a2df-1520-423c-b460-712b1be61fbc" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> sfoo = Symbol(</span><span style="background:#ffffff;color:#a31515">"foo"</span><span style="background:#ffffff;color:#000000">);</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> a = {};</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">a[sfoo] = </span><span style="background:#ffffff;color:#a31515">"sfoo"</span><span style="background:#ffffff;color:#000000">;</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">console.log(a.sfoo); </span><span style="background:#ffffff;color:#008000">// undefined</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">console.log(a[sfoo]);   </span><span style="background:#ffffff;color:#008000">// sfoo</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">console.log(a[Symbol(</span><span style="background:#ffffff;color:#a31515">"foo"</span><span style="background:#ffffff;color:#000000">)])    </span><span style="background:#ffffff;color:#008000">// undefined</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Podemos asignar una propiedad de un objeto a un símbolo, y entonces requerimos del símbolo para poder obtener el valor de la propiedad. Observa que **necesitamos el símbolo original (almacenado en sfoo), crear otro símbolo a partir de la misma cadena, no sirve**. Los símbolos son únicos.

Por lo tanto ahora podemos simular variables privadas de la siguiente manera:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:ab437ff3-2707-47a1-8a14-2bc8292703bc" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#0000ff">const</span><span style="background:#ffffff;color:#000000"> s_name = Symbol(</span><span style="background:#ffffff;color:#a31515">"name"</span><span style="background:#ffffff;color:#000000">);</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#0000ff">export</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">default</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">class</span><span style="background:#ffffff;color:#000000"> Foo {</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">constructor(name) {</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">this</span><span style="background:#ffffff;color:#000000">[s_name] = name;</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">get name() {</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">this</span><span style="background:#ffffff;color:#000000">[s_name];</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Los objetos de la clase Foo tienen una variable cuyo nombre **es el símbolo guardado en s_name**. Esa variable “no es realmente pública”, pero no hay manera de encontrar su nombre fuera del módulo que declara la clase (la variable s_name que contiene el símbolo, es privada del módulo por lo que no es accesible desde el exterior). Observa que no basta que alguien mire el código fuente del módulo y se cree _otro_ Symbol(“name”) para acceder a la variable “privada”: los símbolos son únicos, o lo que es lo mismo: _Symbol("a") != Symbol("a")._

Este método no genera variables totalmente privadas. Las variables que están almacenadas en una propiedad cuyo nombre es un símbolo, **no se listan en Object.getOwnPropertyNames** y **tampoco participan de la iteración de for..in** pero se pueden obtener todos los símbolos asociados a un objeto mediante Object.getOwnPropertySymbols, que devuelve un array con todos los símbolos u
  
sados como nombre de propiedad del objeto. Basta con usar los símbolos devueltos para acceder a las variables “privadas”.

Por lo tanto este sistema, vendría a equivaler a la visibilidad private de C# para entendernos: Por defecto es imposible acceder a la propiedad, pero tenemos un mecanismo (en C# es Reflection) que nos permite saltarnos esa protección y acceder al contenido de la variable “privada”.

**Weak maps**

El uso de weak maps nos permite tener variables completamente privadas. Para ello usamos el nuevo tipo WeakMap de ES2015, que básicamente es un diccionario clave, valor, pero es especial porque no impide al GC de JavaScript recolectar todos aquellos valores almacenados en el weak map, siempre y cuando la referencia a la clave que tenga el weak map sea la única restante. Es decir, a todos los efectos el weak map no cuenta para el GC: aunque un objeto esté almacenado en él, si la clave usada no está referenciado por ningún otro objeto, el GC lo podrá eliminar.

El código usando WeakMap es el siguiente:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:6084b55c-7959-4532-b375-c4549761f8a8" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#0000ff">const</span><span style="background:#ffffff;color:#000000"> privates = </span><span style="background:#ffffff;color:#0000ff">new</span><span style="background:#ffffff;color:#000000"> WeakMap();</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#0000ff">export</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">default</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">class</span><span style="background:#ffffff;color:#000000"> Foo {</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">constructor(name) {</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000">privates.set(</span><span style="background:#ffffff;color:#0000ff">this</span><span style="background:#ffffff;color:#000000">, {name: name});</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">get name() {</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> privates.get(</span><span style="background:#ffffff;color:#0000ff">this</span><span style="background:#ffffff;color:#000000">).name;</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

La idea es muy sencilla: por cada objeto que se cree de Foo, se almacena un valor en el WeakMap _privates_. La clave de dicho valor es _this_, que es el propio objeto Foo que se crea. Este objeto (privates.get(this)) almacena todo el estado “privado” del objeto Foo en cuestión. Gracias al hecho de que _privates_ es un WeakMap no hay _memory leak:_ Cuando el objeto Foo deje de usarse podrá ser eliminado por el garbage collector, aunque esté referenciado por el WeakMap como una de las claves usadas (precisamente por ser un _privates_ Weak Map).

La clave (nunca mejor dicho :P) del asunto está en usar _this_ como clave del Weak Map _privates_.

Al igual que en el caso del símbolo, el Weak Map es una variable privada del módulo, por lo que no se puede acceder a fuera desde él. Y a diferencia del caso del símbolo, ahora si que no hay nada que nadie pueda hacer para acceder a la variable privada que almacena el valor del nombre.

Saludos!