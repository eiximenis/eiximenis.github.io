---
title: XmlSerializer y propiedades ocultadas
description: XmlSerializer y propiedades ocultadas
author: eiximenis

date: 2009-02-20T09:28:00+00:00
geeks_url: /?p=1438
geeks_visits:
  - 1674
geeks_ms_views:
  - 1025
categories:
  - Uncategorized

---
Hola! Ayer un compañero de trabajo me comentó un problema con el que se encontró trabajando con propiedades ocultadas y el serializador xml.

En concreto, quería serializar dos clases tales como las que siguen:

```cs
public class FOO
{
    private List<FOO> _items;
    public List<FOO> Items
    {
        get { return _items; }
        set { _items = value; }
    }
}
public class DerivedFOO : FOO
{
    private List<DerivedFOO> _items;
    public new List<DerivedFOO> Items
    {
        get { return _items; }
        set { _items = value; }
    }
    public DerivedFOO() { Items = new List<DerivedFOO>(); }
}
```

Son dos clases, una que deriva de la otra, que cada una de ellas tiene una lista de elementos de la propia clase.

Al intentar serializar un objeto DerivedFOO el serializador da una excepción: "El miembro DerivedFOO.Items de tipo `System.Collections.Generic.List``1[ConsoleApplication10.DerivedFOO] oculta al miembro de clase base FOO.Items de tipo System.Collections.Generic.List``1[ConsoleApplication10.FOO]`. Utilice `XmlElementAttribute` o `XmlAttributeAttribute` para especificar un nombre nuevo."

La solución que propone la propia excepción (el uso de `[XmlElement]` o `[XmlAttribute]`) no funciona, y el uso de `[XmlArray]` tampoco.

Analicemos un poco la situación: Tenemos dos clases, una derivada de la otra, donde la derivada oculta una propiedad de la clase base. Declarar la propiedad Items como virtual y redefinirla en la clase derivada, **no** funciona por dos razones. La primera es que C# no acepta propiedades covariantes, lo que implica que desde una clase derivada no podemos redefinir una propiedad de la clase base para que devuelva un tipo más específico (derivado) del que declara la propiedad base. Es decir, esto no compila en C#:

```cs
class A
{
    public virtual A Self { get; set; }
}
class B : A
{
    // C# no tiene propiedades covariantes!
    public override B Self { get;set;}
}
```

La segunda razón por la cual, declarar la propiedad como virtual en la clase base tampoco funcionaría, es que incluso _suponiendo_ que C# tuviese propiedades covariantes, `List<FOO>` y `List<DerivedFOO>` son dos tipos completamente distintos. Recordad que aunque `B` derive de `A`, `List<B>` **no** deriva de `List<A>`.

Está claro que debemos buscar otro enfoque: Una posible solución pasa por el uso de genéricos, es decir definir una sola clase base que tenga la propiedad Items, y que esta sea genérica:

```cs
public class FOOBase<T>
{
    private List<FOOBase<T>> _items;
    public List<FOOBase<T>> Items
    {
        get { return _items; }
        set { _items = value; }
    }
    public FOOBase()
    {
        Items = new List<FOOBase<T>>();
    }
}
public class FOO : FOOBase<FOO> { }
public class DerivedFOO : FOOBase<DerivedFOO> {}
```

En este caso definimos _tres_ clases: La genérica `FOOBase<T>` que es la que contiene la definición de la propiedad, y luego dos especializaciones, que hemos llamado `FOO` y `DerivedFOO`.

Esto funciona, pero no podemos negar que hemos modificado la _relación_ entre FOO y DerivedFOO. Inicialmente la segunda era derivada de la primera, pero ahora entre FOO y DerivedFOO **no** hay ninguna relación. Si queremos poder trabajar indistintamente con ambas clases (FOO y DerivedFOO) debemos trabajar a nivel de `FOOBase<T>`, lo que según el caso puede ser problemático:

```cs
// error CS0246: The type or namespace name 'T' could not be found
static void f(FOOBase<T> t) {}
```

Para que esto compile el método f debe ser a su vez genérico:

```cs
static void f<T>(FOOBase<T> t) { }
```

Por suerte, al menos, Visual Studio puede inferir el tipo genérico del método a partir del argumento de llamada, por lo que _al menos_ esto funciona:

```cs
DerivedFOO df = new DerivedFOO();
f(df);
```

Para evitar tener que arrastrar tantos métodos genéricos es interesante tener una clase base, que **no** sea genérica y que defina todas las propiedades (y métodos) base que no dependen en absoluto del tipo genérico:

```cs
public class FOOBase     
{ 
    // Nueva clase base de la jerarquía
}
public class FOOBase<T> : FOOBase
    where T : FOOBase
{
    private List<FOOBase<T>> _items;
    public new List<FOOBase<T>> Items
    {
        get { return _items; }
        set { _items = value; }
    }
    public FOOBase()
    {
        Items = new List<FOOBase<T>>();
    }
}
public class FOO : FOOBase<FOO> { }
public class DerivedFOO : FOOBase<DerivedFOO> {}
``` 

Ahora la clase `FOOBase` pasa a ser la clase principal, de la que deriva `FOOBase<T>`. Esto nos permite definir métodos que acepten `FOOBase` y que no deben ser genéricos. Evidentemente esto tiene un _precio_: no podemos acceder a la propiedad Items desde una referencia a `FOOBase`, por lo que si necesitamos acceder a esta propiedad, si que no tenemos más remedio que trabajar con métodos genéricos...

Ahora podemos serializar objetos FOO y DerivedFOO y que ambos contienen una propiedad `List<T>` siendo `T` el propio tipo, además de otras propiedades que vendrían heredades de FOOBase. 

¿Satisface esto los requerimientos de mi compañero? Pues no lo sé, pero a mi no se me ha ocurrido ninguna idea más...

¡Saludos!
