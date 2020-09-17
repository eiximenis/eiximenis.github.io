---
title: "C#9 - Top Level Programs"
author: eiximenis
description: "Esa es una nueva característica de C# que, a priori, no aporta casi nada, pero que puede sentar bases para un futuro."
date: 2020-09-17T18:00:00
draft: false
categories:
  - csharp
---

Una de las novedades de C#9 a la que se está prestando (y probablemente con razón) menos atención es la característica denominada "Top Level Programs", lo que básicamente viene a decir que, el siguiente programa es válido en C#9:

```csharp
Console.WriteLine("Hello World!");
``` 

**Eso es todo lo que necesitas para ejecutar un _Hello World_ en C#9**. Vamos, que **ya no necesitas el método estático `Main` definido en una clase** como hasta ahora.

## ¿Como funciona esa característica?

Bueno, como te puedes imaginar es simple "azúcar sintáctico", el compilador genera la clase `Program` y el método `Main` automáticamente por nosotros. En concreto la clase generada se llama `<Program>$` y el método generado se llama `<Main>$`. En ambos casos son nombres "imposibles" de generar usando C#. El código generado es el siguiente:

```csharp
[CompilerGenerated]
internal static class <Program>$
{
  private static void <Main>$(string[] args)
  {
    Console.WriteLine("Hello World!");
  }
}
```

## Usando argumentos (args)

Como puedes observar por el código generado que he puesto arriba, puedes usar `args` sin ningún problema:

```csharp
using System.Linq;
var name = args.FirstOrDefault() ?? "World";
System.Console.WriteLine($"Hello {name}");
```

El siguiente código funciona correctamente. Mostrando `Hello edu` si lo invocas usando `dotnet run edu` p.ej.

El código compilado se corresponde a los `using` y luego la clase `<Program>$` y el método `<Main>$`. Nada especial aquí.

## Definiendo funciones

El siguiente código es también un programa C#9 completamente válido:

```csharp
using System.Linq;
var name = GetName(args.FirstOrDefault());
System.Console.WriteLine($"Hello {name}");

string GetName(string name)
{
    return name ?? "World";
}
```

Aquí las cosas se ponen más interesantes, **ya que el compilador tendría dos opciones para generar ese código**:

1. Generar la función estática `GetName` en la clase `<Program>$`
2. Generar una [función local](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/local-functions?WT.mc_id=AZ-MVP-4039791) a `<Main>$`.

Bien, pues parece ser que el código generado es el siguiente:

```csharp
[CompilerGenerated]
internal static class <Program>$
{
  private static void <Main>$(string[] args)
  {
    Console.WriteLine("Hello " + <Program>$.<<Main>$>g__GetName|0_0(((IEnumerable<string>) args).FirstOrDefault<string>()));
  }
  internal static string <<Main>$>g__GetName|0_0(string name)
  {
    return name ?? "World";
  }
}
```
Asó pues el compilador **genera el mismo código que generaría si la función `GetName` fuese una función local a `<Main>$`. Esos sufijos tipo `|0_0` son típicos de cuando usamos funciones locales. Y es que, las funciones locales se terminan convirtiendo en funciones de la clase**.

## Funciones locales

Vamos a rizar el rizo... qué ocurre si creo una función dentro de otra función? Vamos a verlo...

```csharp
using System.Linq;
var name = GetName(args.FirstOrDefault());
System.Console.WriteLine($"Hello {name}");

string GetName(string name)
{
    return GetFormatted(name ?? "world");
    
    string GetFormatted(string value) {
        return value.ToUpper();
    }
}
```

**Dado que en _runtime_ no existen las funciones locales, la función `GetFormatted` se "promociona" a una función de la clase**:

```csharp
internal static string <<Main>$>g__GetName|0_0(string name)
{
    return <Program>$.<<Main>$>g__GetFormatted|0_1(name ?? "world");
}
internal static string <<Main>$>g__GetFormatted|0_1(string value)
{
    return value.ToUpper();
}
```

Así pues, las funciones se generan todas como funciones estáticas de la clase `<Program>$` y esos índices tipo `|0_0` son para desambiguar posibles nombres que se pudieran repetir (entiendo que si dos funciones distintas definen otras funciones locales con el mismo nombre).

## Usando variables

Vale, ahora el código de nuestro programa es tal y como sigue:

