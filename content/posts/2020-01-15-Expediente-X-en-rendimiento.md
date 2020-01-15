---
title: Expediente X en rendimiento
author: eiximenis
description: Probando demos de mi charla sobre rendimiento me he encontrado hoy con un "expediente X". Os lo cuento, aunque ya os avanzo que el culpable era... ¡yo! xD
date: 2020-01-15T18:20:00
categories:
  - netcore
---

El expediente es el siguiente: contrariamente a todo sentido común **copiar una estructura de 21 campos `double` tardaba menos, pero mucho menos, que copiar una clase**.

## El expediente X

Tengo una clase y una estructura equivalentes: simplemente 21 propiedades de tipo `double` con `get` y `set`. Nada más. La clase se llama `Point21D` y la estructura `Point21DStruct`.

Luego, tengo dos classes (`World21D` y `World21DStruct`) con **el mismo código** salvo, que la primera usa `Point21D` y la segunda `Point21DStruct`:

```cs
class World21D
{
    private Point21D[] _points;

    public World21D()
    {
        _points = new Point21D[65535];
        for (var i = 0; i < 65535; i++) { _points[i] = new Point21D() { X = i, Y = i, Z = i }; }
    }

    public Point21D GetAt(int i)
    {
        return _points[i];
    }
}
```

Ahora viene lo bueno... Ejecuto el siguiente código dos veces (en la primera `_world` es la clase `World21D` y en la segunda es la estructura `World21DStruct`:

```cs
for (var i=0; i< 65535; i++)
{
    var p = _world.GetAt(i);
    p.X = 1;
}
```

Estaba preparado para **que la versión con `struct` fuese más lenta**, porque copiar los 21 campos es más lento que copiar la referencia... Pero usando [BenchmarkDotNet](https://github.com/dotnet/BenchmarkDotNet) obtuve eso:

```
|                 Method |      Mean |    Error |   StdDev |
|----------------------- |----------:|---------:|---------:|
| ReadAllPointsAsClasses | 447.11 us | 8.348 us | 8.199 us |
| ReadAllPointsAsStructs |  27.57 us | 0.606 us | 0.722 us |
```

¡No tiene sentido alguno! Estoy usando .NET Core 3.1 y compilado en _Release_ y a ver, por muy optimizado que esté, no puede ser que copiar una referencia tarde 20 veces más que copiar la estructura de 21 campos ;)

No era un problema de _BenchmarkDotNet_, el profiler me daba exactamente el mismo resultado. En este caso el método `One` usa la versión con la clase y el método `Two` la versión con la estructura:

![Salida de dotTrace donde se observa como tarda más la versión con clases que con structs](/images/posts/2020-01-15-dotTraceOutput.png)

Se puede ver como la versión con clases tarda unas 20 veces más que la versión con estructuras (23.16% vs 1.39%). Los tiempos variaban cada vez, pero el patrón era siempre ese. Está claro que ahí había algo.

Quité _BenchmarkDotNet_ del proyecto (raro, pero podría ser que igual algo de lo que añade al proyecto causara esa variación) ,pero el resultado era exactamente el mismo.

## El "culpable"

Debía entender qué es lo que estaba ocurriendo, por qué la lectura en el caso de las clases tardaba tanto...Era hora de sacar la artillería pesada.

Lo primero verificar con [IlSpy](https://github.com/icsharpcode/ILSpy) que no hubiera nada raro (alguna optimización del compilador que pudiera explicar eso), pero no, todo parecía "correcto":

![Salida de IlSpy donde se ve que el código decompilado coincide con el código fuente](/images/posts/2020-01-15-IlSpyOutput.png)

Descartada cualquier optimización del compilador, usé `ildasm` para ver el código decompilado de ambos métodos:

![Salida de ildasm con el IL de ambos métodos](/images/posts/2020-01-15-ildasm.png)

El listado superior es el IL del método `ReadAllPointsAsClasses` y el segundo el del método `ReadAllPointsAsStructs`. Son bastante parecidos, **pero hay una diferencia fundamental**. En la versión que usa clases tenemos:

```
callvirt   instance void ConsoleApp2.Point21D::set_X(float64)
```

Mientras que en la versión que usa estructuras:

```
call       instance void ConsoleApp2.Point21DStruct::set_X(float64)
```

Esa línea **es la que invoca el _setter_ de la propiedad `X` del objeto**. Recuerda que los métodos `ReadAllPointsAsClasses` /`ReadAllPointsAsStructs` tenían el mismo código fuente:

```cs
for (var i=0; i< 65535; i++)
{
    var p = _world.GetAt(i);
    p.X = 1;
}
```

Es este `p.X = 1;` el que marca la diferencia. En el caso de clases usa `callvirt` y en el de estructuras usa `call`. ¿Cual es la diferencia? Pues bien, hay **dos diferencias fundamentales** entre `callvirt` y `call`:

1. `callvirt` permite llamar a métodos que **pueden ser redifindos en una clase hija**. Es decir, a los métodos virtuales.
2. `callvirt` verifica que el puntero **no sea `null`** (`call` no comprueba nada).

A pesar de que el nombre de `callvirt` sugiera que solo se usa en mètodos marcados con `virtual`, la realidad es que dado que `callvirt` verifica que `this` no sea `null` el compilador lo usa en cualquier llamada a método... excepto si el objeto es un _Value object_ (o sea una estructura) que nunca puede valer `null`. Es por eso que en la versión que usa estructuras se usa `call`.

Bueno, para verificarlo lo más sencillo es eliminar esta línea `p.X = 1` y ver qué ocurre:

![Nueva ejecución de dotTrace. Ahora el 97% del tiempo es leyendo las estructuras](/images/posts/2020-01-15-dotTraceOutput2.png)

¡Vaya cambio! Ahora la lectura de estructuras ocupa el 97% de mi tiempo, mientras que la lectura de las clases ocupa menos del 2%.

¿Y en tiempos? Pues habilitando _BenchmarkDotNet_ otra vez, esos son los resultados:

```raw
|                 Method |        Mean |     Error |    StdDev |
|----------------------- |------------:|----------:|----------:|
| ReadAllPointsAsClasses |    26.81 us |  0.204 us |  0.181 us |
| ReadAllPointsAsStructs | 1,305.62 us | 16.884 us | 15.793 us |
```

**Leer las estructuras es mucho más costoso que leer las clases**. ¡Misterio resuelto!

## Corolario

Ese expediente X no hubiese sido tal si yo no me hubiera saltado las reglas más fundamentales cuando quieres analizar código. A saber:

1. Analiza **una sola cosa cada vez**. En mi caso analizaba dos (obtener un objeto y llamar a una propiedad).
2. **No des NADA por sentado**. En mi caso supuse que la línea `p.X=1` no afectaba, ya que "hacía" lo mismo. Y bueno, ya véis... ni por asomo xD

Y recordad: **¡medid siempre!** (pero medid bien, no os pase como yo xD), que con temas de rendimiento... ¡nunca se sabe! ;-)




