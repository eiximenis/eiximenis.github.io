---
title: Algunas consideraciones sobre las structs

author: eiximenis

date: 2016-07-27T19:48:11+00:00
geeks_url: /?p=1783
geeks_ms_views:
  - 2520
categories:
  - .net
  - 'C#'

---
El otro día un tweet de <a href="https://twitter.com/jc_quijano" target="_blank" rel="noopener noreferrer">Juan Quijano</a>, animó una pequeña discusión sobre la diferencia entre clases y estructuras en .NET. Este no es el primer post que escribo al respecto, pero bueno, aprovechando la coyuntura vamos a comentar algunas de las cosas que se mencionaron en el pequeño debate que generó <a href="https://twitter.com/jc_quijano/status/757967853970722817" target="_blank" rel="noopener noreferrer">el tweet de Juan</a>.

<!--more-->

Seguramente todos ya sabréis que la diferencia fundamental entre clases y estructuras en .NET es que las primeras son tipos por referencia y las segundas tipos por valor. Es decir cuando trabajamos con una clase lo hacemos siempre a través de una referencia, que “apunta” al objeto real. Por otro lado cuando trabajamos con una estructura podemos asumir que no hay referencia&nbsp; y que trabajamos con el “objeto real”, sin intermediarios visibles.

De esta diferencia se deduce que si no hay referencias **es imposible que dos variables distintas apunten a la misma estructura**. Por lo tanto, siendo S una estructura, el código _“S a = b;”_ clona la estructura contenida en la variable b en la variable a. Por lo que después de la asignación tenemos **dos estructuras idénticas. Pero tenemos dos**. Eso no ocurre en las clases donde el mismo código (si S fuese una clase) haría que tuviesémos dos referencias (a y b) apuntando a un único objeto.

De esto mismo se deduce que el valor “null” no es válido en una estructura. El valor null es un valor de referencia, no de objeto. Es decir, un objeto no puede ser null, es la referencia que vale null para indicar que no apunta a ningún objeto. En el caso de las estructuras, dado que no hay referencia, no puede haber el valor null: **todas las variables de tipo struct contienen su propio y único objeto**.

Igualmente cuando pasamos una estructura como parámetro de una función, lo que recibe la función no es la estructura original si no una copia. Y por supuesto lo mismo ocurre cuando devolvemos una estructura como valor de retorno. Son casos análogos al de la asignación que veíamos antes.

Para inicializar una variable de tipo struct se usa una sintaxis unificada con la de las clases:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:076a70e8-e052-4016-9054-569c0d2b6a20" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="color:#0000ff">var</span><span style="color:#000000"> p = </span><span style="color:#0000ff">new</span><span style="color:#000000"> </span><span style="color:#2b91af">Point</span><span style="color:#000000">();</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

De hecho leyendo este código no sabemos si “Point” es una estructura o una clase: el lenguaje unifica la inicialización de ambos tipos para una mayor coherencia. Eso es&nbsp; bueno, porque tenemos una sola forma de inicializar objetos (con new) pero a cambio nos puede confundir y hacernos creer que ambos tipos (clases y estructuras) son más parecidos de lo que realmente son.

