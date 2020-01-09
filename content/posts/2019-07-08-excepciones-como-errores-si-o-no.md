---
title: 'Excepciones como errores: ¿Sí o no?'
description: 'Excepciones como errores: ¿Sí o no?'
author: eiximenis

date: 2019-07-08T17:43:59+00:00
geeks_url: /?p=2397
geeks_ms_views:
  - 574
categories:
  - 'C#'

---
Bueno, he aquí un dilema que es más o menos como el tipado estático vs el dinámico o el preferir espacios o tabuladores: es decir, preferencia personal. Pero a veces las preferencias personales se ven influenciadas por lo que conocemos (o más precisamente por lo que desconocemos)... Así que dejadme que os cuente cuatro cosas al respecto y ya si eso, luego lo discutimos en los comentarios o en un bar...
  
<!--more-->


  
En C# es habitual **usar excepciones para gestionar errores**. Si llamas a _File.Open_ y el fichero no existe, pues se lanza una excepción. Si intentas convertir la cadena &#8220;pepe&#8221; a un entero, con _int.Parse_, pues se genera una excepción. Por lo general las funciones en C# no devuelven un error: devuelven un resultado y si hay alguna causa que les impide generar dicho resultado, se lanza una excepción.
  
Otra aproximación clásica es **devolver un código de error**, si hay eso... un error. P. ej. _File.Open_ podría devolver el _FileStream_ asociado o _null_ si no ha podido abrir el fichero. Claro, que la aproximación de devolver un código de error tiene un problema: **a veces no hay valor de retorno posible para indicar un error**. A veces todo el espectro de los posibles valores de retorno son valores válidos. Por ejemplo la función _int.Parse_... ¿qué código de error puede devolver? Cualquier entero es un valor válido que la función puede devolver. Si _int.Parse_ devuelve un _int_, no hay valor que pueda indicar el código de error. Esa limitación se puede resolver con dos feas aproximaciones, ninguna de las cuales es perfecta:

  * Asumiendo que exista un tipo de retorno con un rango de valores mayor, devolver un valor de tipo de retorno. P. ej. en C, la función [getchar][1], devuelve un int. Podría devolver un _char_, pero devuelve un _int_. El motivo es que el rango de valores de int es superior (e incluye en su totalidad) el de _char_ y eso nos da valores adicionales a usar en caso de error. Así _getchar_ devuelve un -1 (que es un valor _int _válido, pero fuera del rango de _char_) para indicar un error. Eso bueno... es más feo que pegarle a un padre, porque tu lees un carácter y obtienes un _int_, que luego debes convertir a _char_.
  * Se asume un valor como error. Un ejemplo de eso es la función [VAL][2] de BASIC, que convierte una cadena a un entero. Si la cadena no es un número válido, VAL devuelve 0. Pero la cadena &#8220;0&#8221; (que es un número válido), devuelve, obviamente, también 0. Por lo tanto saber si recibes un 0 porque la cadena era &#8220;0&#8221; (o &#8220;00&#8221; o &#8220;0000000&#8221;) o bien era inválida se complica.

Ante esos problemas, **parece que lanzar excepciones es una buena solución**: no me obliga a devolver &#8220;otros tipos de datos&#8221; y siempre sé que si obtengo resultado es que todo ha ido bien. Pero, honestamente, no sé si es la mejor de las soluciones. **Porque las excepciones deberían ser eso... casos excepcionales**. Que abrir un fichero falle, NO es un caso excepcional. Es un caso muy normal, especialmente si la ruta del fichero la ha entrado el usuario. Que convertir un entero a cadena falle, porque la cadena no contiene ningún número, no tiene nada de excepcional, especialmente si esa cadena la obtenemos mediante entradas que no controlamos. Y eso sin hablar de las penalizaciones en el rendimiento que tienen las excepciones.
  
Bueno, voy a matizar: como siempre lo que es una buena o no-tan-buena solución debe entenderse en el contexto de cada lenguaje... y aquí está la clave. Porque el contexto de C#8 y el de C#1 poco tienen que ver. Lanzar excepciones podía ser quizá la mejor solución en C#1, pero... ¿lo sigue siendo con C#7?. ¿Qué otra alternativa podríamos tener? Pues, podríamos imitar lo que hace Golang: **devolver tuplas (bool, resultado)**. El único problema es que un bool, solo dice si ha ido o bien o mal, pero podríamos tener una clase Error que nos diera más información y devolver tuplas (Error, resultado):

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">struct Error
    {
        public bool IsError { get; }
        public string Message { get; }
        public static (Error error, T value) OK&lt;T&gt;(T t) =&gt; (new Error(error: false), t);
        public static (Error error, T value) Build&lt;T&gt;(string message) =&gt; (new Error(error: true, message), default(T));
        public static implicit operator bool(Error error) =&gt; error.IsError;
        private Error(bool error, string msg = null)
        {
            IsError = error;
            Message = msg;
        }
    }
    static (Error error, int value) ParseInt (string s)
    {
        if (int.TryParse(s, out int vs))
        {
            return Error.OK(vs);
        }
        return Error.Build&lt;int&gt;($"Invalid string {s}");
    }
}</pre>

