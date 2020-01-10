---
title: 'C# Varianza en delegados'

author: eiximenis

date: 2018-02-05T15:02:39+00:00
geeks_url: /?p=2007
geeks_ms_views:
  - 1213
categories:
  - 'C#'

---
¡Buenas! A raíz de una situación en la que me he encontrado en un proyecto real (de la que luego hablaré) me he decidido a escribir este post para comentar algunas cosillas sobre varianzas en los delegados mismos.
  
Cuando hablamos de varianzas en delegados hay que contemplar dos aspectos:

  * Varianzas entre los _tipos definidos por el delegado_ y los _tipos de la función asignada al delegado_
  * Varianzas entre _el tipo del delegado _y otros tipos (en este caso _object_).
  * Las combinaciones entre esos dos puntos.

<!--more-->


  
**Los delegados son tipos por referencia**, como si fuesen una clase. Eso significa que _null_ es un valor aceptado y que no hay boxing/unboxing al convertir a/desde object. **Como todos los tipos un delegado hereda de object**:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">public delegate void MyDelegate(int i, object o);
// Luego en cualquier función
object o = new MyDelegate((i, o) =&gt; { });</pre>

Eso compila sin problemas: creamos un _MyDelegate_ vinculado a una función anónima (expresada con una expresión lambda) y asignamos el valor a una variable de tipo object.
  
Observa ahora el siguiente código:

<pre class="EnlighterJSRAW" data-enlighter-language="null">MyDelegate d = new MyDelegate((i, o) =&gt; { });</pre>

Es obvio que funciona, pero podríamos intentar simplificarlo, eliminando la creación explícita del objeto _MyDelegate_ puesto que la expresión lambda ya define una función válida:

<pre class="EnlighterJSRAW" data-enlighter-language="null">MyDelegate d = (i, o) =&gt; { };</pre>

Eso es también correcto. En resúmen, ya sea creando explícitamente el objeto delegado (primer caso) o no (segundo caso) podemos asignar una lambda a un delegado, siempre y cuando los parámetros de la lambda sean &#8220;compatibles&#8221; con la firma del delegado. Para eso, claro está, el compilador debe &#8220;inferir&#8221; los tipos de los parámetros de la lambda. En este caso asume que el parámetro _i_ es de tipo int y el parámetro _o _es de tipo object, ya que esa es la manera de cumplir con la firma del delegado.
  
Vale, entonces **podemos asignar delegados a objetos y podemos crear delegados a partir de lambdas y el compilador infiere los tipos de los parámetros**. Pero, ¿qué ocurre si intentamos asignar un delegado a una función que no se corresponde con la firma del delegado, pero donde hay varianza entre sus parámetros?
  
Es decir, imagina eso:

<pre class="EnlighterJSRAW" data-enlighter-language="null">MyDelegate d = (int i, string s) =&gt; { };</pre>

Aquí estamos forzando que el parámetro _s_ de la expresión lambda sea una string. ¿Eso es correcto? Veamos:

  * string hereda de object
  * Por lo tanto cualquier función que tenga como parámetros (int, object) le podemos pasar (int, string) sin ningún problema.
  * Por lo tanto en un delegado (int, object) podemos guardar una función (int, string). (_¡Ojo! Que este punto es el que contiene el razonamiento incorrecto. Ahora mismo lo vemos_).

A priori parece que eso debería compilar. **Pero la realidad es que no**. Este código no compila. Vas a recibir un error CS1661:_ Cannot convert lambda expression to delegate type &#8216;MyDelegate&#8217; because the parameter types do not match the delegate parameter types._
  
El problema es que hemos asumido, que dado que a cualquier función (int, object) le podemos pasar (int, string), las funciones (int, string) son &#8220;como un subtipo&#8221; de las funciones (int, object). O dicho de otro modo más técnico: **hemos asumido que, como hay covarianza entre los tipos string y object** (podemos usar strings donde se esperan objects) **habrá covarianza entre los tipos &#8220;función (int, object) y &#8220;función (int, string)&#8221;**. Y no es ese el caso.
  
**No hay manera de asignar una función (int, string) a un delegado cuya firma es (int, object)**. ¡Y con razón! Vamos a verlo con un ejemplo:
  
Eso **no funciona**:

<pre class="EnlighterJSRAW" data-enlighter-language="null">public delegate void MyStringDelegate(int i, string s);
public delegate void MyObjectDelegate(int i, object o);
// Luego en cualquier sitio...
MyStringDelegate d = (int i, string s) =&gt; { };
MyObjectDelegate t = d;
</pre>

La última línea da error, diciendo que no puede convertir un delegado (int, string) a un delegado (int, object). Da igual que la covarianza entre string y object. Aquí no aplica.
  
Podrías tener la tentación de usar _in_ para forzar la covarianza:

<pre class="EnlighterJSRAW" data-enlighter-language="null">public delegate void MyDelegate&lt;in  T&gt;(int i, T o);
MyDelegate&lt;string&gt; d = (int i, string s) =&gt; { };
MyDelegate&lt;object&gt; t = d;        // Error CS0266</pre>

Eso tampoco te va a compilar.
  
