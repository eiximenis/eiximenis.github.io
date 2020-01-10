---
title: Meltdown y Spectre… algunos comentarios

author: eiximenis

date: 2018-01-05T12:27:46+00:00
geeks_url: /?p=1978
geeks_ms_views:
  - 1243
categories:
  - otros

---
dBueno, todo el mundo está hablando sobre Meltdown y Spectre los dos ataques que aprovechan fallos de los microprocesadores &#8220;modernos&#8221; Intel (el primero) y de cualquier fabricante (el segundo). Como digo muchos medios ya han hablado de ambos ataques, en muchos casos con titulares grandilocuentes y sensacionalistas y contando apenas nada.
  
En este post voy a intentar explicar como funcionan ambos ataques, aunque me centraré en Meltdown por ser el más fácil de explicar, entender y aplicar. Spectre se basa en los mismos conceptos, pero es un ataque bastante más complejo.
  
Voy a intentar explicarlo de un modo &#8220;entendible&#8221;, pero obviamente eso requiere ciertos conocimientos técnicos, pero bueno... ese es un blog de desarrollo, así que seguramente los tienes 😉
  
<!--more-->


  
**La causa: optimizaciones**
  
Empecemos por la causa que abre la puerta a esos dos ataques: las _optimizaciones_. Pero no optimizaciones del compilador, si no del propio micro-procesador. En este caso parece ser que la raíz de Meltdown es el _out of order execution_ y la de Spectre es _branch prediction_. Ambas optimizaciones son un tipo concreto de otra más general conocida como _speculative execution_.
  
Todos tenemos en mente la idea de que un microprocesador va ejecutando las instrucciones (en código máquina) una tras una. Eso sería lo normal, pero tiene un problema: el **microprocesador es condenadamente rápido en comparación con todo lo demás**. Si dada una instrucción para leer un dato de la memoria RAM, el microprocesador debe esperarse a que el dato llegue al registro, estaría la mayor parte de su tiempo parado. La RAM es rápida pero el procesador lo es mucho más.
  
**Meltdown y Spectre**
  
Por eso hay esas opimizciones y una es la de **out of order execution**. Básicamente esto quiere decir que dado una secuencia de instrucciones el microprocesador ejecuta cada una de ellas tan rápido puede, **es decir una vez sus operandos están listos**. Por supuesto si hay interdependencias no puede hacer esto, pero si no las hay es posible que el procesador ejecute la primera instrucción (que accede a RAM) y mientras no llegue el dato ejecute la siguiente si es que esta siguiente accede a datos de los registros.
  
Vamos a ver un ejemplo, pero no en ensamblador si no en &#8220;alto nivel&#8221; (asumimos que cada línea es una instrucción del procesador) para entenderlo mejor:

<pre class="EnlighterJSRAW" data-enlighter-language="null">throw new Exeception();
var a = b[1000];</pre>

¿Es obvio que la segunda línea nunca se va a ejecutar verdad? Es tan obvio que es falso. El procesador empieza a ejecutar la primera instrucción pero como tarda porque está a la espera de un recurso, mientras tanto ejecuta la segunda **y accede a la posición 1000 del array**. Posteriormente la primera instrucción termina y se lanza la excepción, **en este momento el procesador echa todo lo demás para atrás y queda exactamente en el MISMO estado en el que estaría si solo hubiera ejecutado la primera instrucción.**
  
Desde un punto de vista lógico todo es correcto: **no queda rastro alguno de que se haya accedido a la posición b[1000]**. Aunque se haya accedido a ella, el procesador echa toda la ejecución para atrás. Si este valor estaba en un registro, el registro vuelve a su estado anterior. Es **como hacer un rollback en una base de datos**.
  
La otra optimización es la de _branch prediction_ que se basa en cada vez que el procesador llega a una instrucción de bifurcación, **empieza a ejecutar una de las dos ramas _antes_ de evaluar la comparación**. De este modo, si ha acertado, todo eso que ha ganado. Y si ha fallado, pues bueno, el procesador hace el _rollback_ y empieza a ejecutar la otra rama. Los procesadores actuales usan heurísticas y son muy buenos en acertar que rama es la que se ejecutará y eso permite ahorrar tiempo.
  
Como digo, es como hacer un rollback de base de datos... excepto que sí que queda un rastro. Y este es el rastro que explotan _Meltdown y Spectre_. Y tiene que ver con otra optimización: la cache.
  
Y es que **cuando el procesador accede a la RAM usa una _cache_ (interna del procesador y mucho más rápida que la RAM) y si el dato no está en la _cache_ lo recupera de la RAM y lo mete en la _cache_ para posibles posteriores usos**. Hasta aquí todo correcto: para eso tenemos las _cache_. Pero **existe un &#8220;problemilla&#8221;: cuando el procesador hace el &#8220;rollback&#8221; no echa para atrás los datos de la _cache_.**
  
