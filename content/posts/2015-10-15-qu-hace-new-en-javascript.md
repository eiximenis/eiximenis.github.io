---
title: ¿Qué hace ‘new’ en JavaScript?
description: ¿Qué hace ‘new’ en JavaScript?
author: eiximenis

date: 2015-10-15T13:09:01+00:00
geeks_url: /?p=1707
geeks_visits:
  - 915
geeks_ms_views:
  - 1212
categories:
  - javascript

---
Si preguntas a mucha gente cual es el objetivo de _new_ en JavaScript te dirán que el de crear objetos. Y estarán en lo cierto, aunque esa definición no es del todo precisa. Y es que mucha gente no entiende **exactamente** que hace _new_ así que vamos a dedicarle este post.

**Creación de objetos sin new**

La verdad es que _new_ no es necesario en JavaScript. Se pueden crear objetos perfectamente sin necesidad de usar new:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:1b2d20d4-5c3c-419c-acd3-a5cdd769aa18" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> foo = {</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">name: </span><span style="background:#ffffff;color:#a31515">'foo'</span><span style="background:#ffffff;color:#000000">,</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">getUpperName: </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> () {</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">this</span><span style="background:#ffffff;color:#000000">.name.toUpperCase();</span>
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

Esta manera de crear objetos la conocemos como _object notation_ y nos permite definir objetos con campos y métodos. Por supuesto, en este caso solo creamos un objeto pero basta con declarar una función que tenga ese código para crear tantos objetos como queramos:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:05b103b0-199e-4d92-9863-d7584af11963" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> Foo = </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> (name) {</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> foo = {</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000">name: name || </span><span style="background:#ffffff;color:#a31515">''</span><span style="background:#ffffff;color:#000000">,</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000">getUpperName: </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> () {</span>
        </li>
        <li>
                      <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">this</span><span style="background:#ffffff;color:#000000">.name.toUpperCase();</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> foo;</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> f2 = Foo(</span><span style="background:#ffffff;color:#a31515">"f2"</span><span style="background:#ffffff;color:#000000">);</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">f2.getUpperName();  </span><span style="background:#ffffff;color:#008000">// devuelve F2</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Nos podemos preguntar cual es el prototipo de cada objeto creado mediante una llamada a Foo. Pues, como es de esperar, el prototipo de f2 será Object.prototype. 

Podríamos indicar que los objetos creados por Foo heredan de Foo.prototype, con un código similar al:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:039f9f90-cad2-4184-9d04-cf87962762b6" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> Foo = </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> (name) {</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> foo = {</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000">name: name || </span><span style="background:#ffffff;color:#a31515">''</span><span style="background:#ffffff;color:#000000">,</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000">getUpperName: </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> () {</span>
        </li>
        <li>
                      <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">this</span><span style="background:#ffffff;color:#000000">.name.toUpperCase();</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">Object.setPrototypeOf(foo, Foo.prototype);</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> foo;</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> f2 = Foo(</span><span style="background:#ffffff;color:#a31515">"f2"</span><span style="background:#ffffff;color:#000000">);</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">f2.getUpperName();  </span><span style="background:#ffffff;color:#008000">// devuelve F2</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">f2 </span><span style="background:#ffffff;color:#0000ff">instanceof</span><span style="background:#ffffff;color:#000000"> Foo; </span><span style="background:#ffffff;color:#008000">// devuelve true</span
>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Lo interesante es que establecer como prototipo del objeto creado el objeto _Foo.prototype_, automáticamente hace que _instanceof Foo_ devuelva _true_ sobre todos los objetos creados por la función Foo. Vale, estoy usando _Object.setPrototypeOf_ que es una función nueva de ES2015, pero en ES5 clásico tenemos _Object.create_ que nos permite hacer lo mismo:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:a8d44d89-26c4-407a-8db1-f62d8257cfed" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> Foo = </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> (name) {</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> foo = Object.create(Foo.prototype);</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">foo.name = name || </span><span style="background:#ffffff;color:#a31515">''</span><span style="background:#ffffff;color:#000000">;</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">foo.getUpperName = </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> () {</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">this</span><span style="background:#ffffff;color:#000000">.name.toUpperCase();</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> foo;</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> f2 = Foo(</span><span style="background:#ffffff;color:#a31515">"f2"</span><span style="background:#ffffff;color:#000000">);</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">f2.getUpperName();  </span><span style="background:#ffffff;color:#008000">// devuelve F2</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">f2 </span><span style="background:#ffffff;color:#0000ff">instanceof</span><span style="background:#ffffff;color:#000000"> Foo; </span><span style="background:#ffffff;color:#008000">// devuelve true</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

