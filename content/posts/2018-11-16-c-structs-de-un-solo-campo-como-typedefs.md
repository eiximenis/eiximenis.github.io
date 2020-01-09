---
title: 'C#: Structs de un solo campo como typedefs'
description: 'C#: Structs de un solo campo como typedefs'
author: eiximenis

date: 2018-11-16T09:21:26+00:00
geeks_url: /?p=2223
geeks_ms_views:
  - 829
categories:
  - .net
  - 'C#'

---
No hace mucho me preguntaba si usar structs de un solo campo tenía alguna penalización respecto a usar, simplemente, una variable del tipo del campo. Es decir, me preguntaba si tener:

<pre class="EnlighterJSRAW" data-enlighter-language="null">struct Sint {
   public int value;
}</pre>

Tenía alguna penalización al respecto de usar, simplemente, una variable _int_.
  
A nivel de memoria sospechaba que no: una struct ocupa lo mismo que la suma de todos sus campos, más los _paddings_ que se agregan para que los campos estén alineados, más el _padding_ final que se agrega para que, en el caso de un array, los elementos estén alineados. En el caso de una estructura que solo tenga un _int_ esos _paddings_ no existen (*):

<pre class="EnlighterJSRAW" data-enlighter-language="null">namespace PaddingTest
{
    struct NoPadding
    {
        public int i1;
    }
    struct Padding
    {
        public bool b1;
    }
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine($"{sizeof(int)} vs {Marshal.SizeOf&lt;NoPadding&gt;()}");
            Console.WriteLine($"{sizeof(bool)} vs {Marshal.SizeOf&lt;Padding&gt;()}");
            Console.ReadLine();
        }
    }
}</pre>

Si ejecutas este código, lo más habitual es que veas la primera línea &#8220;4 vs 4&#8221; y la segunda línea &#8220;1 vs 4&#8221;. Eso es porque, a pesar de que un bool ocupa un solo byte, una estructura que contenga un bool ocupa 4 bytes, debido al _padding_ para alinear la memoria. Esos alineamientos son importantes, ya que no tener los datos alineados supone una pérdida de rendimiento importante (el procesador debe efectuar operaciones adicionales).
  
**(*)** Cuando digo que usando un _int_ esos _paddings_ no existen me estoy refiriendo a mi arquitectura (x64). Eso es importante. Observa que no uso _sizeof_ para medir el tamaño de una _struct_. Eso es porque no se puede. Las _structs_ no tienen un tamaño predefinido (a diferencia de los tipos propios como Int32 que siempre son 32 bits). Eso es debido, precisamente a ese _padding_: el CLR puede decidir de empaquetar una estrucutra con distintos _paddings_ en función de la arquitectura sobre la cual se ejecuta el programa y por ello, al no ser el tamaño constante, no podemos usar sizeof.
  
La otra pregunta era si el rendimiento afectaba. Encontré [este post][1] donde parece ser que demuestra que no. De todos modos el test que se hace en este post se limita a comprobar si acceder a un campo de la estructura penaliza. Pero yo quiero más que eso: yo quiero ver si recibir, crear y devolver la estructura tiene impacto en el rendimiento. En definitiva quiero comprobar la diferencia entre esas dos funciones:

<pre class="EnlighterJSRAW" data-enlighter-language="null">static int IntOp(int i)
{
    var j = i + 10;
    return j;
}
static Sint SintOp(Sint i)
{
    var j = new Sint(i.Value + 10);
    return j;
}</pre>

La primera opera solo con ints y la segunda solo con mi estructura (que solo tiene un int). Así que elaboré un test que ejectuaba esas funciones 100 millones de veces:

<pre class="EnlighterJSRAW" data-enlighter-language="null">struct Sint
{
    public readonly int Value;
    public Sint(int v) =&gt; Value = v;
}
class Program
{
    static void Main(string[] args)
    {
        const int MAX_LOOP= 100000000;
        const int MAX_REPS = 10;
        long[] intTimes = new long[MAX_REPS];
        long[] sintTimes = new long[MAX_REPS];
        for (var rep = 0; rep&lt;MAX_REPS; rep++)
        {
            var (tint, tsint) = Test(MAX_LOOP);
            intTimes[rep] = tint;
            sintTimes[rep] = tsint;
        }
        for (var idx=0; idx &lt; MAX_REPS; idx++)
        {
            var diff = sintTimes[idx] - intTimes[idx];
            var percent = ((double)diff / (double)intTimes[idx]) * 100;
            Console.WriteLine($"int took {intTimes[idx]}. Sint took {sintTimes[idx]}. Diff is {diff} ({percent}%)");
        }
        Console.ReadLine();
    }
    private static (long, long) Test(int times)
    {
        var s = new Stopwatch();
        s.Start();
        for (var idx = 0; idx &lt; times; idx++)
        {
            var r = IntOp(idx);
        }
        s.Stop();
        var ms1 = s.ElapsedMilliseconds;
        s.Reset();
        s.Start();
        for (var idx = 0; idx &lt; times; idx++)
        {
            var r = SintOp(new Sint(idx));
        }
        s.Stop();
        var ms2 = s.ElapsedMilliseconds;
        s.Reset();
        return (ms1, ms2);
    }
    static int IntOp(int i)
    {
        var j = i + 10;
        return j;
    }
    static Sint SintOp(Sint i)
    {
        var j = new Sint(i.Value + 10);
        return j;
    }
}</pre>

El programa repite el test 5 veces y al final muestra los resultados. Lo ejecuté varias veces, pero el patrón es siempre muy similar. En mi máquina, compilado en Release esos fueron los resultados:

