---
title: '[WaveEngine] Primeros pasos…'
description: '[WaveEngine] Primeros pasos…'
author: eiximenis

date: 2013-05-21T15:20:00+00:00
geeks_url: /?p=1639
geeks_visits:
  - 1805
geeks_ms_views:
  - 1256
categories:
  - Uncategorized

---
Buenas! Hace algunos días, no muchos, que me estoy _pegando_ (en el buen sentido de la palabra) con <a target="_blank" href="http://waveengine.net/" rel="noopener noreferrer">WaveEngine</a>, esta maravilla que han creado los chicos de Plain Concepts.

> **Disclaimer:** Este post (y todos los que puedan venir) no pretenden sustituir la documentación oficial. No me considero un experto en Wave ni de lejos, realmente soy un aprendiz de nivel 1 🙂 Simplemente voy a expresar mis experiencias y lo iré haciendo a medida que las vaya teniendo, así que bueno... puede haber inexactitudes, errores, omisiones, etc... en estos posts. Así que comentarios son más que bienvenidos.

**Introducción &ndash; ¿Qué es Wave Engine?**

Bueno, pues básicamente WaveEngine es un motor multiplataforma de videojuegos. No es el único hay una larga lista de ellos (algunos más multiplataforma que otros) como <a target="_blank" href="http://cocos2d.org/" rel="noopener noreferrer">cocos2d</a> (y sus derivados tales como <a target="_blank" href="http://www.cocos2d-x.org/" rel="noopener noreferrer">cocos2dx</a>), <a target="_blank" href="http://deltaengine.net/" rel="noopener noreferrer">Delta Engine</a> o el todopoderoso <a target="_blank" href="http://unity3d.com/" rel="noopener noreferrer">Unity3D</a>. Todos ellos nacen con filosofías distintas, lo que termina redundando en características, y precios, distintos.

Wave Engine es totalmente gratuito: la descarga es gratuita y no hay que pagar licencia de ningún tipo ni royalty por juego publicado ni nada parecido. El único detalle a tener en cuenta es que Wave Engine permite desarrollar para iOS y Anrdoid a través de Monotouch y Monodroid (de <a target="_blank" href="http://xamarin.com/" rel="noopener noreferrer">Xamarin</a>) y esos productos no son libres. Aquí pues hay <a target="_blank" href="https://store.xamarin.com/" rel="noopener noreferrer">un coste, que es la licencia de Monotouch y Monodroid</a>. Por supuesto esto os aplica tan solo si quereis desplegar en Android o iOS.

Como todo motor de videojuegos, Wave nos ofrece una API de relativo alto nivel para evitar tener que lidiar directamente con DirectX (o OpenGL), además de integrar muchas otras facetas: animaciones, motores de física, etc. En definitiva, un ahorro de tiempo considerable.

**Panorama actual del desarrollo de videojuegos en Windows 8**

