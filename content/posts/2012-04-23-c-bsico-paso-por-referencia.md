---
title: '[C# Básico] Paso por referencia'
author: eiximenis

date: 2012-04-23T10:50:45+00:00
geeks_url: /?p=1594
geeks_visits:
  - 8845
geeks_ms_views:
  - 8157
categories:
  - Uncategorized

---
¡Buenas! Este es un nuevo post de la serie [C# Básico][1], que como su propio nombre indica trata sobre aspectos digamos elementales del lenguaje. Cada post es independiente y el orden de publicación no tiene porque ser el de lectura. Los temas los voy sacando de los foros o consultas que se me realizan 🙂

Hoy vamos a tratar un tema que veo que causa mucha confusión: el paso de parámetros por referencia. Como en todos los posts de esta serie lo haremos de forma didáctica y paso a paso.

**1. Paso por valor**

Para entender que es el paso por referencia, primero es necesario ver que significa el _paso por valor_. Que salida genera este programa?

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">static</span> <span style="color: blue">void</span> Main(<span style="color: blue">string</span>[] args)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">var</span> inicial = 10;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; Incrementa(inicial);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #2b91af">Console</span>.WriteLine(<span style="color: #a31515">"Valor DESPUES de incrementar es "</span> + inicial);
  </p>
  
  <p style="margin: 0px">
    }
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue">static</span> <span style="color: blue">void</span> Incrementa(<span style="color: blue">int</span> num)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; num = num + 1;
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

El sentido común dice que el programa imprimirá “_Valor DESPUES de incrementar es 11_”, pero la realidad es otra:

 <img style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_4FC5DB1D.png" width="488" height="75" />

¿Como es posible esto? Pues porque la variable _incial_ ha sido pasada por valor. Pasar una variable por valor _significa hacer una copia de dicha variable_ (de ahí el nombre, ya que se pasa el _valor_ y no la variable en sí). De este modo el parámetro num toma el valor de la variable inicial, es decir 10. Pero num es una copia de inicial, así que modificar num, no modifica para nada inicial. Al salir del método, efectivamente num vale 11 pero inicial continua valiendo 10 (además al salir del método la variable num es destruída ya que su alcance es de método).

**2. Paso de objetos**

Veamos ahora el siguiente código, donde en lugar de pasar un entero, pasamos un objeto de la clase Foo que tiene una propiedad entera:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">class</span> <span style="color: #2b91af">Foo</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">public</span> <span style="color: blue">int</span> Bar { <span style="color: blue">get</span>; <span style="color: blue">set</span>; }
  </p>
  
  <p style="margin: 0px">
    }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue">class</span> <span style="color: #2b91af">Program</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">static</span> <span style="color: blue">void</span> Main(<span style="color: blue">string</span>[] args)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">var</span> inicial = <span style="color: blue">new</span> <span style="color: #2b91af">Foo</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; inicial.Bar = 10;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; Incrementa(inicial);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #2b91af">Console</span>.WriteLine(<span style="color: #a31515">"Valor DESPUES de incrementar es "</span> + inicial.Bar);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">static</span> <span style="color: blue">void</span> Incrementa(<span style="color: #2b91af">Foo</span> foo)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; foo.Bar = foo.Bar + 1;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

¿Cual es la salida del programa ahora? Si nos basamos en lo que vimos en el punto anterior deberíamos responder que va a imprimir “_Valor DESPUES de incrementar es 10_”, ya que el parámetro foo debería ser una copia de inicial y por lo tanto modificar foo no debe afectar para nada a inicial.

Pero la realidad es otra:

 <img style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_6E109213.png" width="400" height="75" />

Pasar un objeto **no crea** una copia del objeto. Es por eso que decimos que los objetos no se pasan por valor, se pasan por referencia. Así pues modificar un objeto desde un método modifica el objeto original. No hay manera en C# de pasar una copia entera de un objeto entre métodos (a no ser que se haga manualmente).

