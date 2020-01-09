---
title: Unicode y encodings
description: Unicode y encodings
author: eiximenis

date: 2018-10-01T09:40:40+00:00
geeks_url: /?p=2189
geeks_ms_views:
  - 715
categories:
  - .net
tags:
  - unicode

---
Uno de los conceptos que hoy en día siguen causando más confusión es el de Unicode y sus distintos tipos de codificación. Pero... ¿qué es realmente Unicode? Para ello, déjame que remonte unos cuantos años atrás...
  
**El inicio: ASCII**
  
Los ordenadores los inventaron los americanos y como suele ocurrir se preocuparon de lo suyo: que un ordenador pudiese presentar textos en su idioma. Tampoco hay tantos caracteres en el inglés: las veinte y poco letras (en mayúsculas y minúsculas), los símbolos de puntuación, paréntesis, operaciones matemáticas y poco más. En total eran menos de 128 carácteres. Genial, porque esos espacios que sobraban se podían aprovechar para colocar otros caracteres que no tienen representación gráfica pero que eran (y son) necesarios para controlar el teletipo: retornos de carro, saltos de línea y similares. Había nacido el [código ASCII][1].
  
<!--more-->


  
Aquí ya hay confusión: mucha gente cree, erróneamente que el código ASCII es un código de 8 bits por caracter, pero es falso. ASCII es un código de 128 valores y por lo tanto se usan 7 bits por cada caracter (00-7F). Ten presente, **y esa es la clave de todo el lío que los ordenadores NO entienden de carácteres. Solo de bytes**. Los carácteres son cosa de humanos y el meollo del asunto es &#8220;como pasamos de un churro de M bytes a una cadena de N carácteres&#8221;.
  
Ahora bien dado que los ordenadores trabajan habitualmente con bytes, y un byte son 8 bits con ASCII nos sobra uno. Puede parecer que un bit no da para mucho, pero bueno... ¡nos permite doblar el número de caracteres! Obvio: si en lugar de 7 bits usamos 8 bits por caracter, pasamos de 128 a 256 valores. Y la pregunta está clara: **qué hacemos con los 128 valores restantes?** Pues, literalmente, cada cual hizo lo que le dió la gana. Salieron así los códigos &#8220;ASCII extendidos&#8221; (los que mucha gente cree que son ASCII). Pero fíjate que hablo en plural! ¡Es que hay más de uno! Como te digo cada cual aprovechó para meter sus propios 128 caracteres adicionales. En windows tenemos el comando &#8220;chcp&#8221; que nos permite cambiar &#8220;la página de códigos del terminal&#8221; para que interprete los bytes (de entrada y salida) basándose en uno de estos códigos ASCII extendidos. Así, bajo páginas de código distintas el mismo fichero (churro de bytes) lo verás con contenido diferente!
  
**El primer lío: SBCS y DBCS**
  
Como digo todo el mundo se puso a redefinir los 128 carácteres adicionales que ASCII nos dejaba libre y por supuesto los japoneses no quisieron ser menos. Pero ellos se dieron de bruces con un problema: ¡tienen más de 128 carácteres! Así que ni con los 128 carácteres libres les bastaba.
  
¿La solución? Si un byte no te da para almacenar todos tus carácteres... ¡usa dos! Eso te da la capacidad teórica de 65536 carácteres distintos, pero la realidad no es tan sencilla: las páginas de códigos deben ser compatibles en los 127 primeros carácteres. Eso significa que un fichero que tenga solo bytes en el rango 0x00-0x7f se debe ver bien en todas las páginas de códigos. Por lo tanto se inventaron una codificación donde &#8220;algunos&#8221; de los valores por encima de 0x7f indicaban que el siguiente byte también formaba parte del carácter. Así nacían las codificaciónes DBCS (Double Byte Character Set). Bajo DBCS no tenemos 65536 carácteres distintos, si no bastantes menos y no todos los carácteres ocupan dos bytes, pero bueno, a los japoneses ya les bastaba. Por contraposición a todas las otras páginas de códigos de idiomas mortales, que les bastaba con un byte por caracter se las conoció como SBCS (Single Byte Character Set).
  
**A poner orden: Unicode**
  
Por supuesto hay más de una página de códigos DBCS: los chinos, los coreanos y demás no quisieron ser menos y también se crearon sus propias páginas de códigos con sus carácteres. Eso se nos estaba yendo de las manos.
  
