---
title: 'C#9 – Type classes y extensiones'
author: eiximenis

date: 2019-06-07T10:58:11+00:00
geeks_url: /?p=2366
geeks_ms_views:
  - 851
categories:
  - 'C#'

---
Estaba yo revisando [algunas de las nuevas características que quizá incorpore C# 9][1] y me he encontrado con la [propuesta de **type classes**][2] (_shapes_ en la teminología de C#), que me parece bastante interesante y sobre la cual me gustaría hacer algunos comentarios 🙂
  
<!--more-->


  
Un _type class_ (voy a dejarlo en inglés) es a grandes rasgos la declaración de un conjunto de métodos. Parece ser que _shape_ va a ser la palabra clave apra crear un _type class_:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">public shape SGroup&lt;T&gt;
{
    static T operator +(T t1, T t2);
    static T Zero { get; }
}</pre>

A primera vista no parece que se diferencie mucho de una interfaz: un conjunto de métodos sin implementar. Pero observa que **a diferencia de una interfaz un _type class_ puede declarar métodos estáticos**. Por supuesto, puede declarar también métodos de instancia.
  
Un _type class_ no es un &#8220;tipo de datos&#8221; como sí lo es una interfaz, una clase o una estructura. No puedes declarar variables cuyo tipo sea un _type class_ ni usarlos como valores de retorno o como parámetros. Entonces... ¿para qué sirve?
  
**Restricciones de genéricos**
  
**Es imposible hacer en C# una función genérica que sume dos valores usando el operador de suma:**

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">public T Add&lt;T&gt;(T t1, T t2) =&gt; t1 + t2;</pre>

Este código no compila. Claro **que hay buenas razones para que no lo haga**: Esta función solo funciona si el operador + está definido para el tipo T. **Algunos tipos lo definen** (como Int32 o String) mientras que muchos otros no. Así, como el compilador &#8220;no sabe que es T&#8221; se cura en salud. Bien hecho.
  
Sabemos que a los tipos genéricos les podemos aplicar restricciones. Así, la función anterior podría reescribirla como:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">interface IAdd&lt;T&gt;
{
    T AddTo(T t1);
}
struct A : IAdd&lt;A&gt;
{
    public int Value { get; }
    public A(int value) =&gt; Value = value;
    public A AddTo(A a) =&gt; new A(Value + a.Value);
}
class Program
{
    public static T Add&lt;T&gt;(T t1, T t2)
        where T : IAdd&lt;T&gt; =&gt; t1.AddTo(t2);
    static void Main(string[] args)
    {
        var result = Add(new A(10), new A(20));
    }
}</pre>

Al forzar que el parámetro genérico de Add<T> debe ser un tipo que implemente IAdd<T>, entonces ya podemos usar los métodos definidos en la interfaz IAdd<T> dentro de la función Add<T> (como AddTo). Esta aproximación funciona bastante bien en algunos casos pero tiene dos problemas fundamentales:

  * No vas a poder usar nunca Add<T> con tipos que no implementen IAdd<T>: Por lo tanto olvídate de los tipos que no sean tuyos.
  * No hay acceso a métodos estáticos (de hecho las interfaces no pueden declararlos). Eso impide usar p. ej. operadores (que se definen siempre como un método estático).

Con esto en mente, volvamos a nuetro type class inicial:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">public shape SGroup&lt;T&gt;
{
    static T operator +(T t1, T t2);
    static T Zero { get; }
}</pre>

Como dije antes **eso no declara tipo alguno. Eso le indica al compilador qué cualquier clase que tenga el operador+ y una propiedad estática llamada Zero** **se puede considerar un SGroup<T>**.
  
Y **voy a poder usar SGroup<T> como restricción de tipo genérico:**

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">public static AddAll&lt;T&gt;(T[] ts) where T : SGroup&lt;T&gt;
{
    var result = T.Zero;
    foreach (var t in ts) { result = result + t; }
    return result;
}</pre>

Como puedes ver el segundo problema (uso de métodos estáticos) se ha solucionado.
  
Ahora la duda es qué ocurre con el primero. Pues la realidad es que **los tipos deben explícitamente declarar que implementan un _type class_.** Al igual que en el caso de interfaces, no &#8220;basta&#8221; con poseer los métodos (**no hay tipado estructural**).
  
Así las clases podrán **implementar un _type class_ del mismo modo que implementan una interfaz**. La diferencia está en que deberan implementar también los métodos estáticos. Obviamente, eso **no nos soluciona el primer problema**: los tipos existentes **no** implementaran el _type class_ así que nos quedamos igual. Aquí es donde entra en juego un nuevo mecanismo que se añade al lenguaje:
  
**Extensiones**
  
Las **extensiones son una mejora al mecanismo de métodos de extensión**. Actualmente ya disponemos en C# de un mecanismo para agregar métodos a un tipo existente: los métodos de extensión. A nivel de lenguaje &#8220;básico&#8221; los métodos de extensión no añaden nada nuevo: son simplemente métodos estáticos añadidos a una clase estática. Nada nuevo, salvo que el compilador &#8220;nos deja llamarlos&#8221; como si fuesen métodos del tipo base. Pues bien, se propone **dotar a las extensiones de un elemento del lenguiaje**:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">public extension IntGroup of int: SGroup&lt;int&gt;
{
 public static int Zero =&gt; 0;
}</pre>

Este código **declara una extensión llamada IntGroup**. Esta extensión **implementa SGroup<int>** (el _type class_) y se aplica sobre el tipo _int_. Observa, **y eso es importante**, que la extensión **no declara el operator+**. ¿Por qué? La razón es que el tipo int ya tiene ese operador. La **extensión simplemente extiende el tipo int para que implemente el _type class_.** El _type class_ pide un operator+ y una propiedad estática _Zero_. En este ejemplo el propio tipo proporciona el operator+ y la extensión le añade aquello que hace falta.
  
Hay que tener presente que **las extensiones pueden usarse independientemente de los _type classes_, **no es necesario que una extensión termine &#8220;implementando&#8221; un _type class_. Aunque, en muchos casos van a usarse para eso: ambas características tienen una gran convergencia. Pero las extensiones por si solo ya añaden valor, al permitir añadir propiedades y métodos estáticos a un tipo (algo imposible hasta ahora). **Por lo que he entendido las extensiones permitirán que un tipo implemente un _type class_ pero NO una interfaz**. Como hasta ahora, las interfaces deben ser implementadas por el propio tipo.
  
Como se ha dicho, las extensiones pueden añadir propiedades y métodos de instancia. En este caso, se puede usar _this_ exactamente igual que en una clase:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">public shape SComparable&lt;T&gt;
{
    int CompareTo(T t);
}
public extension IntComparable of int : SComparable&lt;int&gt;
{
    public int CompareTo(int t) =&gt; this - t;
}</pre>

También **se puede extender una interfaz para que implemente un _type class_.** Incluso, si la interfaz **ya posee** los métodos, la extensión es trivial (observa como _Comparable<T>_ queda vacía, ya que _IComparable<T>_ ya contiene los métodos de _SComparable<T>_):

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">public shape SComparable&lt;T&gt;
{
    int CompareTo(T t);
}
public interface IComparable&lt;T&gt;
{
    int CompareTo(T t);
}
public extension Comparable&lt;T&gt; of IComparable&lt;T&gt; : SComparable&lt;T&gt; ;</pre>

Cualquier tipo que implemente _IComparable<T>_, implementa tambiém _SComparable<T>_ (gracias a la extensión vacía). Igual te preguntas entonces por qué narices usar _SComparable<T>_. Pues muy fácil: si usas _IComparable<T>_ como restricción de genéricos solo aquellos tipos que lo implementan podrán ser usados y las extensiones no te podrán ayudar. Si usas el _type class_, los tipos que implementen el _type class_ los podrás usar y para el resto podrás crear una extensión.
  
**Extendiendo _type classes_**
  
Rizando el rizo **podemos crear extensiones que extiendan un _type class_. **Eso **nos permite añadir comportamiento a todos aquellos tipos que implementen (directamente o indirectamente vía una extensión)** el _type class_. No es necesario que el resultado de extender un _type class_ sea otro _type class_:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">public extension Comparison&lt;T&gt; of SComparable&lt;T&gt;
{
    public bool operator ==(T t1, T t2) =&gt; t1.CompareTo(t2) == 0;
    public bool operator !=(T t1, T t2) =&gt; t1.CompareTo(t2) != 0;
    public bool operator &gt; (T t1, T t2) =&gt; t1.CompareTo(t2) &gt;  0;
    public bool operator &gt;=(T t1, T t2) =&gt; t1.CompareTo(t2) &gt;= 0;
    public bool operator &lt; (T t1, T t2) =&gt; t1.CompareTo(t2) &lt;  0;
    public bool operator &lt;=(T t1, T t2) =&gt; t1.CompareTo(t2) &lt;= 0;
}</pre>

Esta extensión **añade todos esos operadores** a cualquier tipo que implemente SComparable<T>.
  
**Type classes en ejecución**
  
Por lo que se está diciendo, no parece que haya un soporte directo en el CLR para los _type classes_ y que casi toda la magia estará en el compilador. Eso significa que en ejecución los _type classes_ desaparecen y solo habrá interfaces y clases estáticas generadas por el compilador. Desconozco como eso afectará a _reflection_ y como &#8220;persistirá&#8221; la información de un _type class_ en un ensamblado.
  
**¿Es eso tipado estructural?**
  
El tipado estructural es un sistema de tipos en el cual la validez de un tipo como parámetro, se basa únicamente en lo que se use de dicho tipo. Cualquier tipo que tenga definidos los métodos, propiedades, operadores... que se usen, será un tipo válido. P. ej. **C++ usa tipado estructural en los _templates_ **(el equivalente a genéricos):

<pre class="EnlighterJSRAW" data-enlighter-language="cpp">template &lt;typename T, typename U&gt; T Add (T a, U b) {
   return a+b;
}</pre>

Esta función _Add_ se puede invocar con cualquier par de tipos T y U para los cuales haya definido un operator+. El **tipado estructural hace innecesarios los _type classes_ a costa de trasladar la responsabilidad al llamante**. Es decir, si en C++ intentas llamar a esta función con un par de tipos (T,U) para los cuales no haya el operator+ definido, el código no compilará. Insisto: no compilará. Tipado estructural no es duck typing, los errores los sigues teniendo en tiempo de compilación.
  
Observa que el compilador no puede asumir nada sobre los tipos T y U en el contexto de la función _Add_. Eso significa que no te puede ayudar si te equivocas en un nombre de un método de T o U y es imposible que herramientas como _intellisense_ puedan aplicarse.
  
Por lo tanto, el tipado estructural es bueno, pero viene con el precio de que el compilador no te puede ayudar mucho _dentro de la función template_ y que **no hay una manera que puedas especificar qué requiere una función**. La gente de C++ está al tanto de esa limitación del tipado estructural y por ello en C++20 se incorpora lo que se conoce como &#8220;[conceptos][3]&#8220;. Los conceptos permiten especificar restricciones de parámetros _templates_ (equivalentes a las restricciones que podremos especificar con los _type classes_) a la vez que se mantiene el tipado estructural.
  
**En resúmen**
  
Si esa característica se concreta me parece **uno avance importantísimo al lenguaje**. Los genéricos tal y como están ahora mismo sufren de grandes limitaciones que con los _type classes_ se podrán solucionar en gran medida. Por supuesto quedan varios flecos por definir y habrá que estar atentos a como evoluciona todo, pero de momento, solo el hecho de que estén pensando en ello ya es buena señal 🙂

 [1]: https://github.com/dotnet/csharplang/milestone/15
 [2]: https://github.com/dotnet/csharplang/issues/110
 [3]: http://www.stroustrup.com/good_concepts.pdf