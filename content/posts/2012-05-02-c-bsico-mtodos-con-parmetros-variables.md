---
title: '[C# Básico] Métodos con parámetros variables'

author: eiximenis

date: 2012-05-02T12:07:57+00:00
geeks_url: /?p=1595
geeks_visits:
  - 13178
geeks_ms_views:
  - 4973
categories:
  - Uncategorized

---
¡Hey! Dos entradas de la serie [C# Básico][1] en menos de un mes… ¿Señal de algo? Quien sabe… 😛

Antes que nada el aviso típico de esta serie: En esos posts exploramos elementos, digamos, básicos del lenguaje. No es un tutorial ni un libro ni nada. Cada post es independiente del resto y pueden ser leídos en el orden en que prefiráis… Dicho esto, al tajo.

Bueno, en este post veremos como funcionan los métodos con parámetros variables en C#. Cuando digo parámetros variables me refiero a que el número de parámetros es variable (o sea le puedes pasar 1 parámetro, ó 10 ó 100).

**Primera opción: sobrecarga**

Cuando aprendemos C# por norma general nos dicen que al definir un método debemos indicar que parámetros acepta, y que todos esos parámetros deben pasarse para invocar el método. Tranquilo, no te han engañado, es cierto. Todos sabemos que si tengo un método:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">class</span> <span style="color: #2b91af">Foo</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">public</span> <span style="color: blue">void</span> DoBar(<span style="color: blue">int</span> baz, <span style="color: blue">string</span> bazbar) { }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Si quiero invocar el método DoBar, debo pasarle los dos parámetros: un int y una cadena. No hay otra. O se hace o el código no compila.

Entonces… ¿como puedo hacer métodos con parámetros variables? Pues, la primera opción, muy usada, para proporcionar _la ilusión_ de un número variable de parámetros (pues en este caso se trata de una ilusión) es aprovechar el mecanismo de sobrecarga. Sobrecargar un método significa que hay dos métodos métodos **independientes** pero que tienen el mismo nombre. El compilador tan solo pedirá una cosa: que la lista de parámetros sea distinta, o bien en número, o bien en el tipo de parámetros. Luego, cuando se llama el método, el compilador invocará al método que toque según el tipo y número de parámetros que se le pasen. Si alguien viene de C (pero no de C++) igual le sorprende esto, pero es realmente útil. Recuerdas toda aquella pléyade de funciones itoa, ltoa, ultoa y similares? Todas hacían lo mismo, convertir un número a cadena, pero claro la primera convertía un int, la segunda un long y la tercera un unsigned long. Esto mismo en C# se puede conseguir declarando tres funciones, pero todas ellas llamadas igual, p.ej. numberToString y teniendo cada una un tipo de parámetros variables. Así tenemos que recordar un solo nombre de método en lugar de n-mil.

El framework está lleno de métodos sobrecargados. ¿Un ejemplo? El método ToInt32 de la clase Convert:

 <img style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_6AEF6B21.png" width="264" height="66" />

Aquí lo veis, tiene 19 sobrecargas! Eso significa que hay 19 métodos llamados ToInt32 en la clase Convert. Insisto en que se trata de métodos independientes, si se quisiera el tipo de retorno podría ser distinto en cada caso y por supuesto podrían hacer cosas completamente distintas&#160; (aunque hay que estar un poco mal de la cabeza para poner dos métodos que se llamen igual pero hagan cosas distintas :p).

Sobrecargando un método puedo dar la ilusión de que el número de parámetros es variable:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">class</span> <span style="color: #2b91af">Foo</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">public</span> <span style="color: blue">void</span> DoBar(<span style="color: blue">int</span> baz, <span style="color: blue">string</span> bazbar) { }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">public</span> <span style="color: blue">void</span> DoBar(<span style="color: blue">int</span> baz) { }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">public</span> <span style="color: blue">void</span> DoBar(<span style="color: blue">string</span> bazbar) { }
  </p>
  
  <p style="margin: 0px">
    }
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
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">var</span> foo = <span style="color: blue">new</span> <span style="color: #2b91af">Foo</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; foo.DoBar(10);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; foo.DoBar(<span style="color: #a31515">"edu"</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; foo.DoBar(20, <span style="color: #a31515">"edu"</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p></p>
</div>