En Windows 7 y anteriores la situación del desarrollo de videojuegos era relativamente sencilla. Básicamente, motores de terceros aparte, había dos opciones básicas:

  1. Usar C++ y DirectX directamente. La opción más potente y la menos productiva ya que DirectX es una API de bajo nivel. 
  2. Usar .NET (C#) y XNA. Una opción que ha sido muy usada por desarrolladores indie y pequeños estudios ya que XNA es una API de medio nivel, que evita que uno tenga que pegarse con DirectX directamente. 

Con Windows 8 y la aparición de las nuevas aplicaciones para la Windows Store, el panorama ha cambiado. XNA no permite realizar aplicaciones para la Windows Store y además MS lo ha discontinuado. No habrá una futura versión de XNA.

El panorama oficial para desarrollar videojuegos para la Windows Store ahora es:

  1. Usar XAML y C#. No es óptimo ni de lejos, ya que no se usa toda la potencia gráfica del ordenador. 
  2. Usar C++ y DirectX... Lo que después de venir usando XNA es un paso atrás en productividad descomunal. 

Por suerte, la comunidad no se está quieta, y así ha surgido el proyecto <a target="_blank" href="http://sharpdx.org/" rel="noopener noreferrer">SharpDX</a>. SharpDX es un wrapper en .NET para DirectX. Usándolo podemos desarrollar videojuegos en C# y DirectX. Aunque es una mejora no te creas que es la panacea: DirectX es de bajo nivel por lo que SharpDX también lo es. Otra alternativa interesante es <a target="_blank" href="http://monogame.codeplex.com/" rel="noopener noreferrer">MonoGame</a> que es un port de XNA. Como su nombre indica usa Mono (Monotouch y Monodroid) para permitir desarrollar videojuegos para iOS y Android y usa por debajo SharpDX para permitir hacer lo mismo para aplicaciones Windows Store.

Y finalmente un escalón por encima están los motores de videojuegos, como Wave. Por supuesto Wave por debajo usa SharpDX pero nosotros quedamos completamente al margen.

**Estructura de un proyecto de Wave**

Cuando instalamos Wave Engine nos aparecen nuevas plantillas de proyecto en VS2012:

[<img height="140" width="504" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_669D7F12.png" alt="image" border="0" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" title="image" />][1]

Si seleccionamos la opción de &ldquo;Game Project&rdquo; VS2012 nos añadirá **dos** proyectos a nuestra solución. Uno con el nombre que hayamos elegido y otro con el añadido &ldquo;Project&rdquo; al final. No sé todavía porque se crean esos dos proyectos pero realmente el primero es el ejecutable y es una lanzadora del segundo. Supongo que esto es porque el primero es específico por cada plataforma mientras que el segundo (que tiene realmente todo el código) es el mismo por todas las plataformas. Sospecho que los tiros van por ahí.

A partir de ahí Wave usa conceptos muy simples de entender:

  1. Escena: Es toda la información de nuestro juego en un momento dado. P. ej. un videojuego que tuviese varios niveles&nbsp; podría tener varias escenas. Otra opción sería tener una escena para el menú principal y otra para el juego en sí. En un momento dado se está _ejecutando_ (por decirlo de algún modo) una escena. 
  2. Componente: Es la unidad de modularización de Wave. Los componentes son como &ldquo;piezas&rdquo; que se añaden a las entidades. P.ej. para posicionar algo en pantalla (si estamos haciendo un juego 2D) vamos a necesitar un componente llamado Transform2D. Todas las entidades que tengan una posición 2D tendrán una instancia de este componente. 
  3. Entidad: Cada uno de los elementos de los que se compone tu juego. El héroe, la princesa o los nubarrones del fondo son entidades. 
  4. Comportamientos (Behaviors): Son componentes que permiten que una entidad tenga lógica, es decir se comporte de una manera u otra. Que hace que la princesa sea una princesa indefensa y el dragón un dragón que escupa fuego? Pues sus comportamientos. 

Por defecto la plantilla de proyecto de Wave nos crea la clase que representa el juego y una escena vacía. Nuestra misión es crear entidades (con sus componentes y comportamientos) y añadirlas a la escena. Y con esto tendremos un juego 🙂

**Hello World con Wave Engine**

Venga, empecemos por lo básico de lo básico. Vamos a crear un pequeño programa en 2D que simplemente muestre un sprite. Luego más adelante veremos como animarlo y darle un poco de vida 😉

Lo primero que debemos hacer es crear un nuevo proyecto de tipo WaveEngine Game Project. En mi caso he llamado &ldquo;Mai&rdquo; al proyecto.

Con esto VS2012 me va a crear los proyectos &ldquo;Mai&rdquo; y &ldquo;MaiProject&rdquo;. Como he dicho antes el segundo es el que contendrá &ldquo;toda la chicha&rdquo; 🙂

En MaiProject se me habrán creado los ficheros Game.cs que contiene la clase que pone en marcha el juego y MyScene.cs, la única escena que (de momento) tiene nuestro juego.

Ahora, lo único que vamos a hacer es mostrar un gráfico en pantalla. Para ello debemos introducir otro concepto de Wave: los assets.

Un asset no es nada más que un elemento que proviene de un fichero externo y que forma parte de nuestro juego. P. ej. si quiero mostrar un fichero .png este .png será un asset. Pero lo mismo ocurrirá si tengo un modelo 3D exportado en formato .x p.ej. WaveEngine **no** entiende de formatos gráficos o de formatos 3D o de cualquier otro formato externo. Wave entiende tan solo de un formato de asset genérico, el .wpk. Por lo tanto NO podemos usar directamente un .png, si no que debemos convertirlo antes a este formato .wpk.

Para ello debemos usar la herramienta (que se instala junto con Wave) llamada Assets Exporter. Si la ponemos en marcha veremos una interfaz muy, muy negra:

[<img height="184" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_5956F5FF.png" alt="image" border="0" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-top-width: 0px" title="image" />][2]

En mi caso tengo un fichero .png, llamado mai_idle (0).png y quiero convertirlo a un .wpk para poder usarlo desde Wave. Para ello, debo crear un proyecto nuevo de assets exporter. Así que le doy a File->New Project y selecciono una carpeta. Ello me crea un fichero .wproj y una estructura de carpetas dentro de la carpeta seleccionada. Una de esas carpetas es llamada Assets y contendrá los ficheros de origen (en mi caso el .png). Para añadir assets al proyecto basta con pulsar el botón de &ldquo;+&rdquo; (el primero por la izquierda) y seleccionar el fichero. Al hacerlo el fichero es copiado automáticamente a la carpeta Assets.

Una vez tenemos todos los Assets podemos darle a exportar (Project &ndash;> Export) y en la carpeta &ldquo;Exports&rdquo; dentro de la carpeta que hemos elegido al crear el proyecto del assets exporter tendremos el fichero .wpk.

Ahora debemos copiar este fichero a la carpeta &ldquo;Content&rdquo; del proyecto de VS2102 y establecer en las propiedades del fichero &ldquo;Copy to output folder&rdquo; a &ldquo;Always&rdquo;:

[<img height="244" width="215" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_378B1D6E.png" alt="image" border="0" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-top-width: 0px" title="image" />][3]

Bien! Esta es la forma habitual de proceder con los assets 🙂

A partir de ahora ya tan solo nos queda codificar. En nuestro caso vamos a mostrar tan solo una imagen. Para ello vamos a crear una entidad (todo son entidades en Wave) que va a tener varios componentes. Vamos a construirlo paso&nbsp; a paso. Todo el código va en el método CreateScene de MyScene. Empezamos por crear la entidad:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">var</span> <span style="color: white">mai</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">Entity</span>(<span style="color: #d69d85">&#8220;Mai&#8221;</span>);
  </p>
</div>

Y ahora vamos a irle añadiendo componentes. Empezaremos por una posición. Haremos que mai aparezca en la esquina inferior izquierda de la pantalla. En Wave una posición es un componente de tipo Transform2D (estamos en un videojuego 2D, los 3D son otro mundo):

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: white">mai</span><span style="color: #b4b4b4">.</span><span style="color: white">AddComponent</span>(<span style="color: #569cd6">new</span> <span style="color: #4ec9b0">Transform2D</span>()
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: white">X</span> <span style="color: #b4b4b4">=</span> <span style="color: #b5cea8;">50</span>,
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: white">Y</span> <span style="color: #b4b4b4">=</span> <span style="color: #4ec9b0">WaveServices</span><span style="color: #b4b4b4">.</span><span style="color: white">Platform</span><span style="color: #b4b4b4">.</span><span style="color: white">ScreenHeight</span> <span style="color: #b4b4b4">&#8211;</span> <span style="color: #b5cea8">46</span>,
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: white">Origin</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">Vector2</span>(<span style="color: #b5cea8">0.5f</span>, <span style="color: #b5cea8">1</span>)
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; });
  </p>