Bueno &#8220;problemilla&#8221; lo he puesto entre comillas, porque esa _cache_ es interna. No hay manera de acceder a ella. No hay instrucción en ensamblador para obtener un dato de la _cache_, todo eso lo hace el procesador entre bambalinas. Así que no limpiar la cache no debería suponer ningún problema. Por eso precisamente **y de forma consciente se optó por no hacerlo**. Revertir el estado de la _cache_ sería muy lento. Tanto que echaría al traste la ganancia de tiempo de _out of order execution_ en el caso de que tuviéramos que ir para atrás. No compensa y total nadie puede acceder a la _cache_ para saber que hay en ella.
  
**Y es cierto: nadie puede leer de la _cache_**.
  
Pero sí que podemos hacer otra cosa mucho más perversa e inteligente: **medir cuanto tiempo tardamos en leer un dato de la memoria**. Si tardamos &#8220;poco&#8221; tiempo, es que este dato estaba en la _cache_ y si tardamos más tiempo es que no estaba. Es decir, **no podemos saber que hay en la _cache_, pero midiendo tiempos podemos saber si un dato en concreto está en ella**. Y este es el vector de ataque.
  
Es decir, **no nos basaremos en leer datos de la _cache_ (eso sería un error de seguridad gravísimo), se basa en averiguar si un dato está o no en base al tiempo que tardamos en acceder a él**. Vale... pero de nuevo: ¿por qué esto nos permite acceder a memoria no autorizada (la del kernel u otro proceso)?. **Si yo intento leer un dato prohibido para ver si está o no en la _cache_, nunca lo sabré porque recibiré siempre una excepción**. Es decir _a priori_, otra vez, parece que no hay posibilidad de ataque. Pero, como siempre, la hay.
  
A grandes rasgos Meltdown (y una de las variantes de Spectre) se basan en algo similar (salvando las distancias, claro) a hacer lo siguiente:

  1. Crear dos arrays del mismo tamaño
  2. Comprobar que el índice pasado está dentro del array1
  3. Si es OK leer el array1 a este índice
  4. A partir de un bit del resultado de 3 **calcular otro índice del segundo array**
  5. Comprobar que este segundo índice está dentro del array2
  6. Acceder al array2 con este índice

¿Os parece poco malicioso, verdad? ¡Es que lo es! Pero este código es la base de Meltdown. La pregunta és **¿qué ocurre con este código cuando accedemos a un índice inválido al array1?** Pues que el código termina en el paso 2. El paso 3, 4, 5 y 6 no se ejecutan nunca.
  
Vale, pero hemos visto que hay _out of order execution_ lo que significa que estos pasos sí que pueden ejecutarse y ya luego, cuando se demuestre que el paso 2 termina el procesador hará su &#8220;rollback&#8221;. Ahora, y aquí está la sutileza, observa que hemos leído de la memoria una posición del array2 **y lo hemos hecho en base a un bit del valor en una posición del array1**, a la que no teníamos acceso. Algo como, si el primer bit es 1 accedes a array2[1] y si es 0 accedes a array2[1]. Esto es lo que hace el punto 6.
  
¿Y eso qué significa? Pues que **o bien el valor de array2[1] o el de array2[0] están ahora en la _cache_.** ¿Por lo tanto qué debes hacer? Muy sencillo: acceder a array2[0] y hay dos posibilidades:

  * La lectura tarda &#8220;mucho&#8221;, porque array2[0] no estaba en la cache, _ergo_ eso significa que el primer bit del valor de array1 al que no tienes acceso era un 1.
  * La lectura tarda &#8220;poco&#8221;, porque array2[0] estaba en la cache (recuerda que no se limpia tras el &#8220;rollback&#8221; del procesador&#8221;. Eso significa que el valor del primer bit de array1 al que no tienes acceso era un 0

Ahora solo tienes que repetir este proceso N veces y usar cada vez un bit distinto del valor &#8220;prohibido&#8221; de array1 para obtener los N bits de array1.
  
Es decir **obtienes todos los bits (uno a uno) de una dirección de memoria a la que NO tienes acceso, mediante la medición de tiempos de la lectura de otra dirección válida (a la que SI tienes acceso), que está o no en la cache del procesador, en función del valor de uno de los bits de la primera dirección**.
  