**¿Entonces… qué hace new?**

Bueno, new se introdujo para dar una “sintaxis familiar” a los desarrolladores que venían de Java, pero básicamente hace lo mismo que hemos hecho nosotros. En JavaScript hablamos de _funciones constructoras_, pero no hay sintaxis para declarar una función constructora. Una función constructora es aquella _pensada para ser llamada con new_, pero que quede claro: es usar o no new al invocar una función lo que convierte a esa en constructora.

Cuando usamos new en una función, llamémosla Bar, ocurre lo siguiente:

  1. Se crea automáticamente **un objeto vacío que hereda de Bar.prototype** 
  2. Se llama a la función Bar, con el valor de **this enlazado al objeto creado en el punto anterior** 
  3. Si (y solo sí) la función Bar devuelve undefined, new devuelve el valor de this dentro de Bar (es decir, el objeto creado en el punto 1). 

Como, gracias al punto 2, dentro de una función llamada con _new_ el valor de _this_ es el objeto creado en el punto 1, decimos que en una función constructora el valor de _this_ es “el objeto que se está creando”. Y es por ello que las codificamos de esa manera:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:f03c0df0-a834-4e02-930c-01365639b129" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> Foo = </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> (name) {    </span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">this</span><span style="background:#ffffff;color:#000000">.name = name || </span><span style="background:#ffffff;color:#a31515">''</span><span style="background:#ffffff;color:#000000">;</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">this</span><span style="background:#ffffff;color:#000000">.getUpperName = </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> () {</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">this</span><span style="background:#ffffff;color:#000000">.name.toUpperCase();</span>
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

Pero para que veas exactamente hasta que punto _new_ no era “imprescindible” es porque incluso podríamos hacer una especie de polyfill para new:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:8a1078e7-29b0-49a8-8fb7-8540861eb8ed" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> functionToCtor = </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> (ctor) {</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> o = Object.create(ctor.prototype);</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> args = Array.prototype.slice.call(arguments)</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">args.splice(0, 1);</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> v = ctor.apply(o, args);</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#0
00000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> v === undefined ? o : v;</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

La función functionToCtor **hace lo mismo que new**. Es decir, en este caso dada la _función constructora Foo_ ambas expresiones son equivalentes:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:150f988e-d54b-4afe-9bdb-9bf8adc5dd05" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> foo = functionToCtor(Foo, </span><span style="background:#ffffff;color:#a31515">"foo"</span><span style="background:#ffffff;color:#000000">);</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> foo2 = </span><span style="background:#ffffff;color:#0000ff">new</span><span style="background:#ffffff;color:#000000"> Foo(</span><span style="background:#ffffff;color:#a31515">"foo2"</span><span style="background:#ffffff;color:#000000">);</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Tanto foo como foo2:

  * Tienen como prototipo Foo.prototype 
  * Tienen como constructor la función Foo 
  * Devuelven true a la expresión instanceof Foo 

**¿Y si una función constructora devuelve un valor?**

