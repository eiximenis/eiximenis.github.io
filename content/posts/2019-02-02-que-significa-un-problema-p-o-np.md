---
title: ¿Qué significa un problema P o NP?
description: ¿Qué significa un problema P o NP?
author: eiximenis

date: 2019-02-02T12:07:51+00:00
geeks_url: /?p=2254
geeks_ms_views:
  - 1177
categories:
  - otros

---
No suelo escribir en el post artículos sobre teoría de la computación, aunque es un tema apasionante. Que ahora recuerde solo escribí uno acerca del concepto del [Orden de un algoritmo][1]. Pero hace algunos días (he tardado más de lo esperado en escribir eso xD) buceando por _twitter_ leí unos comentarios donde se mencionaba el concepto de un problema NP y leyendo el desconocimiento al respecto, me he animado a escribir esto... para que, si alguna vez te topas con ese concepto, sepas de que se habla. **Por supuesto voy a hacer varias simplificaciones al respecto, así que si alguien conoce los fundamentos matemáticos que subyacen debajo, espero que me perdone** 😉
  
<!--more-->


  
Cada disciplina teórica tiene sus grandes preguntas y sus grandes dilemas. La teoría de la computación no podía ser menos y, sin duda, la **gran** pregunta, es &#8220;_P es igual a NP?_&#8220;. Quien encuentre la respuesta a esa pregunta tendrá ganada un lugar en el Olimpo donde viven genios como Gödel o Turing. El primero es posible que te suene y el segundo casi seguro que sí, aunque sea solo por sus logros (aunque, eso sí, exagerados en la película _The Imitation Game_) en la titánica tarea de desencriptar [Enigma][2]. No, no te creas que menosprecio a Turing: si sigues leyendo verás que, aunque parezca imposible, hizo cosas mucho más espectaculares que contribuir decisivamente a que los aliados ganasen la segunda Guerra Mundial.
  
P y NP son grupos de lo que llamamos &#8220;problemas decidibles&#8221; o &#8220;problemas computables&#8221;, así que antes de meternos con ellos, hablemos de qué es un problema decidible...
  
**El problema de la decibilidad**
  
Hasta principios del sXX se creía que todo problema matemático se podía resolver siguiendo una serie de pasos determinados. No se cuestionaba la dificultad de encontrar dichos pasos, pero su existencia se daba por supuesta fuese cual fuese el problema.
  
Fue Leibniz quien planteó la cuestión por primera vez (allá por el sXVII) : &#8220;¿Es posible encontrar una manera de <span style="text-decoration: underline;"><em>decidir</em></span> si un _<span style="text-decoration: underline;">problema matemático</span>_ cualquiera tiene solución?&#8221;.
  
No te confundas! No se trata de encontrar un método que nos permita encontrar la solución a un problema. Se trata de encontrar un método que **nos permita comprobar solo si un problema tiene solución o bien es un problema indecidible**.
  
Que quede claro que en todo caso hablamos de &#8220;problema matemático&#8221;, no pretendemos saber si las dudas metafísicas o teológicas tienen solución. Y un problema matemático es algo muy definido: Una afirmación X qué hay que determinar si es cierta o falsa. ¿Un ejemplo? Pues mira, la conjetura de Goldbach dice que &#8220;todo número par superior a 2 se puede escribir como la suma de dos números primos&#8221;. Es una afirmación concreta y simplemente debemos decidir si es cierta o falsa. ¿Otro ejemplo? La cuadratura del círculo que dice que &#8220;Se puede construir un cuadrado, utilizando regla y compás, que posea un área igual a la de un círculo dado&#8221;.
  
Por lo tanto **no buscamos un mecanismo para saber si la conjetura de Goldbach es cierta o no o si la cuadratura del circulo es posible o no**. Buscamos un mecanismo para saber **si es posible saber si la conjetura de Goldbach es cierta o no**. Buscamos un mecanismo saber **si es posible saber si la cuadratura del círculo es cierta o no.** Porque oye, igual resulta que hay problemas (matemáticos... de los otros, ya no me meto) que no tienen solución.
  
Así, todo se reduce en saber si se puede decidir siempre que una afirmación matemática es cierta o falsa o bien que hay afirmaciones que son indemostrables.
  
