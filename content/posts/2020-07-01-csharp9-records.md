---
title: "C#9 Records: Mis impresiones"
author: eiximenis
description: "He estado jugando un poco con la nueva de C#9 llamada records y esas son mis impresiones."
date: 2020-07-01T18:00:00
draft: false
categories:
  - netcore
---

Estos días, preparando la charla del [Talks4Kids](https://www.talks4kids.com/) he estado jugando un poco con el futuro C#9 y su característica más destacada: los records. He aquí mis impresiones, pero **antes un aviso**:

{{< alert danger>}}Este post se basa en una _preview_. En la versión final las cosas pueden cambiar (o no).{{</ alert >}}

Para jugar con C#9 puedes usar la última versión que haya del .NET5 SDK, junto con la última versión que haya de Visual Studio Preview. A la hora de escribir este post la última versión del SDK de .NET5 es `5.0.100-preview.6.20318.15` y la de VS es la `16.7.0 Preview 3.1`, pero tiene un error que lo invalida para hacer algunas pruebas. Pero, para probar cosillas, **la mejor opcion de lejos es [LinqPad6 Beta](https://www.linqpad.net/LINQPad6.aspx) y [configurarlo para usar las dayli builds de Roslyn](https://www.linqpad.net/LINQPad6.aspx#beta)**. De ese modo tienes acceso a lo último de lo último (incluso a características que no están implementadas todavía en VS Preview).

Y ahora sí... ¡Al tajo!

## Records

Los records son la gran novedad de C#9. Es más una amalgama de ideas que una característica por sí sola, pero bueno... intentaré desgranar (lo que he visto) junto con mis opiniones al respecto.

Se trata de un nuevo tipo de datos (además de delegates, interfaces, structs y clases) que **nos permite definir de forma fácil tipos con semántica de valor**. Definir tipos con semántica de valor en C# era posible sin records, pero todo el trabajo corría de nuestra parte. Ahora con los records compartimos el trabajo con el compilador.

```csharp
public record Beer {
	public string Name {get; set;}
	public double Abv {get; set;}
}
```

Para definir un record se usa la nueva palabra clave `record` (en versiones anteriores era `data class`). ¿Y qué implica `record`? Pues lo siguiente:

1. El tipo `Beer` **obtiene un método Equals generado por el compilador** que compara por valor:
```csharp
var mahou = new Beer() {Name = "Mahou", Abv=4.3};
var mahou2 = new Beer() {Name = "Mahou", Abv=4.3};
var equals = mahou.Equals(mahou2);                    // equals vale true
```
2. Del mismo modo el tipo `Beer` obtiene un `GetHashCode` generado que es coherente con el `Equals`.
3. Se define un método que clona el objeto, pero ese método no se puede llamar directamente. Más sobre eso, luego.

Al margen de eso, la palabra clave `record` no parece implicar nada más. Ahora empecemos a desgranar más cosillas...

## ¿Tipos por referencia o por valor?

Los records parecen ser tipos por referencia, por debajo hay clases no estructuras. El CLR no es consciente de que un tipo era un record, es todo en tiempo de compilación. Así el siguiente código:

```csharp
public record Beer
{
    public int Name { get; set; }
}
```

Se traduce en algo parecido a (elimino el código de los métodos):

```csharp
public class Beer : IEquatable<Beer>
  {
    [SpecialName]
    public virtual Beer <>Clone();
    protected virtual Type EqualityContract {get;}
    protected virtual Type get_EqualityContract();
    public int Name { get; set; }
    public override int GetHashCode();
    public override bool Equals([In] object obj0);
    public virtual bool Equals([In] Beer obj0);
    protected Beer([In] Beer obj0);
    public Beer();
    bool IEquatable<Beer>.Equals(Beer other);
  }
```

Como puedes ver se usa `class` para implementar un record. Por lo tanto una variable de un tipo record puede contener el valor `null`. Eso es algo curioso: **se usa un tipo por referencia para implementar un tipo de datos que tiene semántica de valor**. No es la primera vez que ocurre eso en C# (hola `System.String`). La decisión la podría entender en el contexto de C# 7.1 y anteriores. Pero ahora que tenemos `in` para pasar _value-types_ por referencia de forma segura y transparente, no termino de ver el motivo de usar siempre una clase en lugar de una struct (seguro que lo hay y se me escapa algo). En la sintaxis anterior se podía elegir (`data class` o `data struct`) pero ahora no parece que haya la opción de que puedas tu decidir cuando implementar un record usando una struct (siempre se usan clases). Por supuesto eso implica que no puedes pasar records a métodos genéricos que tengan la restricción `struct` sobre el tipo genérico.

Otra consideración es que **el compilador no sobrecarga `operator==`**. Eso a mi me convence muy poco. Si la semántica del tipo es por valor, el operador `==` debería estar sobrecargado, al igual que lo está para las cadenas por poner un ejemplo. Así (siendo `Beer` un record):

```csharp
var mahou = new Beer() {Name = "Mahou", Abv=4.3};
var mahou2 = new Beer() {Name = "Mahou", Abv=4.3};
var equals2 = mahou == mahou2;          // equals2 vale false
```

No sé, pero **que `==` y `Equals` me devuelvan valores distintos no me gusta nada en tipos que tienen semántica de valor**. La semántica de valor implica precisamente que la comparación es por valor y que la referencia no me importa nada.

## ¿Inmutables o no?

**Un record por si mismo no es inmutable**, aunque la verdad es que hay mecanismos para hacerlos inmutables de forma bastante sencilla. Para ello se apoyan en otra característica novedosa de C#9, las llamadas _init-only properties_ (propiedades init para abreviar). Una propiedad init es una propiedad que **solo se puede establecer su valor en el constructor o bien usando la sintaxis de inicialización de objeto**:

```csharp
public record Beer {
	public string Name {get; init;}
	public double Abv {get; init;}
}
```

Observa el uso de `init` en lugar de `set`. Ahora podemos usar el siguiente código:

```csharp
var mahou = new Beer() {Name = "Mahou", Abv=4.3};
mahou.Abv = 4.0;    // ERROR, no se puede modificar una propiedad init.
```

Así, usando `init` en lugar de `set`, podemos conseguir que nuestros records sean inmutables **sin necesidad de empezar a poner parámetros en el constructor**. Es una idea muy elegante y que me gusta mucho. Las propiedades init son aplicables también a clases normales (y a structs) y una propiedad init pueda estar respaldada por un _backing field_ que sea `readonly` (ya que sabemos que una vez establecido al iniciar el objeto no se puede modificar, con lo que encaja con la semántica de `readonly`).

## Records posicionales

Como este caso se presupone (el querer un record inmutable) muy común **se puede usar una sintaxis distinta para definir records**:

```csharp
public record Beer (string Name, double Abv);
```

Eso es lo que conocemos como record posicional. Es parecido al caso anterior, salvo que eso **genera un constructor con dos parámetros (uno por cada propiedad)**:

```csharp
var mahou = new Beer("Mahou", 5.4);
mahou.Abv = 4.0;    // Error: Abv es una init property.
```

Observa pero, que eso sí que funciona (a pesar de que no tiene mucho sentido lógico):

```csharp
var mahou = new Beer("Mahou", 5.4) { Abv=7.2 };
mahou.Abv;          // 7.2
```

El valor final de `Abv` es `7.2` ya que al ser `Abv` una propiedad es posible establecerla en la inicialización del objeto. Creo que el compilador debería avisar de esto, ya que es digamos... raro.

Si tenemos un record posicional, **el constructor por defecto deja de estar disponible**. Así, ahora ya no podemos usar `new Beer()` para crear una cerveza, hay que pasar siempre ambos parámetros. Se pueden combinar ambos estilos usando valores por defecto en el record posicional:

```csharp
public record Beer (string Name="", double Abv=default);
var mahou = new Beer("Mahou", 5.4);
var estrella = new Beer("Estrella") { Abv=3.2 };
var vollDamm = new Beer(Abv: 7.4) { Name = "Voll Damm" };
```

Ese mecanismo, pese al coñazo de tener que declarar esos valores por defecto, me parece una manera muy flexible de declarar tipos inmutables.

## Definir constructores adicionales en records posicionales

Se pueden definir constructores **adicionales** en los records posicionales. Recuerda que todo record posicional tiene ya un constructor definido en base a los tipos especificados y este constructor **se mantiene incluso aunque definamos constructores adicionales**. Cuando definimos un constructor adicional **debemos SIEMPRE llamar antes al constructor generado usando `this`**:

```csharp
public record Beer(string Name) 
{
	public Beer(int n) : this(n.ToString()) {}
}

var b2 = new Beer("estrella");      // b2.Name == "estrella".
var b3 = new Beer(10);              // b2.Name = "10"
```

Observa como a pesar de definir el constructor que soporta un `int`, el constructor generado con la `string` sigue estando disponible.

## Records posicionales y tuplas

Los records posicionales **soportan deconstrucción** de forma automática:

```csharp
public record Beer (string Name, double Abv);
var mahou = new Beer("Mahou", 5.4) { Abv=7.2 };
var (_,abv) = mahou;
```

Quizá has observado que eso **muy parecido a usar una tupla**:

```csharp
var estrella = (Name: "Estrella", Abv: 4.5);
var (_, abv2) = estrella;
```

En efecto, los records posicionales y las tuplas comparten varios aspectos (en ambos casos el método `Equals` compara por valor), aunque también hay algunas diferencias:

1. Las propiedades de una tupla son de lectura y escritura y las de un record posicional son propiedades init. 
2. El soporte para `with` es solo para records (ver a continuación)
3. El operador `==` en tuplas también compara por valor (no así en los records).
4. Las tuplas se implementan usando tipos por valor (`ValueTuple<>`) mientras que los records se implementan con clases.

## Uso de with

Un patrón muy común en objetos inmutables es crear otro objeto que sea copia de un anterior con alguna modificación. Por ejemplo en el caso de las cadenas:

```csharp
var trimmed = name.Trim();
```

Eso, como ya sabrás, no modifica `name` si no que `trimmed` es una copia del valor de name con una modificación. Bien, los records tienen soporte explícito del lenguaje para esos casos:

```csharp
var mahou = new Beer() {Name = "Mahou", Abv = 4.3};
var mahou5 = mahou with { Name = "Mahou 5 estrellas"};
```

Observa la construcción `mahou with`. Eso lo que hace es aplicar las modificaciones de propiedades que se indiquen dentro del bloque `with` a un objeto nuevo que se crea clonando a `mahou`. Es decir, se clona el objeto original, se aplica el código del bloque `with` sobre el clon y se devuelve el valor clonado. **¿Recuerdas cuando comenté que los records tenían un método para clonarlos, pero que no se podía llamar directamente?** Pues ese método es invocado a través de `with`. Y eso nos lleva a un concepto nuevo: el constructor de copia.

Se puede usar `with` tanto con records normales (llamados "nominales") como con records posicionales.

## El constructor de copia

¿A que si conoces C++ has levantado una ceja? En C++ el constructor de copia es un concepto que ya tiene bastantes años y que ahora aterriza a C#. Como su nombre indica **se trata de un constructor que sirve para devolver una copia** de un objeto que recibe como parámetro. Lo genera automáticamente el compilador para todos los records:

```csharp
protected Beer([In] Beer obj0);
```

Este es el constructor de copia. Recibe como parámetro una referencia al objeto a clonar y devuelve el nuevo objeto, copia del pasado como parámetro. Este constructor es llamado por el método `<>Clone`, que es el usado por `with` para clonar el record. El método `<>Clone` es algo que escapa a nuestro control (su nombre no es un identificador válido en C#) pero el constructor de copia lo podemos redefinir:

```csharp
public record Beer {
	public string Name {get; init;}
	public double Abv {get; init;}
	public Beer(){}
	protected Beer (Beer other) {Name = other.Name.ToUpper(); Abv=other.Abv;}
}

var beer = new Beer {Name="mahou", Abv=5.3};
var beer2 = beer with {Abv=3.2};
beer2.Name    // MAHOU
```

Eso nos permite definir nuestra propia copia en el caso de que la generada por el compilador (una `shallow copy`) no sea suficiente. Observa lo siguiente:

* No se usa `override` para redefinir el constructor de copia. Simplemente se define. Eso es lógico ya que **no se puede definir un constructor como `virtual`**.
* Al definir explícitamente el constructor de copia, **el constructor por defecto deja de estar generado**, por lo que hay que definirlo explícitamente. Eso es bastante peñazo, aunque es coherente con el lenguaje (siempre que definimos un constructor, el constructor por defecto deja de estar generado). Pero igual en records se podría reconsiderar esa opción.

En un record posicional también puedes definir el constructor de copia, pero recuerda que siempre debes usar `this` para llamar al constructor generado:

```csharp
public record Beer(string Name) 
{
	protected Beer (in Beer other) : this(other.Name) {}
}
```

## Más sobre el constructor de copia

Imagina el siguiente código:

```csharp
public record Beer
{
    private string _name;
    public string Name
    {
        get => _name;
        set
        {
            Console.WriteLine("SETTER!");
            _name = value;
        }
    }
    public Beer() {}
    protected Beer(in Beer other) { Name = other.Name; }
}

var beer = new Beer { Name = "mahou" };
var beer2 = beer with { Name = "mahou2" };
```

**¿Cuantas veces se imprimirá `SETTER!` en la consola?**. Si has dicho 3 estás en lo cierto:

1. Cuando se asigna `Name` de `beer`
2. Cuando se ejecuta el constructor de copia por el `with`
3. Cuando se asigna el `Name` de `beer2`.

Y si ahora **eliminamos el constructor de copia**? Cuantas veces se imprimrá `SETTER!` por la pantalla? La intuición te puede hacer decir 3, ya que el constructor de copia generado se supone que debe asignar las propiedades ¿verdad? Pues no, se imprime solo 2 veces. El constructor de copia generado por el compilador no usa las propiedades. Accede directamente a los _backing fields_:

```
IL_0000:  ldarg.0     
IL_0001:  call        System.Object..ctor
IL_0006:  ldarg.0     
IL_0007:  ldarg.1     
IL_0008:  ldfld       UserQuery+Beer._name
IL_000D:  stfld       UserQuery+Beer._name
IL_0012:  ret          
```

Lo mismo ocurre para las propiedades autogeneradas: el compilador pasa de los _setters_ y accede a los campos directamente.

## Mutabilidad, Equals y GetHashCode

Esto nos retrae a la primera versión de .NET. Por definición `GetHashCode` tiene un problema con los objetos mutables: Si para calcular el valor del hash devuelto usas los campos mutables, puede ocurrir que el hash del objeto cambie. Si el objeto era usado como clave en una colección eso generará errores.

La documentación de Microsoft [se cura bastante en salud](https://docs.microsoft.com/en-us/dotnet/api/system.object.gethashcode?view=netcore-3.1#notes-to-inheritors), cuando pone literalmente: `You can ensure that the hash code of a mutable object does not change while the object is contained in a collection that relies on its hash code.`. Pero este `You` se refiere **al usuario del tipo que es quien sabe cuando lo ha añadido como clave en una hash table** no al creador del tipo que no puede controlar eso. La otra opción que se recomienda es calcular el hash solo basándose en campos inmutables. Esa es la opción que deberías considerar siempre. Pero... ¿qué hace la implementación por defecto de los records?

Bueno, pues **parece que los record usan todos los campos para calcular el valor de `GetHashCode()`**:

```csharp
public record Beer {
	public string Name {get; init;}
	public int MutableAbv {get; set;}
}

var beer = new Beer() {Name="estrella", MutableAbv=4};
var hash1 = beer.GetHashCode();
var beer2 = new Beer() {Name="estrella", MutableAbv=7};
var hash2 = beer2.GetHashCode();
var sameHash = hash1 == hash2;        // false :(
```

Claro que usar solo los campos inmutables plantea sus dudas: ¿qué hacemos si el record NO tiene campos inmutables?

Y hasta ahí lo que he podido explorar sobre los records. ¿A vosotros, qué opinión os merecen?

¡Un saludo!
