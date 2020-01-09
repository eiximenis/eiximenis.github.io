---
title: 'C# Básico: Covarianza en genéricos'
description: 'C# Básico: Covarianza en genéricos'
author: eiximenis

date: 2010-11-18T14:49:00+00:00
geeks_url: /?p=1542
geeks_visits:
  - 5535
geeks_ms_views:
  - 2796
categories:
  - Uncategorized

---
Muy buenas! Hacía tiempo que no escribía nada de la [serie C# Básico][1]. En esta serie voy tratando temas (sin ningún orden en particular) que considero que son fundamentos más o menos elementales del lenguaje. No es un tutorial al uso, cada post es independiente del resto y como digo no están ordenados por nada en particular.

El post de hoy nace a raíz de una pregunta que vi en los foros de msdn (<http://social.msdn.microsoft.com/Forums/es-ES/vcses/thread/daf808ed-a0aa-4e1e-88ed-64ee60cce918>), donde un usuario preguntaba porque el intentar convertir una List<LogVehiculos> a List<Log> le daba error teniendo en cuenta que LogVehiculos derivaba de Log.

Mi respuesta fue que en C# 3.5 los genéricos no son covariantes, y este post es para explicarlo todo un poco más 🙂

**Antes que nada**, covarianza y contravarianza son dos palabrejas muy molonas para explicar dos conceptos que son muy básicos pero que tienen implicaciones muy profundas. El **mejor artículo en español que he leído sobre covarianza y contravarianza** es el del _Doctor_ (maestros hay algunos, doctores muchos menos) **[Miguel Katrib][2]** que salió publicado en la [DotNetMania][3] número 62 y titulado &ldquo;La danza de las varianzas&rdquo;. Es un artículo que debe leerse con atención pero sin duda de lo mejorcito que he leído nunca. Este post no entrará ni mucho menos en la profundidad de dicho artículo, así que si os interesa el tema, ya sabeis: haceros con dicha DotNetMania. 

En este post nos vamos a centrar sólamente en la covarianza.

**Covarianza**

Llamamos covarianza a algo muy simple: Cuando permitimos sustituir un tipo D por otro tipo B. Para que eso sea posible debe cumplirse una condición: Que no haya nada que pueda hacerse con B y NO pueda hacerse con D.

Vamos a suponer que tenemos una clase Animal, de la cual deriva la clase Perro:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff;">class</span> Animal<br />{<br />   <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">void</span> Comer() { ... }<br />   <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">void</span> Dormir() { ... }<br />}<br /><br /><span style="color: #0000ff;">class</span> Perro : Animal<br />{<br />   <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">void</span> VigilarCasa() { ... }<br />}</pre>
</div>

Si tenemos un método cualquiera que devuelva un perro, nosotros podemos convertir el resultado a un animal:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">Perro ComprarPerro() { ... }<br /><span style="color: #008000;">// entonces eso es válido:</span><br />Animal animal = ComprarPerro();</pre>
</div>

_Eso_ es covarianza: el poder sustituir la clase derivada (Perro) que devuelve el método con la clase base (Animal). C# soporta covarianza entre una clase derivada y su clase base (como hacen de hecho _todos_ los lenguajes orientados a objetos).

Tiene lógica, porque fijaos que **no** hay nada que pueda hacerse con un Animal (B) que no pueda hacerse con un Perro (D): Dado que Perro deriva de Animal hereda todos sus métodos y propiedades.

Pero la covarianza se da también en más casos y algunos de ellos están soportados en C#. Veamos...

**Covarianza en delegados**

Se trata de poder asignar a un delegado que devuelve un Animal un método que devuelve un Perro:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff;">delegate</span> Animal AnimalDelegate();<br /><span style="color: #0000ff;">class</span> Program<br />{<br />    <span style="color: #0000ff;">static</span> Perro ObtenerPerro() { <span style="color: #0000ff;">return</span> <span style="color: #0000ff;">new</span> Perro(); }<br />    <span style="color: #0000ff;">static</span> Animal ObtenerAnimal() { <span style="color: #0000ff;">return</span> <span style="color: #0000ff;">new</span> Animal(); }<br />    <span style="color: #0000ff;">static</span> <span style="color: #0000ff;">void</span> Main(<span style="color: #0000ff;">string</span>[] args)<br />    {<br />        Animal animal = ObtenerPerro();<br />        AnimalDelegate ad = <span style="color: #0000ff;">new</span> AnimalDelegate(ObtenerAnimal);<br />        AnimalDelegate ad2 = <span style="color: #0000ff;">new</span> AnimalDelegate(ObtenerPerro);        <br />    }<br />}</pre>
</div>

Fijaos en la segunda declaración (ad2): Aunque el delegate está declarado para métodos que devuelven un Animal podemos usar este delegate con métodos que devuelvan un Perro. Por eso decimos que los delegates son covariantes en C#.

**Covarianza en arrays**

El siguiente código en C# funciona y es totalmente válido:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">Animal[] animales = <span style="color: #0000ff;">new</span> Perro[100];</pre>
</div>

Es decir podemos asignar un array de Perros a un array de Animales. De nuevo los arrays son covariantes en C#. Esta decisión se tomó en su día para, bueno... luego hablaremos más sobre ella 🙂

**Covarianza en genéricos**

El siguiente código **no compila** en C#:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #008000;">// error CS0029: Cannot implicitly convert type 'System.Collections.Generic.List&lt;ConsoleApplication8.Perro&gt;' to 'System.Collections.Generic.List&lt;ConsoleApplication8.Animal&gt;'</span><br />List&lt;Animal&gt; animales = <span style="color: #0000ff;">new</span> List&lt;Perro&gt;();<br /></pre>
</div>

Es por ello que decimos que los **genéricos NO son covariantes en C#**.

Y ahora viene la pregunta... ¿por que?

Bien, recordad que si yo quiero sustituir un tipo D por otro tipo B eso significa que en un objeto de tipo D _debo poder hacer cualquier cosa que haga en un objeto de tipo B_. Es decir, si hay algo, llamémosle f(), que pueda hacer para un objeto de tipo B que no pueda hacer con un objeto de tipo D, no puedo aplicar covarianza... Ya que entonces podría hacer D.f() que no sería válido (recordad que f() es válido para B y no para D).

Cojamos el caso de _List<Animal>_ y _List<Perro>_ (recordad que Perro deriva de Animal). La pregunta es... hay alqo que podemos hacer con List<Animal> y que NO podamos hacer con List<Perro>? Veamos...

  * Con List<Animal> puedo contar cuantos animales hay. Con List<Perro> también.
  * Con List<Animal> puedo obtener todos los Animales que hay. Con List<Perro> puedo obtener los Perros, pero dado que Perro deriva de Animal, si obtengo un Perro estoy obteniendo un Animal (primer ejemplo que hemos visto). Así pues ningún problema.
  * Con List<Animal> puedo añadir un Animal. Con List<Perro> puedo añadir... un Perro. Ojo que eso es importante: A List<Animal> puedo añadirle **cualquier** Animal... puede ser un Perro, puede ser un Gato. A List<Perro> no puedo añadirle cualquier animal, _debe_ ser un Perro forzosamente.

Por lo tanto ya hemos encontrado que se puede hacer con List<Animal> que _no_ pueda hacerse con List<Perro>: Añadir un Gato.

Si C# nos dejara aplicar covarianza entonces eso sería válido:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">List&lt;Animal&gt; animales = <span style="color: #0000ff;">new</span> List&lt;Perro&gt;();<br />animales.Add(<span style="color: #0000ff;">new</span> Gato());       <span style="color: #008000;">// EEehhh... estoy añadiendo un Gato a una lista de Perros?</span><br /></pre>
</div>

Por lo tanto, para evitar eso y asegurar que las listas de perros sólo tendrán perros el compilador no nos deja hacer esa conversión: Los genéricos **no** son covariantes.

**Y los arrays?** Recordáis que los arrays sí son covariantes. El siguiente código **es válido** y legal:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">Animal[] animal = <span style="color: #0000ff;">new</span> Perro[100];<br />animal[0] = <span style="color: #0000ff;">new</span> Gato(); <span style="color: #008000;">// Un Gato en una jauría de Perros!</span><br /></pre>
</div>

Si ejecutas el siguiente código obtendrás una [ArrayTypeMismatchException][4] en tiempo de ejecución. Es decir el código compila pero luego rebienta.

Alguien podría decir que hubiesen aplicado eso mismo a las Listas... dejar que fuesen covariantes y luego rebentar en tiempo de ejecución si añado un Gato a una List<Perro>. Porque no lo han hecho así? Pues porque repetir errores no es nunca una buena solución. Los arrays jamás debieron haber sido covariantes. Si los crearon así fue para dar soporte a lenguajes tipo Java dentro del CLR (Java tiene arrays covariantes). Y así estamos: un error de diseño de Java propagado a .NET. Fijaos que eso obliga a que cada vez que añadimos un elemento en un array el CLR en tiempo de ejecución deba comprobar que el elemento _realmente_ es del tipo del array. Viva la eficiencia!

**Y con todo eso... llegó el Framework 4**

Bien... Ahora analicemos el siguiente código:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff;">static</span> IEnumerable&lt;Perro&gt; JauriaDePerros()<br />{<br />    <span style="color: #0000ff;">return</span> <span style="color: #0000ff;">new</span> List&lt;Perro&gt;();<br />}<br /><span style="color: #0000ff;">static</span> <span style="color: #0000ff;">void</span> Main(<span style="color: #0000ff;">string</span>[] args)<br />{<br />    IEnumerable&lt;Animal&gt; perritos = JauriaDePerros();<br />}</pre>
</div>

Ya os lo avanzo: el siguiente código **no compila con el Framework 3.5**. Recordad: los genéricos no son covariantes y hemos visto la razón. Pero _tiene_ sentido en este caso? Hay algo que pueda hacer con un IEnumerable<Animal> y que no pueda hacer con IEnumerable<Perro>? Veamos...

  1. Con un IEnumerable<Animal> puedo obtener todos los Animales. Con un IEnumerable<Perro> puedo obtener todos los Perros, pero como hemos visto ya, los Perros los puedo ver como Animales.

Y ya está. No puedo hacer nada más con un IEnumerable<> salvo obtener sus elementos. Entonces porque no compila el código en C#? Pues bien, porque pagan justos por pecadores: En el framework 3.5 los genéricos no son covariantes. Nunca, aunque _por lógica_ pudiesen serlo.

Para tener una solución a estos casos donde la covarianza tiene sentido, debemos usar el Framework 4 (VS2010). Una de las novedades que incorpora C# en esta versión es precisamente esta: covarianza de genéricos _en según que casos_.

Veamos: la covarianza en genéricos _es segura cuando el parámetro genérico se usa sólamente de salida_. Es decir cuando _ningún método acepta ningún parámetro del tipo genérico, como mucho sólo lo devuelven_. El problema en el caso de List<> estaba en que podía añadir un Gato a una lista de Perros. Y eso es posible porque uno de los métodos de la clase List<T> es Add(T item). Es decir el tipo genérico se usa como valor de entrada a los métodos. En cambio con IEnumerable<T> hemos visto que no hay ningún problema: En un IEnumerable<T> sólo puedo obtener sus elementos, pero no puedo añadirle elementos nuevos. No hay ningún método que reciba un parámetro del tipo genérico. Como mucho hay métodos que devuelven ojetos del tipo genérico. En este caso la covarianza es segura.

Para indicar en C# 4.0 que una clase genérica es covariante respecto a su tipo genérico, usamos la palabra clave _out._ P.ej. IEnumerable<T> en C# 4.0 está definido como:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff;">public</span> <span style="color: #0000ff;">interface</span> IEnumerable&lt;<span style="color: #0000ff;">out</span> T&gt; : IEnumerable<br />{<br />    <span style="color: #008000;">// Métodos...</span><br />}</pre>
</div>

Fijaos en el uso de _out_ para indicarle al compilador: Este tipo es covariante respecto al tipo genérico T. Entonces **este código que en VS2008 no compilaba, es válido en C# 4.0**:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff;">static</span> IEnumerable&lt;Perro&gt; JauriaDePerros()<br />{<br />    <span style="color: #0000ff;">return</span> <span style="color: #0000ff;">new</span> List&lt;Perro&gt;();<br />}<br /><span style="color: #0000ff;">static</span> <span style="color: #0000ff;">void</span> Main(<span style="color: #0000ff;">string</span>[] args)<br />{<br />    IEnumerable&lt;Animal&gt; perritos = JauriaDePerros();<br />}</pre>
</div>

Por supuesto, esto sigue sin compilar:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff;">static</span> List&lt;Perro&gt; JauriaDePerros()<br />{<br />    <span style="color: #0000ff;">return</span> <span style="color: #0000ff;">new</span> List&lt;Perro&gt;();<br />}<br /><span style="color: #0000ff;">static</span> <span style="color: #0000ff;">void</span> Main(<span style="color: #0000ff;">string</span>[] args)<br />{<br />    List&lt;Animal&gt; perritos = JauriaDePerros();<br />}</pre>
</div>

Ya que List<T> no es covariante respecto al tipo genérico T (lógico, si lo fuese podría añadir un Gato a una lista de Perros).

En cambio eso **si que es correcto en VS2010**:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff;">static</span> List&lt;Perro&gt; JauriaDePerros()<br />{<br />    <span style="color: #0000ff;">return</span> <span style="color: #0000ff;">new</span> List&lt;Perro&gt;();<br />}<br /><span style="color: #0000ff;">static</span> <span style="color: #0000ff;">void</span> Main(<span style="color: #0000ff;">string</span>[] args)<br />{<br />    IEnumerable&lt;Animal&gt; perritos = JauriaDePerros();<br />}</pre>
</div>

Aunque el método JauriaDePerros() devuelve una List<Perro>, el código funciona porque:

  1. List<T> implementa IEnumerable<T>
  2. IEnumerable<T> es covariante respecto a T

En el fondo, fijaos que no hay problema: con _perritos_ lo único que puede hacerse es obtener sus elementos, así que de nuevo no hay peligro de que añada un Gato a _perritos_.

**Declaración de mis clases genéricas covariantes**

Si yo creo una clase que quiera que sea covariante con su tipo genérico, simplemente debo usar _out_. La única restricción es que **ningún método de mi clase podrá aceptar un parámetro del tipo genérico**:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff;">interface</span> Foo&lt;<span style="color: #0000ff;">out</span> T&gt;<br />{<br />    T Bar() { ... }<br />    <span style="color: #0000ff;">void</span> Baz(T t) { ... }<br />}</pre>
</div>

Este código no compila (_error CS1961: Invalid variance: The type parameter &#8216;T&#8217; must be contravariantly valid on &#8216;ConsoleApplication5.Foo<T>.Baz(T)&#8217;. &#8216;T&#8217; is covariant._). Ese mensaje de error largote lo único que quiere decir es que T es covariante, y por lo tanto no podemos aceptar parámetros de tipo T.

Finalmente tened presente que sólo las interfaces pueden declarar que su tipo genérico es covariante (las clases no).

Bueno... dejémoslo aquí. Hay otro termino ligado a la covarianza que es la contravarianza, aunque no es tan _común_ como la covarianza y quizá algún día hablemos de ella 🙂

Un saludo y recordaros lo que digo siempre en los posts de esta serie: Si tenéis temas sobre el lenguaje C# que queráis tratar, hacédmelo saber y haré lo que pueda!!!

 [1]: /blogs/etomas/archive/tags/c_2300_+basico/default.aspx
 [2]: http://miguelkatrib.sys-con.com/
 [3]: http://www.dotnetmania.com/
 [4]: http://msdn.microsoft.com/en-us/library/system.arraytypemismatchexception.aspx