No fue hasta bien entrado el sXX que David Hilbert formalizó ese problema y planteó tres cuestiones fundamentales:

  1. Son las matemáticas completas? Es decir **se puede probar que cualquier sentencia (matemática) es cierta o falsa?**
  2. Son las matemáticas consistentes? Es decir **no se puede probar como cierto algo falso**?
  3. Son las matemáticas decidibles (o computables)? Es decir **se puede probar que cualquier sentencia (matemática) es cierta o falsa siguiendo un número finito de pasos**?

Cuando Hilbert formalizó las tres pregundas su intuición era que la respuesta a las tres era Sí. Es decir las matemáticas, el lenguaje universal, eran completas, consistentes y decidibles.
  
Poco duró esa esperanza, porque las dos primeras suposiciones las desmontó Gödel, cuando formuló su teorema de incompletitud donde venía a decir que las matemáticas no podían ser a la vez consistentes y completas. Fue un doloroso mazazo al orgullo de las matemáticas pero al menos todavía quedaba el punto 3. Es decir antes de Gödel se esperaba que se pudiera demostrar que cualquier cosa era cierta o falsa siguiendo un algoritmo con un número finito de pasos. Después de Gödel lo que se esperaba era que en todas aquellas sentencias que se podían demostrar ciertas o falsas, se pudieran hacerlo siguiendo un algoritmo finito.
  
Ah... por cierto... si corriste a pillar un compás, déjalo porque está demostrado que la cuadratura del círculo es imposible 🙂
  
**Algoritmos**
  
Antes he mencionado la palabra &#8220;algoritmo&#8221;. Esto es es un blog de desarrollo así que no creo que sea muy necesario definir el concepto de _algoritmos_. Más que menos trabajamos con ellos. Ya sabes, **un conjunto ordenado de pasos que resuelven un problema**. A la práctica tenemos ordenadores que ejecutan algoritmos (y muy rápido ciertamente), pero en la teoría tenemos incluso algo mucho mejor: una máquina de Turing. Una máquina de Turing pese a su nombre de &#8220;máquina&#8221; no es una construcción física. Es una construcción teórica, concretamente un modelo matemático en forma de autómata capaz de _ejecutar_ cualquier algoritmo. De nuevo la palabra &#8220;autómata&#8221; te puede hacer pensar en un &#8220;robot&#8221; o máquina física, pero no. En matemáticas, un autómata es una &#8220;máquina teórica&#8221; que lee ciertos símbolos y en función de esos cambia de estado. Se trata pues de un modelo que tiene una entrada (un símbolo) y un estado (que va cambiando en función del estado actual y el símbolo de entrada).
  
El conjunto de símbolos que le damos al autómata conforman nuestro algoritmo. Hay varios tipos de autómatas y la máquina de Turing es uno que cumple las siguiente condiciones:

  * Tiene una cinta (infinita) de símbolos
  * Lee cada vez un símbolo de dicha cinta
  * Lo sobreescribe por otro
  * Se mueve para la izquierda o para la derecha en esta cinta (para leer el símbolo anterior o siguiente y volver a empezar)

Y ya. No puede hacer nada más. La máquina de Turing se &#8220;programa&#8221; mediante una tabla de entradas de la forma:
  
(estado, valor) <span class="mwe-math-element"><img class="mwe-math-fallback-image-inline" src="https://wikimedia.org/api/rest_v1/media/math/render/svg/53e574cc3aa5b4bf5f3f5906caf121a378eef08b" alt="\rightarrow " aria-hidden="true" /></span> (nuevo estado, nuevo valor, dirección)
  
Es decir, estando en un estado E, si llega el valor v, la máquina pasa al estado E2, escribe el valor v2 en la cinta y se mueve para la dirección (adelante o atrás) indicada. El algoritmo es la lista complerta de esos pasos. Observa que dado que la máquina avanza de un estado a otro, puede volver a un estado anterior por lo que potencialmente puede entrar en bucles infinitos de estados (y no terminar de ejecutar nunca el algoritmo).
  
A esta máquina de Turing la llamamos **máquina de Turing determinista y ejecuta algoritmos deterministas**. Los algoritmos deterministas son con los que normalmente trabajamos: les das una entrada, realizan ciertas cosas y devuelven un resultado.
  
