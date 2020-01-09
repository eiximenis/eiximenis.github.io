---
title: 'Terminales y millones de colores: una historia complicada'
description: 'Terminales y millones de colores: una historia complicada'
author: eiximenis

date: 2019-04-05T17:28:27+00:00
geeks_url: /?p=2342
geeks_ms_views:
  - 883
categories:
  - .net
  - consola
  - netcore

---
Los que más o menos me seguís por [Twitter][1], quizá os habréis enterado de que estoy escribiendo una [librería _cross-platform_ (netstandard2) para desarrollar aplicaciones de consola][2]. Evidentemente [no es la única][3], es simplemente otra más y puedo asegurar que me lo paso genial desarrollándola.
  
Uno de los objetivos principales cuando empecé **era permitir usar true color** (es decir 16 millones de colores) en aquellos terminales que lo soportan y la verdad es que la historia del soporte de colores en terminales da para un post... y aquí estamos 😉
  
<!--more-->


  
En la [pasada netcoreconf de barcelona][4] dí una charla titulada &#8220;Aplicaciones de consola fáciles? Más quisieramos&#8221; donde contaba **las enormes diferencias entre el terminal de *NIX y la consola de Windows**, así como algunas novedades interesantes que trae Windows 10 al respecto ([aquí está la presentación][5]).
  
De todo lo que comenté brevemente en la charla, me quiero centrar hoy en el soporte de colores en los terminales y en especial en el soporte de true color.
  
**Estado del arte en *NIX**
  
El mundo *NIX representa el escenario más complejo que pueda haber: si uno quiere adaptarse a todas las situaciones debe tener presente que su aplicación puede estar ejecutándose en terminales _antiguos_ que no soporten todas las funcionalidades. En el mundo *NIX el terminal es un dispositivo que solo lee y envía carácteres. Eso significa que todo debe codificarse como una cadena de carácteres. Esta analogía es perfecta para texto (si recibo AB, pues sé que se ha pulsado la tecla A y luego la B) pero lo mismo ocurre para establecer comandos y opciones del terminal. Así para cambiar el color de un texto **se manda una secuencia predeterminada de carácteres**. Por razones históricas estas secuencias <span style="text-decoration: underline;"><em>suelen</em></span> empezar con el carácter [ESC][6] (código ASCII 27) y por eso se conocen generalmente con el nombre de &#8220;[secuencias de escape][7]&#8220;. La situación inicialmente **es que cada terminal definía sus propias secuencias de escape**, así que para poder lidiar con la variedad de terminales existentes, lo que se hizo **fue distribuir junto al SO una &#8220;base de datos&#8221; (en forma de ficheros) llamada termcap** que contenía una lista de terminales y para cada terminal que _capacidades_ (opciones y acciones posibles) tenía y que secuencia de escape había que mandar en cada caso. Posteriormente termcap evolucionó a _terminfo_ y eso es lo que se usa hoy en día. Los ficheros _terminfo_ son binarios (se generan a partir de ficheros de texto con una herramienta llamada tic) y su formato binario no tiene por qué ser compatible entre versiones de sistemas *NIX (aunque suele serlo). Tampoco su ubicación está definida por POSIX (y p. ej. distintas distros Linux tienen los ficheros en distintos sitios).
  
Una cosa **muy importante de _terminfo_, es que forma parte de tu SO**. Eso significa que versiones distintas de SO pueden tener versiones distintas de los ficheros_ __terminfo_. Y, dado que esos ficheros no los han creado los creadores de terminales si no usuarios de *NIX, **hay ficheros _terminfo_ con errores**, especialmente en terminales viejos que nadie tiene muy claro qué secuencias aceptan para qué casos. Con el tiempo se van arreglando esos errores, pero depende de qué versión de *NIX tengas tus ficheros _terminfo_ estarán más o menos actualizados.
  
