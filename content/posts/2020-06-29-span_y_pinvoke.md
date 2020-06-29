---
title: "Span<T> y P/Invoke"
author: eiximenis
description: "Span<T> es una ref struct que habilita de forma fácil escenarios nuevos permitiendo usar la pila donde antes solo podíamos usar el heap. Eso redunda en menos reservas del heap y por lo tanto menos presión sobre el GC."
date: 2020-06-29T12:00:00
draft: false
categories:
  - netcore
---

Uno de los movimientos más nuevos en .NET Core consiste en lo que podemos llamar "zero-allocation code" o código sin reservas. Eso consiste en tener código que **evite al máximo (hasta llegar al ideal de eliminar) las reservas de objetos en el heap**. Para ello es necesario que tanto el lenguaje como el entorno de ejecución lo permitan. Pero... ¿por qué es beneficioso evitar las reservas en el heap?

## Predictibilidad

La razón principal es **la predictibilidad de tu código**, en este caso predictibilidad en el tiempo de ejecución, claro. No se trata de qué tu código funcione más rápido, se trata de que minimizas (idealmente eliminas) un elemento impredecible: el Garbage Collector. Nunca sabes cuando se va a ejecutar y no controlas cuando se va a demorar en su tarea. Es cierto que con cada versión de .NET el GC es algo mejor, más rápido, menos intrusivo, más optimizado. Pero al final tiene una tarea que hacer y tarda cierto tiempo en hacerla y partes de esa tarea requieren que tu código no se esté ejecutando. 

![La famosa frase de Donald Knuth: "premature optimization is the root of all evil"](/images/posts/2020-06-29-donald-knuth.jpg)