Una vez tenemos esta infrastructura montada, lo podemos consumir de varias maneras. P. ej. aprovechando que _Error_ se transforma en  _bool_:

<pre class="EnlighterJSRAW" data-enlighter-language="null">var x2 = ParseInt("edu");
if (x2.error)
{
    Console.WriteLine("Error: " + x2.error.Message);
}
else
{
    Console.WriteLine("Result is: " + x2.value);
}</pre>

O usando el _pattern matching_ de C#7:

<pre class="EnlighterJSRAW" data-enlighter-language="null">var result = ParseInt("9");
switch (result)
{
    case var x when x.error:
        Console.WriteLine("Error: " + x.error.Message);
        break;
    case var x when !x.error && x.value &lt; 10:
        Console.WriteLine($"Value {x.value} is too small");
        break;
    default:
        Console.WriteLine($"Value {result.value} is OK");
        break;
}</pre>

Este código mejorará cuando tengamos C#8, gracias al uso de _pattern matching _recursivo:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">var result = ParseInt("9");
switch (result)
{
    case var (e, _) when e:
        Console.WriteLine("Error: " + e.Message);
        break;
    case var (_, v) when v &lt; 10:
        Console.WriteLine($"Value {v} is too small");
        break;
    default:
        Console.WriteLine($"Value {result.value} is OK");
        break;
}</pre>

Por supuesto, puede haber casos en que no queremos comprobar el error:

<pre class="EnlighterJSRAW" data-enlighter-language="null">var (_, result) = ParseInt("100");
Console.WriteLine($"Result is {result}");</pre>

Claro, que ahora perdemos una de las supuestas ventajas de usar excepciones: que es no tener mezclado el código de gestión de errores, con el código normal. Efectivamente, tengo que usar _ifs_ para comprobar si todo ha ido bien, **aunque hay que decir que el uso de _pattern matching_ me limita bastante estos ifs**. Por eso decía antes que el lenguaje importa: en C#7 tenemos un soporte para tuplas muy bueno y un pattern matching que mejora con C#8. En versiones anteriores de C# no teníamos eso.
  
Usando excepciones parece que todo queda más ordenado (con el código normal dentro del _try_ y el de gestión de erroes en el _catch_), pero ese modelo también tiene sus problemas: el _catch_ se ejecuta en un ámbito distinto del _try_, y muchas veces no tienes acceso a variables definidas dentro del try y que necesitarías usar dentro del _catch_. Estoy suponiendo ciertamente que puedes tratar el error (es decir que puedes hacer algo más en el _catch_ que no sea guardar en un log y propagar la excepción).
  
Además usando excepciones, a veces puedes terminar con un varios bloques try/catch dentro de un mismo método (si el método puede hacer varias cosas):

<pre class="EnlighterJSRAW" data-enlighter-language="null">void Process()
{
    try { Step1();}
    catch (Exception ex) {...}
    try { Step2();}
    catch (Exception ex) {...}
    try { Step3();}
    catch (Exception ex) {...}
}</pre>

En este caso la función Process() ejecuta tres pasos, y si falla el primero, debe ejecutar igualmente el segundo y el tercero. Las cosas se complican si la ejecución del segundo paso depende del tipo de error que ha lanzado el primero: al final terminas con ifs... exactamente que en el caso de devolver el código de error.
  
En mi opinión el modelo de excepciones es solo superior al de devolver el código de error en aquellos casos en que habitualmente no podrás hacer nada con el error... salvo como mucho mostrarlo por pantalla o guardarlo en un log. Pero eso debería ser en casos excepcionales, que es para lo que se creó el modelo de excepciones. Es cierto que en C# la situación es (en mi opinión) mejor que la de Java, ya que no tenemos la cláusula _throws_, así que al final optamos por ignorar las excepciones... menos aquellas que podemos tratar y que muchas veces terminan en bloques try/catch en medio de un método 😉
  
¿Y vosotros, qué pensáis al respecto? 🙂

 [1]: https://www.tutorialspoint.com/c_standard_library/c_function_getchar
 [2]: https://hwiegman.home.xs4all.nl/gw-man/VAL.html