Pero existen **algoritmos no deterministas**, que pueden devolver distintos resultados ante la misma entrada. Si imaginas que un algoritmo determinista es una lista de pasos que se siguen para, a partir de una entrada, llegar a una salida, un algoritmo no determinista sería como un árbol: a partir de una entrada se puede llegar a varias salidas distintas, y no **puede saberse a _priori_ cual de esas salidas se devolverá**. A nivel matemático tenemos otra construcción (un autómata no determinista) que nos permite lidiar con esos algoritmos. Y sí, un timpo de automáta no determinista es la Máquina de Turing no determinista.
  
Para los propósitos de este post solo nos interesa saber que **es posible convertir cualquier algoritmo no determinista en uno equivalente determinista **aunque la complejidad temporal no tiene que ser equivalente. Eso **significa que cualquier algoritmo computable (es decir resoluble) en una máquina de Turing no determinista es computable en una máquina de Turing determinista** (otra cosa es que tarde más).
  
Seguramente pienses que esta &#8220;máquina de Turing determinista&#8221; (recuerda que es un concepto teórico) es muy limitada y que no podrá más que ejecutar y solucionar problemas sencillos. Bien, simplemente que sepas que **una máquina de Turing puede ejecutar cualquier algoritmo determinista existente** (y, por ende, cualquier no determinista). Un ordenador actual, incluso un futuro ordenador cuántico, no tiene más capacidad de cómputo que dicha máquina (en este contexto &#8220;capacidad de cómputo&#8221; significa capacidad para terminar resolviendo un problema con independencia del tiempo y/o memoria que se requiera). Eso significa que **si hay un problema que no es resoluble mediante una máquina de Turing, no es resoluble aunque pongamos todos los ordenadores de la Tierra a trabajar en él durante toda la eternidad**.
  
Alan Turing se enfrentó al llamado &#8220;**problema de la parada**&#8220;, que consiste en saber si una determinada Máquina de Turing M (programada de determinada manera) ante un determinado dato de entrada se quedaría eternamente ejecutando o bien terminaría y daría con un resultado. Traducido a palabras más mundanas, lo que Turing buscaba es si existe un algoritmo P al que le pases como parámetros otro algoritmo A y unos datos de entrada E y te devuelva un booleano que indica si A(E) finaliza en un número finito de pasos, o bien A(E) se queda ejecutando eternamente (p. ej. en un bucle infinito).
  
Así que si pensabas en forrarte creando un analizador estático de código que te indicara si una llamada a una función entra en un bucle infinito... olvídalo. No vas a poder hacer eso, nunca. La única manera sería ejecutando tu la función, pero claro si esa entra en un bucle infinito, tu también entrarías...
  
Y así, demostrando la indecibilidad del problema de la parada, Turing se cargó la tercera suposición de Hilbert.
  
Un problema indecidible es aquel **para el cual no podemos construir un algoritmo (finito) que nos lleve a un resultado**. Pero ojo, no podemos no porque no sepamos, si no porque no existe. A día de hoy **sabemos que hay infinitos problemas indecidibles**.
  
**Ahora sí: P vs NP**
  
Como he dicho antes P y NP son dos grupos (formalmente dos clases de complejidad) que agrupan problemas **decidibles **distintos. Es decir, tanto **para cualquier problema P como para cualquier NP hay un algoritmo que lo resuelve en un tiempo finito**.
  
Ante un problema podemos hacer dos cosas: calcular los resultados o verificarlos. P. ej. ante el problema &#8220;dado x obtener su cuadrado&#8221;, podemos hacer dos cosas: crear un algoritmo qué encuentre el cuadrado de cualquier x o bien hacer un algoritmo que verifique si un y determinado es el cuadrado de un x.
  
Hay problemas que son fáciles de _solucionar y de verificar_. En este contexto &#8220;fáciles&#8221; significa mediante un algoritmo de orden polinomial. A esos problemas **los llamamos problemas de tipo P**. Vale... un apunte rápido porque usaré muchas veces &#8220;orden polinomial o algoritmo polinómico&#8221;. Bien, un algoritmo tiene orden polinomial (o es polinómico) cuando dado unos datos de entrada de tamaño N, resuelve el problema en un tiempo que es un polinomio de N. Así p. ej. si tengo un problema que para ordenar listas de N elementos tarda un tiempo que es, pongamos por ejemplo, **10N<sup>4</sup>+2N<sup>2</sup>+100N** entonces ese agoritmo es polinómico. Ahora bien, si el tiempo que tarda fuese **2<sup>N</sup>**, entonces ya no (ese sería exponencial). Los algoritmos polinómicos los consideramos &#8220;sencillos&#8221; en el sentido de que los podemos computar y el tiempo no nos explota literalmente de las manos cuando los tamaños de datos aumentan. Y cuando digo explotar, digo literalmente explotar:

