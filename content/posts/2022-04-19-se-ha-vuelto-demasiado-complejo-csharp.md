---
title: "¿Se ha vuelto demasiado complejo C#?"
author: eiximenis
description: "Cada versión de C# añade nuevas características que aumentan la riqueza del lenguaje. Pero, en contrapartida aumentan también su complejidad. Así que es lícito formularse la pregunta de si C# está deviniendo un lenguaje demasiado complejo."
date: 2022-04-19
draft: false
categories:
  - dotnet
tags: ["csharp", "opinion"]
---

Este post viene motivado a raíz de una respuesta mía a un tweet de [CampusMVP](https://twitter.com/campusMVP):

{{< tweet user="eiximenis" id="1516049083454132225" >}}

Las reacciones posteriores son las que me han motivado a escribir este post, aunque es algo que ya hace tiempo que barrunta en mi cabeza: ¿Toda la evolución que está sufriendo C# es buena o mala? ¿Se está conviertiendo en un lenguaje demasiado complejo, o por el contrario toda esa evolución nos permite escribir aplicaciones de forma más sencilla? Por supuesto este es un post más de opinión que otra cosa, ya que cualquiera puede tener su forma de pensar al respecto de como es la evolución de C#.

## Los inicios del lenguaje y su evolución temprana

En el inicio C# era "poco más" que un clon de Java. Filosóficamente ambos lenguajes eran casi idénticos: con referencias y sin punteros, orientados a objetos, usando clases con tipado estático, herencia simple y polimorfismo múltiple a base de interfaces. Es cierto que C# añadía algunas mejoras de las que Java adolecía: propiedades, delegados (y eventos), posibildad de redefinir algunos operadores, paso por referencia (usando `ref` y `out`) y quizás la más importante: la diferenciación entre "tipos por referencia" (clases) y "tipos por valor" (estructuras). Pero, vamos, dejando esas pequeñas diferencias de lado ambos lenguajes eran hermanos.

C# sufrió su primera evolución importante en 2005, con su versión 2.0. En esta versión se añadieron varias funcionalidades menores, una relativamente importante (iteradores) y otra muy importante (genéricos). En cierto modo, continuaba la misma evolución que tuvo Java, que había añadido iteradores y genéricos poco tiempo atrás. En general, todas las características se integraban bien en el lenguaje, aunque algunas (como las clases parciales) estaban más diseñadas a facilitar la vida a la propia Microsoft que a solucionar una necesidad real de los desarrolladores.

## La evolución durante la era del .NET Framework

A cada versión de .NET Framework se le solía corresponder una versión nueva del lenguaje que añadía características nuevas. Así en 2007, salió .NET Framework 3.0 que venía acompañado de una nueva versión de C#, la 3.0, que traía interesantes novedades. En mi opinión **esta versión trae quizás las novedades más importantes que ha tenido siempre el lenguaje** y que empiezan a marcar un cambio de filosofía: empezar a incorporar fundamentos sacados de la programación funcional al lenguaje.

Casi todas las novedades que incorporaba C# 3.0, se pueden resumir en una palabra: LINQ. Para soportar LINQ se modificó el lenguaje añadiendo métodos de extensión, árboles de expresión, tipos anónimos, inferencia de tipos (`var`) y expresiones lambda. Visto en retrospectiva es increíble como todas esas novedades se integran perfectamente de forma que, literalmente, si solo una de ella no existiese las demás pasarían a ser muy limitadas (quizás la excepción serían los árboles de expresión cuyo impacto en el resto de características es bastante limitado). Pero vamos, hoy en día es habitual ver código como:

```csharp
var data = _beers.Where(b => b.Price < 20).Select(b => new {Name = b.Name, TotalPrice = b.Price});
```

Este código combina todas las características anteriormente mencionadas anteriormente. Sin métodos de extensión la posibilidad de implementar métodos como `Where` o `Select` quedaba muy limitada, sin tipos anónimos no se puede crear el objeto final con los campos (`Name` y `TotalPrice`), sin expresiones lambda pasar los delegadoes usados como parámetros en `Where` y `Select` sería muy tedioso y sin inferencia de tipos es imposible declarar una variable cuyo tipo coincida con el devuelto por el método `Select`. En resumen, todas las características se integran perfectamente, a la vez que ofrecen valor por si mismas.

Junto con esas novedades C# empezó un camino (quen sigue hoy en día) para eliminar código _boilerplate_ y aparecieron las propiedades auto-implementadas y la sintaxis de inicialización de objeto.

Con la versión 3.0 creo que se puede decir que, finalmente, C# había tomado la delantera a Java (y de hecho algunas de las novedades que incorporaba el lenguaje se añadieron a Java en sucesivas versiones), pero la evolución no se para aquí. 

Llegamos a 2010 y a la versión 4.0. En mi opinión esta es una versión semi-fallida del lenguaje con novedades que aportan poco, pero es ventajista decir eso ahora. Hay que tener presente el contexto de entonces: aunque Microsoft llevaba 10 años intentando erradicarlo, la verdad es que VB6 aguantaba y seguía existiendo un ecosistema enorme de aplicaciones y componentes desarrollados para él. Bueno, quizás no _para él_, si no que existía un ecosistema enorme de componentes COM/COM+ y la verdad es que VB6 fue un lenguaje diseñado con la interoperabilidad con COM/COM+ en mente... y C# no. Aunque era posible interoperar con COM/COM+ usando C#, era pesado así que básicamente C# 4.0 añade mejoras a este punto. En especial el uso de `dynamic` que añade tipado dinámico al lenguaje. También aparecen en esa versión los valores por defecto en los parámetros y los parámetros nombrados. En esta versión se empieza a ver alguna fricción entre novedades del lenguaje y APIs ya existentes: muchas clases tenían el constructor sobrecargado (para simular parámetros opcionales), pero eso ahora ya no debería ser necesario. Pero, por compatibilidad, esos constructores seguían existiendo. Ahora como desarrollador uno debe conocer la diferencia entre esos dos códigos:

```csharp
class Foo() {
    private readonly _i;
    Foo(int i) {_i=i;}
    Foo() : this(0) {}
}
```

```csharp
class Foo() {
    private readonly _i;
    Foo(int i=0) {_i = i;}
}
```

¿Qué diferencias existen entre los códigos? ¿Hacen *exactamente* lo mismo?

Llega VS2012 y con él una nueva versión del lenguaje: C# 5.0. Esa nueva versión aporta apenas una novedad pero de gran calado: el uso de las palabras clave `async` y `await` para facilitar la creación y consumo de métodos asíncronos basados en la TPL. 

## La evolución durante .NET Core

Bueno... Microsoft se lleva la manta a la cabeza y se dedica a reimplementar .NET "desde cero" y hacerlo _cross-platform_ y así nos aparece .NET Core que hace su primera aparición con VS2015. Fueron tiempos convulsos (abandonamos el `csproj` en favor de un mucho más sencillo `project.json`, empezamos a complicar el `project.json`, abandonamos `project.json` y volvemos a un `csproj` refactorizado...), así que Microsoft tampoco tuvo mucho tiempo para dedicarle a C#. Y por ello C# 6.0 presenta un grupo de novedades de pequeño calado (aunque algunas como las interoplación de cadenas o el operador `?.` para propagar `null` se usan extensamente). Pero, gota a gota, la complejidad del lenguaje va aumentando. P. ej. ahora podemos usar expresiones lambda para definir métodos y tenemos nuevas formas de inicializar propiedades, lo que da lugar a nuevas posibles confusiones:

```csharp
class Foo
{
    public int Bar => 5;
    public int Baz() => 5;
    public int FooBar { get; } = 5;
    public int FooBaz { get => 5; }
}
```

¿Qué diferencias hay entre , `Bar`, `Baz`, `FooBar` y `FooBaz`?

**Me atrevo a decir que la mayoría de código que escriben un gran porcentaje de desarrolladores de C# es compatible con esa versión**. Aunque C# ha seguido evolucionando, la realidad es que muchos desarrolladores se han "quedado" en C# 6.0 y apenas han adoptado alguna novedad de las versiones posteriores. También es destacable el hecho de que Microsoft se empieza a dar cuenta de que `null` es algo molesto. Lamentablemente no lo podemos eliminar, pero nos empieza a dar mecanismos para facilitarnos lidiar con él. El operador `?.` es el primer paso en esta dirección.

Bueno... sigamos que ya llega 2017 y C# 7.0. Ahora ya tenemos a .NET Core funcionando a toda máquina y Microsoft tiene un nuevo foco de atención: rendimiento, rendimiento y rendimiento. Es por ello que empieza a dotar a C# de herramientas para ayudarnos a maximizar dicho rendimiento. Así, entre C# 7.0 y las revisiones posteriores (7.1, 7.2 y 7.3) nos aparecen conceptos como `ref locals`, `readonly structs`, el modificador `in`, las `ref structs` y modificaciones a `stackalloc` para poder trabajar con el nuevo tipo de datos `Span<T>`. Venga... ¿cuantos de vosotros usáis esas novedades? Y la complejidad sigue aumentando:

```csharp
readonly ref struct FooS
{
    public int X { get; }
    public FooS(int x) => X = x;
}

class FooC
{
    private int _i;
    public int One(FooS foos) => foos.X + 1;
    public int Two(in FooS foos) => foos.X + 1;
    public int Three(ref FooS foos) => foos.X + 1;
    public ref int RefM() => ref _i;
    public int NoRefM() => _i;
}
```

¿Qué diferencias hay entre `One`, `Two` y `Three`? ¿Y si `FooS` fuese una clase y no una `struct`, entonces qué diferencias habría? ¿Qué significa `readonly ref struct`? ¿Qué diferencias hay entre `RefM` y `NoRefM`?

Pero si debo elegir las novedades claves de C# 7.x serían **que convierten a las tuplas en ciudadanos de primer orden**. Aunque ya teníamos tuplas en .NET, su uso era residual. Pero gracias a C# 7.x ahora usarlas es una delicia. Aunque... de nuevo, eso añade nuevas redundancias al lenguaje (además de que la sintaxis se va complicando, observa el uso de `=` para asignar valores en el primer caso, pero el uso de `:` en el segundo):

```csharp
var x = new { Value = 42, Question = "Meaning of everything" };
var y = (Value: 42, Question: "Meaning of everything");
```

¿Qué diferencias hay entre `x` e `y`? ¿Cuando es mejor usar un objeto anónimo? ¿Cuando es mejor usar una tupla? Si me preguntas a mi, estoy plenamente convencido de que los objetos anónimos existen únicamente porque por aquel entonces no teníamos tuplas. Eso sí, las tuplas deben tener un mínimo de dos valores, no es posible crear tuplas de un solo valor (p. ej. algo como `(5)`) o bien tuplas vacías (que podrían jugar el rol de `void` sin los inconvenientes de `void`).

Además las tuplas permiten que un método devuelva varios valores a la vez, lo que convierte a `out` en redundante. Así, ¿ahora como sería mejor declarar `TryParse`?

```csharp
public static bool ClassicTryParse(string value, out int parsed) { ... }
public static (int Parsed, bool Ok) NewTryParse(string value) { ... }
```

Otra novedad de C#7 es la posibilidad de declarar funciones dentro de otras funciones. Esas funciones internas tienen acceso al ámbito local de la función que las declara:

```csharp
class Test
{
    public int Foo()
    {
        int i = 10;
        return Bar();
        int Bar() => i + 1;
    }
}
```

Nada, que llega .NET Core 3 y Microsoft lanza C# 8.0 junto con él. **Se trata de la primera versión de C# que sólo funciona con .NET Core (7.x todavía iba con .NET Framework)**. C# 8.0 trae varias novedades interesantes que me gustaría comentar. Empecemos por el uso de índices y rangos que nos permite ¡por fin! hacer algo como lo siguiente:

```csharp
var s = "hello World";
var x = s[2..5];        // x es System.String con valor "llo"
```

Ya... ¿quién necesita los métodos `substring` y similares ahora? En fin, que ahora tenemos unos tipos (`System.Index` y `System.Range`) junto con sintaxis C# para crearlos y varios tipos que los soportan. Guay, pero el problema es que, a estas alturas, arrastramos ya muchos métodos que siguen aceptando `ints` como parámetros y que no se actualizan a esos dos tipos. Por lo que, están ahí, pero no se pueden usar siempre.

La segunda novedad que me gustaría comentar es la de "default interface methods", o lo que es lo mismo, **los interfaces ahora pueden tener métodos implementados**. Eso parece algo "contra natura" pero tiene su utilidad, aunque claro a costa de una nueva redundancia. Y es que ahora... ¿qué es mejor, usar una interfaz con algunos métodos implementados o una clase abstracta? ¿Qué diferencias hay?

La tercera novedad destacable es la introducción de _pattern maching_ en el lenguaje. Aunque, para ser sinceros, eso empezó en C# 7.x es en C# 8.0 donde hay las mayores novedades en ese aspecto. De hecho cada versión de C# posterior ha ido incorporando novedades al respecto. P. ej. ahora en C# 8.0 podemos hacer lo siguiente:

```csharp
enum Size { Small, Medium, Big };
Size CalculateIntSize(int i)
{
    var y = x switch
    {
        0 => Size.Small,
        1 => Size.Medium,
        _ => Size.Big
    };
    return y;
}
```

¿Qué aporta eso a una sentencia `switch` tradicional? Pues muy poco **pero no te confundas: eso es la base de algo mucho más grande**. Las mejoras paulatinas que se van incorporando a C# en temas de pattern matching pueden convertir en innecesarias la mayoría de sentencias `if` o `switch` tradicionales. Ya, en C# 8.0 podemos hacer cosas como la siguiente:

```csharp
static Quadrant GetQuadrant(Point point) => point switch
{
    (0, 0) => Quadrant.Origin,
    var (x, y) when x > 0 && y > 0 => Quadrant.One,
    var (x, y) when x < 0 && y > 0 => Quadrant.Two,
    var (x, y) when x < 0 && y < 0 => Quadrant.Three,
    var (x, y) when x > 0 && y < 0 => Quadrant.Four,
    var (_, _) => Quadrant.OnBorder,
    _ => Quadrant.Unknown
};
```

Ahora sí que se ve la mejora, ¿verdad?. La verdad es que _pattern matching_ se trata de uno de los añadidos más espectaculares que tiene el lenguaje. Lástima que "llegue tarde", porque p. ej. estoy convencido de que la sentencia `switch` tradicional no existiría si eso hubiera estado desde los inicios.

Y la última **gran novedad** que trae C# 8.0 tiene que ver con de nuevo "ayudarnos en nuestra cruzada contra `null`" y se trata de las "nullable references". Ese es **un cambio de gran calado, que realmente cambia el comportamiento del lenguaje**. Tanto es así, que esa es una funcionalidad que se puede desactivar si no se desea (y yo os recomiendo desactivarla a no ser que empecéis proyectos nuevos). Pero, asumiendo que está activada eso significa que ahora `string s` significa que `s` puede contener cualquier objeto de tipo `string`, **pero no _debería_ contener el valor de `null`**. Si queremos indicar, explícitamente, que `null` es un valor válido para `s`, entonces su tipo no es `string` si no `string?`. Pero, en ejecución, no hay tipos distintos, tanto `string s` como `string? s` se convierten en `System.String`. Observa quen eso no ocurre cuando usamos el `?` a un tipo por valor como `int` (algo que podemos hacer desde C# 2.0). Así `int i` se convierte en `System.Int32` pero `int? i` es del tipo `System.Nullable<System.Int32>`. Así pues, más complejidad al lenguaje: tenemos tipos que no pueden ser nunca `null`, pero al usar `?` pueden valer `null`. Y por otro lado tenemos tipos que ya pueden valer `null`, pero al usar `?` le indicamos al compilador que ok, que `null` está bien... La cosa se empieza a complicar un poco. En fin, las "nullables references" es una herramienta básicamente de análisis estático de código, que ayuda a que el compilador nos avise de casos en qué deberíamos verificar si algo es `null`. P. ej. estamos asignando el valor de retorno de un método que devuelve `string?` a una `string` y no verificamos antes que dicho valor no sea `null`.

Vale, sigamos avanzando. Microsoft se olvida del nombre de ".NET Core" y unifica todo en .NET5. Y con esa nueva versión de .NET5 nos llega C# 9.0. Y con él otra pléyade de novedades, de las que quiero destacar varias.

La primera son las "init properties". Se trata de propiedades que se pueden establecer **sólo mediante la notación de inicialización de objeto**:

```csharp
class Bounds
{
    public int Width { get; init; }
    public int Height { get; init; }
}
var bounds = new Bounds() { Width = 10, Height = 20 };
bounds.Width = 200; // Error: No se puede asignar Width
```

Esa característica es genial y para ser sincero, convierte en obsoletos todos aquellos constructores que se limitan a asignar propiedades. Pero claro, al venir tarde no hay una forma "canónica" en C# de inicializar objetos. ¿Qué es mejor, usar un constructor que reciba dos parámetros y los asigne, o usar "init properties" y que el usuario use la notación de objeto?

La otra característica destacable son los `records`. Un **nuevo tipo de datos** (aunque realmente es una clase), pensada para aquellos tipos que son clases pero tienen "comportamiento de valor". Los `records` tienen muchas cosas buenas, pero a la vez añaden más duplicidad al lenguaje. Exactamente, ¿qué diferencias hay entre un `record` y una clase? ¿Y entre un `record` y una `struct`? ¿O una `readonly struct`?

 Y por supuesto, _pattern matching_ sigue ganando potencia, pero eso es a costa **de meter más complejidad y de duplicar escenarios**:

```csharp
public static bool IsLetterOrSeparator(this char c) =>
    c is (>= 'a' and <= 'z') or (>= 'A' and <= 'Z') or '.' or ',';

public static bool IsLetterOrSeparatorOld(this char c) =>
    (c >= 'a' && c <= 'z') || (c >= 'A' && c<= 'Z') || c == '.' || c ==  ',';
```

¿Cuando usar una o la otra? ¿Hay diferencias? ¿Si es así, cuales?

De todos modos, como he comentado anteriormente, _pattern matching_ va camino de "obsoletizar" la mayoría de sentencias `if` o `switch`. Fijaos lo que es posible hacer:

```csharp
static string Classify(double measurement) => measurement switch
{
    42 or 67 => "Magic numbers",
    < -4.0 => "Too low",
    > 10.0 => "Too high",
    double.NaN => "Unknown",
    _ => "Acceptable",
};
```

Bueno... y llegamos a la que es (en el momento de escribir eso), la última versión del lenguaje: C# 10 que vino de la mano de .NET6. En esta versión del lenguage se han focalizado en novedades "para escribir menos". En este ámbito entran novedades como "file scoped namespaces", es decir que ahora podemos hacer:

```csharp
namespace Foo;
class Bar {}
```

en lugar del clásico:

```csharp
namespace Foo 
{
    class Bar {}
}

```

Nos ahorramos las llaves del `namespace` (y la indentación de la clase). Otras novedades que van alineadas a "escribir menos" son los using globales (podemos añadir usar `global using` en cualquier fichero de código fuente y será como si ese `using` estuviera en todos los otros ficheros) y los "usings implícitos" (ciertos namespaces vienen ya importados de serie). También tenemos los "top-level statements" que signific que en un fichero de código fuente podemos empezar a poner código (sin declarar clase, ni namespace, ni función ni nada) y el compilador declarará la clase y la función `Main` automáticamente.

Los `records` ahora pueden ser `structs`, por lo que tenemos `record`, `record struct` `readonly record struct` y también para liarla `record class` (que es lo mismo que `record` a secas).

Bueno, claro... y _pattern matching_ sigue ganando potencia. Como muestra, un ejemplo:

```csharp
public record Point(int X, int Y);
public record Segment(Point Start, Point End);

static bool IsAnyEndOnXAxis(Segment segment) =>
    segment is { Start.Y: 0 } or { End.Y: 0 };
```

## El futuro del lenguaje

Pero claro, eso no se para aquí. C#11 llegará con varias novedades. Por supuesto, todo lo que se comenta a partir de ahora "es provisional". Pero de momento parece que algunas de las confirmadas son:

- Miembros estáticos en interfaces: Eso parece otra aberración, pero la idea es poder definir métodos estáticos en interfaces y permitir p. ej. que las interfaces declaren operadores (que en C# son métodos estáticos). Eso, combinado con genéricos, permitiría crear funciones genéricas y usar operadores con tipos genéricos, algo imposible hoy en día.
- "Raw String Literals": Oye, ya tenemos cadenas (`"x"`), cadenas _verbatim_ (`@"x\y"`), cadenas interpoladas (`$"x:{x}"`), cadenas _verbatim_ interpoladas (`$@"x\y={x\y}"`), así que otro tipo de cadenas no vendrá mal ¿verdad?. Ahora podremos tener cadenas que tengan cualquier carácter, ya que la secuencia de carácteres inicial y final de la cadena la podremos definir nosotros. Ya basta de escapar las comillas con `\"` o con `""`.
- Literales de cadena UTF8, que van a permtiir generar en tiempo de compilación, un `byte[]` a partir de sus carácteres en UTF8.

Y curiosamente una de las que generó más revuelo (el uso de `!!` para generar automáticamente la comprobación de que un argumento no es `null`) y que llegó a estar presente en las primeras versiónes _preview_ de C#11 se ha caído.

## Conclusiones

Bueno, visto lo visto... ¿qué te parece? En este post he explorado un poco las novedades más importantes (a mi juicio) de cada versión de C#. Hay más que se me han quedado fuera, pero creo que da un poco la idea de que C# se ha convertido en un lenguaje radicalmente distinto del que era cuando nació. A pesar de ello, Microsoft sigue manteniendo la compatibilidad, lo que genera muchas situaciones de duplicidad y hace que la complejidad del lenguaje siga aumentando.

Es obvio que Microsoft no va a restear C# y empezar otro lenguaje, pero si lo hiciera algunas de las cosas que me gustaría ver en él serían las siguientes:

- Inmutabilidad por defecto
- Sin constructores: inicialización de objetos sería la forma canónica de crear objetos y usar métodos estáticos en aquellos casos que sea necesario
- Sin clases abstractas (interfaces pueden definir métodos y propiedades)
- Sin objetos anónimos (tuplas pueden cubrir todos sus usos)
- Parámetros nombrados obligatorios (pienso que ayuda mucho a la legibilidad del código)
- Sin excepciones (usar tuplas u objetos tipo `Result<R, E>` para gestión de errores)
- Sin `null`. Usar construcciones tipo `Option<R>` para sustituir a `null`
- Sin `void`. Todo método devuelve algo. Usar construcciones tipo `Unit` (o tupla vacía `()`) en su lugar
- Con `enum` más potentes: que un enum pudiese contener más de un tipo, tal y como tienen por ejemplo _Rust_ o _Kotlin_. Eso, combinado con _pattern matching_ simplifica muchísimo la vida y tiene una potencia brutal.

Y tu, ¿qué opinas? ¿Cuantas de esas características usas de forma habitual? ¿Y cuales crees que aportan realmente valor?