</div>

El siguiente paso es tener un asset gráfico. De hecho Wave, como buen motor, nos da el concepto de sprite, es decir un conjunto de gráficos:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: white">mai</span><span style="color: #b4b4b4">.</span><span style="color: white">AddComponent</span>(<span style="color: #569cd6">new</span> <span style="color: #4ec9b0">Sprite</span>(<span style="color: #d69d85">&#8220;Content/mai_idle (0).wpk&#8221;</span>));
  </p>
</div>

Al constructor de Sprite se le pasa el nombre del fichero .wpk que contiene el gráfico (técnicamente la textura). Un tema a destacar, que ya veremos en otro post, es que un fichero .wpk puede contener más de un gráfico de nuestro sprite.

Finalmente tan solo nos queda añadir el renderizador, es decir el componente que se encarga de &ldquo;dibujar&rdquo; en pantalla. Te puede parecer extraño que los renderizadores sean componentes, pero esto permite que una misma entidad se dibuje (se renderice) en pantalla de formas distintas. &iexcl;Modularidad ante todo!

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: white">mai</span><span style="color: #b4b4b4">.</span><span style="color: white">AddComponent</span>(<span style="color: #569cd6">new</span> <span style="color: #4ec9b0">SpriteRenderer</span>(<span style="color: #4ec9b0">DefaultLayers</span><span style="color: #b4b4b4">.</span><span style="color: white">Alpha</span>));
  </p>
</div>

> **Nota:** No la he usado en este ejemplo, pero Wave tiene una API fluent, de forma que en lugar de ir haciendo mai.XXX cada vez, podeis encadenar las llamadas a AddComponent una tras de otra.

Finalmente debemos agregar esta entidad que hemos creado a la escena:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: white">EntityManager</span><span style="color: #b4b4b4">.</span><span style="color: white">Add</span>(<span style="color: white">mai</span>);
  </p>
</div>

&iexcl;Y listos! Hemos terminado, ya podemos darle a F5 para ver nuestra obra de arte:

[<img height="192" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_155311E8.png" alt="image" border="0" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; border-left: 0px; display: inline; padding-right: 0px" title="image" />][4]

En el siguiente post veremos como darle un poco de movimiento... que si alguien se merce ser vista en pleno movimiento es Mai Shiranui 😛 😛

 [1]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_5D24C09C.png
 [2]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_6C7822AB.png
 [3]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_11956D18.png
 [4]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_5A6BDF1E.png