<table width="463">
  <tr>
    <td width="64">
      N
    </td>
    
    <td width="108">
      10N<sup>4</sup>+2N<sup>2</sup>+100N
    </td>
    
    <td width="108">
      (años)
    </td>
    
    <td width="71">
      2<sup>N</sup>
    </td>
    
    <td width="112">
      (años)
    </td>
  </tr>
  
  <tr>
    <td>
      1
    </td>
    
    <td>
      112
    </td>
    
    <td>
      3.5515E-06
    </td>
    
    <td>
      2
    </td>
    
    <td>
      6.34196E-08
    </td>
  </tr>
  
  <tr>
    <td>
      2
    </td>
    
    <td>
      368
    </td>
    
    <td>
      1.16692E-05
    </td>
    
    <td>
      4
    </td>
    
    <td>
      1.26839E-07
    </td>
  </tr>
  
  <tr>
    <td>
      10
    </td>
    
    <td>
      101200
    </td>
    
    <td>
      0.003209031
    </td>
    
    <td>
      1024
    </td>
    
    <td>
      3.24708E-05
    </td>
  </tr>
  
  <tr>
    <td>
      11
    </td>
    
    <td>
      147752
    </td>
    
    <td>
      0.004685185
    </td>
    
    <td>
      2048
    </td>
    
    <td>
      6.49417E-05
    </td>
  </tr>
  
  <tr>
    <td>
      50
    </td>
    
    <td>
      62510000
    </td>
    
    <td>
      1.982179097
    </td>
    
    <td>
      1.126E+15
    </td>
    
    <td>
      35702051.84
    </td>
  </tr>
  
  <tr>
    <td>
      100
    </td>
    
    <td>
      1000030000
    </td>
    
    <td>
      31.71074328
    </td>
    
    <td>
      1.268E+30
    </td>
    
    <td>
      4.01969E+22
    </td>
  </tr>
</table>

Observa que como, incluso aunque nuestro algoritmo polinómico aumenta bastante (no en vano es un polinomio de cuarto nivel) se puede ver como la función exponencial aumenta... bueno, eso, exponencialmente (para un elemento tarda 2 segundos y para 100 más de 10 elevado a 22 años).
  
Por ejemplo calcular el cuadrado de un número es un problema de tipo P (hay un algoritmo en tiempo polinomial para hacerlo: multiplicar el número por si mismo). La verificación también sencilla. ¿Es y el cuadrado de x? Nos basta con hacer la raíz cuadrada de y verificando que su valor es x.
  
**Por lo tanto: aquellos problemas qué podemos solucionar y verificar con un algoritmo de orden polinómico, los llamamos problemas P.** Esos son los buenos, los que podemos solucionar &#8220;fácilmente&#8221;.
  
Por otra parte tenemos el grupo de problemas NP. Este grupo está formado por aquellos problemas **que podemos verificar en tiempo polinomial pero ****que para solucionarlos puede ser que requiramos un algoritmo de orden superior **(formalmente se dice que se pueden solucionar con un algoritmo de orden polinómico usando una máquina de Turing no determinista, pero un algoritmo de orden polinómico en una máquina de Turing no determinista equivale, matemáticamente, a un algoritmo de orden superior en una máquina de Turing determinista).
  
Un ejemplo informal es un puzzle: difícil de solucionar pero basta un vistazo para saber si el puzzle está bien resuelto o no. Pues a **esos tipos de problemas los llamamos NP. Observa que no queremos tratar con ese tipo de problemas.**
  
Observa un detalle: **todos los problemas P son a su vez problemas NP**. Es decir **P es un subconjunto de NP**. ¿Qué por qué? Muy sencillo: recuerda que un problema P es aquel que tiene un algoritmo de orden polinómico que lo resuelve. Mientras que un problema NP tiene un algoritmo de orden polinómico que lo verifica. Pues bien, entonces dado un problema de tipo P puedo verificarlo... corriendo el algoritmo que lo soluciona y comprobando la solución. Por lo tanto, un problema de tipo P es, por definición, un NP.
  
