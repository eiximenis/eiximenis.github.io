---
title: "EF memoria y BBDD"
author: eiximenis
description: Una entrada para repasar conceptos básicos, pero que pueden darte más de un quebradero de cabeza como no vigiles. En concreto como asegurarte de que todas tus consultas LINQ en EF se ejecutan realmente en la BBDD. ¡Vamos allá!
date: 2020-04-08T12:00:00
categories:
  - ef
---

Si tienes experiencia con Entity Framework, es posible que esta entrada no te aporte mucho, pero tras ver los mismos errores en más de un proyecto me he decidido a escribirla. En concreto se trata de **asegurarte de que todas tus queries LINQ (con EF) se ejecutan en la BBDD**.

## Evaluación en cliente

Las primeras versiones de EF (hasta, sin incluir, la 3.0), tenían la posibilidad de realizar lo que se llamaba "evaluación en cliente". Antes, recordemos vagamente lo que hace EF: debe traducir un **árbol de expresión** a una sentencia SQL. Un arbol de expresión en C# es una instancia del tipo `Expression<T>` donde `T` es un delegado. Por ejemplo `Expression<Func<int, bool>>` sería un árbol de expresión. Esos árboles se pueden evaluar en tiempo de ejecución y eso es lo que hace EF para generar el SQL. Como desarrolladores nunca creamos directamente objetos `Expression<T>`, en su lugar dejamos que lo haga el compilador por nosotros, a partir del delegado, que, usualmente, ponemos en forma de expresión lambda. Así, por poner un ejemplo, el compilador puede convertir `x => x+1` a una `Expression<T>` compatible como `Expression<Func<int, int>>`:

```csharp
Expression<Func<int, int>> expr = x => x + 1;   // OK
``` 

Lo dicho, EF usa esas expresiones para generar el SQL final, pero dado que esas expresiones se construyen a partir de los delegados que pasamos con LINQ, ya se ve que hay un posible problema: ¿qué ocurre si LINQ no sabe generar el SQL de una determinada expresión?

Para ver un ejemplo, partimos de la siguiente aplicación de consola (netcore 3):

```csharp
        static void Main(string[] args)
        {
            Console.WriteLine("Creating DbContext...");
            var lf = LoggerFactory.Create(c => c
                .AddFilter("*", LogLevel.Debug)
                .AddConsole());
            var builder = new DbContextOptionsBuilder<FooContext>()
                .UseSqlServer("Data Source=(LocalDb)\\MSSQLLocalDB;Initial Catalog=foo;Integrated Security=SSPI")
                .UseLoggerFactory(lf);
            var ctx = new FooContext(builder.Options);
            ctx.Database.EnsureCreated();
            if (!ctx.Persons.Any())
            {
                ctx.Persons.Add(new Person() { Name = "Baby Monster", Age = 2 });
                ctx.Persons.Add(new Person() { Name = "Young Monster", Age = 14 });
                ctx.Persons.Add(new Person() { Name = "Adult Monster", Age = 30 });
                ctx.Persons.Add(new Person() { Name = "Senior Monster", Age = 70 });
                ctx.SaveChanges();
            }
            Console.WriteLine("'Simple query'");
            var adults = ctx.Persons.Where(p => p.Age > 17).ToList();
        }
```

La clase `FooContext` contiene un solo DbSet de `Person`:

```csharp
    public class FooContext : DbContext
    {
        public FooContext(DbContextOptions<FooContext> options) : base(options)
        {
        }
        public DbSet<Person> Persons { get; set; }
    }
    public class Person
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public int Age { get; set; }
    }    
```

El código saca por pantalla las queries que genera EF y podemos ver el siguiente log:

```
Executing DbCommand [Parameters=[], CommandType='Text', CommandTimeout='30']
SELECT [p].[Id], [p].[Age], [p].[Name]
FROM [Persons] AS [p]
WHERE [p].[Age] > 17 
```

Vale, ¡perfecto! EF ha generado la sentencia esperada a partir de nuestra sentencia LINQ. Ahora bien, imagina que decidimos refactorizar esto y sacar este 17 feote de ahí. Así que te creas una función:

```csharp
private static bool IsAdult(Person p) => p.Age > 17
```

Y luego modificas tu código LINQ:

```csharp
var adults = ctx.Persons.Where(p => IsAdult(p)).ToList();
```

¿Qué puede salir mal? Pues bien **si usas EF 2.x o anterior, aparentemente nada**. La aplicación se sigue ejecutando sin problemas y da el mismo resultado. Pero, si ahora observas el log de EF verás lo siguiente:

```
Executing DbCommand [Parameters=[], CommandType='Text', CommandTimeout='30']
SELECT [p].[Id], [p].[Age], [p].[Name]
FROM [Persons] AS [p]
```

