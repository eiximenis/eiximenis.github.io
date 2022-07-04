---
title: "Sobre el prefijo I en las interfaces..."
author: eiximenis
description: "Una de las convenciones más arraigadas en C# es la de usar el prefijo I para las interfaces. De hecho, es una convención recomendada por la propia Microsoft. Pero hay quien la cuestiona, pero ¿con razón? "
date: 2022-07-04
draft: false
categories:
  - dotnet
tags: ["csharp"]
---

El otro día estuve viendo el [vídeo de CodelyTV sobre código sostenible](https://www.youtube.com/watch?v=my17Y9z5gB0), el cual os recomiendo porque se comentan unas "reglas" para ayudar a escribir código que sea fácil de entender y modificar. Probablemente si ya lleváis algunos añitos en este mundillo os sonarán todas ya que se refieren a aspectos bastante elementales: que haya tests **y que sean significativos**, que cuidemos el nombre de las cosas (ya se sabe "naming things is hard") y que el código no haga más de lo que dice (lo que vendría a entroncar directamente con el SRP). Bueno, este es un resumen un poco chusquero claro, en el vídeo lo detallan todo un poco más. [Existe también un libro](https://savvily.es/libros/codigo-sostenible/) al respecto, escrito por [Carlos Blé](https://twitter.com/carlosble) donde se ahondan en todos esos temas, pero no lo he podido leer todavía, así que de momento no puedo opinar.

Bueno, a lo que vamos que me pierdo por las ramas... En un momento de la charla, Carlos, cuando habla sobre la importancia del _naming_ menciona dos ejemplos de cosas que "no deberían hacerse". Primero atiza a los Javeros, diciendo que el sufijo "Impl" para las clases no debería usarse nunca, y luego le toca el turno a los dotneteros cuando comenta que el prefijo "I" en las interfaces no tiene sentido y tampoco debería ser usado. Y esa afirmación es la que me da pie a escribir este post :)

