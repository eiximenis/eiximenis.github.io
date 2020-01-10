---
title: 'C#: Conversiones (explícitas o implícitas) e interfaces'

author: eiximenis

date: 2018-12-01T15:12:14+00:00
geeks_url: /?p=2232
geeks_ms_views:
  - 1019
categories:
  - 'C#'

---
Una de las características más útiles, aunque más potencialmente peligrosas de C# es la posibilidad de sobrecargar los operadores de conversión (_casting_) y concretamente el de conversión implícita.
  
Poder sobrecargar el operador de conversión explícita, aunque lo entiendo como una característica que agrega ortogonalidad al lenguaje, no es algo que me guste. Antes de eso prefiero crear un método AsXXX(). De hecho me parece que un cliente de mi clase encontrará más lógico un método AsXXX() que no &#8220;un casting a XXX&#8221; que debes saber que se puede hacer para hacerlo.
  
<!--more-->


  
Pero el casting implícito ya es otra historia: es especialmente útil cuando tenemos clases que son poco más que un envoltorio simple de otro tipo y queremos que usarlo en el entorno del otro tipo sea lo más sencillo posible. Aunque te pueda parecer una aberración que un tipo se convierta de forma implícita (es decir, sin que tu hagas nada) eso es algo relativamente habitual:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">long l=10;</pre>

Ale, aquí ya tienes una conversión implícita, donde un valor de tipo _int_, se convierte automáticamente en un valor de tipo _long_.
  
Bien, sobrecargar el operador de conversión implícita tiene una limitación, y es que solo funciona con tipos concretos, no con interfaces. Lo siguiente no funciona (te copio parte de la clase para que tengas el contexto):

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">class DynamicViewport
{
    private readonly Func&lt;IViewportFactory, IViewport&gt; _vpFunc;
    private readonly IViewportFactory _viewportFactory;
    public DynamicViewport(IViewportFactory factory, Func&lt;IViewportFactory, IViewport&gt; viewportFunc)
    {
        _vpFunc = viewportFunc;
        _viewportFactory = factory;
    }
    public static implicit operator IViewport(DynamicViewport dp)
    {
        return dp._vpFunc(dp._viewportFactory);
    }
}</pre>

Observa que el código es _a priori_ coherente: __vpFunc_ es un delegado que devuelve un _IViewport_ así que por ahí no se prevee un problema.
  
Bueno **la razón por la que eso no funciona es porque está explícitamente prohibido por la especificación de C#**. Ya, pero... ¿por qué está prohibido? ¿Hay alguna razón?
  
Permitir sobrecargar el operador de casting es cuanto menos peliagudo ya que hay varios casos que nos pueden hacer confundir fácilmente. Uno es el de la pérdida de identidad de los objetos. Si, dado el siguiente código:

<pre class="EnlighterJSRAW" data-enlighter-language="null">object obj = new MyClass();
var myObj = (MyClass)obj;
var same = Object.ReferenceEquals(obj, myObj);</pre>

Os pregunto por el valor de _same_, supongo que una gran mayoría de vosotros me diréis que _same_ vale true.  El operador de casting explícito existente en el lenguaje no modifica los objetos, solo las referencias. Ahora, imaginad que el siguiente operador estuviera permitido:

<pre class="EnlighterJSRAW" data-enlighter-language="null">public static explicit operator MyClass(object o) { return new MyClass(); }</pre>

Entonces, el valor de _same_ del código anterior sería _false_. Así, que para evitar convertir el desarrollo de C# en algo parecido a la física cuántica, y que algunas conversiones modificasen la identidad y otras no, los desarrollades del lenguaje decidieron **prohibir cualquier conversión explícita o implícita siempre que ya exista una** (y la conversión de tipo base a tipo derivado existe ya en C#).
  
Bueno, pues el mismo criterio aplica a las interfaces:

<pre class="EnlighterJSRAW" data-enlighter-language="null">interface IMyClass { }
class MyClass : IMyClass { }
class OtherClass { }
class Program
{
    static void Main(string[] args)
    {
        var other = new OtherClass();
        var iFirst = (IMyClass)other;
    }
}</pre>

El siguiente código compila, ya que C# nos permite hacer _casting_ de una clase cualquiera a un interfaz y sabemos que esa conversión tiene dos posibles resultados:

  1. O bien rebienta en ejecución si OtherClass no implementa realmente IMyClass
  2. O bien iFirst es un IMyClass pero Object.ReferenceEquals(iFirst, other) es true. Es decir son &#8220;el mismo&#8221; objeto.

Es un caso análogo al de pasar de clase base a derivada, a pesar de que ser una conversión entre dos tipos que no tienen relación alguna (OtherClass y IMyClass).
  
Pues, a caso análogo, razonamiento análogo. Si pudiéramos tener una conversión (implícita o no) propia podríamos hacer eso:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">class OtherClass
{
    public static explicit operator IMyClass (OtherClass oc) =&gt; new MyClass();
}</pre>

Y entonces _other_ y _iFirst_ del código anterior **no tendrían la misma referencia**. Así que para evitar esas situaciones, los diseñadores del lenguaje decidieron cortar por lo sano.
  
Leyendo el último ejemplo igual te preguntas como narices C# lo permite (es decir, que compila). Lo copio otra vez:

<pre class="EnlighterJSRAW" data-enlighter-language="null">interface IMyClass { }
class MyClass : IMyClass { }
class OtherClass { }
class Program
{
    static void Main(string[] args)
    {
        var other = new OtherClass();
        var iFirst = (IMyClass)other;
    }
}</pre>

Yo no sé vosotros, pero a mi no me hace falta un CLR que me diga que _other_ no implementa la interfaz. Es decir:

  * Tengo una variable _other_ que es de tipo _OtherClass_
  * La clase _OtherClass_ no implementa _IMyClass_
  * Luego es de cajón que el casting fallará.

Estamos todos de acuerdo, ¿no?. ¿En qué pensaban los diseñadores del lenguaje cuando decidieron soportar esas conversiones sin sentido?
  
Ah, calla que igual pensaban en soportar eso:

<pre class="EnlighterJSRAW" data-enlighter-language="null">interface IMyClass { }
class MyClass : IMyClass { }
class OtherClass
{
}
class OtherClassChild : OtherClass, IMyClass {}
class Program
{
    static void Main(string[] args)
    {
        OtherClass other = new OtherClassChild();
        var iFirst = (IMyClass)other;
    }
}</pre>

Como podéis ver cuando tenemos interfaces por en medio, todo se complica...
  
Bueno, en fin, resumiendo, que no hay una razón técnica por la que las conversiones propias desde/a interfaces no estén permitidas. La razón es porque abren la puerta a determinados escenarios que los diseñadores del lenguaje no quisieron abrir y por ello las prohibieron.
  
Y, ya, por último, para que veáis que las conversiones no son tan sencillas, os presento el siguiente código:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">class A
{
    public int Foo { get; set; }
}
class B
{
    public static implicit operator A(B b) =&gt; new B();
}
class Program
{
    static void Main(string[] args)
    {
        A a = new B();
        var x = a.Foo;
    }
}</pre>

Qué créis que hace este código? Compila? No? Y en caso de qué compile... Qué ocurre? Y por qué?
  
Saludos!
  
PD: Por supuesto, yo no soy diseñador del lenguaje de C#, así que cuando afirmo la decisión que tomaron y las razones es porque ellos mismos así lo han comentado: https://stackoverflow.com/questions/9229928/more-on-implicit-conversion-operators-and-interfaces-in-c-sharp-again/9231325#9231325 🙂