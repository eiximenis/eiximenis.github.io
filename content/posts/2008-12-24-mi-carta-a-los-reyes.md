---
title: Mi carta a los reyes…
description: Mi carta a los reyes…
author: eiximenis

date: 2008-12-24T13:56:35+00:00
geeks_url: /?p=1429
geeks_visits:
  - 901
geeks_ms_views:
  - 893
categories:
  - c#

---
Aunque sería para unos reyes que pasaran más allá del 2010, pero bueno… por pedir que no quede 😛

Estas son las cosas que me gustaría que algun dia se incorporasen a C#. Como son fiestas mágicas, pues aquí van para ver si los chicos de Redmond se animan y para algún milenio las tenemos…

Pongo ideas al margen de si son posibles/factibles con el CLR actual… simplemente son cosas que me gustarían que estuviesen en el lenguaje. Solo pongo las ideas y un ejemplo de lo que me gustaría, sin desarrollarlas a fondo… 🙂

## 1. Conceptos en genéricos

La idea es poder representar como restricción de un tipo genérico algo más allá que una interfaz. Por ejemplo:

```cs
concept HasHandle
{
    IntPtr Handle{ get; }        
}
class Foo
{
    public static void Main(string[] args)
    {
        new Foo().Bar(new Form());
    }

    public void Bar<T> (T t)
        where T : HasHandle
        {
            DoWork(t.Handle);
        }
}
```

Este código deberia compilar porque el concepto HasHandle define una propiedad llamada Handle de tipo IntPtr. El mètode genérico Bar<T> utiliza esta propiedad. Y la llamada a Bar con un paramétro Form satisface el concepto, puesto que la clase Form tiene una propiedad IntPtr llamada Handle.

Los conceptos deberían poder ser genéricos a su vez:

```cs
concept HasXXX<T>
{
    T XXX{ get; set;}        
}
class Foo
{
    public void Bar<T,U> (T t)
        where T : HasXXX<U>
        {
            U u = t.XXX;
        }
}
```

El método `Bar<T,U>` debería poder llamarse con cualquier tipo `T` que tuviese una propiedad `U XXX {get; set;}`

Los conceptos deberían poder hacer referencia a la existencia de métodos estáticos:

```cs
concept Comparable<T>
{
    bool operator > (T t1, T t2);
}
class Foo
{
    void Bar<T> (IEnumerable<T> items)
        where T : Comparable<T>
    {
    }
}
```

El método Bar<T> debería poder llamarse con cualquier tipo T que tuviese definido el operador ‘>’ (aunque este esté definido de forma estática).

## 2. Constantes binarias

Pues eso mismo…

```cs
int i = 0b00001100;
```

Fácil, no??

## 3. Tipos anónimos que puedan implementar una interfaz

Los tipos anónimos tal y como estan ahora sirven para poco más que para LINQ. Sería interesante que pudiesen implementar una interfaz y que pudiesen ser devueltos a través de referencias a dicha interfaz:

```cs
interface ISomeInterface
{
    int Foo(int);
}
class Bar
{
    ISomeInterface Baz()
    {
        return new ISomeInterface  { void Foo(int i) { return i+1;} };
    }
}
```

El tipo anónimo creado en el método Baz, implementa la interfaz ISomeInterface y es devuelto a través de una referencia a dicha interfaz.

## 4. Referencias const

Al estilo de C++. Mira que esto es simple y útil y no se porque no lo han metido… Para más info: [http://www.cprogramming.com/tutorial/const_correctness.html][2].

## 5. Enumeraciones con cualquier tipo

Estaría bien que los enums pudieran tener cualquier tipo base:

```cs
class Planeta
{
    long Diametro { get; set; }
    long DistanciaSol { get; set; }
}
enum SistemaSolar<Planeta>
{
    Mercurio = new Planeta() { Diametro=4879, DistanciaSol=46000000},
    Tierra = new Planeta() { Diametro=12742, DistanciaSol=150000000}
}
class Foo()
{
    Bar () { this.Baz(SistemaSolar.Mercurio);}
    Baz(SistemaSolar p)
    {
        if (p.Diametro > 10000) { ... }
    }
}
```

El enum SistemaSolar se define como una enumeración de objetos de la clase Planeta.

El método Foo.Baz acepta como parámetro sólo SistemaSolar.Mercurio o SistemaSolar.Tierra. Pasarle cualquier otro objeto Planeta no debería compilar.

## Felices fiestaaaaaaaaaaaaaaaaaaaaaaas!

Bueno… paro ya de pedir cosas, jejeeee… 🙂

Solo desearos a todos los geeks unas felices fiestas, llenas de alegría, familia, ceros y unos…

… y que el [**tió os cague**][3] muchos regaloooooooooooooooooooooooooooooooos!!!!

 [1]: http://11011.net/software/vspaste
 [2]: http://www.cprogramming.com/tutorial/const_correctness.html "http://www.cprogramming.com/tutorial/const_correctness.html"
 [3]: http://es.wikipedia.org/wiki/Canciones_del_Ti%C3%B3