> **Nota:** En el código anterior, simplemente modifica “class” por “struct” cuando declaramos Foo. ¿Qué ocurre entonces? Pues que el programa ahora muestra “_Valor DESPUES de incrementar es 10_”. ¿A que es debido esto? Pues a que las estructuras se pasan por valor (¡es decir se copia su contenido!). Ver el punto (4) para más detalles.

Pero si nos quedamos en este punto obviamos una pregunta muy importante: Efectivamente los objetos se pasan por referencia… ¿pero las propias referencias como se pasan? Pues la respuesta es que **las referencias se pasan por valor**. Es decir la referencia **_foo_ es una _copia_ de la referencia _inicial_**. Pero copiar una referencia no es copiar su contenido (el objeto). Copiar una referencia significa que ahora tenemos _dos referencias_ distintas que apuntan al mismo objeto. Por ello debemos tener muy claro que **no es lo mismo modificar el contenido de una referencia (el objeto) que modificar la referencia misma**. Si modificamos el contenido (es decir una propiedad del objeto apuntado, en este caso la propiedad Bar), este cambio es compartido ya que ambas referencias apuntan al mismo objeto. Pero si modificamos la referencia este cambio no será compartido:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">static</span> <span style="color: blue">void</span> Main(<span style="color: blue">string</span>[] args)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">var</span> inicial = <span style="color: blue">new</span> <span style="color: #2b91af">Foo</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; inicial.Bar = 10;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; Incrementa(inicial);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #2b91af">Console</span>.WriteLine(<span style="color: #a31515">"Valor DESPUES de incrementar es "</span> + inicial.Bar);
  </p>
  
  <p style="marg
in: 0px">
    }
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue">static</span> <span style="color: blue">void</span> Incrementa(<span style="color: #2b91af">Foo</span> foo)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">int</span> valor = foo.Bar;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; foo = <span style="color: blue">new</span> <span style="color: #2b91af">Foo</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; foo.Bar = valor + 1;
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Este código _modifica la referencia foo**.**_ No modifica el contenido, modifica la referencia ya que asigna un nuevo objeto a la referencia foo. Al salir del método tenemos:

  1. Una referencia (inicial) que apunta a un objeto 
  2. Otra referencia (foo) que apunta a un objeto _nuevo_ 
  3. El valor de la propiedad “Bar” del objeto apuntado por inicial es 10. 
  4. El valor de la propiedad “Bar” del objeto apuntado por foo es 11. 

Al salir del método la referencia foo se pierde, y su contenido (el objeto cuya propiedad Bar vale 11) al no ser apuntado por ninguna otra referencia será destruido por el Garbage Collector. Y efectivamente ahora la salida del programa es:

<img style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_4FC5DB1D.png" width="488" height="75" />

Si entiendes la diferencia entre modificar una referencia y modificar el _contenido_ de una referencia, entonces ya estás listo para el siguiente punto…

**3. El paso por referencia**

En el punto anterior hemos visto que los objetos se pasan por referencia, pero las propias referencias se pasan por valor. Así que la pregunta obvia es: ¿hay alguna manera de pasar las referencias por referencia?

Y la respuesta es sí: usando la palabra clave ref:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">static</span> <span style="color: blue">void</span> Main(<span style="color: blue">string</span>[] args)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">var</span> inicial = <span style="color: blue">new</span> <span style="color: #2b91af">Foo</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; inicial.Bar = 10;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; Incrementa(<span style="color: blue">ref</span> inicial);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #2b91af">Console</span>.WriteLine(<span style="color: #a31515">"Valor DESPUES de incrementar es "</span> + inicial.Bar);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #2b91af">Console</span>.ReadLine();
  </p>
  
  <p style="margin: 0px">
    }
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue">static</span> <span style="color: blue">void</span> Incrementa(<span style="color: blue">ref</span> <span style="color: #2b91af">Foo</span> foo)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">int</span> valor = foo.Bar;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; foo = <span style="color: blue">new</span> <span style="color: #2b91af">Foo</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; foo.Bar = valor + 1;
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Fíjate que ref debe usarse tanto al declarar el parámetro como al invocar al método. El uso de ref significa que queremos pasar el parámetro por referencia. Si el parámetro es un objeto (como el caso que nos ocupa), ref no significa “pasar el objeto por referencia”, pues eso se hace siempre (como hemos visto en el punto (2)). En este caso ref significa “pasar la referencia por referencia”.