Un tema interesante es que **las estructuras siempre tienen el constructor por defecto**. No se puede evitar, ni **aún declarando otro constructor**. Así dada la siguiente estructura:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:e6cbbbd7-baf9-4c26-8d83-890829d26cd1" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="color:#000000"></span><span style="color:#0000ff">struct</span><span style="color:#000000"> </span><span style="color:#2b91af">Point</span>
        </li>
        <li>
          <span style="color:#000000">{</span>
        </li>
        <li>
              <span style="color:#000000"></span><span style="color:#0000ff">public</span><span style="color:#000000"> </span><span style="color:#0000ff">int</span><span style="color:#000000"> X;</span>
        </li>
        <li>
              <span style="color:#000000"></span><span style="color:#0000ff">public</span><span style="color:#000000"> </span><span style="color:#0000ff">int</span><span style="color:#000000"> Y;</span>
        </li>
        <li>
              <span style="color:#000000"></span><span style="color:#0000ff">public</span><span style="color:#000000"> Point(</span><span style="color:#0000ff">int</span><span style="color:#000000"> x, </span><span style="color:#0000ff">int</span><span style="color:#000000"> y)</span>
        </li>
        <li>
              <span style="color:#000000">{</span>
        </li>
        <li>
                  <span style="color:#000000">X = x;</span>
        </li>
        <li>
                  <span style="color:#000000">Y= y;</span>
        </li>
        <li>
              <span style="color:#000000">}</span>
        </li>
        <li>
          <span style="color:#000000">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Se pueden crear objetos Point usando new Point(). **Observa que eso no es cierto en una clase: si se declara un constructor en una clase, el constructor por defecto deja de existir**.

Por supuesto, **si el constructor por defecto existe siempre, es lícito preguntarnos qué hace**: Pues **inicializa todos los miembros de la struct a su valor por defecto**. En este caso “new Point()” nos devolverá un punto en el que el miembro X y el miembro Y tienen el valor default(int) que es 0. Es posible que este comportamiento no te guste o lo quieras cambiar. **Pues mala suerte: C# no permite redefinir el constructor por defecto de una estructura**. El siguiente código no compila:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:4f40a16a-cbc0-47ef-a075-2966505bde0d" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="color:#000000"></span><span style="color:#0000ff">struct</span><span style="color:#000000"> </span><span style="color:#2b91af">Point</span>
        </li>
        <li>
          <span style="color:#000000">{</span>
        </li>
        <li>
              <span style="color:#000000"></span><span style="color:#0000ff">public</span><span style="color:#000000"> </span><span style="color:#0000ff">int</span><span style="color:#000000"> X;</span>
        </li>
        <li>
              <span style="color:#000000"></span><span style="color:#0000ff">public</span><span style="color:#000000"> </span><span style="color:#0000ff">int</span><span style="color:#000000"> Y;</span>
        </li>
        <li>
              <span style="color:#000000"></span><span style="color:#0000ff">public</span><span style="color:#000000"> Point(</span><span style="color:#0000ff">int</span><span style="color:#000000"> x, </span><span style="color:#0000ff">int</span><span style="color:#000000"> y)</span>
        </li>
        <li>
              <span style="color:#000000">{</span>
        </li>
        <li>
                  <span style="color:#000000">X = x;</span>
        </li>
        <li>
                  <span style="color:#000000">Y= y;</span>
        </li>
        <li>
              <span style="color:#000000">}</span>
        </li>
        <li>
              <span style="color:#000000"></span><span style="color:#0000ff">public</span><span style="color:#000000"> Point()</span>
        </li>
        <li>
              <span style="color:#000000">{</span
>
        </li>
        <li>
                  <span style="color:#000000">X = 0;</span>
        </li>
        <li>
                  <span style="color:#000000">Y = 0;</span>
        </li>
        <li>
              <span style="color:#000000">}</span>
        </li>
        <li>
          <span style="color:#000000">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

El error, además, es muy claro y no deja lugar a dudas: “_CS0568 Structs cannot contain explicit parameterless constructors_”. Una de las novedades que se preveyeron para C#6 fue precisamente que se pueda definir el constructor por defecto de las structs (y estuvo presente en algunas RCs), pero al final se descartó. Así que de momento no es posible.

Esta explicación **debería ya resolver <a href="https://twitter.com/jc_quijano/status/757967853970722817" target="_blank" rel="noopener noreferrer">la duda de Juan</a>**. Él se preguntaba porque new Guid() devolvía un Guid vacío (todo ceros) y para obtener un Guid único era necesario hacer Guid.NewGuid(). Pues la razón es, obviamente, que Guid es una struct. Por lo tanto “new Guid()” invoca al constructor por defecto que tienen todas las estructuras y que inicializa todos los miembros a su valor por defecto y que no podemos cambiar (una limitación a mi parecer un poco ridícula, pero es lo que hay).

Otro de los <a href="https://twitter.com/luisxkimo/status/758045175532888069" target="_blank" rel="noopener noreferrer">comentarios típicos que se hacen sobre las estructuras es que estas residen en la pila, mientras que las clases residen en el heap</a>__. Por supuesto los objetos que son instancias de clases residen en el _heap_. Es obvio ya que son, por definición, “long-lived objects”, es decir la vida del objeto está desligada del ámbito de la referencia que lo contiene. Así dado el siguiente código **y asumiendo que “Rect” es una clase:**

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:1b2619df-6ed2-4d33-8456-2ae6b7b0cd75" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="color:#000000"></span><span style="color:#2b91af">Rect</span><span style="color:#000000"> Foo()</span>
        </li>
        <li>
          <span style="color:#000000">{</span>
        </li>
        <li>
              <span style="color:#000000"></span><span style="color:#0000ff">var</span><span style="color:#000000"> obj = </span><span style="color:#0000ff">new</span><span style="color:#000000"> </span><span style="color:#2b91af">Rect</span><span style="color:#000000">();</span>
        </li>
        <li>
              <span style="color:#000000"></span><span style="color:#008000">// Do something</span>
        </li>
        <li>
          <span style="color:#000000">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

El objeto asignado en la referencia “obj” sigue existiendo incluso después de salir de la función Foo. Quizá en este caso se podría pensar que el compilador _podría_ ser lo suficientemente inteligente como para detectar que la referencia “obj” no se copia hacia ningún sitio de la función y podría eliminar el objeto creado al salir de Foo, pero… ¿para qué complicar el compilador teniendo ya un elemento que se encarga de eliminar objetos inaccesibles? Al salir de Foo, el objeto Rect es inaccesible (no hay referencia alguna que apunte a él) y el _garbage collector_ ya lo eliminará.

Observa que **es imprescindible** que los objetos instancias de clases tengan un ciclo de vida independiente del de las referencias que las contienen. Dado este código:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:7dda819a-6b3f-42bf-9846-ab47008ffe73" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="color:#000000"></span><span style="color:#2b91af">Rect</span><span style="color:#000000"> Foo()</span>
        </li>
        <li>
          <span style="color:#000000">{</span>
        </li>
        <li>
              <span style="color:#000000"></span><span style="color:#0000ff">var</span><span style="color:#000000"> obj = </span><span style="color:#0000ff">new</span><span style="color:#000000"> </span><span style="color:#2b91af">Rect</span><span style="color:#000000">();</span>
        </li>
        <li>
              <span style="color:#000000"></span><span style="color:#008000">// Do something</span>
        </li>
        <li>
              <span style="color:#000000"></span><span style="color:#0000ff">return</span><span style="color:#000000"> obj;</span>
        </li>
        <li>
          <span style="color:#000000">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

El objeto Rect creado **no puede ser eliminado al salir de la función Foo** porque en este caso dado un código “var f = Foo()” la referencia “f” terminaría apuntando a un objeto eliminado.

Eso misma en una structura no es necesario, porque en este caso el objeto creado dentro de la función se copia cuando devolvemos de la función, por lo que el original puede ser destruído sin miedo.

Vale, queda claro que unos (clases) son objetos “long-lived” y otros (structs) no tienen porque serlo. ¿Pero eso implica que las structs deban guardarse en la pila y las clases en el _heap_?. No es del todo cierto… la realidad es que las clases sí que deben guardarse en el _heap_ pero las estructuras no tienen por qué guardarse en la pila. Ojo, se puede ¿eh? Pero no es imprescindible. **La especificación de .NET no obliga a que los objetos struct se guarden en la pila**. Es algo que depende de la implementación. De hecho, realmente **a veces puede ocurrir que objetos estructura estén en el _heap_.** Y la realidad es que, como desarrollador, debe darte igual si un objeto estructura está en la pila o el heap. Esto es un detalle de la implementación y no es importante. Lo que debes entender es la **semántica de valor** de las estructuras.

**La asignación de this**

<a href="https://twitter.com/eiximenis/status/758046799731027968" target="_blank" rel="noopener noreferrer">Como comenté</a> en el hilo que se generó a raíz del tweet de Juan, las stucts tienen otras curiosidades, y una de ellas es la asignación de this. Básicamente, en una struct, this no es una constante si no una variable, y por lo tanto puede asignarse. Así, el siguiente código es válido:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:c0fe9e32-2c5d-4710-8e85-ec8f0bdcd0b7" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="color:#000000"></span><span style="color:#0000ff">struct</span><span style="color:#000000"> </span><span style="color:#2b91af">Point</span>
        </li>
        <li>
          <span style="color:#000000">{</span>
        </li>
        <li>
              <span style="color:#000000"></span><span style="color:#0000ff">public</span><span style="color:#000000"> </span><span style="color:#0000ff">int</span><span style="color:#000000"> X;</span>
        </li>
        <li>
              <span style="color:#000000"></span><span style="color:#0000ff">public</span><span style="color:#000000"> </span><span style="color:#0000ff">int</span><span style="color:#000000"> Y;</span>
        </li>
        <li>
              <span style="color:#000000"></span><span style="color:#0000ff">public</span><span style="color:#00
0000"> Point(</span><span style="color:#0000ff">int</span><span style="color:#000000"> x, </span><span style="color:#0000ff">int</span><span style="color:#000000"> y)</span>
        </li>
        <li>
              <span style="color:#000000">{</span>
        </li>
        <li>
                  <span style="color:#000000">X = x;</span>
        </li>
        <li>
                  <span style="color:#000000">Y= y;</span>
        </li>
        <li>
              <span style="color:#000000">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
              <span style="color:#000000"></span><span style="color:#0000ff">public</span><span style="color:#000000"> </span><span style="color:#0000ff">void</span><span style="color:#000000"> CloneInc(</span><span style="color:#2b91af">Point</span><span style="color:#000000"> p) {</span>
        </li>
        <li>
                  <span style="color:#000000"></span><span style="color:#0000ff">this</span><span style="color:#000000"> = p;</span>
        </li>
        <li>
                  <span style="color:#000000">X++;</span>
        </li>
        <li>
                  <span style="color:#000000">Y++;</span>
        </li>
        <li>
              <span style="color:#000000">}</span>
        </li>
        <li>
          <span style="color:#000000">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Observa como el método “CloneInc” asigna a this el valor del parámetro p. Recuerda que hablamos de estructuras, por lo que eso **asigna a this una copia del objeto estructura contenido en p**. Por eso tiene sentido esa asignación. En una clase eso no es posible, porque esa asignación implicaría que la referencia “this” se modifica, y que pasa a apuntar a otro objeto. En cambio en una estructura eso no es así. Simplemente significa que se modifica el valor del objeto contenido en this. Por eso es legal y tiene sentido poder hacer eso.

Y nada más, os dejo con otros posts sobre structs con más información:

  * [http://www.forcode.es/lenguaje/net-categoria/structs-vs-clases-tan-parecidas-tan-distintas/][1] (post que escribí yo mismo en el blog de forcode) 
      * [http://blog.koalite.com/2012/02/curiosidades-con-structs-en-c/][2] (post de <a href="https://twitter.com/gulnor" target="_blank" rel="noopener noreferrer">Juanma</a> sobre otras curiosidades de las structs) 
          * [http://www.campusmvp.es/recursos/post/clases-y-estructuras-en-net-cuando-usar-cual.aspx][3] (post del blog de Campus MVP sobre cuando usar clases o estructuras) 
              * [https://blogs.msdn.microsoft.com/ericlippert/2010/09/30/the-truth-about-value-types/][4] Post de Eric Lippert donde, entre otras cosas, comenta que no es obligatorio que las estructuras estén en la pila.</ul> 
            Un saludo!

 [1]: http://www.forcode.es/lenguaje/net-categoria/structs-vs-clases-tan-parecidas-tan-distintas/ "http://www.forcode.es/lenguaje/net-categoria/structs-vs-clases-tan-parecidas-tan-distintas/"
 [2]: http://blog.koalite.com/2012/02/curiosidades-con-structs-en-c/ "http://blog.koalite.com/2012/02/curiosidades-con-structs-en-c/"
 [3]: http://www.campusmvp.es/recursos/post/clases-y-estructuras-en-net-cuando-usar-cual.aspx "http://www.campusmvp.es/recursos/post/clases-y-estructuras-en-net-cuando-usar-cual.aspx"
 [4]: https://blogs.msdn.microsoft.com/ericlippert/2010/09/30/the-truth-about-value-types/ "https://blogs.msdn.microsoft.com/ericlippert/2010/09/30/the-truth-about-value-types/"