Por ello al final se decidió **crear un estándard** para unificar todas las páginas de códigos en una. Se usarían 2 bytes por carácter, lo que nos da, ahora sí, 65536 carácteres diferentes que nos deben bastar para todas las lenguas. Se creó un consorcio, el [consorcio Unicode][2] y  se les pagó para que definieran una lista con los 65536 valores y así que nuestra vida fuese mucho más fácil.
  
La gente de Unicode tenían dos virtudes: la primera es que tontos no eran, y para evitar follones, los primeros 127 valores Unicode son el código ASCII (en caso contrario se hubiese armado un follón que ríete tu del [efecto 2000][3]). La segunda virtud es que eran hijos de abogados y por eso encontraron una manera de complicarlo todo hasta el límite para que los informáticos tuviéramos algo de que vivir y poder escribir posts como este.
  
**La complejidad de Unicode**
  
Mucha gente se cree que Unicode es un estándard que define (un máximo de) 65536 carácteres (2 bytes por carácter) y que se trata de una lista numerada donde a cada valor le corresponde un carácter. Esta visión tiene dos problemas: el primero es que es una simplificación muy burda y el segundo que es falsa.
  
Es falsa porque simple y llanamente **Unicode tiene más de 65536 carácteres** (la última versión de Unicode, la 11.0, tiene 137374 carácteres) y es una simplificación muy burda porque en Unicode es posible definir carácteres como combinaciones de otros carácteres (parece una idea muy loca pero tiene su utilidad). Mucha gente sigue creyendo que Unicode define un máximo de 65536 carácteres **porque empezó así**, pero más adelante se vió que eso era insuficiente y se modificó el estándard. A cada nueva versión se le han ido agregando carácteres.
  
A partir de ahora voy a ponerme un poco más estricto y usaré la siguiente nomenclatura:

  * _code point_ para referirme a un carácter Unicode.
  * _code points BMP_ (Basic Multilingual Plane) para referirme a aquellos _code points_ en el rango U+0000 &#8211; U+FFFF.
  * _code points no-BMP** **_para referirme a aquellos _code points _fuera del rango BMP (a partir de U+10000 para adelante)

Para **codificar un _code point _Unicode necesitamos 4 bytes** (aunque no usamos todos los 32 bits porque hay menos de 2<sup>32</sup> carácteres Unicode). Si **para cada _code point _usamos siempre los 32 bits, tenemos lo que se llama UTF32.** UTF32 es la codificación más simple... y la menos usada. Simple porque a cada carácter Unicode le corresponde un único valor de 32 bits, y la menos usada porque ocupa gran cantidad de espacio. Observa que usando UTF32 cada carácter ocupa 32 bits, incluso aquellos con códigos bajos (p. ej. ASCII), que son los carácteres más usados. Así un texto en UTF32 que tenga 10 carácters ocupará 40 bytes, con independencia de que esos 10 carácteres sean 10 &#8216;A&#8217;s o bien 10 carácteres raros (creéme: hay carácteres muy raros en Unicode).
  
Es por ello que, para evitar este uso de espacio, se usan otras codificaciones como UTF7, UTF8 y UTF16. Que como habrás adivinado usan 7 bits, 8 bits o 16 para cada... ¿para cada qué? Está claro que para cada _code point _Unicode no (porque no cabe). **Cuando usamos estas otras codificaciones debemos entender el concepto de <span style="text-decoration: underline;"><em>code unit</em></span>**. Así UTF7 usa _code units_ de 7 bits, UTF8 las usa de 8 bits y UTF16 de 16 bits. **Y un _code point_ (carácter) Unicode se compone de una o más _code units_ combinadas**. Así en UTF8 hay algunos carácteres Unicode (p. ej. los ASCII) que ocupan solo 1 byte, hay otros que ocupan dos y los hay que ocupan 3 y hasta 4 bytes. Así un texto codificado en UTF8 siempre ocupa menos que el mismo texto en UTF32. Algo parecido ocurre con UTF16 (aunque en este caso, cada _code unit_ ocupa 2 bytes, por lo que el ahorro de espacio no es tan sensible).
  
**Unicode y .NET**
  
Si has estado leyendo el post, quizá te haya sorprendido que Unicode defina más de 65536 carácteres. Ya que ¿no se supone que .NET soporta Unicode? Pero un _char_ de .NET son 16 bits. Es decir **no hay espacio en un char de .NET para representar la totalidad de _code points_ de Unicode**. ¿Entonces... qué ocurre?
  
