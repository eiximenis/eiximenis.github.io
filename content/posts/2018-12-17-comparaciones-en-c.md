---
title: 'Comparaciones en C#'
description: 'Comparaciones en C#'
author: eiximenis

date: 2018-12-17T07:00:28+00:00
geeks_url: /?p=2237
geeks_ms_views:
  - 1645
categories:
  - .net
  - 'C#'

---
¡Buenas!
  
Este _post_ pertenece al &#8220;[calendario de adviento de C#][1]&#8220;, y me gustaría hablaros de un tema que parece sencillo pero que bueno, esconde sus cosillas. En concreto sobre **comparaciones en C#**.
  
<!--more-->


  
Sabemos que en C# tenemos dos formas básicas de comparar objetos. Por un lado el operador de igualdad (==) y por otro el método _Equals_ que se define en Object y por lo tanto está disponible en cualquier objeto. Si te preguntas el por qué hay dos métodos de comparar y qué diferencia hay, mucha gente viene con esa respuesta:
  
_&#8220;El operador == compara referencias (es decir si dos variables apuntan al mismo objeto). Por otro lado Equals compara valores (es decir si dos objetos son idénticos)&#8221;._
  
Pero **esta afirmación es rotundamente falsa.** Ni == compara forzosamente referencias, ni Equals compara siempre valores. Simplemente, **en C# hay dos métodos de comparación porque uno (==) lo proporciona el lenguaje y otro (Equals) el Framework**. Lenguaje y Framework son cosas distintas: cualquier lenguaje que exista en .NET ofrecerá el método Equals. Que tenga también un operador de igualdad, eso ya depende del lenguaje.
  
La confusión de que _Equals_ compare valores y == referencias viene, creo yo, por una comparación con Java, donde en efecto eso es así... para las cadenas. Cualquier javero conoce el consejo elemental de &#8220;no uses == para comparar cadenas, usa siempre _equals_&#8220;. En Java, si comparas cadenas usando ==, lo que verificas es que dos variables sean la misma cadena, no que dos cadenas sean iguales. Si comparas una variable con una constante cadena, puedes terminar comparando una referencia (la variable) que contiene una cadena, con otra referencia temporal (la constante cadena) que contiene otra cadena. Aunque las cadenas sean idénticas, son dos cadenas diferentes, por lo que == devolverá _false_. Por su lado _equals_ compara el contenido de ambas cadenas.
  
Pero ¡ojo! _equals_ en Java compara contenidos de cadenas, porque la clase _java.lang.String_ redefine dicho método. Porque la **implementación base de _equals _en Java hace exactamente lo mismo que la implementación base de Equals en C#: comparar si dos objetos son el mismo.**
  
Veamos un ejemplo en C#, claro:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">class A
{
    public int Value { get; set; }
}
class Program
{
    static void Main(string[] args)
    {
        var a1 = new A() { Value = 42 };
        var a2 = new A() { Value = 42 };
        var equals = a1.Equals(a2);
    }
}</pre>

En este código el valor de la variable _equals_ es &#8220;false&#8221;, porque a1 y a2 son dos objetos distintos. Por lo tanto **Equals no compara por Valor**.
  
**Redefiniendo Equals**
  
De hecho, por defecto, Equals y == se comportan casi igual. Devolverán _true_ y _false_ en los mismos casos, pero por supuesto, dado que _Equals_ es un método virtual lo podemos redefinir:

<pre class="EnlighterJSRAW" data-enlighter-language="null">public override bool Equals(object obj)
{
    return obj is A a ? a.Value == Value : base.Equals(obj);
}</pre>

Ahora hemos redefinido _Equals_ para que compare por valor. Por lo tanto, la comparación anterior ahora devolvería _true_. Por supuesto, dado que _Equals_ acepta un _object_ como parámetro, en nuestro método Equals, verificamos que realmente el parámetro _obj_ es de tipo A. En caso contrario pasamos el control al método _Equals_ de la clase base (en nuestro caso Object).
  
**Redefiniendo ==**
  