```csharp
using System.Linq;
var name = args.FirstOrDefault();
System.Console.WriteLine($"Hello {GetName()}");

string GetName()
{
    return name ?? "World";
}
```

Aquí hay una implicación importante y es que `GetName()` accede a la variable `name`, definida más arriba. Como parece ser que el compilador usa funciones locales, eso debería compilar ¿verdad?. Pues **en efecto, compila** y el código generado ahora es el siguiente:

```csharp
[CompilerGenerated]
internal static class <Program>$
{
  private static void <Main>$(string[] args)

  {
    <Program>$.<>c__DisplayClass0_0 cDisplayClass00;
    cDisplayClass00.name = ((IEnumerable<string>) args).FirstOrDefault<string>();
    Console.WriteLine("Hello " + <Program>$.<<Main>$>g__GetName|0_0(ref cDisplayClass00));
  }

  internal static string <<Main>$>g__GetName|0_0(
    [In] ref <Program>$.<>c__DisplayClass0_0 obj0)
  {
    return obj0.name ?? "World";
  }

  [StructLayout(LayoutKind.Auto)]
  private struct <>c__DisplayClass0_0
  {
    public string name;
  }
}
```

El compilador **genera una estructura para contener la variable**, declara una variable de esa estructura en la función `<Main>$` y luego la pasa al método `g__GetName|0_0`.

Esa estrucutra la usará para definir todo el estado que se comparte de la función principal a sus funciones locales, por lo que en nuestro caso si yo declaro dos variables en mi programa, entonces esa estructura tendrá dos campos. Así que vayamos a verificarlo:

```csharp
using System.Linq;
var name = args.FirstOrDefault();
var lastname = args.LastOrDefault();
System.Console.WriteLine($"Hello {GetName()}");

string GetName()
{
    return (name ?? "World") + (lastname ?? "!");
}
```

Y ahora, efectivamente, la estructura tiene los dos campos:

```csharp
[StructLayout(LayoutKind.Auto)]
private struct <>__DisplayClass0_0
{
    public string name;
    public string lastname;
}
```

Ahora bien, nos queda una duda por resolver: el compilador mete **todas las variables declaradas ahí o solo las que se usan en una función adicional?**. Veamoslo:

```csharp
using System.Linq;
var name = args.FirstOrDefault();
var lastname = args.LastOrDefault();
// lastname solo se usa en <Main>$
System.Console.WriteLine($"Hello {GetName()} {lastname}");

string GetName()
{
    return name ?? "World";
}
```

En este código la variable `lastname` no se usa en la función `GetName()`. ¿Qué hará el compilador? Pues parece ser que es lo suficientemente inteligente y **solo genera el campo name** en la estructura, mientras que la variable `lastname` la declara como local `<Main>$`:

```csharp
[CompilerGenerated]
internal static class <Program>$
{

  private static void <Main>$(string[] args)
  {
    <Program>$.<>c__DisplayClass0_0 cDisplayClass00;
    cDisplayClass00.name = ((IEnumerable<string>) args).FirstOrDefault<string>();
    string str = ((IEnumerable<string>) args).LastOrDefault<string>();
    Console.WriteLine("Hello " + <Program>$.<<Main>$>g__GetName|0_0(ref cDisplayClass00) + " " + str);
  }

  internal static string <<Main>$>g__GetName|0_0(
    [In] ref <Program>$.<>c__DisplayClass0_0 obj0)
  {
    return obj0.name ?? "World";
  }

  [StructLayout(LayoutKind.Auto)]
  private struct <>c__DisplayClass0_0
  {
    public string name;
  }
```

Que el compilador sea capaz de hacer esa optimización genera otra duda: qué ocurre si una variable X se usa en una función F1 y otra distinta se genera en una función Y? ¿Generará dos estructuras (una para F1 y otra para F2) o bien lo encapsulará todo junto? Veamos:

```csharp
using System.Linq;
var name = args.FirstOrDefault();
var lastname = args.LastOrDefault();
System.Console.WriteLine($"Hello {GetName()} {GetLastName()}");

string GetName()
{
    return name ?? "World";
}
string GetLastName()
{
    return lastname ?? "!";
}
```

La variable `name` se usa solo en `GetName` mientras que `lastname` se usa solo en `GetLastName`. Veamos qué hace el compilador en este caso:

