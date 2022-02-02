---
title: "De runas y grafemas en Unicode y NET6"
author: eiximenis
description: "Un carácter en Unicode es algo complejo de definir. Veamos como está la situación en NET6"
date: 2021-01-31
draft: false
categories:
  - dotnet
tags: ["unicode", "dotnet", "c#"]
---

Hace algún tiempo (varios años xD) escribí un post sobre [encodings en Unicode](https://www.eiximenis.dev/posts/2018-10-01-unicode-y-encodings/) en el que comentaba un poco el lío de codificaciones que tenemos en Unicode, y que significa UTF-8 o UTF-16. Te lo puedes leer si quieres pero si no aquí tienes un pequeño resúmen de lo más importante que comentaba y que es necesario para entender de lo que quiero hablarte hoy.

Lo más importante de aquel post eran los siguientes tres puntos:

* El concepto _code point_ en Unicode que es lo que equivaldría a "carácter". Hay muchos (más de 2 a la 16) _code points_ distintos en Unicode
* El concepto de _code unit_ (valor de 16 bits) que **es propio de UTF-16**. Un _code point_ puede estar formado por varios _code units_.
* En .NET un `System.Char` representa un _code unit_ y no un _code point_. 

El tercer punto es clave ya que significa que:

* Una variable `char` es posible que no se corresponden a ningún carácter real (no todos los _code unit_ se corresponen a un _code point_).
* Hay carácteres Unicode que no se pueden representar usando un `char` y necesitamos más de uno.

## Runas (aka code points)

El nombre de runa suena muy esotérico **y no está definido en ningún estándard** pero quedémonos con que una runa es exactamente lo mismo que un _code point_ pero en nomenclatura C# (eso viene de que esa parte de la API de .NET está fuertemente inspirada en su equivalente de Golang y en Golang eligieron dicho nombre).

Así una runa es un valor de 32 bits (como podría ser un `System.Int32`) que se mapea a un _code point_, es decir a un carácter Unicode. Vamos a verlo. Imagina el siguiente código:

```csharp
var s = "A";
Console.WriteLine(s.Length);
```

¿Cuál es el resultado de ese programa? **Supongo que todos habéis dicho 1, ya que la cadena `"A"` tiene un carácter**. Efectivamente la salida es 1. Sigamos:

```csharp
var s = "😀";
Console.WriteLine(s.Length);Rune is 😀 with value 128512
```

¿Y ahora cual es la salida? Uno podría pensar que debería seguir siendo 1, ya que tenemos un solo carácter (recuerda que en Unicode los emojis son carácteres como los demás). Pero la realidad **es que la salida es 2**. ¿Y por qué es 2 la salida, si solo hay un carácter? Pues muy sencillo: **porque en .NET una cadena no contiene carácteres (_code points_) si no que contiene los _code units_ de UTF-16**. El tipo `System.Char` representa un _code unit_, no un _code point_ y para el carácter Unicode '😀' se requieren 2 _code units_ de UTF-16 para codificarlo. **Las cadenas en .NET están en UTF-16 y este hecho se filtra en la API: NO tenemos una API orientada a carácteres, tenemos una API orientada a _code units_ de UTF-16**. No te creas que eso es un problema de los emojis solo: muchos carácteres de lenguas extrangeras requieren más de un _code unit_ para almacenarlos por lo que necesitamos más de un `System.Char` de .NET.

Para solucionar este problema, en netcore 3 se introdujo la API de runas:

```csharp
var s = "😀";
var runes = s.EnumerateRunes();
foreach (var rune in runes) {
    Console.WriteLine("Rune is {0} with value {1}", rune, rune.Value);
}
```

El método `EnumerateRunes` devuelve un `StringRuneEnumerator` que nos permite iterar sobre las runas de la cadena. Recuerda que una runa es un _code point_ de Unicode, es decir un carácter.

La salida de este programa ahora si que nos indica que sólamente hay una runa. La salida es como sigue:

```
Rune is 😀 with value 128512
```

El valor numérico que nos aparece (128512 o 0x1f600) es el valor del [_code point_ correspondiente de Unicode](https://www.compart.com/en/unicode/U+1F600) y es de 32 bits. Por lo tanto **el código correcto para saber cuantos carácteres reales tiene una cadena NO es usar `String.Length` si no usar `String.EnumerateRunes().Count()`**.

Si quisieras repetir este ejercicio pero en lugar de usar el emoticono metido en el fichero, quisieras usar su codificación Unicode, la cosa no es tan sencilla:

```csharp
var s = "\u1f600";
Console.WriteLine(s);
```

Este código **NO imprime el emoticono** si no que en su lugar imprime `ὠ0`. ¿Qué ocurre aquí? Pues sencillamente que la notación `\u` nos permite indicar el código Unicode, pero el código Unicode ¿de qué? No de un _code point_ si no de un _code unit_ de UTF-16 (es decir de un `System.Char`). Por lo tanto después de `\u` siempre van 4 dígitos hexadecimales (de `0000` a `ffff`) que nos cubren los 16 bits possibles. Por lo que la cadena `"\u1f600"` se toma como una cadena con dos _code units_, el primero con [código Unicode `1f60`](https://www.compart.com/en/unicode/U+1F60) y el segundo es el carácter `0`. Si quieres construir el emoji **debes saber como se codifica en UTF-16**:

```csharp
var s = "\ud83d\ude00" 
Console.WriteLine(s);
```

Este código si que imprime el emoticono por pantalla ya que `s` ya que el emoticono 😀 en UTF-16 se codifica mediante dos _code units_ (`0xd83d` y `0xde00`), y como un `System.Char` de .NET equivale a un _code unit_ de UTF-16 y un `System.String` es una colección de `System.Char` pues nos toca poner la codificación UTF-16 tal cual. Obviamente da igual si usamos los dos `System.Char` explícitos en la cadena o el emoji directo, el código que detecta las runas funciona igual (ya que realmente ambas cadenas son la misma). Así el siguiente código imprime `True`, ya que la condición es cierta:

```csharp
Console.WriteLine("\ud83d\ude00" == "😀");
```

Resumiendo: la API de Runas de C# nos permite obtener los carácteres Unicode (_code points_) de la cadena, en lugar de los _code units_ de UTF-16. La realidad es que aquí se nota que la API de cadenas de .NET tiene sus añitos (recuerda que en nada .NET cumple 20 años xD) ya que, estoy seguro, si se diseñase ahora desde 0, el valor de `System.Char` se correspondería a un _code point_ de Unicode y en todo caso tendríamos métodos para obtener una codificación UTF-16 (o UTF-8 o lo que sea) de un `System.Char`.

Igual te preguntas cuando deberías usar runas y no `System.Char` al trabajar con cadenas. La realidad es que deberías hacerlo siempre que quieras soportar cualquier posible carácter Unicode. Usando `char` entras en riesgo si el carácter cae fuera de cierto rango (el que llamamos rango BMP). Por ejemplo, el siguiente código falla miserablemente al contar las letras de la cadena `s`:

```csharp
var s = "𐓏𐓘𐓻𐓘𐓻𐓟 𐒻𐓟";
var letters = s.Where(c => char.IsLetter(c)).Count();
```

El valor de `letters` es 0. Por otro lado si usamos la API de runas todo funciona correctamente:

```csharp
var s = "𐓏𐓘𐓻𐓘𐓻𐓟 𐒻𐓟";
var letters = s.EnumerateRunes().Where(r => Rune.IsLetter(r)).Count();
```

Ahora sí que `letters` tiene el valor correcto (`8`). La clase `System.Text.Rune` tiene muchos de los métodos estáticos que hay en `System.Char` incluyendo `ToUpper` o `ToLower` por ejemplo.

## Grafemas

Vale, ya tenemos claro que si una cadena tiene 3 runas, es que el usuario verá 3 carácteres en pantalla ¿no? **Pues no**. Bienvenido al séptimo círculo infernal de Unicode: los grafemas.

Un grafema (el nombre técnico es `grapheme cluster`) es el equivalente de carácteres percibidos en pantalla por el usuario. O sea, si una cadena contiene 2 grafemas el usuario verá dos carácteres en pantalla. Así que poniéndolo todo junto:

* Una `System.String` puede contener varios `code units` de UTF-16. 
  * Cada `code unit` se representa mediante un `System.Char`.
* Todos esos _code units_ son la representación en UTF-16 de varios _code points_ de Unicode.
  * Cada _code point_ se representa mediante un `System.Text.Rune`.
* Todos esos _code points_ de Unicode son la representación de varios grafemas
  * Por cada grafema el usuario percibirá un carácter

Vamos a ver un ejemplo de grafema. Por ejemplo el carácter "👩‍👩‍👧‍👧". Esta adorable família de dos madres y dos niñas **no es un _code point_** de Unicode. No hay ningún carácter Unicode que represente a esta família. En su lugar **existe una combinación de _code points_ que forman un grafema**. Concretamente en este caso **hay 7 _code points_ que se combinan para formar ese grafema**. Los podemos ver uno a uno usando la API de runas de C#:

```csharp
var s = "👩‍👩‍👧‍👧";
var runes = s.EnumerateRunes();
foreach (var rune in runes) {
    Console.WriteLine("Rune is {0} with value {1}", rune, rune.Value);
}
```

La salida de ese programa es tal y como se muestra a combinación:

```
Rune is 👩 with value 128105
Rune is ‍ with value 8205
Rune is 👩 with value 128105
Rune is ‍ with value 8205
Rune is 👧 with value 128103
Rune is ‍ with value 8205
Rune is 👧 with value 128103
```

El _code point_ 8205 es un _code point_ "especial", se llama ZWJ (Zero Width Joiner) y se usa, precisamente, en este tipo de combinaciones: para combinar varios _code points_ y formar un grafema. ¡Ojo! Que el 8205 no es el único _code point_ especial, hay muchos más que se usan en otros grafemas. El estándard Unicode pues, no solo define las tablas de carácteres (_code points_) y como se codifican (UTF-16, UTF-8, etc) si no que también define posibles combinaciones de _code points_ para formar distintos grafemas. De hecho, puedes ver que 👩‍👩‍👧‍👧 está formado por varios carácteres si, usando Visual Studio Code, te situas justo después del carácter y empiezas a borrar:

![Imagen animada donde se va pulsando la tecla de borrar y el grafema va cambiando](/images/posts/2022-02-02-vscode-delete-grafema.gif)

Cada vez que borro, se elimina un _code point_ por lo que el grafema es cada vez distinto, de ahí que el emoji vaya cambiando cada vez borramos.

Igual te preguntas si es posible en C# saber cuantos grafemas tiene una cadena. Pues la realidad es que sí. **Podemos iterar sobre los grafemas de una cadena**:

```csharp
var s = "👩‍👩‍👧‍👧";
var graphemes = StringInfo.GetTextElementEnumerator(s);
var count = 0;
while (graphemes.MoveNext()) {
    count++;
}
Console.WriteLine("We have {0} graphemes", count);
```

Este código imprime `We have 1 graphemes` en pantalla, indicando que efectivamente la cadena "👩‍👩‍👧‍👧" contiene un solo grafema. También podemos obtener cada uno de los grafemas:

```csharp
while (graphemes.MoveNext()) {
    var grapheme = graphemes.Current;   
}
```

Ahora, aquí tenemos un pequeño problema y es que la variable `grapheme` es de tipo `object?`, ya que no hay un tipo específico para representar un grafema. De hecho el objeto devuelto por `Current` es de tipo `System.String`, o sea que podemos usar el siguiente código:

```csharp
var grapheme = (string)graphemes.Current;
```

Por supuesto, puedes usar `as` o incluso usar `graphemes.Current?.ToString()`.

¿Y como es que para representar un grafema terminamos con una cadena? Eso es porque, al menos en la versión actual de la API (NET6), con los grafemas poco podemos hacer: imprimirlos por pantalla, contarlos y por supuesto (dado que son cadenas) saber sus runas:

```csharp
var s = "👩‍👩‍👧‍👧";
var graphemes = StringInfo.GetTextElementEnumerator(s);
var count = 0;
while (graphemes.MoveNext()) {
    count ++;
    Console.WriteLine($"Grapheme {count}:");
    var grapheme = (string)graphemes.Current;   
    var runes = grapheme.EnumerateRunes();
    foreach (var rune in runes) {
        Console.WriteLine($"Rune is {rune} with code {rune.Value}");
    }
}
Console.WriteLine("We have {0} graphemes", count);
```

La salida de este código es tal y como sigue:

```
Grapheme 1:
Rune is 👩 with code 128105
Rune is ‍ with code 8205
Rune is 👩 with code 128105
Rune is ‍ with code 8205
Rune is 👧 with code 128103
Rune is ‍ with code 8205
Rune is 👧 with code 128103
We have 1 graphemes
```

Por supuesto, un grafema, dado que es una cadena lo podemos imprimir tal cual por consola:

```csharp
var s = "👩‍👩‍👧‍👧";
var graphemes = StringInfo.GetTextElementEnumerator(s);
while (graphemes.MoveNext()) {
    Console.WriteLine("Grapheme:" + graphemes.Current);
}
```

La salida de este código **debería ser** la siguiente:

```
Grapheme:👩‍👩‍👧
```

Pero **la realidad es que la salida puede ser cualquier cosa** ya que el soporte de grafemas en terminales está un poco verde. Es decir, es posible que en tu terminal no lo veas bien. Pero la culpa no es de .NET, es del terminal que no gestiona la visualización de la combinación de _code units_ como un grafema.

Algunas muestras. Muestro una captura de pantalla de la salida del programa junto con el contenido del fichero `Program.cs`. Empecemos por Windows Terminal en su versión 1.11.3471.0:

![Salida en WT donde el grafema se ve bien pero con espacios alrededor](/images/posts/2022-02-02-wt.png)

En esta segunda imagen se muestra como se ve en el terminal integrado de VSCode:

![Salida en el terminal de vscode donde el grafema se muestra como sus runas individuales](/images/posts/2022-02-02-vscode.png)

Y en esta tercera como se ve en el terminal por defecto de Ubuntu 21.10:

![Salida en el terminal de Ubuntu](/images/posts/2022-02-02-ubuntu.png)

Como puedes ver, cada terminal lo gestiona distinto. Visual Studio Code es curioso porque en el editor el grafema se ve perfecto, pero en el terminal no.

En resumen, en este post hemos visto las diferencias entre un `System.Char` de .NET, un carácter Unicode y un grafema. Espero que este post te haya podido aclarar un poco las ideas y de paso hacerte ver que el soporte que tenemos en .NET para tratar con Unicode es ahora mismo muy bueno.