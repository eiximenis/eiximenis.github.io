---
title: "C#: Equals y ref structs"
author: eiximenis
description: En C# las "ref struct" son estructuras muy peculiares. Siempre deben estar en la pila, nunca en el heap. Esto, impide que por ejemplo se pueda hacer boxing con ellas, lo que a la práctica significa que "ya no todo se puede guardar en una referencia de tipo object".
date: 2020-02-05T10:00:00
categories:
  - csharp
---

En .NET / C# sabemos que **todos los tipos heredan de `System.Object`** (aka `object`) por lo que, dada una referencia de tipo `object` podemos almacenar cualquier valor. Pero, también sabemos que en el fondo hay dos jerarquías de tipos claramente separadas, con características distintas: Por un lado los tipos por referencia (clases) y por otro lado los tipos por valor (estructuras). Los primeros, se almacenan en el _heap_ y las variables son meras referencias (apuntadores) al objeto que está en el _heap_. Los segundos se suelen almacenar en la pila (aunque pueden terminar en el _heap_) y la variable es el valor en sí mismo, no una referencia.

`System.Object` es una clase, por lo tanto es un tipo por referencia y, recordemos, que todos los tipos heredan de `object`. Esto plantea un punto interesante, y es lo que ocurre cuando usamos una referencia de tipo `object` para un tipo por valor (una estructura). Lo que ocurre es que el valor es copiado al _heap_ y la nueva variable de tipo `object` contiene una referencia al valor del _heap_. Este proceso es conocido como "boxing":

```csharp
int i = 10;
object o = i;       // boxing
```

Al finalizar este código, terminamos con dos objetos `System.Int32` (aka `int`): El primero es `i` que está almacenado en la pila. El segundo está en el _heap_, es una copia del primero y podemos acceder a él usando la referencia `o`. El segundo objeto, será reclamado por el GC a su debido momento, el primero será eliminado cuando se elimine la pila, al devolver de la función.

Este proceso de "boxing" ocurre automáticamente y no lo podemos personalizar de ninguna manera.

En C#7.2 se introdujo un concepto nuevo llamado `ref struct`. Como su nombre indica es un tipo de estructura (por lo tanto tipo por valor), pero tiene una particularidad fundamental: **una `ref struct` nunca puede terminar en el _heap_**. El compilador lo va a evitar por todos los medios. Y eso tiene una implicación directa: **no se puede hacer boxing de un objeto _ref struct_**:

```csharp
ref struct Foo {}

Foo foo = new Foo();
object o = foo;
```

Este código **no compila** (_cannot implicity convert type `Foo` to `object`_). Si añades una conversión explícita (`object o = (object)foo`) el código sigue sin compilar (_cannot convert type `Foo` to `object`_). La razón es que si eso funcionara, una copia del objeto de tipo `Foo` terminaría en el _heap_ y eso está prohibido, ya que `Foo` es una `ref struct`.

Lo interesante es que a nivel del CLR, `Foo` sigue heredando de `System.Object` (a fin de cuentas seguimos en .NET), lo que implica que podemos, por ejemplo, redefinir `Equals`:

```csharp
ref struct Foo
{
    public override bool Equals(object obj)
    {
        if (obj is Foo)
        {
            // Convertimos obj a Foo y comparamos
            return true;
        }

        return false;
    }
}
```

Esta es la implementación típica de `Equals` y la que puedes ver en muchas clases y estructuras. Pero **en una `ref struct` esa implementación no tiene sentido**. La razón básica, es que dado que no hay manera que una referencia `object` pueda llegar a apuntar a un objeto `Foo` (recuerda, el _boxing_ no es posible), entonces nunca se podrá llamar a `Equals` pasando un `Foo` como parámetro. Eso **no compila**, ya que implicaría un _boxing_:

```csharp
Foo foo = new Foo();
Foo foo2 = new Foo();
foo.Equals(foo2);
```

Además, la expresion `obj is Foo` **es siempre `false`**. Dado que el _boxing_ está prohibido, es por definición, imposible que en una referencia `object` tengamos un objeto `Foo`. En resumidas cuentas: **no hay manera de implementar `Equals` en una `ref struct`, pero no es necesario porque no hay manera de llamarlo con una variable de la propia `ref struct`**. Si necesitas comparar `ref struct` entre ellas, debes redefinir el operador `==`:

```csharp
public static bool operator ==(Foo one, Foo other)
{
    // Comparar y devolver true o false
}

public static bool operator !=(Foo one, Foo other)
{
    !(one == other);
}
```

Así, que en el caso de una `ref struct`, la recomendación de Visual Studio de implementar `Equals` te la puedes saltar libremente, o si quieres hacerlo feliz puedes implementar `Equals` para que devuelva siempre `false`, ya que una `ref struct` nunca será igual a un `object`, pues es imposible que ninguna referencia `object` termine apuntando a un objeto que sea una `ref struct`.

¡Saludos! :)