```
[StructLayout(LayoutKind.Auto)]
private struct <>c__DisplayClass0_0
{
public string name;
public string lastname;
}
```

Pues lo encapsula todo junto, lo que tiene cierta lógica, ya que si no el compilador puede meterse en un berenjenal (si tenemos más funciones y más variables y cada función usa algunas de las variables las combinaciones pueden explotar). La idea es que **el compilador usa una de esas estructuras por cada función que tenga funciones locales. En este caso la única función que tiene funciones locales es `<Main>$`, por lo que hay una sola estructura para contener el estado de `<Main>$` que es accedido en cualquiera de sus funciones locales**.

> **Nota**: Ten presente que el uso de esas estructuras `<>c__DisplayClass` no es algo propio de esa característica de "top level programs" si no que forma parte de la implementación de funciones locales. Pero bueno, dado que nunca había hablado de funciones locales en el blog, pues bienvenido sea.

## Comportamiento de nameof

Por lo general sabemos que `nameof` nos devuelve el nombre de un identificador. Qué ocurre en este caso (en el qué los identificadores generados no se corresponden con los del código fuente)? Vamos a ver:

```csharp
using System.Linq;
var name = args.FirstOrDefault();
WriteName();
void WriteName()
{
    System.Console.WriteLine(nameof(name) + "=" + name);
}
```

Pues bien, `nameof` se comporta tal y como se espera y eso imprime por pantalla el mensaje `name=<valor de name>`. De hecho en el código ya generado por el compilador el `nameof(name)` se ha convertido en la cadena `"name"`:

```csharp
internal static void <<Main>$>g__WriteName|0_0(
    [In] ref <Program>$.<>c__DisplayClass0_0 obj0) 
{
    Console.WriteLine("name=" + obj0.name);
}
```

## Uso de async/await

Podemos usar `await` sin problemas en nuestro "top level program":

```csharp
using System.Threading.Tasks;
await StuffAsync();
System.Console.WriteLine("Done");
Task StuffAsync() => Task.CompletedTask;
```

En este caso el compilador genera toda la parafernalia necesaria para poder usar `await` en nuestro método main (no pongo el código decompilado porque no es `async Main` si no que, lógicamente, ya incluye la máquina de estados).

## Definiendo clases

Nuestros "top level programs" pueden definir sus propias clases! ¿Como traslada eso el compilador?

```csharp
var a = new A() {Foo = 10};
System.Console.WriteLine(a.Foo);
class A {
    public int Foo;
}
```

¿El compilador generará el tipo `A` como una clase separada, o lo generará como una inner class de `<Program>$`?

Pues en este caso, hace lo más sencillo que es generar el tipo `A` como una clase separada, sin nada que ver con `<Program>$`. Por cierto que la clase `A` podría ser `public` y se generaría como publica al ensamblado. Sin ningún problema.

# Usos de esa característica

Vale, hemos visto un poco como el compilador implementa esa característica, y como la implementa el compilador (básicamente se basa en la característica de funciones locales y un poco de azúcar sintáctico adicional) pero ¿qué usos tiene?

Honestamente, **no espero ver grandes programas que eliminen su método `Main`**. Para grandes proyectos, esta característica no tiene apenas relevancia. Es interesante eso sí para entornos de aprendizaje y, sobretodo **permite enfocar C# como lenguaje para scripting**. Viene a ser como "un sustituto" de los antiguos ficheros `.csx`, aunque seguimos necesitando de un `csproj`, pero eso es subsanable con un poco de tooling. Y aquí tenemos un gran potencial... **Aprovechando esa característica se podría incorporar el soporte de [shebangs](https://en.wikipedia.org/wiki/Shebang_(Unix)) a C#**.

Es decir, que eso fuese un fichero válido de C#:

```chsarp
#!/usr/bin/dotnet
System.Console.WriteLine("Hello from terminal!");
```

De ese modo podríamos invocar directamente ficheros .cs desde el shell de Linux... ¡y esos se ejecutarían directamente! Exactamente como ocurre con Python p. ej.

Si eso te parece interesante, que sepas que ¡[ya se está evaluando en esta issue](https://github.com/dotnet/csharplang/issues/3507)!

Y bueno... nada más, no sé si vas a usar esta nueva característica de C# o no, pero bueno... que sepas que aquí está! xD



