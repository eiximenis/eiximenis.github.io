---
title: "NetCore 3 - NativeLibrary: ¡p/invoke a tope!"
author: eiximenis
description: NetCore ha soportado P/Invoke desde siempre a través de [DllImport] que funciona tanto en Mac, como en Linux como en Windows. Pero DllImport tiene algunas limitaciones que pueden ser frustrantes. Por suerte con NetCore 3 tenemos NativeLibrary que viene a solventar esos problemas de una vez por todas.
date: 2020-01-31T18:20:00
categories:
  - netcore
---

¿Conoces P/Invoke? Se trata de la posibilidad de **realizar llamadas nativas al sistema operativo** desde código .NET. Para habilitarlo debemos definir en C# una función externa (usando la palabra clave `extern`) y luego usar `[DllImport]` para indicarle al _CLR_ en qué librería del SO se encuentra esta función implementada. Por supuesto **eso genera código que no es _cross platform_**, ya que estás llamando explícitamente a una librería en concreto del SO. Veamos un ejemplo:

```csharp 
static class ConsoleNative {
    [DllImport("kernel32.dll", SetLastError = true)]
    public static extern COORD GetConsoleFontSize(IntPtr hConsoleOutput, uint nFont);
}
```

Este código define la función `ConsoleNative.GetConsoleFontSize` pero el modificador `extern` indica que dicha función está implementada "fuera de .NET" y es por ello que no tiene código. El atributo `[DllImport]` indica donde está implementada esa función (en `kernel32.dll`). Una de las cosas **más complicadas en P/Invoke y que NO voy a tratar en este _post_** es como "mapear" de los tipos nativos a los tipos C#. Es decir, si miras la documentación de Win32, la función `GetConsoleFontSize` está definida como:

```c
COORD WINAPI GetConsoleFontSize(
  _In_ HANDLE hConsoleOutput,
  _In_ DWORD  nFont
);

```