Pues la respuesta **es que char NO ES un _code point _(carácter_) _de Unicode. <span style="text-decoration: underline;">Char es un <em>code unit </em>de UTF16</span>**. Y sí, las cadenas de .NET no son secuencias de carácteres Unicode: son secuencias de _code units_ de UTF16. Por lo tanto **para algunos carácteres de Unicode se requiere más de un char de .NET para representarlos**. Es por eso que se dice siempre que **.NET está basado en UTF16**. Cuanto antes asimiles esta idea menos problemas tendrás.
  
Pero... como podemos representar mediante .NET un carácter Unicode que requiera más de un _code unit_ en UTF16. P. ej. el carácter **U+1000A** (65546 decimal) se representa de la siguiente manera:

  1. 0x0001000A en UTF32 (recuerda en UTF32 cada valor es un _code point_ de Unicode)
  2. 0xD800, 0xDC0A en UTF16 (dos _code units_)
  3. 0xF0, 0x90, 0x80,0x8A en UTF8 (cuatro _code units_)

Observa que **no podemos crear una cadena que contenga este carácter &#8220;tal cual&#8221;:**

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">var impossible = "\u0001000A";</pre>

Esto funciona, pero **_impossible_ no contiene el carácter U+1000A**, si no que es una cadena con cinco carácteres (0x1, 0x30 (&#8216;0&#8217;), 0x30, 0x30, 0x65 (&#8216;A&#8217;)). La sintaxis &#8216;\uxxxx&#8217; define un carácter cuyo código es &#8220;xxxx&#8221; pasado a hexadecimal. Siempre son cuatro valores (0x0000 &#8211; 0xffff), así que lo que sigue se consideran carácteres normales. Así, pues es como si hubieramos hecho &#8220;\u0001&#8221; + &#8220;000A&#8221;.  Por supuesto si hago _impossible = &#8220;\u1000A&#8221;_ (sin los cero iniciales), la cadena tiene dos carácteres (el primero es 0x1000 y el segundo una letra A).
  
Para representar una carácter Unicode como U+1000A, debo partir de su representación en UTF32 y usar el método [ConvertFromUtf32][4] de la clase char. Este método **toma un int (32 bits) con el código UTF32 y devuelve UNA CADENA que contiene el carácter Unicode**:

<pre class="EnlighterJSRAW" data-enlighter-language="null">var possible = char.ConvertFromUtf32(0x1000A);
foreach (var x in possible)
{
    Console.Write("({0:X4})", (int)x);
}</pre>

La salida de este código es (D800)(DC0A) que son los dos _code units _en UTF16. **Es por eso que el método devuelve una string: hay carácteres Unicode (_code points_) que requieren más de un carácter .NET (_code unit_ de UTF16) para representarse**. En este caso la propiedad Length de _possible_ es 2.
  
**No hay en .NET una API orientada a _code points_ (carácteres) de Unicode que nos permita saber cuantos carácteres Unicode tiene una secuencia de bytes. **La API de .NET (las clases String y char) está orientada a UTF16. Sí que tenemos herramientas para saber si un char de .NET es directamente un _code point_ de Unicode o bien se trata de un _code unit_ de UTF16 que forma parte de la representación de un _code point _de Unicode que toma más de un _code unit_ en UTF16. El método _IsSurrogate_ (estático de char) nos dice si un _code unit_ de UTF16 (char en .NET) es un _surrogate_ (¿sustituto?)  o no (más sobre _surrogates_ en breve).
  
Tener una API orientada a UTF16 tiene un problema: **dada una cadena Unicode, representada como String en .NET, no podemos fácilmente acceder al i-ésimo carácter Unicode**. Ya, es cierto que las cadenas tienen índice, pero este índice nos devuelve el i-ésimo _code unit_ de UTF16, no el i-ésimo _code point_ de la cadena Unicode. Por lo tanto debemos recorrer toda la cadena y parsearla.
  
**Surrogates en Unicode y UTF16**
  
Un _surrogate_ es un _code point_ (es decir carácter Unicode) en el rango U+D800 &#8211; U+DFFF (hay pues 2048 surrogates), que se dividen en dos grupos:

  * U+D800 &#8211; U+DBFF (1024 surrogates) conocidos como _high surrogates_
  * U+DC00 &#8211; U+DFFF (los otros 1024) conocidos como _low surrogates_

En UTF16 aquellos _code units no BMP_ (a partir de U+10000) se representan mediante un par de _surrogates_, siempre un _high surrogate_ primero y un _low surrogate_. Observa el ejemplo anterior en el que U+1000A se representaba mediante U+D800 (el primero de los _high surrogates_) seguido de U+DC0A (en el rango de los _low surrogates_).
  
Y ahora algo importante: **los _surrogates _estan reservados para su uso en UTF16**. Fuera de una cadena en UTF16 no se pueden usar y además siempre deben aparecer en pares (_high, low_). Una cadena Unicode que contenga un _surrogate_ sin su pareja es inválida. Y, insisto: sólo en UTF16. Eso significa que **en UTF8 no se necesitan _surrogates_**_. _A pesar de qué técnicamente es posible codificar una cadena Unicode que contenga _surrogates_ en UTF8, esta cadena **debería ser considera inválida**. Un parser de UTF8 puede negarse a codificar/decodificar dicha cadena: está fuera del estándard Unicode. Es decir, **hay secuencias de carácteres Unicode que NO SON cadenas válidas Unicode**.
  
Y quiero insistir en algo que he dicho al inicio de dicha sección pero que igual se te ha pasado: los _surrogates_ son _code points_, es decir son carácteres Unicode por sí mismos, a los que el estándard da un trato especial (en este caso formar parte de una secuencia UTF16).
  
**Nota: **UTF8 no necesita _surrogates _porque usa otro mecanismo para codificar los _code points _de Unicode (en función de si determinados bits de cada byte están establecidos o no, se sabe si este byte forma parte de un _code point _que ocupa más de un byte).
  
Vale... ya casi hemos diseccionado el tema, pero nos queda una cosilla más para tener el puzle más o menos completo...
  
**UCS-2, UCS-4**
  
[UCS][5] significa &#8220;Universal Character Set&#8221;  y es un estándard que define una lista de carácteres y su código. Así UCS-2 define una lista de carácteres que ocupan cada uno 2 bytes y UCS-4 lo extiende a 4 bytes.
  
¿Te suena muy parecido a Unicode, eso? ¡Es que lo es! Por suerte, y a pesar de que Unicode y UCS empezaron distinto, se han coordinado **y tenemos una única lista de _code points_. **UCS-2 define todos los _code points_ del rango BMP (0x0000-0xffff) mientras que UCS-4 define todos esos y los siguientes (hasta 31 bits). Lo importante es que los códigos son los mismos que Unicode.
  
Entonces... en que se diferencia UCS de Unicode? ¿Y p. ej. que diferencia hay entre UCS-2 y UTF16 o entre UCS-4 y UTF32?
  
Entre Unicode y UCS la diferencia está que el segundo es una mera lista de _code points_ con su código y Unicode va mucho más allá: define reglas de codificación (p. ej. _surrogates_) pero también de normalización de cadenas, _collation_ y demás. Es decir Unicode no es (sólo) una lista de _code points_, es todo un estándard que define como trabajar con ellos. Todo eso queda fuera del alcance de UCS.
  
Entre UTF16 y UCS-2, ambos usan un tamaño de _code unit_ de 16 bits. La diferencia está en que el primero puede representar todo el rango de _code points_ de UCS-4, mientras que el segundo, no (el segundo solo puede representar _code points_ en el rango BMP y en UCS el concepto de _surrogate_ no existe).
  
Entre UTF32 y UCS-4, ambos usan un tamaño de _code unit_ de 32 bits. La diferencia está en que el primero es, de hecho, un subconjunto del segundo. En UCS-4 el rango de carácteres válidos va de 0x0 &#8211; 0x7fffffff (31 bits), mientras que UTF32 al ser una representación de Unicode está limitado por el rango de carácteres definido en Unicode.
  
Bueno... lo dejamos aquí. Espero que este post os haya ayudado a aclarar algunos conceptos si es que los teníais confundidos 🙂

 [1]: https://en.wikipedia.org/wiki/ASCII
 [2]: https://es.wikipedia.org/wiki/Consorcio_Unicode
 [3]: https://es.wikipedia.org/wiki/Problema_del_a%C3%B1o_2000
 [4]: https://docs.microsoft.com/en-us/dotnet/api/system.char.convertfromutf32?view=netframework-4.7.2
 [5]: https://es.wikipedia.org/wiki/ISO/IEC_10646