He aquí **una de las grandes diferencias iniciales de C# con Java**, y es que en C# se puede redefinir el operador ==. Es decir, **puedo hacer que el operador == se comporte como yo quiera**. Eso es un gran poder, ya que estamos &#8220;alterando&#8221; el comportamiento habitual del lenguaje, así que bueno ya sabéis lo de la responsabilidad y todo eso. De hecho, **redefinir el operador == es lo que hace la clase System.String en C#**: lo redefine para que compare valores, no referencias. Por eso en C# podemos comparar cadenas usando == y todo nos funciona sin preocuparnos!

<pre class="EnlighterJSRAW" data-enlighter-language="null">public static bool operator ==(A a1, A a2)
{
    return a1.Value == a2.Value;
}
public static bool operator !=(A a1, A a2)
{
    return a1.Value != a2.Value;
}</pre>

Este código muestra la redefinición el operador== para la clase A para que compare por valor. Un tema al que C# obliga, es que si redefines == **debes redefinir** también el operador !=. Y observa que el operador **se define como una función estática.**
  
De hecho, usando este operador == redefinido, ahora el siguiente código devuelve true:

<pre class="EnlighterJSRAW" data-enlighter-language="null">var a1 = new A() { Value = 42 };
var a2 = new A() { Value = 42 };
var equals2 = a1 == a2;</pre>

Por lo tanto, tanto _Equals_ como == pueden comparar por valor. Todo depende de nuestras necesidades.
  
Ahora bien, **la regla de oro a tener en cuenta** sobre como se comportan _Equals_ o el operador == cuando están redefinidos y es que **en C# los operadores se seleccionan en tiempo de compilación, no de ejecución**... al revés que los métodos virtuales, que se seleccionan en tiempo de ejecución. Es decir, **cuando C# debe decidir que operador == llama, usa el tipo de las variables**. Pero **cuando C# debe decidir que método Equals llama, usa el tipo de los objetos.** La diferencia es fundamental. Así, suponiendo nuestros métodos _Equals_ y el operador == redefinido:

<pre class="EnlighterJSRAW" data-enlighter-language="null">var e1 = a1.Equals(a2);             // true
var o1 = a1 == a2;                  // true
var e2 = ((object)a1).Equals(a2);   // true
var o2 = ((object)a1) == a2;        // false</pre>

La **última comparación devuelve false_,_**_ _no true como sería de esperar. Y la razón es que ((object)a1) es una expresión que se evalúa a object, por lo que, cuando el compilador debe elegir, elige un operador== cuyo primer parámetro sea &#8220;object&#8221; y el segundo &#8220;A&#8221;. Pero ese no es nuestro operador==. Nuestro operador== se definió con ámbos parámetros a &#8220;A&#8221;.
  
**Eh! Lo acabo de comprobar con cadenas y me estás engañando!**
  
Está bien comprobar las cosas y no creértelo todo! Quizá has probado un código como ese:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">var x = ((object)"c# mola") == "c# mola";</pre>

Y esperabas que _x_ fuese _false_ y resulta que no. Pero eso no es porque no te he engañado. Eso es porque el **compilador tiene una optimización que cuando aparece dos veces la misma constante cadena, las reutiliza en un MISMO objeto**. Pero, observa lo siguiente:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">var x = ((object)"c# mola") == new string("c# mola");</pre>

Aquí estoy forzando que se cree otro objeto cadena y ahora sí que el resultado es _false_.  Por otro lado si quitamos el _casting_ a object entonces sí que x vale true, como es de esperar:

<pre class="EnlighterJSRAW" data-enlighter-language="null">var x = "c# mola" == new string("c# mola");</pre>

**El caso especial de null**
  
Con el papa hemos topado. La realidad es que _null_ no es de ningún tipo (o es de todos ellos). Cabe preguntarse, dado el siguiente código:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">if (a1 == null) { }
</pre>

¿Qué operador llamará el compilador? La realidad es que el compilador asume que el tipo de _null_ es el mismo que el del otro operando. Es decir, en este caso el operando a1 es de tipo A, por lo que el compilador asume que este _null_ es de tipo A. Por lo tanto nos llamará a nuestro operador ==, lo que nos dará un error (_NullReferenceException_ ya que usábamos .Value sin verificar que a1 o a2 podían ser _null_). En resumen: **ten en cuenta _null_ cuando redefinas tus operadores de igualdad**.
  
