---
title: Unity? Sí gracias, pero no me abraces demasiado…
description: Unity? Sí gracias, pero no me abraces demasiado…
author: eiximenis

date: 2009-02-02T13:07:00+00:00
geeks_url: /?p=1435
geeks_visits:
  - 2198
geeks_ms_views:
  - 590
categories:
  - Uncategorized

---
No hace mucho, [Jorge Dieguez] escribió [un interesante post sobre Unity](https://geeks.ms/jdieguez/archive/2009/01/25/microsoft-unity-inyecci-243-n-de-dependencias-net.aspx) y el patrón de [Dependency Injection](https://en.wikipedia.org/wiki/Dependency_injection). Resumiendo mucho este patrón permite eliminar las dependencias de nuestro código, trasladandolas todas a un sólo elemento, que se conoce generalmente como &ldquo;contenedor de DI&rdquo;. Este contenedor es el responsable de devolvernos todas las referencias a clases que nostros precisemos.

Las ventajas es que tenemos un código mucho menos acoplado, que por lo tanto es más fácil de probar y de mantener.

Contenedores de DI hay muchos, p.ej. en .NET tenemos a [Unity](https://www.codeplex.com/unity) (que viene de la mano de la gente de [P&P](https://msdn.microsoft.com/en-us/practices/default.aspx)) a [Windsor Container](https://www.castleproject.org/container/index.html) o a [Spring.NET](https://www.springframework.net/) sólo por citar tres ejemplos. Cada uno de ellos (y de los muchos otros que hay) tienen sus características y peculiaridades y dado que estamos pensando en hacer código débilmente acoplado... quizá deberíamos evitar ligarnos a nuestro contenedor DI, porque nunca sabemos cuando nos puede interesar cambiar.

Cojamos el caso de Unity. Imaginemos el siguiente proyecto super sencillo:

```cs
namespace UnityTest
{
    interface IFoo {}
    class FooClass : IFoo { }
    class Program
    {
        static void Main(string[] args)
        {
            UnityContainer uc = new UnityContainer();
            uc.RegisterType<IFoo, FooClass>();
            IFoo foo = uc.Resolve<IFoo>();
            Console.WriteLine(foo.GetType().FullName);
            Console.ReadLine();
        }
    }
}
```

Es una aplicación de consola, donde se declara una interfaz (IFoo), una clase que la implementa (FooClass) y luego en el método Main:

  1. Se crea una instancia de Unity 
  2. Se registra el mapping entre IFoo y FooClass 
  3. Se obtiene una instancia de IFoo. 
  4. Miramos el tipo de la referencia obtenida 

Como os podeis suponer lo que este programa muestra por pantalla es: `UnityTest.FooClass`

Unity nos devuelve una instancia de FooClass cada vez que le pedimos una instancia de IFoo, porque así lo especifica el mapping creado con RegisterType.

Hasta ahora todo perfecto: nuestra interfaz IFoo y nuestra clase FooClass no estan ligadas a Unity en ningún modo... vamos a ver como se nos pueden _torcer_ las cosas...

Modificamos la clase FooClass para que tenga dos constructores:

```cs
class FooClass : IFoo 
{
    public FooClass() 
    { 
        Console.WriteLine("FooClass::Default ctor"); 
    }
    public FooClass(BarClass bar) 
    { 
        Console.WriteLine("FooClass::Bar ctor"); 
    }
}
```

El resto del código es igual que antes, con la excepción de que nos aparece una clase nueva (BarClass) que da igual (puede ser vacía, no nos afecta). La pregunta es obvia: ¿que constructor usará Unity para construir un objeto FooClass?

La respuesta es: el que tenga el mayor número de parámetros (¿la razón? Quien sabe...). Unity utilizará el constructor con mayor número de parámetros e inyectará todos los parámetros (llamando internamente al método Resolve) a dicho constructor. &iexcl;Eh, un momento! Esto está muy bien pero... nosotros no hemos definido ningún mapping para BarClass en Unity... ¿No debería quejarse? Pues no: El método Resolve<T> de Unity es capaz de crear cualquier clase para que lo no haya un mapping definido, siempre y cuando T sea una clase, no una interfaz.

Si alguien se pregunta que pasaría si FooClass tuviese dos constructores con un parámetro, entonces Unity se quejará con una excepción parecida a: _The type FooClass has multiple constructors of length 1. Unable to disambiguate_.

Supongamos que queremos que Unity use un constructor en concreto (bien sea porque tenemos varios constructores con mayor número de parámetros o bien porque queremos usar alguno en concreto). En este caso, una solución es aplicar el atributo InjectionConstructor al constructor que queramos que use Unity:

```cs
class FooClass : IFoo 
{
    [InjectionConstructor]
    public FooClass() 
    { 
        Console.WriteLine("FooClass::Default ctor"); 
    }
    public FooClass(BarClass bar) 
    { 
        Console.WriteLine("FooClass::Bar ctor"); 
    }
}
```

Bueno... esto funciona correctamente, pero **en este momento hemos creado una dependencia entre FooClass y Unity**. Si algluna vez nos cambiamos a cualquier otro conenedor de DI, no podemos esperar que entienda el atributo InjectionConstructor, ya que éste, obviamente, es propio de Unity. Así que ahora nuestro código está acoplado a Unity... lo cual no es la solución ideal (en algunos casos).

Por suerte existe una solución que nos permite que nuestra clase FooClass no tenga dependencias contra Unity: cuando especificamos el mapping podemos indicarle que constructor utilizar. Para ello podemos usar el método Configure de Unity:

```cs
uc.RegisterType<IFoo, FooClass>().Configure<InjectedMembers>().
    ConfigureInjectionFor<FooClass>(new InjectionConstructor());
```

Aquí estamos registrando el mapping entre IFoo y FooClass, y configuramos los miembros inyectados para el tipo FooClass para que use el constructor sin parámetros. Si quisieramos usar el constructor con un parámetro BarClass:

```cs
uc.RegisterType<IFoo, FooClass>().Configure<InjectedMembers>().
    ConfigureInjectionFor<FooClass>(
    new InjectionConstructor(new BarClass()));
```

Con esto Unity cada vez que deba crear un FooClass, usará el constructor que acepta un parámetro BarClass y lo invocará con una instancia de BarClass.&nbsp; Unity usará siempre la misma instancia de BarClass para todos los FooClass que cree. Es decir, si tenemos:

```cs
IFoo foo = uc.Resolve<IFoo>();
IFoo foo2 = uc.Resolve<IFoo>();
```

Tanto foo como foo2 serán creados con el constructor que acepta un BarClass pero el BarClass que ambos reciban será el mismo. Incluso aunque no creemos ningún objeto FooClass, el objeto BarClass sí que se crea.

Si queremos que Unity cree cada vez un BarClass para cada FooClass, entonces podemos utilizar el siguiente código:

```cs
uc.RegisterType<IFoo, FooClass>().Configure<InjectedMembers>().
    ConfigureInjectionFor<FooClass>(
    new InjectionConstructor(typeof(BarClass)));
```

En este caso, Unity creará un BarClass nuevo cada vez que deba crear un FooClass... Bueno, esto no es estrictamente cierto: Realmente Unity cada vez llamará a su método `Resolve<BarClass>`, cada vez que deba obtener un BarClass para llamar al constructor de FooClass. Si no tenemos mapping definido, `Resolve<BarClass>` crea un BarClass nuevo cada vez. Pero, si definiesemos el siguiente mapping:

```cs
uc.RegisterInstance<BarClass>(new BarClass());
```

Aquí estamos registrando una instancia de BarClass como singleton dentro de Unity. Por lo tanto todas las llamadas a `Resolve<BarClass>` devolverán el mismo objeto... volvemos a la situación anterior: todos los constructores de FooClass recibirán el mismo BarClass. E igual que antes el BarClass se crea cuando se llama a RegisterInstance, por lo que aunque no creemos ningún FooClass, el BarClass sí que es creado.

Tenemos una variación de este caso. Si registramos BarClass como singleton, usando RegisterType:

```cs
uc.RegisterType<BarClass>(new ContainerControlledLifetimeManager());
```

Ahora BarClass está registrado como singleton,&nbsp; pero no se creará la primera instancia de BarClass hasta que sea necesario. Así, cuando creemos el primer FooClass, Unity creará el BarClass y lo usará como parámetro del constructor. Para crear el segundo FooClass, Unity usará el BarClass previamente creado.

Pero lo importante es que nuestra clase FooClass no depende para nada de Unity, de forma que podemos reutilizarla en cualquier otro proyecto o bien cambiar de contenedor de DI... Que sí, que Unity está muy bien pero tampoco es plan de que nos atemos a él, no????

Saludos!