¡Espera! ¿Donde está el WHERE? Observa que **EF se está trayendo todos los registros de la tabla**. Bien, lo que ocurre aquí es lo siguiente: Cuando EF debe convertir tu código LINQ en SQL, se encuentra con una llamada a `IsAdult(p)` y **no sabe traducir eso a SQL**. Por lo tanto, en este punto, deja de generar SQL, y continuará la evaluación en memoria. Por lo tanto ocurrirá lo siguiente:

* Se ejecutará el resultado de traducir `Persons` (que es acceder a toda la tabla)
* El resultado se guardará en memoria en un `IEnumerable<Person>`
* En memoria se ejecutará el resto de la consulta LINQ (el `Where`).

Esa característica (de continuar consultas en memoria) es lo que llamamos "evaluación en cliente".

Si la BBDD tiene pocos registros eso apenas lo notarás, pero ahora imagina una tabla con un millón de registros...

La **evaluación en cliente es una pésima característica** que, sospecho, estuvo en las primeras versiones porque EF (especialmente EF 1.x) no era capaz de generar SQL para algunos casos casi triviales. Pero, un consejo, **si usas EF 2.x, desactívala**. Para eso puedes añadir el siguiente código al crear el `DbContextOptionsBuilder`:

```csharp
ConfigureWarnings(w => w.Throw(RelationalEventId.QueryClientEvaluationWarning))
```

Ahora cada vez que EF no pueda traducir LINQ a SQL lanzará una excepción en vez de evaluar en cliente. Lo cual es mucho mejor, porque te das cuenta de que esa consulta no se puede ejecutar en BBDD y mejor darte cuenta en tu fase de pruebas, que no porque se te tumba producción.

**En EF 3.x la evaluación en cliente está desactivada ya de serie**, por lo que sin hacer nada se te generaría la excepción. Esta decisión es un _breaking change_, pero personalmente es una gran decisión. EF 3.x es mucho más maduro y capaz de generar SQL en una gran variedad de escenarios, no hay necesidad de tener una bomba de relojería activa, como es la evaluación en cliente.

## Forzar evaluación en cliente

A veces nos interesa forzar la evaluación en cliente, simplemente porque no hay manera posible de escribir parte de la consulta de una manera traducible. Podemos forzar que EF evalue en cliente parte de una query llamando a `AsEnumerable()`:

```csharp
var adults = ctx.Persons.Where(p => p.Age > 17).
    AsEnumerable().Where(p => p.Name.StartsWith("Se")).ToList();
```

Si ahora miras el log de EF la query será:

```
Executed DbCommand (2ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
SELECT [p].[Id], [p].[Age], [p].[Name]
FROM [Persons] AS [p]
WHERE [p].[Age] > 17
```

Observa como **una vez hemos llamado a `AsEnumerable()` pasamos a evaluar en cliente** y la comprobación de que el nombre empiece por `Se` se realiza en cliente, no en la BBDD.

## Forzar la evaluación en cliente sin querer

Forzar la evaluación en cliente queriendo, está muy bien: es una opción que tenemos. **El problema es cuando la forzamos sin querer** y, como la hemos forzado, EF no nos avisa claro:

```csharp
var adults = ctx.Persons.Adults().ToList();
// ...
static class MyExtensions
{
    public static IEnumerable<Person> Adults (this IEnumerable<Person> source)
    {
        return source.Where(p => p.Age > 17);
    }
}
```

¿Todo bien, no? EF no se queja y la aplicación sigue devolviendo los resultados correctos... Pero observa el log:

```
Executed DbCommand (1ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
SELECT [p].[Id], [p].[Age], [p].[Name]
FROM [Persons] AS [p]
```

Al pasar la condición en un método de extensión sonbre `IEnumerable` se ha forzado la evaluación en cliente. A todos los efectos es como si llamaras a `.AsEnumerable()`. Pero claro, como lo has forzado, EF no te va avisar, se supone que sabe lo que estás haciendo. **He visto este error en muchas (demasiadas) ocasiones**.

Si quieres encapsular consultas en métodos separados, debes declararlos sobre `IQueryable`, no sobre `IEnumerable`:

```csharp
public static IQueryable<Person> Adults (this IQueryable<Person> source)
{
    return source.Where(p => p.Age > 17);
}
```

Declarar el método sobre `IEnumerable` funciona porque `IQueryable` hereda de `IEnumerable`, pero cuando usamos `IEnumerable` estamos usando siempre evaluación en cliente. La razón técnica es la combinación de dos aspectos de C#:

1. El _dispatch_ de los métodos de extensión es en tiempo de compilación
2. Los métodos de LINQ sobre IEnumerable, trabajan con delegados, no con expresiones.

Empecemos por el punto 2. El método `Where` de `IEnumerable` está definido así: 

```csharp
public static IEnumerable<TSource> Where<TSource>(this IEnumerable<TSource> source, Func<TSource, bool> predicate);
```

Mientras que el `Where` de `IQueryable` está definido como:

```csharp
public static IQueryable<TSource> Where<TSource>(this IQueryable<TSource> source, Expression<Func<TSource, bool>> predicate);
```

Observa que en el segundo caso el parámetro es un árbol de expresión (`Expression`) mientras que en el primero es un delegado directo. El `Where` de `IEnumerable` no le da a EF nada que analizar, no hay posibilidad de transformar un delegado a SQL. Necesitamos un árbol de expresión para eso.

Y ahora entra en juego el punto 1 anterior: los métodos de extensión se seleccionan en tiempo de compilación. Eso significa que si tenemos:

```csharp
source.Where(p => p.Age > 17);
```

Cuando el compilador genera código para llamar a `Where` lo hace **en función del tipo de la variable `source`**. Repito, en función del tipo de la variable, no del objeto referenciado por la variable. Da igual que el objeto "real" sea un `IQueryable`, si `source` es de tipo `IEnumerable`, se llamará al método de extensión `Where` definido sobre `IEnumerable`. Porque esa decisión la toma el compilador (no el CLR) y el compilador no tiene otra información que el tipo de la variable.

Por lo tanto, ojo con definir métodos sobre `IEnumerable` porque es una manera de forzar la evaluación en cliente.

Ahora bien **quiero dejar claro que el problema es que se llama al `Where` de `IEnumerable`**, no que el método de extensión trabaja sobre `IEnumerable`. Es un detalle sutil. Por ejemplo el siguiente método de extensión `Adults()` está definido sobre `IEnumerable` pero no fuerza la evaluación en cliente:

```csharp
public static IEnumerable<Person> Adults(this IEnumerable<Person> source)
{
    return source.CWhere(p => p.Age > 17);
}
public static IEnumerable<T> CWhere<T>(this IEnumerable<T> source, Expression<Func<T, bool>> predicate)
{
    if (source is IQueryable<T>)
    {
        return ((IQueryable<T>)source).Where(predicate);
    }
    return source.Where(predicate.Compile());
}
```

La clave es que `CWhere` analiza (en tiempo de ejecución) si el objeto es o no `IQueryable` difiriendo la llamada al método `Where` que toque.

Eso sí, si encadenasemos algo más después de `Adults()` lo que encadenasemos se evaluaría en cliente, porque `Adults()` devuelve una variable de tipo `IEnumerable` y pasamos a estar en el punto anterior.

## Evaluación perezosa

No confundas la evaluación en cliente con la evaluación perezosa. La evaluación perezosa significa que **hasta que no se recorran los objetos de un `IEnumerable` no se evaluará dicho `IEnumerable`**. Eso ocurre también con los `IQueryable`, así que tenemos tanto evaluación perezosa en BBDD como en cliente. Es algo inherente a .NET, no lo podemos desactivar.

Eso significa que si mi consulta LINQ es:

```chsarp
var adults = ctx.Persons.Where(p => p.Age > 17);
```

La variable `adults` contiene el resultado, **pero eso no se generará hasta que lo recorra**. Ese recorrido puede ser, con un foreach, o bien materializando el resultado (p. ej. llamando a `.ToList()` para copiarlo en una lista).

Aquí hay una diferencia super importante entre la evaluación perzosa en cliente y la evaluación perezosa en BBDD:

* La evaluación perezosa en BBDD es **ejecutar el SQL**
* La evaluación perezosa en cliente es **generar los elementos del IEnumerable a medida que se necesitan**

¿Qué quiero decir con eso? Pues que si tienes una consulta tal y como sigue (donde `Adults()` está definido sobre `IEnumerable` y por lo tanto nos fuerza la evaluación en cliente):

```csharp
Console.WriteLine("'Simple query'");
var adults = ctx.Persons.Adults();
Console.WriteLine("Iterating results");
foreach (var a in adults.Take(1))
{
    Console.WriteLine(a.Name);
}
```

El log que verás es parecido a:

```
'Simple query'
Iterating results
Executed DbCommand (1ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
SELECT [p].[Id], [p].[Age], [p].[Name]
FROM [Persons] AS [p]
Adult Monster
```

Observa que **no se ejecuta la consulta SQL hasta que empezamos a iterar**, pero claro esta consulta se trae todos los registros de la BBDD, da igual que luego hagas un `Take(1)`, porque ese `Take` es en cliente. En este caso la situación es que:

1. Empezamos a iterar
2. Se ejecuta una sola vez la consulta BBDD (se trae todos los elementos)
3. Se generan, uno a uno, los elementos del IEnumerable (en este caso solo hay uno por el `Take`).

La ventaja de la consulta perezosa es que te permite realizar las consultas LINQ donde quieras, pero no pagarás el precio hasta que las recorras o las materialices (con una llamada a `ToList()` o similar, ten presente que `AsEnumerable()` no materializa nada, por lo que continúas teniendo evaluación perezosa).

Espero que este post te haya ayudado a entender como funciona la evaluación en cliente de EF.