Ahora la duda de si P es igual a NP ya está más clara, ¿verdad? Si resulta que eso es así**, cosa que nadie ha demostrado****, **eso significa que cualquier problema que se pueda verificar en tiempo polinomial se puede resolver también en tiempo polinomial (otra cosa es que descubramos el algoritmo que lo hace, pero se sabrá que existe). Por otro lado, si P no es igual a NP, **cosa que nadie ha demostrado tampoco**, eso implicará que hay problemas para los cuales nunca existirá una solución en tiempo polinómico. Eso, aunque parece una desgracia, tampoco cambiaría gran cosa: actualmente estamos _dando por sentado_ que P es distinto de NP y así p. ej. la criptografía se basa, precisamente, en eso.
  
Como digo **la creencia &#8220;popular&#8221; hoy en día es que P es distinto de NP**. Eso implica que hay problemas irresolubles en un tiempo polinómico, aunque ¡ojo! puede ser que haya problemas que pensemos que sean NP y sean P (solo que aún no hayamos descubierto el algoritmo).
  
Vale, ahora que tenemos claro lo que son los problemas P y los NP, ha llegado el momento de introducir _otro tipo_ de problemas: los temidos NP-Completos.
  
**NP Completo**
  
En la década de los 70 se soltó una bomba termonuclear cuyos efectos todavía estamos padeciendo: se **descubrieron un &#8220;tipo&#8221; nuevo de problemas**, los llamados NP-Completos.
  
Formalmente decimos que un problema C es NP Completo sí:

  * C es NP
  * Todo problema NP se puede reducir a C en tiempo polinomial.

El segundo punto es la primera parte de la bomba de la que os hablaba: se **demostró que cualquier problema NP se puede reducir a otro tipo de problemas NP**. Esos otros problemas NP son los que llamamos NP-Completos. Decimos que un problema X reduce a X2 si X2 es **más complejo de resolver que X**, de forma que el algoritmo (o parte de él) usado para resolver X2 también resuelve X.
  
Por lo tanto, y esa es la clave, un problema C es NP-Completo si cualquier problema NP se reduce a él, lo que significa que el algoritmo usado para resolver C puede resolver cualquier NP. Por lo tanto **si encontramos un algoritmo para resolver C en un tiempo polinomial este mismo algoritmo (o parte de él) se podrá usar para resolver en tiempo polinomial cualquier NP**.
  
Pero esperad, que queda lo mejor... la segunda parte de la bomba: se demostró **que todos los problemas NP-Completos** eran equivalentes entre sí. Eso significa que **si un solo problema NP-Completo tiene una solución polinómica entonces por definición todos la tienen. Y vicecersa: si se demuestra que tan solo un problema NP-Completo no tiene solución en tiempo polinómico, entonces ninguno la tiene**.
  
Veamos un ejemplo clásico de un problema NP-Completo: el problema del viajante. Dicho problema nos dice que debemos obtener el **camino más corto que pase una vez por cada una de las N ciudades y que vuelva al punto de orígen**. Sabes la distancia entre todas las ciudades y puedes ir de cualquier ciudad a cualquier otra en cualquier orden.
  
Hoy en día se cree que la manera de solucionar dicho problema es calculando todas las posibilidades una a una, lo que nos deja un algoritmo de orden exponencial (concretamente orden factorial). Si tu algoritmo para 20 ciudades tarda 1 segundo en tu ordenador, en el mismo ordenador dicho algoritmo tardará 21 segundos para 21 ciudades (solo una más). Agregar otra ciudad hará que tarde más de 7 minutos y si pasas de 20 a 30 ciudades, lo que antes tardaba 1 segundo ahora tardará unos 3 millones de años.
  
Otro ejemplo clásico: el problema de la suma de subconjuntos. Consiste en, dado un conjunto de enteros, determinar si existe algún subconjunto suya tal que la suma de sus elementos sea cero. Este problema es claramente NP (fácil de verificar si una solución x es cierta o no (basta con comprobar que x es un subconjunto del conjunto original y que sus elementos suman 0)), pero dificil de solucionar. Pues se demostró que ese problema es también NP-Completo.
  
