---
title: "C# funcional con Arrow Sharp"
author: eiximenis
description: "A pesar de que C# no es un lenguaje funcional, cada vez más incluye características propias de los lenguajes funcionales. En este post te presento ArrowSharp una librería que incluye pequeñas construcciones para ayudarte a implementar la programación funcional usando C#"
date: 2020-08-07
draft: false
categories:
  - general
---

[ArrowSharp](https://github.com/eiximenis/ArrowSharp) es una pequeña librería **inspirada en [Arrow-kt Core](https://arrow-kt.io/)** que ofrece algunas utilidades para ayudarte a desarrollar con un estilo más funcional usando C#. Lo mejor es verlo con un ejemplillo. Como ejemplo **voy a basarme en el que muestra Massimo Carli en [este post](https://www.raywenderlich.com/7059961-functional-programming-with-kotlin-and-arrow-more-on-typeclasses)**.

En él partiríamos de un código inicial (C# clásico) como el siguiente:

```csharp
public class FetcherException : Exception
{
    public FetcherException(Exception inner) : base(inner.Message, inner) { }
}
public class Fetcher
{
    public async Task<string> Fetch(Uri uri)
    {
        try
        {
            var sb = new StringBuilder();
            var client = new HttpClient();
            var res = await client.GetAsync(uri);
            res.EnsureSuccessStatusCode();
            var stream = await res.Content.ReadAsStreamAsync();
            using (var sr = new StreamReader(stream))
            {
                string line = null;
                do
                {
                    line = await sr.ReadLineAsync();
                    sb.AppendLine(line);
                } while (line != null);
            }
            return sb.ToString();
        }
        catch(Exception ex)
        {
            throw new FetcherException(ex);
        }
    }
}
```

Este código funciona, pero veamos como podemos mejorarlo desde el punto de vista **funcional**. El primer tema a abordar está en el propio método `Fetch`, este método está declarado como que toma una `Uri` y devuelve una cadena, pero hay un efecto colateral que la firma no menciona: el método puede lanzar una excepción. En concreto una `FetcherException`. No hay manera de que yo pueda saber este efecto colateral si no es leyendo el código: **la firma del método nos oculta información**.

Una manera de lidiar con esto es seguir la filosofía de lenguajes como _Go_ devolver tuplas (resultado, error):

```csharp
 public async Task<(string, FetcherException)> Fetch(Uri uri) { ... }
```

## Representando un resultado O un error: Either

Pero **esta solución también nos miente. El método `Fetch` NO devuelve un par `(string, FetcherException)`**. Este método o bien devuelve una cadena o bien una excepción, pero nunca ambos. Aquí es donde podemos introducir el tipo `Either` que incorpora ArrowSharp. La clase `Either<E,R>` representa un resultado de tipo `E` o un resultado de tipo `R` pero nunca ambos:

```csharp
public async Task<Either<FetcherException, string>> Fetch(Uri uri) { ... }
```

Observa que he intercambiado el orden de los tipos. Eso es porque **el tipo `Either` está sesgado hacia la derecha**: el tipo de la derecha se considera el resultado "más probable" (o el "no error" si prefieres). Usando `Either` el código nos quedaría así:

```csharp
public async Task<Either<FetcherException, string>> Fetch(Uri uri)
{
    try
    {
        var sb = new StringBuilder();
        var client = new HttpClient();
        var res = await client.GetAsync(uri);
        res.EnsureSuccessStatusCode();
        var stream = await res.Content.ReadAsStreamAsync();
        using (var sr = new StreamReader(stream))
        {
            string line = null;
            do
            {
                line = await sr.ReadLineAsync();
                sb.AppendLine(line);
            } while (line != null);
        }
        return Either.Right<FetcherException, string>(sb.ToString());
    }
    catch (Exception ex)
    {
        return Either.Left<FetcherException, string>(new FetcherException(ex));
    }
}
```

> Se usa `Either.Right` o `Either.Left` para construir un `Either`. Lamentablemente C# no puede inferir todos los tipos genéricos, por lo que toca pasarlos. Es un poco fastidio, pero es lo que hay :(

Lo interesante, pero es el **uso que hacemos de Fetch**. Antes debíamos usar un `try/catch` para capturar la posible `FetcherException` pero ahora el resultado es siempre un `Either`. Así podemos pensar en un código como el siguiente:

```csharp
var either = await fetcher.Fetch(new Uri("https://www.google.com"))
Console.WriteLine(either.Right);
```

¡Ojo! Ese código compila, pero no es correcto. Y es que nos estamos lanzando a la piscina! ¿Qué ocurre si no hay resultado porque ha habido un error? En este caso el `Either` contiene un valor de tipo `FetcherException`. Es por ello que dado un `Either<E,R>` las propiedades `Left` y `Right` no son de tipo `E` o `R` como uno puede presuponer rápidamente. En su lugar, la propiedad `Left` es de tipo `Option<E>` y la propiedad `Right` es de tipo `Option<R>`. ¿Y qué es `Option`?

## Representando un valor opcional: Option

El tipo **`Option` es otro tipo de ArrowSharp que representa un valor de un tipo `T` o la ausencia de él**. Es como `null` pero sin los problemas de `null`. Así, la propiedad `Right` de `Either` nos devolverá un `Option` que contiene el resultado derecho o nada si no lo hay. Así, en lugar de usar `either.Right` directamente podríamos hacer lo siguiente:

```csharp
var either = await fetcher.Fetch(new Uri("https://www.google.com"));  
Console.WriteLine(either.Right.GetOrElse("ERROR!"));
``` 

Usamos el método `GetOrElse` de `Option` para obtener un valor si lo hubiera o un valor por defecto en caso de qué no. Por supuesto, también podemos usar _pattern matching_:

```csharp
var either = await fetcher.Fetch(new Uri("https://www.google.com"));
var s = either.Type switch
{
    EitherType.Left => either.FoldLeft("", e => e.Message),
    _ => either.Fold("", v => v)
};  
Console.WriteLine(s);
```

Este código usa la propiedad `Type` que nos devuelve un enum `EitherType` que nos dice si el `Either` tiene resultado izquierdo o derecho. En el caso que tenga resultado izquierdo usa el mçetodo `FoldLeft` que convierte el resultado izquierdo a otro resultado (del mismo u otro tipo). En nuestro caso pasamos de `FetcherException` a `string`, mientras que si el `Either` tiene resultado derecho se usa el método `Fold`.

> Es lo que he comentado antes: la clase `Either` está sesgada a la derecha. Por eso `Fold` (sin sufijo) actua sobre el resultado de la derecha y debemos usar `FoldLeft` para actuar sobre el resultado izquierdo.

En este caso concreto, incluso podríamos haber simplificado el código, usando una sobrecarga de `Fold` que actúa sobre el resultado que exista:

```csharp
var s = either.Fold(e => e.Message, v => v);
```

## Trabajando con Eithers y Options: Sequence

`Sequence<T>` es otro tipo de ArrowSharp que representa una lista de valores. De hecho, implementa `IEnumerable<T>` y no ofrece apenas ningún método adicional. La clave está en que **`Sequence` entiende los tipos `Either` y `Option`** y no agrega ningún `Either` que tenga resultado izquiero o ningún `Option` vacío. 

Eso lo puedes ver fácilmente con ese código:

```csharp
var urls = new[] { new Uri("https://www.google.com"), new Uri("https://www.google.invalid"), new Uri("http://www.microsoft.com") };
var results = await Sequence.OfAsync(urls.Select(u => fetcher.Fetch(u)));
```

La variable `results` contiene una `Sequence<string>` que contiene dos elementos. Solo dos elementos en lugar de tres, porque hay una URL que es inválida y con la que el método `Fetch` devuelve un `Either` con resultado izquierdo. Ese `Either` es ignorado.

Para crear una `Sequence` se usa siempre `Sequence.Of` y puedes crear una `Sequence` de tipos `T` a partir de:

1. Un `IEnumerable<T>`, en este caso la sequencia contendrá los mismos valores, **excepto los `null` que son filtrados**
2. Un `IEnumerable<Option<T>>`, en este caso la sequencia contendrá los valores (de tipo `T`) de los `Option` que tengan valor (los `Option` vacíos se filtran).
3. Un `IEnumerable<Either<L,T>>`, en este caso la secuencia contendrá los valores (de tipo `T`) de aquellos `Either` que tengan valor derecho (los que tengan valor izquierdo son ignorados)

> `Sequence` hace "_unwrap_" de `Either` y de `Option`. Eso significa que a partir de un enumerable de `Option<T>` lo que obtienes es una `Sequence<T>` (no una `Sequence<Option<T>>`) en la cual los `Option` vacíos han sido filtrados. **Recuerda que `Secuence` es la representación de una sequencia de elementos y los `Option` vacíos no se consieran elementos válidos**. Lo mismo ocurre con `Either`: dada una colección de `Either<L, T>` obtienes una `Sequence<T>` donde los `Either` que tienen resultado izquierdo están filtrados.

Existen versiones asíncronas que trabajan con `IEnumerable<Task>` como la he usado en el ejemplo (que trabaja con `IEnumerable<Task<Either<L, T>>>`).

El problema con el código anterior es que tenemos solo los resultados correctos, pero hemos perdido la información de que hay una URL que ha generado un error. Si eso ya nos va bien, pues perfecto, pero... ¿como podemos mantener esa información? Para ello tenemos que combinar la lista de URLs que teníamos con los distintos `Either` que obtenemos para generar una lista de objetos (de otro tipo) que contenga la información necesaria. El método `Fold` de `Either` nos permite transformar el `Either` y el método `Zip` de LINQ hace la combinación entre la lista de URLs y la de Eithers:

```csharp
var data = results.Zip(urls)
    .Select(i => i.First.Fold(
        e => new { Ok = false, Content = e.Message, Url = i.Second }, 
        c => new { Ok = true, Content = c, Url = i.Second }
    ));
```

En data tenemos una lista de objetos (de un tipo anónimo), donde:

1. Si el `Either` tenía resultado derecho (de tipo `string`), el valor de `Ok` será `true`, el de `Content` la propia cadena y el de `Url` la Url.
2. Si el `Either` tenía resultado izquierdo (de tipo `FetcherException`), el valor será `false`, el de `Content` el mensaje de error y como antes en `Url` tendremos la Url.

## Option y Either son monads

Los tipos `Option` y `Either` ofrecidos por ArrowSharp se comportan como _monads_. Para describir lo qué es un _monad_ hay dos maneras. La primera, matemáticamente impecable pero completamente inútil para que nadie la entienda (pero que puedes usar si quieres pecar de petulante) dice que un monad simplemente es un monoide en la categoría de los endofunctores. Como digo esa definición no sirve para nada, así que usaré otra mucho más práctica, completamente sui generis, pero que espero que entiendas a la primera:

Un _monad_ es un envoltorio para tipos X que es capaz de transformarse al mismo tipo de envoltorio pero para tipos Y.

A grandes rasgos eso significa que `Option<T>` es un _monad_ porque puedes transformar un `Option<T>` a un `Option<T'>` y lo mismo aplica a `Either`. El método que realiza esa transformación se llama `Map`:

```csharp
Option.Some(10).Map(i => i.ToString());
```

El método `Option.Some` crea un `Option` con el valor indicado (en este caso un `Option<int>`) y el método `Map` lo transforma un `Option<string>`. En este caso el tipo de envoltorio es `Option` (no se modifica usando `Map`, pasamos de un `Option` a otro `Option`) pero el tipo de datos envuelto si que lo hace (pasamos de `int` a `string`). Esa transformación debe respetar las casuísticas del envoltorio a la que se aplica. P. ej. el siguiente código funciona correctamente y no genera error alguno:

```csharp
var result = Option.Some((string)null).Map(s => s.Length);
```

La clave ahí es que el método `Option.Some` **entiende que `null` no es un valor válido**. Así que en lugar de un _Some_ (así llamamos a los `Option` que tienen valores), obtenemos un _None_ (un `Option` vacío). Cualquier transformación de un _None_ a otro _None_ es inocua, ya que no hay valor qué transformar (solo tipo). Así `result` es un `Option<int>` pero no tiene valor (su propiedad `Type` es `OptionType.None` y la propiedad `IsNone` vale `true`).

> `Option` y `Either` hacen _unwrap_ de si mismos, eso significa que `Option.Some(Option.Some(10))` no devuelve un `Option<Option<int>>` si no un `Option<int>`.

La forma "correcta" de crear un `Option` vacío es usando `Option.None<T>()`, pero que el método `Option.Some` entienda de `null` es para simplificar la interoperabilidad con código "clásico":

```csharp
public static Option<CustomerInfo> GetCustomer(int id)
{
    return Option.Some(LegacyGetCustomer(id));
}

private static CustomerInfo LegacyGetCustomer(int id)
{
    if (id % 2 == 0) return new CustomerInfo() { Id = id, Name = "Customer id " + id };
    else return null;
}
```

El método `GetCustomer` envuelve el método `LegacyGetCustomer` para transformar el `CustomerInfo` a un `Option<CustomerInfo>` que estará vacío si el método ha devuelto `null`. Ahora podemos llamar a `GetCustomer` y olvidarnos de `null`:

Quieres obtener solo el nombre de todos los clientes? Sencillo:

```csharp
var names = Sequence.Of(Enumerable.Range(1, 20).Select(i => GetCustomer(i).Map(c => c.Name)));
```

El resultado es una `Sequence<string>` que contiene los nombres de los 10 clientes. Observa qué ha ocurrido paso a paso:

1. Usando `Enumerable.Range` creamos un enumerable de 1..20
2. Transformamos cada valor al valor correspondiente de `GetCustomer`, lo que tendríamos un `IEnumerable<Option<CustomerInfo>>`
3. Usamos `Map` sobre cada `Option<CustomerInfo>` para transformarlo a un `Option<string>` que contenga solo el Nombre. En este punto siguen habiendo 20 _Options_ en la lista, salvo que 10 de ellos son _Nones_.
4. Usamos `Sequence.Of` que nos filtra los _Nones_ y además nos hace _unwrap_ por lo que pasamos de una colección de `Option<string>` a una colección de `string`, que contiene solo los valores válidos.

¡Ya lo ves! ¡Sin necesidad de preocuparnos de `null` en ningún momento!

## Quiero empezar a jugar con ArrowSharp

Vale... **NO ESTÁ TERMINADA así que no hay NuGet por el momento**. Espero que lo haya en breve, pero por el momento:

1. El [código fuente está en Github](https://github.com/eiximenis/ArrowSharp)
2. Debes usar el SDK de net5 para compilarla
3. De momento no usa "nullables references"... veremos.

Por el momento ArrowSharp hace multi-target a `netstandard2.1` y `net5.0`. Supongo que lo dejaré así pero está por ver.

Finalmente, todos los tipos de ArrowSharp son estructuras, no clases. Eso condiciona el diseño de la librería (p. ej. en Kotlin `None` y `Some` son tipos derivados de `Option<T>`, con lo que puedes hacer _pattern matching_ por tipo en lugar de por una propiedad. No tengo claro que todas las relaciones de herencia que hay en Kotlin se puedan establecer en C# por la diferencia entre como funcionan los genéricos en ambos lenguajes).