---
title: Más sobre Code Contracts…

author: eiximenis

date: 2009-06-09T14:06:00+00:00
geeks_url: /?p=1454
geeks_visits:
  - 1240
geeks_ms_views:
  - 817
categories:
  - Uncategorized

---
Hola a todos... después de que Jorge (en [http://geeks.ms/blogs/jorge/archive/2009/04/26/precondiciones-y-microsoft-code-contracts.aspx][1]) yo mismo (en [http://geeks.ms/blogs/etomas/archive/2009/05/04/pexcando-errores-en-nuestro-c-243-digo.aspx][2]) comentasemos algo de Code Contracts, voy a comentar algunas cosillas más que me he encontrado con Code Contracts usándolos en un proyecto real.

<!--more-->

Aunque están en fase &ldquo;beta&rdquo;, la tecnología está lo suficientemente madura para ser usada en proyectos &ldquo;reales&rdquo;, al menos teniendo en cuenta de que las builds donde suelen activarse todos los contratos son las builds de debug. En nuestro caso tenemos activados todos los contratos en debug y sólo las precondiciones en release.

**Precondiciones**

La última versión de Code Contracts salió el 20 de mayo, y una de las principales diferencias es que la excepción de violación de contrato (ContractException) ha dejado de ser pública para ser interna. Por ello en lugar de Contract.Requires() es mejor usar Contract.Requieres<TEx>() que lanza una excepción de tipo TEx en caso de que la condición no se cumpla:

<pre class="code"><span style="color: blue">public double </span>Sqrt(<span style="color: blue">double </span>d)
{
    <span style="color: #2b91af">Contract</span>.Requires&lt;<span style="color: #2b91af">ArgumentException</span>&gt;(d &gt;= 0, <span style="color: #a31515">"d"</span>);
}</pre>

[][3]

Si la precondición no se cumple la excepción que se generará será una ArgumentException en lugar de la interna ContractException.

Recordad que fundamentalmente las precondiciones vienen a decir que nosotros como desarrolladores de una clase, no nos hacemos responsables de lo que ocurra si el cliente no cumple el contrato... por lo tanto es de cajón que el cliente _debe_ poder comprobar si su llamada cumple las precondiciones o no. O dicho de otro modo: las precondiciones de un método público deben ser todas ellas públicas. Esto **no** es correcto:

<pre class="code"><span style="color: blue">class </span><span style="color: #2b91af">Foo
</span>{
    <span style="color: #2b91af">List</span>&lt;<span style="color: blue">int</span>&gt; values;
    <span style="color: blue">public </span>Foo()
    {
        <span style="color: blue">this</span>.values = <span style="color: blue">new </span><span style="color: #2b91af">List</span>&lt;<span style="color: blue">int</span>&gt;();
    }
    <span style="color: blue">public void </span>Add(<span style="color: blue">int </span>value)
    {
        <span style="color: #2b91af">Contract</span>.Requires&lt;<span style="color: #2b91af">ArgumentException</span>&gt;
            (!values.Contains(value));
        values.Add(value);
    }
}</pre>

[][3]

Como va a comprobar el cliente que su llamada a Add cumple el contrato, si no le damos ninguna manera de que pueda validar que el parámetro que pase no esté en la lista? Lo suyo es hacer algo así como:

<pre class="code"><span style="color: blue">class </span><span style="color: #2b91af">Foo
</span>{
    <span style="color: #2b91af">List</span>&lt;<span style="color: blue">int</span>&gt; values;
    <span style="color: blue">public </span><span style="color: #2b91af">IEnumerable</span>&lt;<span style="color: blue">int</span>&gt; Values {
        <span style="color: blue">get </span>{ <span style="color: blue">return this</span>.values; }
    }
    <span style="color: blue">public </span>Foo()
    {
        <span style="color: blue">this</span>.values = <span style="color: blue">new </span><span style="color: #2b91af">List</span>&lt;<span style="color: blue">int</span>&gt;();
    }
    <span style="color: blue">public void </span>Add(<span style="color: blue">int </span>value)
    {
        <span style="color: #2b91af">Contract</span>.Requires&lt;<span style="color: #2b91af">ArgumentException</span>&gt;
            (!Values.Contains(value));
        values.Add(value);
    }
}</pre>

[][3]

**Interfaces**

Métodos que implementen métodos de una interfaz **NO** pueden añadir precondiciones. Es decir, esto no compila si teneis los contratos activados:

<pre class="code"><span style="color: blue">interface </span><span style="color: #2b91af">IFoo
</span>{
    <span style="color: blue">double </span>Sqrt(<span style="color: blue">double </span>d);
}
<span style="color: blue">class </span><span style="color: #2b91af">Foo </span>: <span style="color: #2b91af">IFoo
</span>{
    <span style="color: blue">public double </span>Sqrt(<span style="color: blue">double </span>d)
    {
        <span style="color: #2b91af">Contract</span>.Requires&lt;<span style="color: #2b91af">ArgumentException</span>&gt;(d &gt;= 0, <span style="color: #a31515">"d"</span>);
        <span style="color: blue">return </span><span style="color: #2b91af">Math</span>.Sqrt(d);
    }
}</pre>

[][3]

El compilador se quejará con el mensaje: _Interface implementations (ConsoleApplication7.Foo.Sqrt(System.Double)) cannot add preconditions._ La razón de esto es que las precondiciones deben añadirse _a nivel de interfaz_ y no a nivel de implementación, por la razón de que la interfaz es en muchos casos lo único que vamos a hacer público.

Dado que C# no nos deja meter código en una interfaz, la sintaxis para definir las precondiciones de una interfaz es un poco... curiosa: consiste en declarar una clase que **no hace nada salvo contener las precondiciones**, y usar un par de atributos (ContractClass y ContractClassFor) para vincular esta clase &ldquo;de contratos&rdquo; con la interfaz:

<pre class="code">[<span style="color: #2b91af">ContractClass</span>(<span style="color: blue">typeof</span>(<span style="color: #2b91af">IFooContracts</span>))]
<span style="color: blue">interface </span><span style="color: #2b91af">IFoo
</span>{
    <span style="color: blue">double </span>Sqrt(<span style="color: blue">double </span>d);
}
[<span style="color: #2b91af">ContractClassFor</span>(<span style="color: blue">typeof</span>(<span style="color: #2b91af">IFoo</span>))]
<span style="color: blue">class </span><span style="color: #2b91af">IFooContracts </span>: <span style="color: #2b91af">IFoo
</span>{
    <span style="color: blue">double </span><span style="color: #2b91af">IFoo</span>.Sqrt(<span style="color: blue">double </span>d)
    {
        <span style="color: #2b91af">Contract</span>.Requires&lt;<span style="color: #2b91af">ArgumentException</span>&gt;(d &gt;= 0, <span style="color: #a31515">"d"</span>);
        <span style="color: blue">return default</span>(<span style="color: blue">double</span>);
    }
}
<span style="color: blue">class </span><span style="color: #2b91af">Foo </span>: <span style="color: #2b91af">IFoo
</span>{
    <span style="color: blue">public double </span>Sqrt(<span style="color: blue">double </span>d)
    {
        <span style="color: blue">return </span><span style="color: #2b91af">Math</span>.Sqrt(d);
    }
}</pre>

[][3][][3]

La clase IFooContracts contiene sólo los contratos para la interfaz IFoo. El valor de retorno usado es ignorado (es sólo para que compile el código). Si ejecutais el código paso a paso, vereis que cuando haceis new Foo().Sqrt() se ejecutan primero los contratos definidos en IFooContracts.Sqrt y luego el código salta al método Foo.Sqrt.

Code Contracts requiere que la implementación de la interfaz sea **explícita**.

**Invariantes**

El invariante de un objeto es un conjunto de condiciones que se cumplen **siempre** a lo largo del ciclo de vida del objeto (excepto cuando es destruído). A nivel de contratos esto significa que son condiciones que se evalúan inmediatamente _después_ de cualquier método público, y que todas ellas deben cumplirse. Si alguna de ellas falla, el invariante se considera incumplido y el contrato roto.

Los invariantes se declaran en un método de la clase decorado con el atributo ContractInvariantMethodAttribute y consisten en varias llamadas a Contract.Invariant con las condiciones que deben cumplirse:

<pre class="code"><span style="color: blue">class </span><span style="color: #2b91af">Foo
</span>{
    <span style="color: blue">public int </span>Value { <span style="color: blue">get</span>; <span style="color: blue">private set</span>;}
    <span style="color: blue">public void </span>Inc()
    {
        <span style="color: blue">this</span>.Value++;
    }
    <span style="color: blue">public void </span>Dec()
    {
        <span style="color: blue">this</span>.Value--;
    }
    [<span style="color: #2b91af">ContractInvariantMethod</span>]
    <span style="color: blue">protected void </span>ObjectInvariant()
    {
        <span style="color: #2b91af">Contract</span>.Invariant(<span style="color: blue">this</span>.Value &gt; 0);
    }
}</pre>

[][3]

En este código hemos definido que el invariante de Foo es que el valor de la propiedad Value debe ser siempre mayor o igual a cero. Esta condición se evaluará al final de cualquier método público de Foo. Por lo tanto en el siguiente código:

<pre class="code"><span style="color: #2b91af">Foo </span>foo = <span style="color: blue">new </span><span style="color: #2b91af">Foo</span>();
foo.Inc();
foo.Dec();
foo.Dec();</pre>

[][3]

La **segunda** llamada a Dec() provocará una excepción de contrato.

**Métodos puros**

Un método &ldquo;Puro&rdquo; es aquel que no tiene ningún &ldquo;side-effect&rdquo;, es decir no modifica el estado de ninguno de los objetos que recibe como parámetro (incluyendo this). Un método puro se puede llamar infinitas veces con los mismos parámetros y se obtendran siempre los mismos resultados.

El código que se ejecuta para evaluar los contratos debe ser código puro. La razón principal es que los contratos pueden desactivarse en función del tipo de build, por lo que añadir código que no sea puro puede causar que el comportamiento cambie en función de si los contratos están o no habilitados. Si llamamos a un&nbsp; método NO PURO desde un contrato nos va a salir un warning. Fijaos en el siguiente código:

<pre class="code">[<span style="color: #2b91af">ContractClass</span>(<span style="color: blue">typeof</span>(<span style="color: #2b91af">IFooContracts</span>))]
<span style="color: blue">interface </span><span style="color: #2b91af">IFoo
</span>{
    <span style="color: #2b91af">IEnumerable</span>&lt;<span style="color: blue">int</span>&gt; Values { <span style="color: blue">get</span>;}
    <span style="color: blue">void </span>Bar(<span style="color: blue">int </span>value);
}
[<span style="color: #2b91af">ContractClassFor</span>(<span style="color: blue">typeof</span>(<span style="color: #2b91af">IFoo</span>))]
<span style="color: blue">sealed class </span><span style="color: #2b91af">IFooContracts </span>: <span style="color: #2b91af">IFoo
</span>{
    <span style="color: blue">private </span><span style="color: #2b91af">IEnumerable</span>&lt;<span style="color: blue">int</span>&gt; values; <span style="color: green">// Para que compile
    </span><span style="color: #2b91af">IEnumerable</span>&lt;<span style="color: blue">int</span>&gt; <span style="color: #2b91af">IFoo</span>.Values { <span style="color: blue">get </span>{ <span style="color: blue">return </span>values; } }
    <span style="color: blue">void </span><span style="color: #2b91af">IFoo</span>.Bar(<span style="color: blue">int </span>value)
    {
        <span style="color: #2b91af">Contract</span>.Requires(CheckValue(value));
    }
    <span style="color: blue">public bool </span>CheckValue(<span style="color: blue">int </span>value)
    {
        <span style="color: blue">return </span>(value % 2) == 0 && !((<span style="color: #2b91af">IFoo</span>)<span style="color: blue">this</span>).Values.Contains(value);
    }
}
<span style="color: blue">class </span><span style="color: #2b91af">Foo </span>: <span style="color: #2b91af">IFoo
</span>{
    <span style="color: blue">private readonly </span><span style="color: #2b91af">List</span>&lt;<span style="color: blue">int</span>&gt; values;
    <span style="color: blue">public </span><span style="color: #2b91af">IEnumerable</span>&lt;<span style="color: blue">int</span>&gt; Values
    {
        <span style="color: blue">get </span>{ <span style="color: blue">return this</span>.values; }
    }
    <span style="color: blue">public </span>Foo()
    {
        <span style="color: blue">this</span>.values = <span style="color: blue">new </span><span style="color: #2b91af">List</span>&lt;<span style="color: blue">int</span>&gt;();
    }
    <span style="color: blue">public void </span>Bar(<span style="color: blue">int </span>value)
    {
        <span style="color: blue">this</span>.values.Add(value);
    }
}</pre>

[][3]

Antes que nada: que no os líe la variable privada IEnumerable<int> values declarada en IFooContracts: es **simplemente** para que compile el código, pero realmente nunca es usada... Realmente nunca se instancian (ni usan) objetos de las clases de contrato: en el contexto de ejecución del método CheckValue el valor de this NO es un objeto IFooContracts, si no un objeto Foo (mirad la imagen si no me creeis :p).

[<img height="104" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_4F4B2AAF.png" alt="image" border="0" title="image" style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" />][4] 

Bueno... que me desvío del tema. A lo que íbamos: La clase de contratos para&nbsp; IFoo, define un método CheckValue que sirve para evaluar la precondición del método Bar. Si compilais os aparecerá un warning:

_Detected call to impure method &#8216;ConsoleApplication7.IFooContracts.CheckValue(System.Int32)&#8217; in a pure region in method &#8216;ConsoleApplication7.IFooContracts.ConsoleApplication7.IFoo.Bar(System.Int32)&#8217;_

Como sabe Code Contracts que mi método CheckValue no es puro? Pues simplemente porque yo no lo he declarado como tal. Para ello basta con decorarlo con el atributo Pure:

<pre class="code">[<span style="color: #2b91af">Pure</span>]
<span style="color: blue">public bool </span>CheckValue(<span style="color: blue">int </span>value)
{
    <span style="color: blue">return </span>(value % 2) == 0 && !((<span style="color: #2b91af">IFoo</span>)<span style="color: blue">this</span>).Values.Contains(value);
}</pre>

[][3]

Actualmente Code Contracts **no comprueba** que mi método que dice ser puro, lo sea de verdad... como en MS no se fían mucho de nosotros (los desarrolladores... ¿por que será? :p) están preparando mecanismos de validación de forma que cuando un método diga ser puro, lo sea efectivamente de verdad. A dia de hoy, tanto si poneis [Pure] como si no, funciona todo igual, así que el atributo Pure sirve para poco, al menos en lo que a ejecución se refiere. De todos modos creo que documentalmente es un atributo muy interesante: Indica que quien hizo el método lo hizo con la intención de que fuese puro, así que quizá debemos vigilar un poco las modificaciones que hagamos en este método... 

Personalmente me hubiese encantado que Pure formase parte de C# a nivel de palabra reservada incluso... para que el compilador nos pudiese avisar si estamos modificando algún estado de algún objeto (o sea llamando a un método no-puro) desde un método puro. Pero si no nos quieren ni dar referencias const, mucho menos todavía nos van a dar esto... 🙁

Bueno... cierro el rollo aquí... han quedado bastantes cosillas de contracts pero espero que al menos esto os ayude y os anime a dar el paso de empezar a utilizarlos en vuestras aplicaciones, porque realmente creo que vale mucho la pena...

... y más cuando Sandcastle sea capaz de sacar la información de los contratos de cada clase en la documentación, tal y como parece ser intención de microsoft ([http://social.msdn.microsoft.com/Forums/en-US/codecontracts/thread/cb5556e9-9dc9-45ed-8016-567294236af3][5]).

Saludos!

 [1]: /blogs/jorge/archive/2009/04/26/precondiciones-y-microsoft-code-contracts.aspx "http://geeks.ms/blogs/jorge/archive/2009/04/26/precondiciones-y-microsoft-code-contracts.aspx"
 [2]: /blogs/etomas/archive/2009/05/04/pexcando-errores-en-nuestro-c-243-digo.aspx
 [3]: http://11011.net/software/vspaste
 [4]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_43248F26.png
 [5]: http://social.msdn.microsoft.com/Forums/en-US/codecontracts/thread/cb5556e9-9dc9-45ed-8016-567294236af3 "http://social.msdn.microsoft.com/Forums/en-US/codecontracts/thread/cb5556e9-9dc9-45ed-8016-567294236af3"