¿Por qué son tan importantes los problemas NP-Completos? Pues bien, precisamente por qué cualquier NP se puede reducir a ellos y también porque son equivalentes entre sí. Por lo tanto **si se demuestra que un NP-Completo tiene solución polinómica, entonces todos los NP-Completos la tienen. Y dado que todos los NP se pueden reducir a ellos, eso significa que todos los NP tienen solución polinómica. O dicho de otro modo: P = NP. **Y lo mismo es cierto a la inversa: si se demuestra que un problema NP-Completo no tiene solución polinómica (ojo, debe demostrarse, no basta con no encontrarla, claro xD) entonces eso significa que P es distinto de NP.
  
A día de hoy **de todos los problemas que se sabe son NP-Completos no se ha encontrado la solución polinómica de ninguno de ellos, aunque no se ha demostrado formalmente que no la posean**.
  
**NP-Intermedios (NPI)**
  
Richard E. Ladner propuso la existencia de problemas que fuesen NP, pero que no fuesen P ni tampoco NP-Completos. A este grupo de problemas se les llama NP-Intermedios y formarían otro subconjunto de NP. El teorema de Ladner dice que si P es distinto de NP, entonces NPI es un conjunto no vacío. Este teorema **se ha demostrado**, por lo que **podemos afirmar que existen los problemas NPI, pero solo si antes podemos demostrar que NP es distinto a P**.
  
Para que queda claro, un NPI es un problema que:

  * Es NP (se puede verificar en tiempo polinómico)
  * No es P (no se puede solucionar en tiempo polinómico)
  * No es NP-Completo

Lo que el teorema de Ladner dice es que si P es distinto de NP, entonces existen sí o sí los problemas NPI. Actualmente se sabe que el problema de determinar si dos grafos son isomorfos es un NPI.
  
**NP-Hard (NP-Complejo)**
  
Un problema NP-Complejo (no confundir con NP-Completo) es aquel problema al cual pueden reducirse en tiempo polinomial todos los problemas NP. Eso significa que si H es un problema NP-Hard entonces cualquier problema L que sea NP se podrá reducir a H.
  
Observa que esta era la segunda condición de NP-Completo. La diferencia entre NP-Hard y NP-Completo, es que un problema NP-Completo es en sí mismo un problema NP y un problema NP-Hard no tiene por qué. De hecho, **el conjunto** **NP-Completo es la interesección entre los conjuntos NP-Hard y NP****.**
  
Por supuesto si P=NP entonces, NP queda definido como un subconjunto de NP-Hard.
  
**Problemas No NP**
  
Existen más clases de complejidad al margen de NP, como p. ej. BQP (aquellos problemas solventables en tiempo polinomial con un algoritmo cuántico) y algunas otras. Todas ellas se engloban dentro de lo que llamamos el PSPACE, que contiene el conjunto de problemas que son resolubles en **espacio polinómico pero con tiempo ilimitado en una máquina de Turing determinista**. Es decir, existe un algoritmo de orden polinómico en cuanto al espacio necesitado (memoria, vamos), pero el tiempo que tarde es irrelevante.
  
Por supuesto existe un NPSPACE que contiene los problemas resolubles con un algoritmo no determinsta con orden polinómico de espacio. Pero se demostró que todo problema resoluble mediante un algoritmo no determinista con orden polinómico de espacio se puede resolver también con un algoritmo determinista con orden polinómico de espacio, por lo que en este caso **sí que sabemos que PSPACE=NPSPACE**. Ya ves tu.
  
De hecho, hay una cuestión de &#8220;nivel superior&#8221; a si &#8220;P es igual a NP&#8221; y consiste en saber si &#8220;P es igual a PSPACE&#8221;. Pero eso ya es harina de otro costal...
  
Y recuerda... si puedes demostrar que P es igual a NP o viceversa... un millón de euros tiene tu nombre. Aunque claro... siempre quedará la duda si precisamente ese es un problema indecidible 🙂
  
Un saludo y espero que os haya resultado interesante!

 [1]: https://geeks.ms/etomas/2012/11/28/el-orden-de-los-algoritmos-esa-gran-o/
 [2]: https://es.wikipedia.org/wiki/Enigma_(m%C3%A1quina)