El hecho de que las comparaciones con null, puedan llamar a operadores== redefinidos, puede causar casos interesantes. El siguiente es el intento de crear un objeto **que siempre es distinto a cualquier otro objeto de la misma clase**:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">class Unique
{
    public Guid Id { get; }
    public Unique() =&gt; Id = Guid.NewGuid();
    public override bool Equals(object obj) =&gt; false;
    public static bool operator ==(Unique first, Unique second) =&gt; false;
    public static bool operator !=(Unique first, Unique second) =&gt; true;
}</pre>

Un objeto _Unique_ siempre es distinto a cualquier objeto _Unique_... **incluso es distinto a sí mismo**:

<pre class="EnlighterJSRAW" data-enlighter-language="null">var u1 = new Unique();
var u2 = new Unique();
var b1 = u1 == u2;          // false;
var b2 = u1 == u1;          // false!!!</pre>

Pero, recuerda que comparando con _null_ también se llamará a nuestro operador. Así, si tenemos el siguiente código:

<pre class="EnlighterJSRAW" data-enlighter-language="null">Unique u1 = null;
if (u1 != null)
{
    Console.WriteLine($"Id: {u1.Id}");
}
</pre>

Eso es lo que ocurre:
  
[<img class="alignnone size-full wp-image-2238" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/12/null-comparacion.png" alt="Comparación con null" width="844" height="311" />][2]
  
**Recibimos una _NullReferenceException_**_. _Eso es porque la comparación &#8220;_u1 != null_&#8221; que tenemos en el _if_, usa nuestro operador redefinido (que recuerda, devolvía siempre _true_). Vale, es un caso extremo, pero... que te sirva de recordatorio de hasta donde podemos llegar.
  
Por supuesto recuerda que si la variable _u1_ ya no es de tipo _Unique_ ya no se llamará a nuestro operador, o lo mismo ocurre si es _null_ el que no es de tipo _Unique. _Así, eso funciona:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">Unique u1 = null;
object o = u1;
if (o != null)
{
    Console.WriteLine($"Id: {u1.Id}");
}</pre>

Lo mismo que eso:

<pre class="EnlighterJSRAW" data-enlighter-language="null">Unique u1 = null;
if (u1 != (object)null)
{
    Console.WriteLine($"Id: {u1.Id}");
}</pre>

Pero... no te líes demasiado. Hay una manera más sencilla y natural de **comparar con null de forma segura, con independencia de cualquier operador redefinido y de la forma más eficiente**_. _Y además la conoces:

<pre class="EnlighterJSRAW" data-enlighter-language="null">Unique u1 = null;
if (!(u1 is null))
{
    Console.WriteLine($"Id: {u1.Id}");
}</pre>

De hecho, **deberíamos usar siempre _is_ (o su equivalente _as_) al comparar con _null_. **Si no usamos _is_ al comparar con _null_, se ejecuta el operador== redefinido, si existe. No sabes que hace este operador, pero piénsalo bien, ¿para qué quieres ejecutar una versión propia del operador== para comparar con _null_? Lo que quieres de verdad es comprobar que la referencia no es nula, y para ello mejor usar _is_.
  
Evidentemente, si no hay operador== redefinido, entonces &#8220;x is null&#8221; y &#8220;x == null&#8221; se comportan igual.
  
Y... ¡nada más! Os dejo algunos _posts_ como lectura complementaria a este por si os apetece seguir leyendo más sobre el tema de las comparaciones.
  
Un saludo y recordad &#8220;abrir&#8221; el resto de posts del [calendario][1]!

  * https://blogs.msdn.microsoft.com/ericlippert/2009/04/09/double-your-dispatch-double-your-fun/
  * https://www.c-sharpcorner.com/UploadFile/3d39b4/difference-between-operator-and-equals-method-in-C-Sharp/
  * https://coding.abel.nu/2014/09/net-and-equals/

 [1]: https://aspnetcoremaster.com/c%23/advientocsharp/2018/11/16/Calendario-adviento-csharp.html
 [2]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/12/null-comparacion.png