Es por ello que ahora _foo_ y _inicial_ **son la misma referencia**. Dado que son la misma referencia, forzosamente las dos deben apuntar al mismo objeto. Por ello cuando hacemos foo = new Foo(); estamos modificando la referencia _foo_ haciendo que apunte a otro objeto distinto. Pero si foo y inicial son la misma referencia, al modificar foo modificamos inicial, por lo que ahora al salir del método Incrementa:

  1. La referencia foo apunta a un objeto nuevo cuya propiedad Bar vale 11. 
  2. La referencia inicial, dado que es la misma que foo, apunta al mismo objeto. 
  3. El antiguo objeto (el que su valor Bar valía 10) al no estar apuntado por ninguna referencia, será destruído por el Garbage Collector. 

Por eso, ahora la salida del programa es:

<img style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_6E109213.png" width="400" height="75" />

**4. Paso por referencia de tipos por valor**

En terminología de .NET llamamos tipos por valor aquellos tipos que no son pasados por referencia. Así los objetos _no_ son tipos por valor, ya que hemos visto en el punto (2) que se pasan por referencia. Pero p.ej. un int es un tipo por valor, ya que hemos visto en el punto (1) que se pasa por valor.

En general son tipos por valor todos los tipos simples (boolean, int, float,…), los enums, las estructuras (como DateTime). No son tipos por valor los objetos (¡ojo, que eso incluye a string!). Hay una forma sencilla de saber si un tipo es por referencia o por valor: si admite null es por referencia y si no admite tener el valor null es por valor.

Pues bien, ref puede usarse para pasar por referencia un tipo por valor, como p.ej. un int:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">static</span> <span style="color: blue">void</span> Main(<span style="color: blue">string</span>[] args)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">var</span> inicial = 10;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; Incrementa(<span style="color: blue">ref</span> inicial);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #2b91af">Console</span>.WriteLine(<span style="color: #a31515">"Valor DESPUES de incrementar es "</span> + inicial);
  </p>
  
  <p style="margin: 0px">
    }
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue">static</span> <span style="color: blue">void</span> Incrementa(<span style="color: blue">ref</span> <span style="color: blue">int</span> valor)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; valor = valor + 1;
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Como ya debes suponer ahora la salida del programa es:

<img style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_6E109213.png" width="400" height="75" />

En este caso la variable _valor_ no es una copia de la variable inicial. Ambas variables son la misma, por lo que al modificar valor, estamos modificando también inicial. Por ello al salir del método, el valor de inicial sigue siendo 11.

**5. Resumen**

En resumen, hay cuatro puntos a tener en cuenta:

  1. Los tipos simples, estructuras y enums se pasan por valor (de ahí que digamos que son tipos por valor). Es decir, se pasa una copia de su contenido.
  2. Los objetos se pasan por referencia. Pero la referencia se pasa por valor, es decir el método recibe _otra</ em> referencia que apunta al mismo objeto.</li> 
    
      * La palabra clave “ref” permite pasar un parámetro por referencia.
      1. Si este parámetro es un tipo por valor se pasa por referencia, es decir no se pasa una copia sino que se pasa _la propia variable_.
      2. Si este parámetro es un objeto, lo que se pasa por referencia es la propia referencia.</ol> 
    
    ¡Espero que este post os haya aclarado un poco el tema de paso por valor y paso por referencia! 😉

 [1]: http://geeks.ms/blogs/etomas/archive/tags/c_2300_+basico/default.aspx