A priori sorprende que alguien cuestione una convención que está tan profundamente arraigada y que además **[está recomendada por la propia Microsoft](https://docs.microsoft.com/en-us/dotnet/standard/design-guidelines/names-of-classes-structs-and-interfaces)**. Pero ojo, y eso es importante, esa recomendación de Microsoft aplica básicamente **a los desarrolladores de librerías**, no a los desarrolladores de aplicaciones. Es decir, si tu vas a desarrollar una librería, tus interfaces deberían empezar todos por "I". ¿Por qué? Por coherencia con el resto del Framework. Los desarrolladores de .NET cuando usan una interfaz, esperan que esa empiece por "I", y si las interfaces de tu librería no lo hacen, tu librería se va a sentir "extraña" y que no se integra bien con el resto. Pero el propio Carlos matiza que su recomendación de no usar "I" no aplica a los desarrolladores de librerías, si no a los desarrolladores de aplicaciones finales. Es decir, básicamente está diciendo que las interfaces que nos creamos nosotros para ser usadas única y exclusivamente en el código de nuestra aplicación, no deberían llevar el prefijo "I". ¿Ahora bien, qué problema hay en usar la "I"? Empecemos por el principio...

## Notación húngara

La notación húngara es un invento que a primera vista parece maravilloso e inocuo: Se trata de añadir información del tipo de la variable en el propio nombre de ésta. Su principal ventaja era que viendo el nombre de la variable, uno podía saber su tipo y por lo tanto ganaba en contexto. A priori no parece nada dañino: la variable se llamará `iAge` en lugar de `age` para dejar claro que es de tipo entero, o `sName` en lugar de `name` para indicar que es de tipo cadena (_string_).

Pero, la realidad es que la notación húngara **intenta solucionar un problema que no existe** y parte de una premisa errónea que es, que para entender un código el tipo de las variables importa. Y no, el tipo de una variable no debería importar a la hora de leer un código. Cuando estoy leyendo un código saber si han usado un `int` o un `short` para la variable de la edad me importa poco o nada. Lo importante es que el nombre de la variable indique su significado. Que se llame `age` es lo importante. La notación húngara al final es contraproducente, ya que "ensucia" los nombres de variables con prefijos que terminan generando más ruído que utilidad. Quizá solo te estás imaginando prefijos de una sola letra, pero cuando el lenguaje que usas tiene conceptos como "long pointer to constant string of wide characters" terminas con variables llamadas `lpcwstrName` y aberraciones similares... Por suerte, en algún momento pusimos criterios, vimos que eso daba más pol saco que aportaba, y hoy en día la notación húngara es algo que se usa más bien poco y se recomienda menos.

Como parte de la notación húngara era habitual que todas las clases empezaran por `C` (para que quede claro que era una clase) y así teníamos `CCustomer` o `CAnimal` entre otras lindezas, así las podíamos distinguir de las estructuras `SCustomer` o `SAnimal` claro...

## La I en las interfaces

Parece obvio que el prefijo "I" en las interfaces es una vieja herencia de la notación húngara en .NET. El por qué Microsoft diseñó el framework usando esa nomenclatura es algo que escapa a mi conocimiento y a mi comprensión: cuando se creó .NET la notación húngara ya estaba de capa caída y además ya existía Java, que tenía interfaces y no los prefijaba de ninguna manera. La realidad es que la "I" en las interfaces sobra, es redundante y por si eso fuese poco nos abre la puerta a otra "mala práctica" muy extendida en .NET: las interfaces con una única implementación. Por si acaso, recuerdo de nuevo, que todo lo que digo aplica al desarrollo de aplicaciones finales, no de librerías pensadas para ser reutilizadas y extendidas por aplicaciones.

Cuando digo que la "I" en las interfaces nos abre la práctica a tener interfaces con una única implementación, es porque nos quita el trabajo mental de pensar en el naming de la clase. Si la interfaz se llama `IBeersRepository`, no hay que estrujarse mucho las neuronas para terminar llamando a la clase `BeersRepository`. Por lo tanto en .NET es muy sencillo tener pares "IFoo/Foo" y listos y muchas veces se abusa de eso y hay mucha gente que casi crea una interfaz para todo (¡yo he llegado a ver interfaces hasta para DTOs!). Por cierto que cuando preguntas el porqué de una determinada interfaz que tiene una única implementación y que no se trata de un punto de extensibilidad de la aplicación, la respuesta suele ser que "es para poder hacer _mocks/stubs_ para los tests". Pero no necesitas para nada una interfaz para poder hacer _mocks/stubs_ en los tests: lo puedes hacer con una clase sin ningún problema, aunque ciertamente ello te obliga a pensar un poco más sobre los puntos de extensibilidad para decidir qué métodos declarar como virtuales... como pensar esto es complejo, solemos tirar por el camino fácil y declarar la interfaz y listos (para ser justos, en Java no tienen ese problema al ser todos los métodos virtuales por defecto).

Volviendo al caso de `IBeersRepository` vamos a suponer que la interfaz es necesaria, ya que tienes o prevees que se puedan tener, varias implementaciones de ella. Ahora bien, si no usas la `I` para la interfaz, entonces el nombre de la interfaz sería `BeersRepository`, lo que te obliga a pensar en el nombre de la clase. Así la clase se puede llamar `EntityFrameworkBeersRepository` dejando claro que es uan implementación usando EF. El par de nombres `BeersRepository/EntityFrameworkBeersRepository` es mucho mejor que el par `IBeersRepository/BeersRepository`. Obviamente que la interfaz se llame `IBeersRepository` no te impide que la clase se llame `EntityFrameworkBeersRepository`, pero la realidad es que **por norma general, en el desarrollo, lo que no se fuerza no sucede**. Es decir, si no te fuerzan a pensar en el nombre de una clase, no lo harás y optarás por el camino fácil (cierto, estoy generalizando, pero creo que coincidirás en que suele ser así).

> Efectivamente, todo esto que he dicho en las "I" de las interfaces, se puede aplicar al subrayado delante de las variables privadas. Tampoco es necesario para nada, pero yo en este caso reconozco que lo uso y es que bueno... ¡hay costumbres que cuestan sacarse de encima! :P

## Otro ejemplo: el sufijo Async

Otro ejemplo parecido (aunque con muchos, muchísimos matices), ya sin nada que ver con lo que se comentaba en el vídeo, es el uso del sufijo `Async` para los métodos asíncronos. Es recomendación de Microsoft el usarlo pero **¿tiene sentido?**. En este caso yo reconozco que **no sigo esa convención**, ya que creo que aporta más bien poco. Qué un método es asíncrono lo puedes ver fácilmente por el tipo de retorno (será `Task<T>`, `ValueTask<T>` o algún otro _awaitable_). **Reconozco que puede ayudar** en un caso:

```csharp
void Foo()
{
    // Código por aquí
    BarAsync();
    // Más código por aquí
}
```

En este caso al leer este código se abre una yellow flag, ya que una llamada a un método asíncrono sin un `await` o sin recojer la tarea devuelta, es muy probablemente un error. En este caso el sufijo `Async` ayuda al leer el código, pero en muchos de esos casos el compilador nos ayuda emitiendo el [CS4014](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-messages/cs4014). Aunque, para ser justos lo emite solo en aquellos casos en que el método sea declarado como `async`. Es decir, el siguiente código compila con 0 errores y 0 warnings:

```csharp
Console.WriteLine("Hello, World!");
Bar();
Task Bar()
{
    return Task.Delay(1000).ContinueWith(t => Console.WriteLine("Done"));
}
```

Pero no funciona correctamente, ya que al no realizar el `await` sobre `Bar`, el programa termina una vez se ha impreso "Hello World". Yo creo que el CS4014 debería emitirse siempre que el método devolviese un awaitable y este fuese ignorado, pero lamentablemente parece que solo lo hace si el método es `async`.

Y por supuesto, en el caso en que tengas una implementación síncrona y otra asíncrona del método, entonces el sufijo `Async` (o cualquier otro) es obligatorio, ya que seguramente esos dos métodos diferirán solo en el valor de retorno, por lo que debern tener nombres distintos.

¿Y vosotros? ¿Qué opináis de todo esto? ¿Alguna otra convención/recomendación que no os convenza?