Saber que `HANDLE` se transforma en `IntPtr` y que `DWORD` es `uint` es parte de la dificultad de P/Invoke. Hay algunos recursos que ayudan, como [pinvoke.net](https://pinvoke.net/), pero no en todos los casos y en general es necesario tener cierto conocimiento de como son los parámetros en la función nativa, pero esto se presupone si vas a llamarla.

## P/Invoke con .NET Core

El código anterior presentado funciona tanto en .NET Framework tradicional, como con .NET Core. P/Invoke forma parte de NetStandard, por lo que puedes compilar una librería NetStandard que use P/Invoke sin problemas. En el caso anterior, si solo tenemos este código, en una librería NetStandard, esta no será multiplataforma, si no solo para Windows. Si en un Linux se ejecuta la aplicación, esta explotará al ejecutar el método `ConsoleNative.GetConsoleFontSize` ya que `kernel32.dll` no existe en Linux.

Bueno... de hecho en Linux no hay manera de obtener el tamaño de la fuente actual, ya que eso depende completamente del emulador de terminal usado. En Windows las cosas están un poco más liadas y el concepto de emulador de terminal es un poco difuso. Eso ilustra las dificultades de hacer aplicaciones _cross platform_ en según que escenarios. Pero, busquemos otro ejemplo. En Windows si quiero mover el cursor a una posición de la consola puedo usar:

```csharp
[DllImport("kernel32.dll", SetLastError = true)]
public static extern bool SetConsoleCursorPosition(IntPtr hConsoleOutput, COORD dwCursorPosition);
```

Por su parte el equivalente Linux es el siguiente:

```csharp
[DllImport ("libncursesw.so.5")]
public static extern int move (int line, int col);
``` 

De nuevo, otra diferencia importante entre Windows y Linux: en Linux hay chorrocientas maneras de mover el cursor a una posición (tantas como tipos de terminales existen, pero esa es otra historia). Para no lidiar con ello, se suele usar [NCurses](https://es.wikipedia.org/wiki/Ncurses) que es una librería que nos abstrae de todo ese lío. Pero claro, NCurses puede no venir instalada y si lo está a saber en qué version. Bienvenido al infierno.

En mi ejemplo dependo de la libreria `libncursesw.so.5` que es NCurses en su versión 5. 

Eso plantea un problemón, porque la definición de `move` es la misma con independencia de la versión de `ncurses` pero al poner la versión (`.5`) en `[DllImport]` quedo atado a esta versión y solo funcionará en ordenadores que tengan esta versión. Para evitar esto, en algunos casos, se crea un enlace simbólico del fichero `xxxx.so` al fichero `xxxx.so.v` (donde `v` es, generalmente, la última versión instalada). Esto permite que si tu código carga el fichero `xxxx.so` se utilice el que esté apuntado por dicho enlace simbólico. De todos modos, eso depende de cada caso (no es algo que se haga para todos los casos). Otro uso de esos enlace simbólicos es "que siempre se use la última versión compatible". Así, p. ej. en mi máquina hay un enlace simbólico de `libncurses.so.5` a `libncurses.so.5.9`.

El hecho de que los ficheros `.so` en Linux estén versionados ayuda a evitar el _Dll Hell_ que hay en Windows ya que podemos tener tantas versiones como sean necesarias. A cambio los ejecutables "quedan atados" a una versión, aunque el uso de enlaces simbólicos permite aplicar "parches" (cambiando el enlace simbólico a otro binario).

`DllImport` aplica un poco de lógica sobre el nombre que nosotros pongamos. Lo buscará también añadiendo el sufijo `.so`, el prefijo `lib` o ambos. Así, si yo uso `[DllImport("libncursesw.so.")]` el _runtime_ de netcore buscará los siguientes ficheros:

* `libncursesw.so.6.so`
* `liblibncursesw.so.6.so`
* `libncursesw.so.6`
* `liblibncursesw.so.6`

## P/Invoke _cross platform_

Para realizar código P/Invoke que sea _cross platform_ debes definir las funciones N veces (donde N probablemente sea 3, una para Windows, otra para Linux y una tercera para MacOS). Tendrás tres funciones `extern` y luego desde cada plataforma debes llamar a la que toque. Una forma sencilla de gestionar eso es a través de interfaces:

```csharp
interface IConsole {
    void SetCursorTo(int x, int y);
}

class WindowsConsole : IConsole {
    public void SetCursorTo(int x, int y) => Native.Win32.SetConsoleCursorPosition(_stdout, new COORD(x, y));
}
class LinuxConsole : IConsole {
    public void  SetCursorTo(int x, int y) => Native.Linux.move(x, y);
}

class Native {
    public class Win32 { /* DllImports de Win32 */ }
    public class Linux {/* DllImports de Linux */ }
}
```

Luego, según el SO en el que se ejecute la aplicación, registras la clase `IConsole` correspondiente en el sistema de inyección de dependencias.

## NativeLibrary (.NET Core 3)

.NET Core 3 viene con `NativeLibrary` un añadido interesante a P/Invoke. El principal problema de `DllImport` es que el nombre de la librería que enlazamos es fijo y constante en tiempo de compilación. Pues bien, `NativeLibrary` viene a cambiar eso, permitiendo especificar reglas que le indiquen al _framework_ que librerías debe buscar y enlazar. Para ello hay que usar el método estático `NativeLibrary.DllImporter` y especificar un delegado que indique que librería enlazar. Veamos un ejemplo:

```csharp
static class LinuxDllResolver
{
    public static IntPtr ResolveLibrary(string libraryName, Assembly assembly, DllImportSearchPath? searchPath)
    {
        Console.WriteLine($"Lib requested: {libraryName}");
        Console.WriteLine("Let's try first v6");
        var ok = NativeLibrary.TryLoad($"{libraryName}.6", out var handle6);
        if (!ok)
        {
            Console.WriteLine("Ups, version 6 not found. Let's try 5");
            return NativeLibrary.TryLoad($"{libraryName}.5", out var handle5) ? handle5 : IntPtr.Zero;
        }

        return handle6;
    }
}
```

El método `ResolveLibrary` es el que se encarga de resolver la librería a enlazar. Recibe como parámetro el nombre de la librería en el `DllImport` y puede usar este nombre para determinar que librería usar. En este caso sigo una lógica sencilla: Intento cargar primero la versión 6 (añadiendo `6` al nombre) y si no me funciona, pruebo con la versión `5`. Aquí es de gran ayuda el método `NativeLibrary.TryLoad` que intenta cargar una libreria determinada. Si la versión `5` no existo, devolvemos `IntPtr.Zero` que hace que se ejecute el comportamiento por defecto de `DllImport`.

Ahora solo debemos usar el método `NativeLibrary.SetDllImportResolver` en el método `Main` para indicar que queremos usar esa lógica para determinar que librería cargar:

```csharp
NativeLibrary.SetDllImportResolver(typeof(Program).Assembly,LinuxDllResolver.ResolveLibrary);
```

Ahora puedo tener un `DllImport` tal y como sigue:

```csharp
[DllImport("libncursesw.so")]
public static extern int move(int line, int col);
```

Y, al ejecutar este código en mi máquina, la salida es la siguiente:

```
Lib requested: libncursesw.so
Let's try first v6
Ups, version 6 not found. Let's try 5
Hello World!
```

Otra opción es poder usar nombres genéricos de librerías (p. ej. un `DllImport("ncurses")`) y que eso se resuelva a `libncursesw.so.x` en Linux y a `libncurses.x.dylib` en Mac. Esto es útil porque en muchos casos las librerías de Mac y Linux son idénticas, pero solo difieren en el nombre del fichero.

Y tranquilo, a nivel de rendimiento, debes tener presente que tu _DllResolver_ se ejecuta solo la primera vez que se usa el método con `DllImport`, por lo que si realizas 1000 llamadas al mismo método tu _DllResolver_ se ejecuta solo la primera vez. Eso sí, si llamas a dos métodos distintos, aunque tengan el mismo `DllImport` se ejecutará dos veces el _DllResolver_. Este código ejecuta dos veces el _DllResolver_ (uno por `move` otro por `getch`):

```csharp

static void Main(string[] args)
{
    NativeLibrary.SetDllImportResolver(typeof(Program).Assembly,LinuxDllResolver.ResolveLibrary);
    Native.move(2, 2);
    Console.WriteLine("Hello World!");
    Native.getch();
    Native.move(2, 1);
    Console.WriteLine("Hello World!");
    Native.getch();
}

static class Native
{

    [DllImport("libncursesw.so")]
    public static extern int move(int line, int col);

    [DllImport("libncursesw.so")]
    public static extern int getch();
}
```

Debes tener presente que solo se puede tener un _DllResolver_, intentar registrar más de uno da un error.

En resúmen: `NativeLibrary` nos da una gran flexibilidad que antes no teníamos y es la de aplicar lógica propia para decidir qué librerías debe cargar `DllImport`. En según que escenarios, puede ser algo extremadamente interesante.