Bueno, esta técnica puede usarse para casos con pocos parámetros, pero en el fondo no tenemos realmente un “número de parámetros variable”, ya que no le puedo pasar 8 parámetros a DoBar.

**Segunda opción: params**

Antes que nada, a alguien se le ocurre un método en C# en el que se le pueda pasar un número arbitrario de parámetros (uno, diez o cien)? Pues hay varios, pero el más conocido es string.Format.

Seguro que todos habéis usado string.Format. Y sabéis que en función del primer parámetro, se le pasan luego tantos parámetros adicionales como sea necesario. La verdad es que string.Format tiene varias sobrecargas, pero te lo aseguro, no tiene todas las posibles. Entonces, ¿como lo hace? Pues usando la palabra clave _params_ en C#:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">class</span> <span style="color: #2b91af">Foo</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">public</span> <span style="color: blue">void</span> DoBar(<span style="color: blue">int</span> baz, <span style="color: blue">params</span> <span style="color: blue">string</span>[] bazbar) { }
  </p>
  
  <p style="margin: 0px">
    }
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
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">var</span> foo = <span style="color: blue">new</span> <span style="color: #2b91af">Foo</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; foo.DoBar(10, <span style="color: #a31515">"hola"</span>, <span style="color: #a31515">"adios"</span>, <span style="color: #a31515">"bye"</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; foo.DoBar(20);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; foo.DoBar(10, <span style="color: #a31515">"hi!"</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p
 style="margin: 0px">
    }
  </p></p>
</div>

Este código compila y es perfectamente válido. El método DoBar tiene realmente dos parámetros: un int y un array de cadenas. Pero el uso de la palabra clave _params_ permite pasar todas las cadenas del array no como un array sino como n parámetros separados por comas. Pero lo que DoBar recibe es un array de cadenas. Así el siguiente programa:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">public</span> <span style="color: blue">void</span> DoBar(<span style="color: blue">int</span> baz, <span style="color: blue">params</span> <span style="color: blue">string</span>[] bazbar)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #2b91af">Console</span>.WriteLine(<span style="color: #a31515">"# cadenas: "</span> + bazbar.Length);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">for</span> (<span style="color: blue">int</span> i = 0; i < bazbar.Length; i++)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #2b91af">Console</span>.Write(bazbar[i] + <span style="color: #a31515">","</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #2b91af">Console</span>.WriteLine();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
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
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">var</span> foo = <span style="color: blue">new</span> <span style="color: #2b91af">Foo</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; foo.DoBar(10, <span style="color: #a31515">"hola"</span>, <span style="color: #a31515">"adios"</span>, <span style="color: #a31515">"bye"</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; foo.DoBar(20);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; foo.DoBar(10, <span style="color: #a31515">"hi!"</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Genera la siguiente salida:

 <img style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_36BB41DB.png" width="480" height="147" />

Así como podéis ver, realmente el método DoBar tiene dos parámetros: un entero que debo pasar siempre y un array de cadenas que _el compilador me deja_ pasar como si fuesen N parámetros, pero que realmente es un array de N cadenas. De hecho, incluso eso es válido:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    foo.DoBar(10, <span style="color: blue">new</span> <span style="color: blue">string</span>[] { <span style="color: #a31515">"hola"</span>, <span style="color: #a31515">"adios"</span> });
  </p></p>
</div>

En resumen, el uso de la palabra clave _params_ me permite pasar un array como si fuesen N parámetros, pero no me obliga a ello (puedo seguir pasando el array como un array). Pero recordad: es un truco del compilador 😉

¡Ah, s¡! Y tened presente que:

  1. Tan solo puedo tener un parámetro _params_ en un método… 
  2. …Y debe ser el último 

Por supuesto, si queréis que los “parámetros variables” puedan ser de cualquier tipo, podéis declarar que vuestro método recibe un params[] object:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">class</span> <span style="color: #2b91af">Foo</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">public</span> <span style="color: blue">void</span> DoBar(<span style="color: blue">int</span> baz, <span style="color: blue">params</span> <span style="color: blue">object</span>[] bazbar) { }
  </p>
  
  <p style="margin: 0px">
    }
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
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">var</span> foo = <span style="color: blue">new</span> <span style="color: #2b91af">Foo</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; foo.DoBar(10, <span style="color: #a31515">"veinte"</span>, 20, <span style="color: blue">new</span> ClaseMia());
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