[<img class="alignnone size-full wp-image-2349" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/04/terminfo-files.png" alt="Terminal que muestra los ficheros terminfo ubicados (en mi caso) en /lib/terminfo" width="650" height="188" />][8]
  
Si tienes un sistema *NIX a mano (un Linux, MacOS o, si estás en Windows, el WSL o cygwin) abre un terminal y teclea &#8220;infocmp&#8221;. Eso te imprimirá las capacidades de tu terminal y las secuencias de escape para cada una de ellas (ojo que esas secuencias de escape son &#8220;plantillas con parámetros&#8221; a partir de las cuales se crea la secuencia de escape real usando la API de terminfo).
  
Por lo tanto en \*NIX debes usar la API de terminfo para leer esas cadenas de escape del terminal actual y enviarlas cuando desees hacer hacer &#8220;cosas&#8221; con el terminal (tales como borrar la pantalla, usar colores o mover el cursor). Para saber cual es el &#8220;terminal actual&#8221; la opción suele ser usar el valor de la variable de entorno &#8220;TERM&#8221;. Si pruebas de ejecutar &#8220;echo $TERM&#8221; verás cual es el terminal actual de tu sistema \*NIX. Lo más probable es que el valor sea &#8220;_xterm-256color_&#8221; que es uno de los terminales más usados en los sistemas actuales, pero no tiene por qué.
  
Para usar colores las capacidades básicas de _terminfo_ que debes usar son _setf, setaf, setb, _y _setab _(si tecleas _infocmp | grep <nombre-capacidad> _verás la plantilla para generar la secuencia de escape a usar en cada caso).
  
La utilidad _tput _recibe como parámetro una capacidad de terminfo, sus parámetros y envía al terminal la secuencia de escape correspondiente. Es un mecanismo rápido para probar las distintas capacidades. Abre tu terminal y teclea &#8220;_echo $(tput setaf 0)$(tput setab 5) hello world_&#8220;. Lo más probable es que veas &#8220;hello world&#8221; en negro y con fondo morado. Bienvenido al mundo de los colores.
  
Si quieres ver las secuencias de escape enviadas por tput puedes usar [xxd][9]. Así en mi terminal si uso &#8220;echo -n $(tput setaf 0)$(tput setab 5) | xxd&#8221; lo que veo es:

<pre>00000000: 1b5b 3330 6d1b 5b34 356d .[30m.[45m</pre>

Donde puedo ver <ESC>[30m y <ESC>[45m que son las dos secuencias de escape mandadas.
  
Divertido ¿verdad? Pues aún hay más. Para empezar la dualidad setb/setab para establecer el color fondo y setf/setaf para establecer el de texto. ¿Por qué hay dos? Pues setf/setb son las primeras versiones y luego cuando salió el estándard ANSI de colores se añadió la capacidad setaf y setab en aquellos terminales que los soportaban. La primera versión de ANSI usaba 3 bits (8 colores) que se expandieron luego a 4 bits (un bit para intensidad). Así en ANSI el color 5 (0101) es el morado, mientras que el color 13 (1101) es &#8220;morado intenso&#8221;. Lo puedes verificar tu mismo jugando con tput:
  
[<img class="alignnone size-full wp-image-2347" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/04/morado-vs-morado-intenso.png" alt="Terminal *NIX donde se muestra texto con &quot;tput setaf 5&quot; en morado y luego con &quot;tput setaf 13&quot; con morado intenso" width="435" height="93" />][10]
  
Así esos 4 bits nos dan 8 \* 2 colores distintos, que no son realmente 8 \* 2, ya que el 0000 es el negro y no tiene sentido que el 1000 fuese el &#8220;negro intenso&#8221;. Por ello el 1000 se reservó para un gris. Así que realmente tenemos 7 colores en su variante normal y brillante, el negro y un gris. Este es el **estándard ANSI de 4 bits**.
  
Luego está la otra gran dualidad: los &#8220;terminales tipo Tektronix&#8221; y los &#8220;terminales tipo HP&#8221;. En los terminales tipo Tektronix tenemos un máximo de N colores y podemos combinarlos como queramos como fondo y texto. Eso nos da un máximo de N \* N combinaciones. Por otro lado en los terminales de tipo HP, tenemos un número N de colores y podemos combinarlos solo de M maneras distintas (donde M es menor que N \* N). Estos terminales no permiten establecer el color de fondo y de texto de forma independiente, si no que debemos crear &#8220;un par (fondo, texto)&#8221; y establecer este par como color del carácter. Y podemos tener hasta &#8220;M pares&#8221; de forma simultanea. La mayoría de terminales actuales son &#8220;tipo Tektronix&#8221; (y son los que usan setf/setaf y setb/setab). Para los terminales &#8220;tipo HP&#8221; (mayoría en la antigüedad) debe usarse la capacidad scp (set color pair).
  
La capacidad &#8220;colors&#8221; te da el valor de colores soportados (en mi terminal xterm+256color &#8220;tput colors&#8221; me devuelve 256). Además si existe la capacidad &#8220;ccc&#8221; (can change color) eso significa que se pueden redefinir los colores: **es decir puedes modificar la paleta de colores**. Si &#8220;infocmp | grep ccc&#8221; te devuelve &#8220;ccc&#8221; significa que puedes redefinir los colores. En este caso **puedes usar la capacidad initc para inicializar un valor x (0...colors-1) con los valores RGB deseados.** Así echo $(tput initc 1 500 500 500)$(tput setaf 1)Hello imprimirá &#8220;Hello&#8221; con el color número 1 que antes hemos definido como RGB (500,500,500) (los rangos suelen ir de 1 a 1000), es decir un gris. Con _initc_ cambias la paleta actual, por lo que **si tienes algo en el terminal impreso con el color x y lo modificas con intc, los carácteres existentes de este color se verán afectados también.**
  
Posteriormente la gama de colores ANSI se amplió a 256 colores (8 bits). En este caso setaf/setab soportan valores de 0 a 255 y hay paleta pre-establecida de colores. De todos modos, por lo general, los terminales que soportan 256 colores suelen soportar también _ccc_ por lo que puedes modificar la paleta a tu gusto. La mayoría de terminales actuales permiten usar paletas de 256 colores simultáneos a elegir entre una gama de 16 millones.
  
Hasta aquí lo básico de colores y terminfo. Pero **no se puede hablar del desarrollo de terminal en *NIX sin hablar de [ncurses][11]**. Esta es una librería que es un _standard de facto_ en el mundo *NIX **que nos ofrece una API unificada** sea cual sea el tipo de terminal que tengamos debajo. Eso significa que en ncurses tengo un método para borrar la pantalla y este método usará la secuencia de escape correcta. Por supuesto ncurses por debajo usa _terminfo_ (e incluso _termcap__ _en versiones *NIX realmente antiguas) pero nos abstrae de él. Cuando crearon _ncurses_, para dar soporte a la dualidad HP/Tektronix optaron **por usar una API orientada en pares de colores**. Es decir, en ncurses no estableces el color de fondo y texto por separado, si no que creas un par de colores (fondo, texto) y estableces un par para el carácter. Vamos, que ncurses expone el modelo de HP. Si el terminal es tipo Tektronix, es ncurses quien traslada el &#8220;par x&#8221; a los colores de fondo y texto correspondientes. La razón de que funcione así es que ncurses es una evolución de curses y cuando curses apareció lo normal es que los terminales fuesen &#8220;tipo HP&#8221;.
  
Claro, el modelo de pares no escala muy bien. En 256 colores tenemos 65536 pares que bueno... aún podemos meter en memoria. Pero... si queremos 16 millones de colores, la cosa se dispara.
  
Vale, vayamos ahora al soporte de **true color**. Lo primero a saber **es que no hay un mecanismo estandarizado para soportar true color**, pero bueno. Lo primero **es saber si tu terminal soporta true color**. No hay entrada en _terminfo_ que te lo diga, así que actualmente se suele usar la variable de entorno COLORTERM. Si existe y su valor es &#8220;truecolor&#8221; o &#8220;RGB&#8221; es que el terminal que usas soporta _true color_. En mi Ubuntu 18.04 el valor de COLORTERM es &#8220;truecolor&#8221; para indicar precisamente este soporte. Pero **el fichero _terminfo_ que viene por defecto (xterm-256color) no tiene las definiciones de las secuencias de escape correctas... **simplemente porque no se ha definido qué entradas de _terminfo_ deben usarse para _true color_.
  
El modo _true color_ se diferencia del modo de 256 colores en algo fundamental: el primero **funciona, como ya hemos visto, con paleta de colores** y el segundo **es de acceso directo**. Con los modos de 256 o bien tenemos una paleta fija (los colores ANSI) o (si se soporta ccc) podemos modificarla (con llamadas a _initc_) y luego mediante _setaf/setab_ seleccionamos cual de esos 256 colores usamos en cada caso. Con un modo de acceso directo no nos interesa esa aproximación: lo que queremos es establecer los valores (R,G,B) de cada carácter de forma independiente. Pero, como hemos visto, terminfo **no define ninguna capacidad que podamos usar**. Cual es la aproximación que puede seguir? La primera es tener _hardcodeadas_ en tu código las secuencias de escape a usar para usar un color (RGB) concreto. Todos los terminales parece ser que usan las mismas, así que esa aproximación funcionaría.
  
Otra aproximación es reutilizar las capacidades setaf/setab y tener un fichero _terminfo_ modificado que genere las nuevas secuencias de escape. Por ejemplo ncurses, en su última versión la 6.1, ha optado por esa aproximación. Así ncurses 6.1:

  * Mira si _terminfo_ tiene una capacidad llamada RGB (sin valor, solo debe existir)
  * Si existe **asume que el terminal &#8220;puede usarse&#8221; en modo directo** (sin paletas) y asume que _setaf/setab_ mandan las secuencias de escape correctas.

Si tu terminal es &#8220;xterm-256color&#8221; puedes encontrar un fichero _terminfo_ con soporte de color directo en https://invisible-island.net/xterm/terminfo-contents.html#tic-xterm-direct. Este es un fichero _terminfo_ que reutiliza xterm-256color pero añade la capacidad RGB y redefine _setab/setaf_. Debes usar [tic][12] para instalar este fichero en tu sistema.
  
De todos modos, **hasta donde yo sé ncurses todavía no está adaptada al modo de color directo**. El problema es que su API sigue estando basada en pares de colores y eso es imposible para 16 millones de colores. Aunque reconozco que no he investigado mucho más, **creo que hoy por hoy, si quieres soportar true color debes abandonar ncurses y usar _terminfo_ a mano** (o si la compatibilidad con terminales antiguos no te importa puedes tener _hardcodeadas _las secuencias de escape que es lo que hace mucha gente).
  
**Estado del arte en Windows**
  
Las cosas son mucho más sencillas en Windows... o al menos lo parecen. A diferencia de \*NIX, Windows no se preocupa de terminales antiguos y a diferencia de \*NIX, Windows no ve el terminal solo como un emisor/receptor de carácteres, si no como un &#8220;objeto del sistema&#8221;. Y por lo tanto, Windows ofrece una [API de consola][13], que desde el punto de vista de desarrollador es una maravilla, aunque, como veremos luego viene con un precio enorme que pagar.
  
La API de consola de Windows, entiende de terminales modernos por lo que, tenemos métodos para mover el cursor, para obtener datos del teclado, del ratón y por supuesto para gestionar la pantalla y los colores. Nada de secuencias de escape: métodos de la API de Win32. Y por supuesto, como forma parte de Windows, entiende que la consola es una ventana, de forma que puedes abrir tantas consolas como necesites (cada proceso puede tener una).
  
Ya te digo: como desarrollador es imposible no enamorarse de la API de consola de Windows. No solo es potentísima, si no que está bien diseñada y da soluciones para casi todo. En un mundo Windows, donde la gente suele estar **físicamente delante del ordenador** todo funciona de mil maravillas. Pero **hay dos grandes limitaciones con la API de Windows actual**.
  
La primera limitación **es que todo aquello que la API no soporte, no vas a poder hacerlo**. En *NIX no hay esa limitación porque siempre puedes mandar secuencias de escape que te hagas tú y oye, si el terminal las entiende todo funcionará. Pero en Windows no **y la API tiene una gran limitación actualmente: solo soporta colores ANSI de 4 bits**. Sí, eso significa **que estás limitado a 16 colores. Game over** (bueno, no del todo... sigue leyendo xD).
  
La segunda limitación tiene que ver con un escenario **muy típico en el mundo *NIX pero muy poco frecuente en el mundo Windows hasta hace relativamente poco**: el acceso desde un terminal remoto. En *NIX es típico usar ssh, conectarse a un ordenador remoto y desde esa terminal ejecutar comandos que pueden incluír programas de consola &#8220;a pantalla completa&#8221; como [vi][14] o [emacs][15]. Y si el programa está &#8220;bien hecho&#8221; (es decir usa _terminfo_) el programa se adaptará al terminal remoto (el tuyo). Eso mismo **en Windows es, con perdón de la expresión, un puto dolor**. Me dirás que hay clientes de ssh en Windows como Putty, pero es que el dolor al que me refiero no es tuyo si no de los pobres desgraciados que han tenido que crear un **servidor ssh** para Windows. De verdad, que programas como [OpenSSH][16] o emuladores de terminal como [ConEmu][17] funcionen en Windows es casi un milagro. ¿Conoces los emuladores de terminal? En *NIX son muy comunes (p. ej. xterm que es el que se usa por defecto en la mayoría de distros de Linux), de hecho son tan comunes **que (casi) siempre usas uno**. La razón es que un terminal es un dispositivo _hardware_ (un monitor y un teclado p. ej.). Un **terminal no es una ventana flotante**. Cada vez que abres una ventana flotante, en un Linux o un MacOs usas un emulador de terminal. Un emulador de terminal lee carácteres, intepreta las secuencias de escape y muestra los resultados.
  
Imagina que quieres crear un emulador de terminal. En \*NIX crearías un programa que básicamente leería carácteres. Carácter que recibe lo muestra en su ventana, a no ser que forma parte de una secuencia de escape que en cuyo caso procesa adecuadamente. Por lo tanto, en \*NIX, abres tu emulador de terminal (como xterm) y ejecutas un programa de consola (como emacs o midnight commander). Este programa empezará a mandar carácteres y secuencias de escape al emulador de terminal (para este programa el emulador de terminal es _el terminal_) y éste los procesará para mostrar los resultados de forma correcta. Pero en Windows..... ¡uy en Windows! El problema es que en Windows tu abres el emulador de terminal y ejecutas un programa. Este programa usará la API de consola de Windows para pintar la pantalla o mover el cursor. Pero **la API de consola de Windows ¡afectará a la consola real de Windows**, no a la ventana del emulador de terminal. Por lo tanto, lo que hacen los emuladores de terminal en Windows es **tener la ventana de consola real del sistema en una posición no visible** y es en esta ventana en la que se ejecuta realmente el programa. Luego, con un temporizador, realizan _scrapping_ del contenido de dicha ventana y refrescan la ventana real que tu estás viendo. En el caso de un servidor ssh para windows ocurre lo mismo (de hecho un _servidor_ ssh es un emulador de terminal)_._ El programa remoto se ejecuta en una ventana de la consola de Windows, el servidor ssh hace scrapping de dicha ventana y manda por red el resultado hacia el cliente. ¡Qué diferencia con *NIX donde el servidor ssh solo debe mandar los carácteres que recibe!
  
En fin... ¿a qué ahora miras a ConEmu con otros ojos?
  
Bueno, tenemos pues las dos limitaciones en Windows: colores ANSI de 4 bits y remoting de aplicaciones muy complejo. Por suerte **Windows 10 viene a cambiar las reglas del juego**.
  
Windows 10 incorpora novedades en el sistema de consola de Windows. Es importante porque, a grandes rasgos, el sistema de consola de Windows 7 es el mismo que el de Windows 2000. Así que el hecho de que Microsoft esté haciendo cambios importantes en él, es algo muy destacable.
  
El primer cambio que Microsoft **ha implementado ya en Windows 10**, es... ¡**soporte para secuencias de escape! **Efectivamente: ahora la consola de Windows 10 entiende de secuencias de escape. Además, como no se pretende soportar un porrón de terminales del pleistoceno, no hay _terminfo_ ni nada que se le parezca: [el SO define unas secuencias y son esas las que puedes usar][18]. En este caso son secuencias definidas por el estándard ANSI (sí, hay un estándard de secuencias de escape al cual todos los terminales modernos, incluídos los emuladores modernos en \*NIX) están adheridos). P. ej. en \*NIX, el emulador xterm-256colors usa secuencias de escape ANSI. Un programa que emita esas secuencias de escape se verá correctamente en el terminal de Windows 10... En el de Windows 7 solo verás horror y destrucción. Eso viene acompañado del **soporte de colores ANSI de 8 bits e incluso _true color_ pero solo a través de secuencias de escape.** Es decir, la API de consola no se ha adaptado para soportar más de 16 colores, pero mediante secuencias de escape, ¡tenemos los que queramos! De hecho **la recomendación es empezar a usar secuencias de escape en lugar de la API de consola.**
  
Si te preguntas por qué le ha dado a Microsoft por incorporar secuencias de escape en su consola después de tantos años de pasar de ellas, la razón es muy simple y tiene tres letras: WSL. Recuerda: WSL ejecuta programas Linux nativos, pero se ejecutan en la consola de Windows... por lo tanto la consola de Windows debe actuar como emulador de terminal y entender las secuencias de escape.
  
La **segunda novedad es incluso de mayor calado y aparece en Windows 1809** y se trata de la API ConPTY. Y, amigos, eso es la bomba: **ConPTY dota a Windows de un modelo de PTY completo como el de *NIX**. Una descripción detallada de ConPTY queda fuera del alcance de este post, pero, para resumir, diremos que **básicamente lo que ConPTY hace es traducir las llamadas de la API de Consola a secuencias de carácteres**. Y eso, está pensado para simplificar el remoting de aplicaciones y los emuladores de terminal. ¿Recuerdas todo lo del _scrapping_ que comentaba antes? Pues con ConPTY ahora todo es mucho más sencillo. La idea es que el emulador de terminal **se conectará con ConPTY que a su vez se conectará con la consola real. **Cuando una aplicación que se ejecuta en la consola real usa la API de consola para hacer algo, ConPTY traducirá ese algo a una secuencia de carácteres y mandará esa secuencia de carácteres a quien esté conectado a él. Por lo tanto nuestro emulador de terminal ya no tiene que hacer _scrapping_ ni nada: envía carácteres a ConPTY y esos carácteres llegan a la consola (y al programa real). Cuando el programa usa la API de Consola, ConPTY se encarga de generar una secuencia de escape correcta y es esa secuencia de escape la que lee el emulador de consola. De hecho el emulador de consola desconoce cual es la consola real asociada: él solo conoce a su ConPTY asociado. En resúmen, con ConPTY Windows está adoptando el modelo que tiene *NIX, al tiempo que mantiene la compatibilidad de la API de consola para no romper todas las aplicaciones actuales.
  
Por lo tanto si quieres hacer una aplicacion de consola para Windows y aprovechar al máximo las capacidades debes:

  1. Usar la API de consola de Windows si estás en Windows 7 o anteriores
  2. Usar las secuencias de escape si estás en Windows 10

Y tener presente que tu aplicación en Windows 7 solo podrá usar 16 colores.
  
**Estado del arte en .NET**
  
En .NET las cosas son muy sencillas: la API de consola de .NET (System.Console vamos) se creó a imagen y semejanza de la API de consola de Windows, así que ha heredado sus limitaciones (p. ej. solo 16 colores) y ha añadido algunas de cosecha propia (no se soporta el ratón).
  
Con .net core, actualmente, tenemos las mismas limitaciones y hay una implementación de System.Console por cada plataforma:

  * En Windows, se usa la API de Windows via P/Invoke
  * En Linux y MacOS se usan secuencias de escape. La implementación para Linux usa _terminfo_ para obtener las secuencias de escape. Eso sí, la gestión es simplificada (p. ej. asumen que el terminal es tipo Tektronix y siempre usan setaf/setab) pero lo realmente interesante **es que no usan la API de terminfo** (las funciones [_tigetstr_ y _putp_][19] básicamente) via P/Invoke si no que **se han currado [su propio parser del fichero terminfo][20]**. Entiendo que es por rendimiento. Lamentablemente esta clase no es pública 🙁

¡Y eso es todo! Ya ves... y parecía que crear una aplicación de consola era sencillo, ¿verdad? Pues solo hemos hablado de colores... cualquier día hablaré de la gestión del teclado y... ¡del soporte de ratón! Otro mundo...

 [1]: https://twitter.com/eiximenis
 [2]: https://github.com/eiximenis/tvision2
 [3]: https://github.com/migueldeicaza/gui.cs
 [4]: http://netcoreconf.com/
 [5]: https://es.slideshare.net/eduardtomas/aplicaciones-de-consola-fciles-ms-quisieramos-139527302
 [6]: https://en.wikipedia.org/wiki/Escape_character
 [7]: https://en.wikipedia.org/wiki/ANSI_escape_code#Escape_sequences
 [8]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/04/terminfo-files.png
 [9]: https://www.unix.com/man-page/all/1/xxd/
 [10]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/04/morado-vs-morado-intenso.png
 [11]: https://es.wikipedia.org/wiki/Ncurses
 [12]: https://www.systutorials.com/docs/linux/man/1-tic/
 [13]: https://docs.microsoft.com/en-us/windows/console/console-functions
 [14]: https://es.wikipedia.org/wiki/Vi
 [15]: https://es.wikipedia.org/wiki/Emacs
 [16]: https://www.openssh.com/
 [17]: https://conemu.github.io/
 [18]: https://docs.microsoft.com/en-us/windows/console/console-virtual-terminal-sequences
 [19]: https://linux.die.net/man/3/tigetstr
 [20]: https://github.com/dotnet/corefx/blob/master/src/System.Console/src/System/TermInfo.cs