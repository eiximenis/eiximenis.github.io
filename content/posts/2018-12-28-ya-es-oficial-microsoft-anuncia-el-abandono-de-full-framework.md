---
title: 'Ya es oficial: Microsoft anuncia el abandono de Full Framework'

author: eiximenis

date: 2018-12-28T14:51:23+00:00
geeks_url: /?p=2245
geeks_ms_views:
  - 1358
categories:
  - Sin categoría

---
A ver, era algo que se veía a venir... Microsoft ha anunciado finalmente el abandono de Full Framework, en favor de Net Core. Cualquiera que estuviese al tanto de los intríngulis de pasillo del gigante de Redmond sabía que ambos equipos andaban bastante a la greña.
  
<!--more-->


  
Parece que .NET Framework 4.8 será la última versión que de hecho saldrá &#8220;<span style="color: #0000ff;"><em>ya la hemos empezado y ahora queda como mal no terminarla</em></span>&#8220;, porque el equipo ya se está desmantelando. El soporte de Winforms y WPF de netcore3 ha sido la puntilla.
  
Es de hecho el paso final hacia esa &#8220;nueva&#8221; Microsoft que lidera con puño de hierro Satya Nadella. Tal y como reconocen de puertas para adentro &#8220;_<span style="color: #0000ff;">.NET Framework es reconocido como un producto de la era de Ballmer y esta era debe ser obliterada completamente</span>&#8220;_. Por supuesto los desarrolladores del equipo de .NET Framework no consideran que .NET Core esté preparado para reemplazar al Full Framework. Varios de sus desarrolladores afirman que &#8220;_<span style="color: #0000ff;">sí, han implementado winforms y wpf ahora que nadie desarrolla ya para escritorio.  Claro, como saben que nadie va a usar eso... Pero con webforms no se atreven. El día que Net Core pueda ejecutar webforms será un framework de verdad, hasta entonces es poco más que un jueguete</span>&#8220;_. Otras opiniones son que eso _&#8220;<span style="color: #0000ff;">ya lo hizo más o menos Mono en su día y eran cuatro matados con De Icaza, así que mérito tampoco tiene</span>_&#8220;.
  
Hay una cierta corriente en Microsoft que opina que con estos cambios, Microsoft está perdiendo toda la innovación. &#8220;<span style="color: #0000ff;"><em>Mira Net Core: es una copia de nodejs. ¡Hasta han copiado el infierno de npm! O fíjate en la &#8216;gran novedad&#8217; de Net Core 3 que podrás distribuír todo tu proyecto en un solo ejecutable. Si eso lo tiene Golang coñe! Solo copian... Y Blazor? No me jodas! Si es Silverlight bajo un nuevo nombre!&#8221;</em>.<span style="color: #000000;"> Son las mismas opiniones que piensan que antes Microsoft sí que innovaba</span>: </span>_<span style="color: #0000ff;">&#8220;Mira webforms... qué existia por aquel entonces? PHP que era igual de mierdoso que ahora, Coldfusion que era peor todavía, JSP con sus custom tags... nadie tenía un framework con viewstate y Page_Load! Rompimos el paradigma de lo que una aplicación web debía ser! Y WPF? XML para definir interfaces gráficas, botones que podían ser redondos y lo sacamos en 2006 sin editor gráfico ni nada. Con dos cojones!</span>&#8220;_
  
Otro de los desarrolladores del equipo de Full Framework opina con pesadumbre que todo lo que hace el &#8220;nuevo&#8221; equipo de _core _y derivados tiene mucha popularidad, pero que no es nada especial: &#8220;<span style="color: #0000ff;"><em>Es como lo de <a style="color: #0000ff;" href="https://blogs.msdn.microsoft.com/dotnet/2013/09/30/ryujit-the-next-generation-jit-compiler-for-net/">RyuJit</a>. Mucho ruído pero la verdad es que nosotros ya estábamos preparando KenJit. Y a ver, todos sabemos que Ryu es el que tiene nombre, pero Ken es el que se quedaba con las rubias de pote. Y no solo eso, también estábamos preparando BlankaJit con algoritmos muy, llamémosles eléctricos, para cargarse los memory leaks. Y por supuesto, nuestro quipo de R&D estaba empezando ChunliJit que le pegaba unas patadas a los objetos de Generación 2, que no veas</em></span>&#8220;.
  
