---
title: Duda metafísica sobre contravarianza en delegates

author: eiximenis

date: 2008-12-18T13:14:02+00:00
geeks_url: /?p=1428
geeks_visits:
  - 1322
geeks_ms_views:
  - 957
categories:
  - c#

---
Hola… hoy voy a poner un post sobre una dudilla _metafísica_ que me ha surgido, concretamente relativa a los _delegates_. Y he pensado… que mejor sitio que ponerla que aquí??? 😉

Los delegates en C# 2.0 son contravariantes, es decir un delegate a un método que espera un parámetro tipo X aceptará cualquier método que espere un parámetro de cualquier tipo base de X.

<!--more-->

Es decir, el siguiente código funciona bien:

```cs
delegate void Foo(Derived d);
public class Base { }
public class Derived : Base { }
public class SomeCode
{
    SomeCode()
    {
        Foo foo = new Foo(this.SomeMethod);
    }
    private void SomeMethod(Base b) { }
}
```

Todos entendemos la lógica que hay tras ello: al ser todos los objetos Derived, objetos Base, es evidente que cualquier método que trate con objetos Base, lo podrá hacer con objetos Derived, y por ello el método SomeMethod es un destino bueno para el delegate Foo.

Ahora bien, imaginemos que tenemos el siguiente código:

```cs
delegate void Foo(Derived d);
public class Base { }
public class Derived 
{
    public static implicit operator Base(Derived d) { return new Base(); }
}
public class SomeCode
{
    SomeCode()
    {
        Foo foo = new Foo(this.SomeMethod);
    }
    private void SomeMethod(Base b) { }
}
```

En este caso el código no compila, el compilador se queja con un `No overload for SomeCode.SomeMethod(Base) matches delegate Foo`.

La duda metafísica es… creeis que debería compilar? En _cierto_ (lo pongo en cursiva) todos los objetos `Derived` tambien son objetos `Base`, puesto que hay definida una conversión implícita de `Derived` a `Base`… con lo que _podríamos_ entender que hay una cierta contravarianza.

O creeis que no? Que ya está bien que no compile puesto que el operador de conversión no puede representar nunca una relación is-a y por lo tanto la contravarianza no tiene sentido…

MMmmm… yo reconozco que no estoy 100% posicionado a favor de ninguna opción…

Saludos!