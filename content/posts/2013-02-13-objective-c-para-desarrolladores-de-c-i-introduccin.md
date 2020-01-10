---
title: 'Objective-C para desarrolladores de C# (i)–Introducción'

author: eiximenis

date: 2013-02-13T18:34:02+00:00
geeks_url: /?p=1631
geeks_visits:
  - 3091
geeks_ms_views:
  - 905
categories:
  - Uncategorized

---
Muy buenas! Con este post inicio una serie de posts (como siempre, ni idea de cuantos van a ser) dedicado especialmente a **desarrolladores de C# que quieran empezar con Objective-C**. No soy un experto en Objective-C ni esta serie pretende que te conviertas en un experto en este lenguaje. Tampoco es un tutorial de Objective-C. Es simplemente una ayuda para todos aquellos desarrolladores con un background de C# y .NET que tengan curiosidad por ver como es el lenguaje _de la manzana_.

A lo largo de esta serie intentaré tocar el máximo número de temas y eso incluirá:

  * Lenguajes: Vamos a comparar las características intrínsecas de ambos lenguajes. Por intrínsecas me refiero a las características que no dependen de ninguna API o librería externa como sintaxis, tipos básicos, etc… 
  * Herramientas: Por mucho que sean posts de Objective-C y de C# ambos lenguajes están muy enfocados a plataformas muy concretas y se suelen usar con herramientas muy concretas. Así que hablaré de XCode y de Visual Studio. Pero eso no quita que existan otras herramientas para ambos lenguajes. 
  * Plataforma: Los lenguajes, si nos ceñimos a su definición, son agnósticos de plataforma. Pero vamos a terminar hablando de plataforma, porque todo lenguaje necesita una que le de soporte. En el caso de C# tenemos la BCL y luego todo .NET. En Objective-C tenemos _Foundation_ y luego Cocoa y las apis de iOS (no hablaré de desarrollo para MacOS porque no tengo ni idea). Así que, en algún momento, compararemos las plataformas asociadas a cada lenguaje. Por supuesto esta comparación no será, porque no lo pretende porque no puede, ser una comparación exhaustiva. 

Todos los posts de la serie estarán escritos des del punto de vista de alguien que viene (y por lo tanto conoce) el mundo de C# y .NET. Esta serie **no busca en ningún momento un ganador**. Por eso que nadie busque en las comparaciones vencedores y perdedores. Eso queda en el juicio de cada uno.

Y finalmente… la pregunta del gritón de dólares: **¿Necesito un ordenador de Cupertino para seguir la serie?**

Pues para seguirla _toda, toda, toda_ pues obviamente sí. Cuando hablemos de XCode o de Cocoa vas a necesitar un Mac si quieres “hacer ejemplos en vivo”.

De todos modos doy por supuesto que si te ha picado la curiosidad por Objective-C es que te estés planteando el desarrollar para iOS, por lo que necesitarás un Mac tarde o temprano 😛 **Pero, es posible hacer muchos de los ejemplos que presentaré en un PC con Windows**. Existe el runtime de Objective-C para Windows, por lo que si lo que pretendes es simplemente trastear con Objective-C podrás hacerlo desde Windows. Obviamente con limitaciones.

Luego ya podrás decidir si prefieres comprarte un Mac o bien prefieres contactar con <a href="https://twitter.com/jfmiranda75" target="_blank" rel="noopener noreferrer">Juan Fco. Miranda</a> quien te guiará por los sinuosos caminos del _Hackintosh_.

**Instalación de Objective-C en Windows**

Bueno, el primer paso es ir a la página de descargas para <a href="http://www.gnustep.org/experience/Windows.html" target="_blank" rel="noopener noreferrer">Windows de GNUStep</a> y descargarte:

  1. GNUstep MSYS System 
  2. GNUstep Core 
  3. GNUstep Devel 

Luego instalatelos **en este mismo orden**. Yo por ejemplo los tengo instalados en D:GNUStep

GNUStep consiste en una serie de herramientas, como gcc para compilar Objective-C y también tiene ports de parte de la plataforma (está Foundation y parte de Cocoa implementado).

GNUStep es para hombres y no viene con ningún IDE ni nada parecido. Aunque técnicamente no necesitas ninguno ya que GNUStep te habrá instalado make.exe. Yo no conozco ningún IDE de Objective-C para Windows por ahí (personalmente no uso Objective-C bajo Windows).

Para probar si GNUStep está bien instalado vamos a realizar nuestro HelloWorld.

Abre tu editor de texto favorito y crea un fichero llamado HelloWorld.m con el siguiente código:

<pre style="background: #dddddd">#import &lt;Foundation/Foundation.h&gt;
int main (void)
{
        NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] init];
        NSLog (@"Hello, World!");
        [pool drain];
        return;
}</pre>

Guardalo como HelloWorld.m (la extensión .m es la que se suele usar para crear archivos Objective-C)

Ahora nos toca usar gcc.exe para compilar este archivo. Ello deberemos hacerlo desde línea de comandos. Abres una línea de comandos y te vas donde tengas el archivo HelloWorld.m y tecleas (todo en una misma línea):

<font size="2" face="Courier New">gcc HelloWorld.m -I D:/GNUstep/GNUstep/System/Library/Headers -L D:/GNUstep/GNUstep/System/Library/Libraries -fconstant-string-class=NSConstantString -o HelloWorld.exe -lobjc -lgnustep-base</font>

Las opciones que estamos usando son las siguientes:

  * -I <path_includes>: Path raíz de los archivos de cabecera 
  * -L <path_libraries>: Path raíz de los archivos de librería 
  * -fconstant-string-class <clase>: Nombre de la clase a usar cuando haya constantes string. 
  * -o <fichero>: Nombre del fichero de salida 
  * -lobjc: Para enlazar con la librería de objective C 
  * -lgnustep-base: Para enlazar con la librería de GnuStep 

Esto te debería compilar y generar el HelloWorld.exe… Si es así, ya estás listo para empezar a programar con Objective-C bajo Windows 😉

En sucesivos posts veremos como compilar usando XCode (por si tienes un Mac) y empezaremos a ver las diferencias entre Objective-C y C#).

Nos vemos!