¿Y por qué no se puede? Pues muy sencillo. Imagina que cualquiera de esos dos códigos fuese posible. Entonces yo podría hacer:

<pre class="EnlighterJSRAW" data-enlighter-language="null">MyDelegate&lt;string&gt; d = (int i, string s) =&gt; { };
MyDelegate&lt;object&gt; t = d;
t(1, DateTime.Now);</pre>

¡He llamado a una función que aceptaba (int, string) y le he pasado (int, DateTime)! (De hecho le he pasado (int, object), por lo que a la práctica le puedo pasar (int, _cualquier cosa_)). ¡**Es obvio que esto no debe compilar!**
  
Claro, **el problema radica en que un delegado me permite invocar a la función que contiene**. Así, realmente lo que parece un tema de covarianza (usar strings en lugar de objects) es realmente un problema de contravarianza (usar objects en lugar de strings). Por lo tanto, a pesar de que una variable de tipo object puede contener una string, un delegado cuya firma sea (object) no puede contener una función con firma(string).
  
Es decir, sí, a cualquier función que acepte (int, object) le podemos pasar (int, string) pero eso ¡¡¡NO es asignar una función (int, string) a un delegado (int, object)!!!
  
De hecho **el uso que hemos intentado con la palabra clave _in_ nos habilita precisamente la conversión contraria**:

<pre class="EnlighterJSRAW" data-enlighter-language="null">MyDelegate&lt;object&gt; d = (int i, object s) =&gt; { };
MyDelegate&lt;string&gt; t = d;
t(1, "");</pre>

Observa como podemos asignar un _MyDelegate<object>_ a un _MyDelegate<string>_. Eso te puede sonar contra-intuitivo (a fin de cuentas una variable string no puede contener un object), pero de nuevo tiene toda la lógica si lo miras desde el punto de vista del delegado: El delegado _d_ contiene una función que acepta (int, object), por lo tanto esta función aceptará parámetros (int, string) que es precisamente lo que exige el delegado t. ¡Todo en orden!
  
En este caso decimos que el delegado _MyDelegate<T> **es contravariante respecto a T**_, que es una forma de decir que dado un MyDelegate<T> se puede asignar a un MyDelegate<U> donde U es un subtipo de T. La palabra clave _in_ habilita la contravarianza.
  
En lugar de contravarianza podemos declarar que un delegado es covariante respecto a un tipo. Es decir que dado MyDelegate<T> podemos asignarlo a un MyDelegate<U> donde U es un &#8220;supertipo&#8221; de T. Para ello usamos _out_ en lugar de _in_ al definir el delegado:

<pre class="EnlighterJSRAW" data-enlighter-language="null">public delegate void MyDelegate&lt;out  T&gt;(int i, T o);    // Error CS1961</pre>

Claro que eso no compila (si lo hiciera volveríamos al punto inicial de pasar DateTimes a funciones que esperan cadenas). De hecho para que ese funcione el parámetro genérico T debe estar **como tipo de retorno** del delegado. Tomemos p. ej. Func<T, R>. Este delegado está declarado así:

<pre class="EnlighterJSRAW" data-enlighter-language="null">public delegate R Func&lt;in T, out R&gt;(R r)</pre>

Es decir, **Func es contravariante respecto a T y covariante respecto a R**. Lo que significa que puedo asignar un Func<T,R> a cualquier Func<T&#8217;,R&#8217;> siempre que:

  * T&#8217; sea un subtipo de T
  * R&#8217; sea un supertipo de R

Así pues el siguiente código es correcto:

<pre class="EnlighterJSRAW" data-enlighter-language="null">class A { }
class B : A { }
class C : B { }
// En cualquier lugar
Func&lt;B, A&gt; f = _ =&gt; new A();
Func&lt;C, object&gt; f2 = f;</pre>

Observa como se cumplen las dos reglas:

  * C es un subtipo de B (T&#8217; es subtipo de T)
  * object es supertipo de A (R&#8217; es supertipo de R)

Eso es lógico ya que:

  * Forzando que T&#8217; sea subtipo de T, evitamos pasar como parámetro un objeto no permitido: todos los parámetros (de tipo T&#8217;) serán de un tipo que será subtipo del original.
  * Forzando que R&#8217; sea un supertipo de R, evitamos que el resultado esté en una referencia errónea. En nuestro caso el resultado &#8220;real&#8221; (tipo A) se guardará en una referencia de tipo object (supertipo de A), si invocamos la función a través de f2.

Resumiendo: es fácil que pensando rápido caigamos en confusiones y cosas que nos parecen &#8220;obvias&#8221; no lo sean tanto. Una discusión muy parecida a esta se da también en interfaces genéricas, de las que ya [hablé hace muuuucho tiempo][1] (aunque centrándome solo en la covarianza).
  
Otra gente que ha hablado en el pasado de lo mismo:

  * Rubén Fernandez: <http://charlascylon.com/2014-07-16-covarianza-y-contravarianza-en-c>
  * Fernando Machado: <https://fernandomachadopiriz.com/2010/04/13/la-explicacin-fcil-para-entender-covarianza-y-contravarianza/>

 [1]: https://geeks.ms/etomas/2010/11/18/c-bsico-covarianza-en-genricos/