Ahora sustituye &#8220;array1&#8221; por &#8220;puntero a la dirección de memoria del _kernel_&#8221; y listos. En este caso el paso 2 que teníamos lanza una excepción (no puedes acceder a memoria del kernel (anillo 0) desde código de usuario (anillo 3)). Pero claro, recuerda que &#8220;gracias a&#8221; la ejecución especulativa el resto de pasos se han ejecutado (y al haber la excepción se hace el &#8220;rollback&#8221;). Así pues puedes &#8220;inferir&#8221; toda la memoria del _kernel_. No la &#8220;lees&#8221; directamente, lo que haces es &#8220;deducirla&#8221; bit a bit. **Pues eso es, a grandes rasgos, Meltdown y Spectre** (al menos una de sus variantes).
  
Por lo tanto son varias las causas que dan lugar al ataque:

  1. _Speculative execution_: El procesador ejecuta instrucciones de forma especulativa, fuera de orden o en otra rama de decisión.
  2. El uso de _cache_
  3. Que en el rollback de la _speculative execution_ no se limpia la _cache_

Los tres puntos tienen una cosa en común: **se prima la velocidad a la seguridad**.
  
El hecho de que Meltdown parece (no está confirmado) que afecta solo a Intel es porque los procesadores Intel permiten que una referencia especulativa en anillo 3 acceda a memoria del anillo 0 (en nuestro caso permiten ejecutar el punto 3 de forma especulativa si asumimos que array1 es la memoria del kernel). Parece ser que AMD no permite eso, de forma que al no ejecutarse nunca el punto 3, no tendríamos nada en la _cache_ con lo que nada podríamos inferir. Precisamente por eso Spectre les afecta, porque en Spectre &#8220;array1&#8221; no es el kernel si no memoria virtual localizada en un espacio de direcciones de otro proceso, pero en el mismo anillo 3. Y sí que se permite una lectura de una referencia especulativa a una dirección del anillo 3 en código de anillo 3 (obviamente, si no, no permitirías ninguna).
  
¿Es eso un _bug_? Pues bueno, depende de como se mire. ¿Es un problema? Eso sí. Sin duda.
  
**El parche a Meltdown**
  
Para Spectre no hay parche a la vista ni lo habrá creo yo. Sí que saldran acciones puntuales, pero veremos. Para Meltdown sí que hay un parche y es literalmente: quitar el kernel de la memoria virtual de los procesos en anilla 3 (lo que se conoce como KAISER).
  
Ahora mismo la memoria del ordenador se divide en dos partes: la de usuario que usan los programas y la del Kernel. Generalmente se asigna una porción de memoria virtual para cada proceso (supongamos 2GB) y el resto (otros 2GB si estamos en 32 bits) para el Kernel. **El Kernel está físicamente en un solo lugar de la memoria y se mapea a cada memoria virtual de cada proceso en la misma dirección**. Así si yo tengo un puntero y sé que el Kernel empieza en la dirección virtual X, puedo intentar leer de ella. Eso me generará una excepción si lo hago &#8220;a saco&#8221; ya que estoy en anillo 3. Si lo hago &#8220;correctamente&#8221;, es decir llamando a métodos del Kernel, en algún momento el código salta a anillo 0 y entonces el Kernel puede leer su memoria. Así es a grandes rasgos como funciona.
  
¿Y el parche que hace? Pues básicamente **quita el Kernel de la memoria virtual del proceso**. La parte reservada al Kernel estará vacía (o casi, es imposible vaciarla toda). Por lo tanto ahora cuando hagamos una llamada al Kernel y pasemos al anillo 0, el Kernel obtendrá su propia memoria virtual y podrá, por supuesto, acceder a ella. KAISER no es nada malo _per se_. De nuevo la pregunta es ¿por qué teníamos la memoria del Kernel mapeada en la memoria virtual de cada proceso? Y la respuesta vuelve a ser: rendimiento.
  
Mapear una dirección virtual a una física es un proceso &#8220;lento&#8221; por lo que para acelerarlo existe... sí, lo has acertado: otra _cache_. Pero es una _cache_ pequeña y una de las formas de hacer que se llene menos es teniendo mapeado el Kernel en la misma dirección virtual de cada proceso. De este modo al cambiar de un proceso al Kernel no añadimos presión a esa _cache_. Pero ahora cuando cambiemos de un proceso al Kernel, esa _cache_ ya no es válida, ya que las direcciones virtuales cambian, lo que significa que perdemos _cache hits_. Es esa pérdida, básicamente, la que genera esa pérdida de rendimiento que se cifra entre un 1 y un 50%... que depende básicamente de cuantos cambios de Kernel a espacio de usuario tengamos.
  
Y bueno... espero que os haya aclarado algo. Como digo es un resúmen un poco basto de lo que es Meltdown y Spectre. Si queréis profundizar más en como funcionan leeros los papers que los describen (disponibles en [Meltdownattack][1] pero son un poco &#8220;duros&#8221;, aviso xD).

 [1]: https://meltdownattack.com/