De todos modos, eso es **una optimización prematura** y por lo tanto, seguramente, causará más problemas de los que arreglará. La gran mayoría de programas (casi su totalidad) no deberían preocuparse de eso, ya tienen bastante con sus otros problemas (que si APIs, que si BBDD, que si sistemas de ficheros). Así que hazle caso al bueno de [Donald](https://es.wikipedia.org/wiki/Donald_Knuth) y no te preocupes de eso. Solo las librerías, y aquellas muy especializadas, deberían preocuparse. Que sé yo, si eres [Javi Cantón](https://twitter.com/jcant0n) y te por **desarrollar un motor 3D** (como [Wave Engine](https://waveengine.net/)) probablemente te interesa reducir al máximo las reservas del heap, ya que quieres proporcionar unos fps constantes, que solo dependan de la complejidad de la escena. No te interesa que los fps caigan cada cierto tiempo, durante un rato, porque el GC está actuando. Lo mismo si haces un servidor web: quieres que el proceso de peticiones dependa, básicamente, del código a ejecutar en cada petición y que el número de rps no caiga porque el GC ha decidido liberar memoria. **Necesitas esa predictibilidad en tiempo de ejecución** y para ello debes adaptar tu código.

No nos engañemos: .NET ha abrazado siempre el GC y el movimiento "zero-allocation" es muy reciente, ya que hasta hace poco ni lenguaje (C#) ni plataforma estaban muy bien preparados para él. El punto de inflexión vino propiciado por la necesidad de aumentar el rendimiento en netcore, lo que hizo que Microsoft desarrollase un conjunto de técnicas y herramientas y las pusiera a disposición de los desarrolladores, o sea a disposición nuestra.

## Span&lt;T&gt;

Una de esas herramientas, es `Span<T>`, que a pesar de que es `netstandard2.0` su verdadero potencial es solo aprovechable en netcore ya que requiere colaboración del runtime para funcionar. Usar `Span<T>` en Full Framework es ciertamente posible y hasta recomendable, y además nos permite tener una sola versión _netstandard_ de nuestra librería, pero su potencial total en cuanto a rendimiento solo se obtiene con netcore 2.1 y posteriores.

Este tipo es una `ref struct`, que por si eso te suena a chino, se trata de una estructura (por lo tanto tipo por valor, habitualmente almacenado en la pila) pero que **solo puede estar almacenado en la pila**. Seguramente sabrás que hay muchos motivos que pueden hacer que una struct termine almacenada en el heap (p. ej. ser miembro de una clase), pues bien en este caso no podrás usar una `ref struct`. Cualquier causa que implique que tu `ref struct` pase por el heap (eso incluye boxing) hace que el código no compile. Por lo general lo único que puedes hacer con una `ref struct` es crear objetos locales a una función y pasarlas como parámetro a otras funciones... a otras funciones que, por cierto, ni sean `async` ni sean lambdas.

Un `Span<T>` representa, más o menos, un puntero a un conjunto de datos (de tipo `T`) de cierta longitud. Nos da una visión unificada que nos independiza del origen de esos datos (algo parecido a un stream). En este caso por origen entendemos la zona de memoria donde están esos datos: la pila, el heap o la memoria no manejada.

## Punteros en P/Invoke

P/Invoke es la capacidad de llamar a métodos nativos (C/C++) que están en librerías nativas (DLLs en windows, o ficheros `.so`/`.dylib` en Linux/MacOS) desde netcore. Es un arte arcano, en el cual uno debe traducir la firma de los métodos nativos a una firma compatible en .NET. Eso requiere entender ambos lados con sus particularidades: el código nativo C/C++ estará plagado de `#define` y `typedef` que deberás ir escrutando para ir traduciendo los tipos finales reales a los equivalentes en .NET (súmale eso que el mismo código fuente C/C++ generará tipos distintos en función de la arquitectura final compilada). Si solo vas a invocar funciones de la API de Win32 las cosas se tranquilizan un poco porque la API usa nomenclaturas bastante consistentes y además [muy bien documentadas](https://docs.microsoft.com/en-us/windows/win32/winprog/windows-data-types). Linux y MacOS no son tan consistentes (o mi conocimiento es menor que también puede ser) y tocará pelearte con la documentación en cada caso.

El caso que quiero mencionar en este post es el de pasar **un puntero a un array de valores** a una función nativa. En el ejemplo que voy a poner la función es de Win32, [concretamente esta](https://docs.microsoft.com/en-us/windows/console/readconsoleinput):

```c
BOOL WINAPI ReadConsoleInput(
  _In_  HANDLE        hConsoleInput,
  _Out_ PINPUT_RECORD lpBuffer,
  _In_  DWORD         nLength,
  _Out_ LPDWORD       lpNumberOfEventsRead
);
```

Ahí el tipo `PINPUT_RECORD` es un array de estructuras (de tipo [`INPUT_RECORD`](https://docs.microsoft.com/en-us/windows/console/input-record-str`) que es rellenada por la función y que contiene la información de los distintos eventos pendientes de leer en el terminal. La definición de `INPUT_RECORD` es tal y como sigue:

```c
typedef struct _INPUT_RECORD {
  WORD  EventType;
  union {
    KEY_EVENT_RECORD          KeyEvent;
    MOUSE_EVENT_RECORD        MouseEvent;
    WINDOW_BUFFER_SIZE_RECORD WindowBufferSizeEvent;
    MENU_EVENT_RECORD         MenuEvent;
    FOCUS_EVENT_RECORD        FocusEvent;
  } Event;
} INPUT_RECORD;
```

Lo que traducido a C# nos queda como:

```csharp
[StructLayout(LayoutKind.Explicit)]
internal struct INPUT_RECORD
{
    [FieldOffset(0)]
    public ConsoleEventTypes EventType;
    [FieldOffset(4)]
    public KEY_EVENT_RECORD KeyEvent;
    [FieldOffset(4)]
    public MOUSE_EVENT_RECORD MouseEvent;
    [FieldOffset(4)]
    public WINDOW_BUFFER_SIZE_RECORD WindowBufferSizeEvent;
    [FieldOffset(4)]
    public MENU_EVENT_RECORD MenuEvent;
    [FieldOffset(4)]
    public FOCUS_EVENT_RECORD FocusEvent;
};
```

En mi versión original del código, la función estaba definida en C# tal y como sigue:

```csharp
[DllImport("kernel32.dll", EntryPoint = "ReadConsoleInputW", CharSet = CharSet.Unicode)]
public static extern bool ReadConsoleInput(IntPtr hConsoleInput, [Out] INPUT_RECORD[] lpBuffer, uint nLength, out uint lpNumberOfEventsRead);
```

Y la llamada al método era tal y como sigue:

```csharp
var buffer = new INPUT_RECORD[numEvents];
ConsoleNative.ReadConsoleInput(_hstdin, buffer, (uint)buffer.Length, out var eventsRead);
```

Ese código se ejecutaba unas 60 veces por segundo, y aunque la mayoría de veces `numEvents` era 0 (y ya no se hacía nada), muchas otras veces el valor de `numEvents` era 1, por lo que terminaba creando un array de un elemento (en el heap, como todos los arrays) y luego llamando a la función nativa. Este código generaba unas trazas de reservas del heap como la siguiente:

![Captura del profiler: 498 objetos INPUT_RECORD[] en el heap totalizando 21.972 bytes](/images/posts/2020-06-29-heap-allocs.png)

Mi idea era ver si podía eliminar todas esas reservas, usando un `Span<T>` que apuntase a un array en la pila.

## Usando Span<T> con P/Invoke

Crear el array en la pila es posible gracias a que `Span<T>` se entiende con `stackalloc`:

```csharp
Span<INPUT_RECORD> buffer = stackalloc INPUT_RECORD[(int)numEvents];
```

¡Eso es simplemente genial! Tengo un apuntador (`buffer`) a un array, pero ese array **no está en el heap, sino en la pila** gracias al uso de `stackalloc`. Lo que es genial no es el uso de `stackalloc`, que existe desde que .NET es .NET, si no el hecho de que **`Span<T>` me permite usar `stackalloc` sin necesidad de usar un contexto `unsafe`**.

Antes de `Span<T>`, si quería usar `stackalloc` me veía obligado a declarar un contexto `unsafe`:

```csharp
unsafe
{
  INPUT_RECORD* pBuffer = stackalloc INPUT_RECORD[(int)numEvents];
}
```

Gracias a `Span<T>` puedo usar `stackalloc` sin necesidad de `unsafe`, pero eso es solo la mitad de la historia.

La otra mitad consiste en **pasar este `Span<T>` a la función nativa, pero P/Invoke no está preparado para ello**. Es una lástima pero P/Invoke no entiende de `Span<T>`. Hay que convertir ese `Span<T>` en otra cosa.

## Primer intento: Obtener una referencia al primer elemento del Span

Esa fue la primera opción que pasó por mi cabeza: Obtener una referencia **al primer elemento del Span<T>** y pasar esa referencia al método nativo. Para ello se puede usar `MemoryMarshal.GetReference()` que dado un `Span<T>` devuelve una `ref T` al primer elemento:

```csharp
Span<INPUT_RECORD> buffer = stackalloc INPUT_RECORD[(int)numEvents];
ref var pBuffer = ref MemoryMarshal.GetReference(buffer);
ConsoleNative.ReadConsoleInput(_hstdin, ref pBuffer, numEvents, out var eventsRead);
```

Por supuesto hay que modificar el método nativo:

```csharp
[DllImport("kernel32.dll", EntryPoint = "ReadConsoleInputW", CharSet = CharSet.Unicode)]
public static extern unsafe bool ReadConsoleInput(IntPtr hConsoleInput, ref INPUT_RECORD lpBuffer, uint nLength, out uint lpNumberOfEventsRead);
```

Esto parecía funcionar... pero solo en el caso que hubiese un solo evento. Si había más de uno, la función nativa daba un error (`Attempted to read or write protected memory. This is often an indication that other memory is corrupt.`). 

Probé varias combinaciones tales como usar `out` en lugar de `ref` y declarar el parámetro con el atributo `[Out]` en la función nativa, pero todas sin éxito. 

## Segundo intento: Usar un Span&lt;byte&gt;

Honestamente no entendía porque la solución anterior no funcionaba, la única razón que se ocurrió es que el _marshaller_ cuando pasaba la memoria se hiciese algún lío, quizá la estructura `INPUT_RECORD` tenía algo o le faltaba algún atributo de P/Invoke. No sé, para probar se me ocurrió **pasar a la función nativa un `Span<byte>` que apuntaase a la misma dirección que el `Span<INPUT_RECORD>`**:

```csharp
Span<INPUT_RECORD> buffer = stackalloc INPUT_RECORD[(int)numEvents];
var byteBuf = MemoryMarshal.AsBytes(buffer);
ConsoleNative.ReadConsoleInput(_hstdin, ref MemoryMarshal.GetReference(byteBuf), numEvents, out var eventsRead);
```

La línea del medio es la clave aquí: `byteBuf` es un `Span<byte>`, pero apunta a la misma memoria que `buffer` (aquí no se copia memoria, ni nada). Cada elemento de `buffer` ocupa 20 bytes en `byteBuf` (ya que `sizeof(INPUT_RECORD)` es 20). Y ahora a la función nativa le pasamos una referencia al primer byte de ese `Span<byte>`, es decir una referencia al primer byte del primer elemento de `buffer`. La función nativa la redefiní de la siguiente manera:

```csharp
[DllImport("kernel32.dll", EntryPoint = "ReadConsoleInputW", CharSet = CharSet.Unicode)]
public static extern unsafe bool ReadConsoleInput(IntPtr hConsoleInput, ref byte lpBuffer, uint nLength, out uint lpNumberOfEventsRead);
```

Y... **¡funcionó!** Ahora aunque hubiese más de un evento, todo funcionaba. Claro que igual te preguntas **como paso el `Span<byte>` a un `Span<INPUT_RECORD>` otra vez**. Bueno, la realidad es que no hay que hacerlo: `byteBuf` y `buffer` apuntan a la misma dirección de memoria. Si se modifica el contenido de uno, se modifica el del otro. Ambos spans son dos visiones distintas de la misma memoria subyacente. Por lo tanto, una vez ejecutada la función, en `buffer` tenía el resultado.

Como "bonus" final os dejo como hacerlo de la forma más clásica (pero con `Span<T>`), usando eso sí, un contexto `unsafe`:

```csharp
Span<INPUT_RECORD> buffer = stackalloc INPUT_RECORD[(int)numEvents];
unsafe
{
    fixed (INPUT_RECORD* pBuf = &MemoryMarshal.GetReference(buffer))
    {
        ConsoleNative.ReadConsoleInput(_hstdin, (IntPtr)pBuf, (uint)buffer.Length, out var eventsRead);
    }
}
// Función nativa
[DllImport("kernel32.dll", EntryPoint = "ReadConsoleInputW", CharSet = CharSet.Unicode)]
public static extern unsafe bool ReadConsoleInput(IntPtr hConsoleInput, IntPtr lpBuffer, uint nLength, out uint lpNumberOfEventsRead);
```

A pesar de que, hasta donde entiendo yo, el uso de `fixed` no sería necesario (porque la memoria de la pila no es reubicable), el compilador me obligaba a ponerlo. Ahí os lo dejo :)

Espero que os haya resultado interesante?

¡Saludos!