Esa es buena… Pues en este caso cuando se use new el valor obtenido será el devuelto por la función constructora (siempre que este valor sea distinto de _undefined_):

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:5b9a3921-387b-4c25-9d77-b6fa9a5e5244" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> Foo = </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> (name) {    </span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">this</span><span style="background:#ffffff;color:#000000">.name = name || </span><span style="background:#ffffff;color:#a31515">''</span><span style="background:#ffffff;color:#000000">;</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">this</span><span style="background:#ffffff;color:#000000">.getUpperName = </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> () {</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">this</span><span style="background:#ffffff;color:#000000">.name.toUpperCase();</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> { a: 10 };</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> f = </span><span style="background:#ffffff;color:#0000ff">new</span><span style="background:#ffffff;color:#000000"> Foo();</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">f.a;        </span><span style="background:#ffffff;color:#008000">//  10</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">f </span><span style="background:#ffffff;color:#0000ff">instanceof</span><span style="background:#ffffff;color:#000000"> Foo </span><span style="background:#ffffff;color:#008000">// false</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Si… a pesar de que f ha sido creado usando _new Foo_, el prototipo de f ya no es Foo.prototype, ya que f vale el objeto devuelto por la función Foo. En este caso el objeto “original” cuyo prototipo es Foo.prototype y al que nos referíamos con _this_ dentro de Foo se ha perdido. Lo mismo ocurre si usas el “polyfill” _functionToCtor_ que hemos visto antes.

**Funciones duales**

Podemos hacer una función que sepa si se ha invocado con _new_ o no? Pues sí:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:de53f0c2-5576-4087-a7ea-da356fee7e1f" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> Foo = </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> (name) {</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">if</span><span style="background:#ffffff;color:#000000"> (</span><span style="background:#ffffff;color:#0000ff">this</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">instanceof</span><span style="background:#ffffff;color:#000000"> Foo) {</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#008000">// Invocada con new</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">else</span><span style="background:#ffffff;color:#000000"> {</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#008000">// Invocada sin new</span>
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

Esto nos permite trabajar con _this_ en el caso de que se haya usado new o bien no trabajar con _this_ y devolver un valor en el caso de que no se haya invocado a la función con _this_. De todos modos si quieres que el usuario obtenga el mismo valor use new o no, lo más fácil es devolver el valor desde la función constructora. En este caso, eso sí, pierdes el hecho de que el prototipo del objeto sea la función constructora. Si quieres asegurarte que el usuario obtiene siempre un objeto con el prototipo asociado a la función constructora, es decir como si siempre usase new, puedes hacer:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:b343dfcb-d8fa-4772-b0c6-9c6b55f13a5a" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; displa
y: inline; padding-right: 0px">
  </p> 
  
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> Foo = </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> (name) {</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> _create = </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> (name) {</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">this</span><span style="background:#ffffff;color:#000000">.name = name || </span><span style="background:#ffffff;color:#a31515">''</span><span style="background:#ffffff;color:#000000">;</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">this</span><span style="background:#ffffff;color:#000000">.getUpperName = </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> () {</span>
        </li>
        <li>
                      <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">this</span><span style="background:#ffffff;color:#000000">.name.toLocaleUpperCase();</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">if</span><span style="background:#ffffff;color:#000000"> (</span><span style="background:#ffffff;color:#0000ff">this</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">instanceof</span><span style="background:#ffffff;color:#000000"> Foo) {</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000">_create.bind(</span><span style="background:#ffffff;color:#0000ff">this</span><span style="background:#ffffff;color:#000000">)(name);</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">else</span><span style="background:#ffffff;color:#000000"> {</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> o = Object.create(Foo.prototype);</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000">_create.bind(o)(name);</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> o;</span>
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

La función __create_ es la que realmente rellena el objeto (le llega ya creado en la variable _this_). La función Foo lo único que hace es mirar si ha sido llamada con new o no. En el primer caso llama a \_create pero vinculando el valor de this dentro de \_create al propio valor de this dentro de Foo (que en este caso es el objeto que se está creando) y no devolver nada. En el caso que el usuario no haya usado new la función Foo crea un objeto vacío, le asigna Foo.prototype como prototipo y luego llama a \_create vinculando el valor de this dentro de \_create al del objeto que acaba de crear. Y finalmente, en este caso, devuelve el objeto creado.

Así, tanto si el usuario hace var x = Foo() como var x2 = new Foo() obtiene lo mismo: un objeto cuyo prototipo es Foo.prototype y que por lo tanto devuelve true a instanceof Foo.

Espero que este post os haya ayudado a entender como funciona new en JavaScript.

Saludos!