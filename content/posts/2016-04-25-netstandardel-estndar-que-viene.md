---
title: netstandard–El “estándar” que viene
description: netstandard–El “estándar” que viene
author: eiximenis

date: 2016-04-25T10:00:59+00:00
geeks_url: /?p=1773
geeks_ms_views:
  - 2894
categories:
  - .net

---
Cuando .NET salió, las cosas eran muy sencillas: había una sola versión de .NET, el .NET Framework, así que como mucho debíamos saber para que versión de .NET era una determinada librería. “Oh, la librería es solo para .NET 2.0 y yo uso .NET 1.1, que mala suerte”. Al margen de eso no había mucho más, todos teníamos claro que significaba .NET.

<!--more-->

Para los desarrolladores de librerías el mundo no era tan sencillo: si uno quería ofrecer una versión de una librería que funcionara tanto en .NET 1.1 como en .NET 2.0, podía optar por lo sencillo que era compilarla solo para .NET 1.1. O podía tener dos versiones de la librería, una para .NET 2.0 (aprovechando todas sus ventajas) y otra para .NET 1.1 y compilar ambas. Tener “dos librerías” es tal y como suena: dos ficheros .csproj, que podían compartir más o menos código (ficheros .cs) en función de las circunstancias. En aquellos tiempos lidiar con varios .csproj, ficheros .cs compartidos entre ellos y por supuesto la compilación condicional era el pan de cada día.

Pero con el tiempo fueron surgiendo “sabores” de .NET. Todos ellos compartían una parte de la API básica de .NET (la BCL – Base Class Library) y diferían en el resto. Empezando por el Compact Framework, siguiendo por Silverlight y luego las distintas reencarnaciones del SDK de Windows Phone hasta llegar a UWP.

Para los desarrolladores de librerías el caos empezaba a ser devastador: tu librería es para .NET4 y también para Silverlight y ya puestos para WP7.5 y WP8. Pues ahí andabas, con cuatro .csprojs. El problema principal era que al final tenías 4 binarios (ensamblados) distintos a pesar de que en algunos casos podían incluso ser compilados a partir de exactamente los mismos ficheros .cs. Tener cuatro ensamblados significaba los 4 .csproj (una limitación de .csproj es que solo permite generar binarios para una plataforma, limitación que desaparece con el nuevo project.json), con sus settings, referencias, etc.

Pero incluso los desarrolladores de aplicaciones se empezaban a ver salpicados por todo este proceso… Que querías compartir código de una librería de negocio en una app .NET y Silverlight? Pues debías tener dos .csproj, uno para que te compilase la librería para .NET y otro para que te la compilara en Silverlight. El problema era ya evidente cuando salieron las “aplicaciones universales”. ¿Os acordáis (seguro que sí) de la plantilla de proyecto de aplicación universal? Aquel pseudo-proyecto “shared” donde meter el código compartido y luego el proyecto por cada plataforma… 

**Round 1 – Portable Class Libraries**

En algún momento alguien en Redmond se dio cuenta que eso no podía aguantar mucho más. La solución: las _portable class libraries_ o PCLs. Las PCLs fueron toda una bendición, en especial para los desarrolladores de librerías: Si antes, para hacer una librería compatible con .NET4, Silverlight y WP necesitaba 3 .csproj, ahora bastaba con uno. Al crear una PCL se definía un _perfil_ contra el que compilar. Los perfiles eran combinaciones válidas de plataformas que compartían una API común. Además VS tenía todas las definiciones de perfiles, por lo que nos impedía que llamáramos a un método que podía estar en .NET pero no en Silverlight.

Con una PCL teníamos un solo proyecto (.csproj) que generaba un solo binario que era compatible con todas las plataformas que conformaran el perfil que se había elegido al crear la librería (por supuesto, VS era lo suficientemente hábil para no preguntarnos el perfil, en su lugar nos preguntaba qué plataformas queríamos soportar y él infería el perfil a partir de nuestra respuesta).

La vida de los desarrolladores de librerías fue mucho más sencilla: con un solo proyecto .csproj podían dar soporte a varias plataformas y solo debían volver a varios .csproj cuando no existía perfil de PCL válido para todas las plataformas que se querían soportar.

Lo mismo para los desarrolladores de aplicaciones: compartir código de negocio entre una aplicación WPF y una aplicación Windows Phone era posible gracias a una PCL. El pseudo-proyecto “shared” de la plantilla de “aplicación universal” ya no era necesario

Esta es la situación hoy en día… y va a cambiar

**Pero… ¿cuál es el problema de las PCLs?**

Las PCLs simplifican la vida a los desarrolladores, pero vienen con un problema de diseño de base: **las definiciones de los perfiles son estáticas e immutables con el tiempo**.

Es decir, si un desarrollador crea una librería PCL contra un determinado perfil, pongamos el 111 que contiene .NET Framework 4.5, Windows 8 y Windows Phone 8.1 estas son las plataformas que esta PCL va a soportar. A nivel de desarrollo significa que el desarrollador de la PCL está limitado a las APIs comunas de esas tres plataformas.

Imagina que en un futuro aparece un sabor nuevo de .NET (.NET para XYZ) que es un _superconjunto_ de la api para Windows 8. Es decir, todas las APIs de Windows 8 están soportadas en .NET para XYZ. A priori no debería haber ningún problema en que nuestra PCL funcione en .NET para XYZ (ya que esta contiene todas las APIs de Windows 8). Pero a la práctica no la vamos a poder usar.