En este caso, dentro de DoBar, bazbar es un array de objects con dos objetos (una cadena “veinte” y un objeto ClaseMia). Por supuesto que el método DoBar sepa de que tipo son los objetos que están dentro de bazbar es otra historia… 😉

**Tercera opción (aunque no reconoceré haberlo dicho): __arglist**

Aunque params por si solo ya es suficientemente interesante, vamos a hablar de una de esos pequeños aspectos de C# que son en general, desconocidos… Me refiero a __arglist (sí, con DOS subrayados delante, lo cual ya indica algo). El uso de esa palabra clave (pues es una palabra clave) significa “y a partir de aquí más argumentos”. En efecto, podemos declarar nuestro método DoBar:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">class</span> <span style="color: #2b91af">Foo</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">public</span> <span style="color: blue">void</span> DoBar(<span style="color: blue">int</span> baz, <span style="color: blue">__arglist</span>) { }
  </p>
  
  <p style="margin: 0px">
    }
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
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">var</span> foo = <span style="color: blue">new</span> <span style="color: #2b91af">Foo</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; foo.DoBar(10, <span style="color: blue">__arglist</span>(<span style="color: #a31515">"veinte"</span>, 20, 30, <span style="color: #a31515">"bye"</span>));
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p></p>
</div>

Bueno, antes de continuar un par de cosas:

  1. Clarísimamente (como diría Cruyff) __arglist está pensada para que **no** la usemos. Es decir se trata de una palabra clave no documentada. La verdad es que func
  
    iona desde, al menos VS2005 y supongo que funcionará en todas las versiones de C#. Pero insisto: no está documentada, no es “oficial” (de ahí el “no reconoceré haberlo dicho” del título). 
  2. Fijaos que debo invocar el método pasándole los parámetros opcionales usando también la palabra clave __arglist. 

Ahora nos queda la cuestión final: como accede DoBar a esos parámetros opcionales. Eso no es como params que tenemos un array. No. Aquí si que tenemos un número arbitrario de parámetros de verdad. Pues nada, vamos a echar mano de la estructura [ArgIterator][2]:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">public</span> <span style="color: blue">void</span> DoBar(<span style="color: blue">int</span> baz, <span style="color: blue">__arglist</span>)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">var</span> args = <span style="color: blue">new</span> <span style="color: #2b91af">ArgIterator</span>(<span style="color: blue">__arglist</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">while</span> (args.GetRemainingCount() > 0)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #2b91af">Console</span>.WriteLine(<span style="color: #a31515">"Param {0} valor {1}"</span>,
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #2b91af">Type</span>.GetTypeFromHandle(args.GetNextArgType()).Name,
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #2b91af">TypedReference</span>.ToObject(args.GetNextArg()).ToString());
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

El uso de ArgIterator nos permite iterar sobre la lista de argumentos. Por cada argumento básicamente obtenemos:

  * Su valor, pero a través de un objeto [TypedReference][3] que se puede convertir a un object usando el método estático ToObject de la clase TypedReference. 
  * Su tipo, pero a través de un objeto [RuntimeTypeHandle][4] que se puede convertir a un Type a través del método estático GetTypeFromHandle de la clase Type. 

Probablemente estés pensando que no vale la pena usar \_\_arglist, ArgIterator y todo este coñazo de TypedReference y RuntimeTypeHandle y tendrás razón. Por algo la palabra clave \_\_arglist no está documentada. El hecho de que exista tiene que ver más con P/Invoke que no con una necesidad propiamente dicha del lenguaje.

Y bueno… eso es todo! 😛 Espero que os haya sido interesante… 😉 

Saludos!

 [1]: http://geeks.ms/blogs/etomas/archive/tags/c_2300_+basico/default.aspx
 [2]: http://msdn.microsoft.com/es-es/library/system.argiterator.aspx
 [3]: http://msdn.microsoft.com/es-es/library/system.typedreference.aspx
 [4]: http://msdn.microsoft.com/es-es/library/system.runtimetypehandle.aspx