<pre class="EnlighterJSRAW" data-enlighter-language="null">int took 254. Sint took 569. Diff is 315 (124.015748031496%)
int took 244. Sint took 447. Diff is 203 (83.1967213114754%)
int took 249. Sint took 436. Diff is 187 (75.1004016064257%)
int took 236. Sint took 460. Diff is 224 (94.9152542372881%)
int took 235. Sint took 441. Diff is 206 (87.6595744680851%)
int took 251. Sint took 427. Diff is 176 (70.1195219123506%)
int took 241. Sint took 446. Diff is 205 (85.0622406639004%)
int took 241. Sint took 452. Diff is 211 (87.551867219917%)
int took 263. Sint took 472. Diff is 209 (79.467680608365%)
int took 258. Sint took 473. Diff is 215 (83.3333333333333%)</pre>

Es decir **hay una pérdida de rendimiento** que oscila entre el 70 y el 100% aproximadamente. Ahora ya se trata de juzgar de si es asumible.
  
En mi caso **decidí que sí**, a pesar de la pérdida de rendimiento, tampoco iba a tener un número de operaciones tan bestia (observad que estamos hablando de alrededor de perder un cuarto de segundo cada 100 millones de operaciones).
  
**¿Y por qué hacer esto?**
  
Esa es una buena pregunta, claro. O sea, **que ventajas tiene usar una estructura de un solo campo respecto a usar una variable**. Es decir, si ya tengo ints, para ¿qué narices quiero usar la estructura Sint?
  
Una  respuesta es **para dar semántica al código**. Te pongo mi ejemplo concreto. Tengo una función al que se le pasa un color. Hay 8 colores predefinidos, con valores de 0 a 7, **pero luego se pueden definir colores propios** a los que se les asignan índices de 8 para arriba (el número de colores que se pueden definir depende de la plataforma de ejecución). Todas las funciones de pintado esperan el color que como puedes ver es un _int_ (cuyo rango va de 0 hasta un valor máximo que no se conoce a priori).
  
Usar una estructura _Color_ con un solo campo _int_, me permite definir una semántica propia: las funciones que reciben/devuelven un _Color_ sé que trabajan con colores. **En definitiva estoy tipando el parámetro o valor de retorno**. Otra opción es **que puedo usar métodos de extensión (o en la propia estructura) para agrupar operaciones que se pueden hacer con colores**. Y además todas aquellas operaciones que se pueden hacer con _ints_ y no con colores (como dividirlos p. ej.) no se pueden realizar.
  
En mi caso, por si te pica la curiosidad, terminé con esa estructura:

<pre class="EnlighterJSRAW" data-enlighter-language="null">public static class TvColorNames
{
    public const int Black = 0;
    public const int Red = 1;
    public const int Green = 2;
    public const int Yellow = 3;
    public const int Blue = 4;
    public const int Magenta = 5;
    public const int Cyan = 6;
    public const int White = 7;
    public static IEnumerable&lt;int&gt; AllStandardColors() =&gt; new[] { Black, Red, Green, Yellow, Blue, Magenta, Cyan, White };
    public const int StandardColorsCount = 8;
}
public struct TvColor
{
    public readonly int Value;
    public TvColor(int value) =&gt; Value = value;
    public TvColor Plus(int valueToAdd) =&gt; new TvColor(Value + valueToAdd);
    public readonly static TvColor Black = new TvColor(TvColorNames.Black);
    public readonly static TvColor Red = new TvColor(TvColorNames.Red);
    public readonly static TvColor Green = new TvColor(TvColorNames.Green);
    public readonly static TvColor Yellow = new TvColor(TvColorNames.Yellow);
    public readonly static TvColor Blue = new TvColor(TvColorNames.Blue);
    public readonly static TvColor Magenta = new TvColor(TvColorNames.Magenta);
    public readonly static TvColor Cyan = new TvColor(TvColorNames.Cyan);
    public readonly static TvColor White = new TvColor(TvColorNames.White);
    public static explicit operator short(TvColor color) =&gt; (short)color.Value;
    public static bool operator ==(TvColor one, TvColor other) =&gt; one.Value == other.Value;
    public static bool operator !=(TvColor one, TvColor other) =&gt; one.Value != other.Value;
    public override bool Equals(object obj)
    {
        if (obj is TvColor) return ((TvColor)obj).Value == Value;
        if (obj is int) return (int)obj == Value;
        return base.Equals(obj);
    }
    public override int GetHashCode() =&gt; Value.GetHashCode();
}</pre>

Y el código queda un poco más legible, ya que las funciones que operan con colores se definen en base a este tipo, no en base a un _int_ como p. ej:

<pre class="EnlighterJSRAW" data-enlighter-language="null">public interface IStyleBuilder
{
    IStyleBuilder DesiredStandard(TvColor fore, TvColor back,CharacterAttributeModifiers attributes = CharacterAttributeModifiers.Normal);
    IStyleBuilder DesiredFocused(TvColor fore, TvColor back, CharacterAttributeModifiers attributes = CharacterAttributeModifiers.Normal);
}</pre>

En general **uso esta estructura para el mismo caso en que, si los valores fueran fijos, usaría un enum**. Si solo hubiese la posibilidad de usar los 8 colores, hubiese creado un enum con esos 8 valores. Pero, al haber la posibilidad de crear más colores y de no saber a priori cuantos son, me impide usar un enum y por ello uso la estructura. Pero en ambos casos la idea es la misma: **dotar el código de semántica, hacerlo más claro**.
  
Que valga la pena o no pagar el precio de rendimiento eso ya queda a criterio de cada cual.

 [1]: https://manski.net/2012/07/net-performance-primitive-type-vs-struct/