Si crearámos un proyecto de tipo .NET para XYZ y intentaramos usar la librería PCL veríamos que no podríamos. Se la intentáramos instalar via nuget, éste nos diría que no hay versión de la PCL compatible con .NET para XYZ. Esto es porque el perfil de la librería PCL se asigna al crear la PCL y el perfil contiene una lista de plataformas soportadas. Estas y solo estas son las que la PCL soportará. El que salga a posteriori otra plataforma que podría ser compatible (a nivel de API ofrecida) es irrelevante.

Esto es lo que pretende solucionar _netstandard_

**Como va a funcionar netstandard**

Cuando ahora se crea una PCL se le asignan distintos _platforms monikers_ que representan las plataformas que soporta nuestra PCL. Así el perfil 111 que comentábamos antes tiene los _monikers_ portable-net45, netcore45 y wpa81 que define las tres plataformas soportadas. Generalmente usamos más los _monikers_ que el número de perfil, así generalmente se habla de _portable-net45+netcore45+wpa81_ en lugar “Perfil 111”. 

Con _netstandard&nbsp;_ lo que aparece es un nuevo _moniker_ que es _netstandard_. Ahora cuando desarrollemos una librería no vamos a vincularla a un conjunto de plataformas como hasta ahora: vamos a vincularla contra una versión de _netstandard_. **Actualmente hay definidas seis versiones de _netstandard_.** Es importante comprender que _netstandard_ es compatible “hacia atrás”: la versión 1.1 de _netstandard_ incorpora todas las apis definidas en _netstandard 1.0_. **Cada versión de _netstandard_ define un conjunto de APIs que pueden usarse**.

La **norma para desarrolladores de librerías es ceñirse a la versión de _netstandard_ más baja posible.** ¿Y eso por qué?

Pues por qué, las _plataformas reales_ (tales como Xamarin, UWP, .NET Framework, y las que sean) implementarán una versión determinada de _netstandard_. Así, una plataforma que implemente _netstandard 1.2_ podrá usar librerías PCL que estén compiladas para la versión 1.2 o anteriores de _netstandard_. Así, si en un futuro aparece “.NET para XYZ” que implementa _netstandard_ 1.2, cualquier PCL existente que haya sido generada contra _netstandard_ 1.2 o anterior (1.1 o 1.0) se podrá usar en proyectos “.NET para XYZ” a pesar de que dicha plataforma no existía cuando se creó la PCL. Así, solucionamos el problema principal de las PCLs: que la lista de plataformas válidas para la PCL era estática.

La **norma para desarrolladores de plataform
  
as es implementar la versión más alta de _netstandard_ posible.** Si la plataforma “.NET para XYZ” implementa _netstandard 1.4_ será mucho mejor que si implementa _netstandard 1.1_, ya que podrá ejecutar muchas más librerías. Por supuesto, eso implica que la plataforma debe implementar muchas más APIs de .NET. Evidentemente una plataforma puede ofrecer más APIs que las que le obliga la versión de _netstandard_ correspondiente. Y habitualmente será así. P. ej. Xamarin.iOS ofrecerá las APIs para iOS además de las de la versión de _netstandard_ que implemente. Eso implica que un desarrollador para Xamarin.iOS podrá usar todas esas APIs (por supuesto), pero alguien que cree una PCL solo podrá usar las APIs de Xamarin.iOS que existan en la versión de _netstandard_ que Xamarin.iOS soporte. La idea es muy simple: **desarrollamos para una plataforma, pero las librerías PCL que consumimos son para _netstandard_. Y todas las plataformas implementan _netstandard_ en alguna versión**.

**Definiciones actuales de _netstandard_**

Como he dicho antes **hay seis versiones ya definidas de _netstandard_ (1.0 – 1.5).** Cada una de ellas define un conjunto de APIs (y recuerda que son acumulativas, es decir _netstandard 1.1_ soporta todas las APIs de _netstandard 1.0_ además de las suyas).

David Fowler, ha creado un gist, <a href="https://gist.github.com/davidfowl/8939f305567e1755412d6dc0b8baf1b7" target="_blank" rel="noopener noreferrer">donde explica con código la relación entre netstandard, las APIs soportadas y las plataformas</a>. No entraré en estos detalles en este post, pero es interesante destacar un par de cosas:

  1. **La primera versión de .NET Framework que implementa _netstandard_ es .NET Framework 4.5**, que implementa _netstandard 1.1_. Por supuesto .NET Framework 4.5 es mucho más grande que _netstandard 1.1_ pero a nivel de PCLs significa que .NET Framework 4.5 podrá ejecutar una PCL compilada contra _netstandard 1.1_ o _1.0_&nbsp; pero no compilada contra _netstandard 1.2_. Esto significa que seguimos necesitando multi-targeting si queremos atacar a versiones del .NET Framework anteriores. 
      * **La última versión de .NET Framework actual (4.6.2) implementa la versión más alta definida de _netstandard_ que es 1.5**. La misma versión que implementan Xamarin.iOS, Xamarin.Android y .NET Core. En cambio UWP implementa _netstandard_ 1.4. Por lo tanto una librería _netstandard 1.4_ se podrá usar en todas esas plataformas, mientras que una librería _netstandard 1.5_ no se podrá usar en UWP.</ol> 
    En resúmen, _netstandard_ es el “estándar” que viene para permitir la creación de librerías que sean consumibles en varios sabores de .NET, incluso sabores que se creen a _posteriori_. Y es que, si algo es seguro, es que ¡iremos viendo cada vez más y más sabores de .NET para distintas plataformas!
    
    Saludos!