Hay incluso quien va más lejos... &#8220;_<span style="color: #0000ff;">La culpa es de Satya, que no es que sea malo, pero tu ya me entiendes... ¿votas a Trump no? Pues ya me entiendes. No es de aquí. Viene con ideas que no encajan con una Gran América. No como Ballmer... Seguro que los de Core hacen donaciones a los demócratas. No son verdaderos patriotas. El Full Framework es americano. Por su parte Net Core... ¿qué es?</span>&#8220;_
  
Otras opiniones del equipo de Full Framework son que Net Core se está orientando demasiado a _startups_ con tanta CLI y cross-platform: &#8220;_<span style="color: #0000ff;">Tenemos que volver a los orígenes. Tenemos que volver a tener herramientas lentas y pesadas y un eco-sistema propio y cerrado. Eso es lo que quieren la mayoría de nuestros clientes. No lo olvidemos: nuestros clientes son cárnicas que cobran por cada día que tienen a un Junior a precio de Senior doctorado. Si lo que antes se tardaba 10 días, ahora con Core se hace en 2... ¿qué ocurrirá con ellos? ¡Es una hecatombe! Y si todas esas consultoras líderes en su sector desaparecen... ¿qué nos queda como cliente? ¡Cuatro startups que comprará Google! Será nuestro fin</span>&#8220;_. Por eso ven con disgusto como incluso el buque insigina de .NET, es decir Visual Studio, está abrazando Net Core: &#8220;_<span style="color: #0000ff;">Al principio con 2015 lo hicieron bien. Crearon el xproj que era una mierda pinchada de un palo, y así nadie podía usar bien Net Core con Visual Studio. Pero los desgraciados se han vendido y con 2017 ya empezaron a soportar Net Core correctamente. Ahí, perdimos la batalla</span>&#8220;. _Lo único que les queda es consolarse con las migajas: _<span style="color: #0000ff;">&#8220;Ahora puedes editar un csproj a mano, que aberración, pero al menos con el formato del sln y los guids infernales que se usan no han podido. Afortunadamente el desarrollador que escribió el código para tratar con ficheros sln se fue de la empresa y no queda nadie que entienda como funciona el puto formato... se quedará así eternamente!!!</span>&#8220;_.
  
Muchos culpan a Helsberg de este cambio: &#8220;_<span style="color: #0000ff;">Anders tiene parte de culpa. Cuando entró en Micro venía de Object Pascal y con C# y .NET hicimos algo bonito. Luego, alguien le comió el tarro y se puso con TypeScript. Por suerte el mundo pasaba de TypeScript, pero Google que siempre la jode, lo adoptó para Angular y ya entonces se convirtió en imparable. ¿Qué les costaba haber adoptado Flow en lugar de TypeScript? Así Helsberg habría dejado TypeScript y el movimiento startupil y open source este de las narices hubiese terminado</span>&#8220;_.
  
Es una pena, pero es así... Full Framework se abandona de forma oficial y los miembros de su equipo ya están buscando trabajo: &#8220;_<span style="color: #0000ff;">Lo ideal sería Oracle, su cultura anti-open source y super enterprise es lo que le llevará a dominar el mercado. Mira J2EE: con sus herramientas pesadas y ahí sigue... ¡en todas las Fortune 50!<span style="color: #000000;">&#8220;</span></span>_
  
En fin, toda empresa grande (y no tan grande) tiene sus batallas internas: esa no ha sido la primera en Microsoft ni será la última... ¡veremos que nos depara el futuro!
  
**Nota innecesaria:** Por supuesto, esto es una inocentada como ya habrás deducido... xD