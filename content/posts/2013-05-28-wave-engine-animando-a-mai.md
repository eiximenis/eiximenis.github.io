---
title: '[Wave Engine] Animando a Mai'
author: eiximenis

date: 2013-05-28T23:01:40+00:00
geeks_url: /?p=1642
geeks_visits:
  - 1839
geeks_ms_views:
  - 773
categories:
  - Uncategorized

---
Buenas! Este post es el segundo sobre WaveEngine, esta maravilla que han parido los chicos de Plain Concepts 🙂

En el post anterior, vimos los fundamentos de Wave Engine y terminamos con un programa que mostraba a [Mai Shiranui][1] en la esquina inferior izquierda de la pantalla. Pero Mai Shiranui gana mucho cuando se mueve (¿por qué será?) así que vamos a ver como podemos hacerlo para que nuestra bella protagoniste se anime.

Aunque a tenor del post anterior pueda parecer que tan solo hemos mostrado una imagen por pantalla, desde el punto de vista de Wave hicimos mucho más: definimos una entidad y le asociamos entre otros el componente de sprite.

Hoy vamos a añadir un componente nuevo, el de animación. Y es que, obviamente, en Wave los sprites pueden estar animados.

**Sprites sheets**

En Wave una animación consta de varios “estados” (varias animaciones realmente) y cada estado consta de varias imágenes. Así podemos tener una animación (realmente un componente de tipo Animation2D) que defina los estados “Idle” (cuando el personaje no hace nada) y&#160; “walk” (cuando el personaje camina). Cada uno de esos estados consta de varios gráficos que conformarán cada de esos estados.

En este post la animación tendrá un solo estado (que llamaremos “idle”) y que constará de 12 imágnes (12 pngs que se llaman mai\_idle (0).png hasta mai\_idle (11).png).

Por supuesto podríamos usar la herramienta Wave Exporter para generar 12 .wpks (es decir uno por cada png tal y como hicimos en el post anterior) pero es mucho más eficiente usar un _sprite sheet_. Los que vengáis del mundo web probablemente conocereis este concepto ya que se usa mucho en CSS. La razón es que es mucho más eficiente bajarse una sola imagen grande y mostrar (por CSS) solo la parte que nos interesa que descargarse varias imágenes pequeñas. Pues en videojuegos ocurre lo mismo: es mucho mejor tener una sola textura (recordad que las imágenes son realmente texturas) grande y mostrar tan solo una parte que tener varias texturas pequeñas.

El primer paso consiste en generar una sola imagen (un png) que combine todas las imágenes que formaran parte de mi _sprite sheet_. Ni se te ocurra hacerlo a mano (cortando y pegando), ya que por suerte hay herramientas para ello. En los tutoriales de Wave [hay una explicación muy buena de como hacerlo paso a paso][2] utilizando [Texture Packer][3]. Para no repetirme yo haré un ejemplo utilizando otra herramienta, llamada [Shoebox][4]. La verdad es que Texture Packer es más completa (pero hay funcionalidades de pago), pero usar Shoebox me va a permitir contaros una cosilla más sobre Wave 🙂

Lo primero es generar el .png que contenga los 12 pngs, esto con Shoebox es muy sencillo. Arranco Shoebox, selecciono los 12 pngs desde el explorador de archivos y los arrastro sobre el icono “Sprite Sheet” de la ventana de Shoebox. Hecho esto me aparece una ventana con el sprite sheet:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; display: inline; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_33853C2E.png" width="215" height="244" />][5] 

Con el botón se settings puedo modificar varias propiedades, como p. ej. si la textura debe ser cuadrada, debe tener un tamaño que sea potencia de dos (eso es deseable en según que motores gráficos por temas de rendimiento) y también el formato del fichero XML que nos generará:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; display: inline; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_5B5815C0.png" width="187" height="244" />][6] 

Sí, he dicho formato XML, porque tener un png grande con todos los pngs pequeños incrustados no sirve de nada si no tenemos manera de saber donde empieza y termina cada “sub-imagen”. ShoeBox nos genera un fichero XML con toda esa información (Texture Packer también, por supuesto).

En mi caso el contenido del fichero XML es el siguiente (en el formato por defecto que genera Shoebox):

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: gray"><</span><span style="color: #569cd6">TextureAtlas</span><span style="color: gray"> </span><span style="color: #92caf4">imagePath</span><span style="color: gray">="</span><span style="color: #c8c8c8">sheet.png</span><span style="color: gray">"></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray">&#160;&#160;&#160; <</span><span style="color: #569cd6">SubTexture</span><span style="color: gray"> </span><span style="color: #92caf4">name</span><span style="color: gray">="</span><span style="color: #c8c8c8">mai_idle (0).png</span><span style="color: gray">" </span><span style="color: #92caf4">x</span><span style="color: gray">="</span><span style="color: #c8c8c8"></span><span style="color: gray">" </span><span style="color: #92caf4">y</span><span style="color: gray">="</span><span style="color: #c8c8c8">189</span><span style="color: gray">" </span><span style="color: #92caf4">width</span><span style="color: gray">="</span><span style="color: #c8c8c8">77</span><span style="color: gray">" </span><span style="color: #92caf4">height</span><span style="color: gray">="</span><span style="color: #c8c8c8">93</span><span style="color: gray">"/></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray">&#160;&#160;&#160; <</span><span style="color: #569cd6">SubTexture</span><span style="color: gray"> </span><span style="color: #92caf4">name</span><span style="color: gray">="</span><span style="color: #c8c8c8">mai_idle (1).png</span><span style="color: gray">" </span><span style="color: #92caf4">x</span><span style="color: gray">="</span><span style="color: #c8c8c8"></span><span style="color: gray">" </span><span style="color: #92caf4">y</span><span style="color: gray">="</span><span style="color: #c8c8c8"></span><span style="color: gray">" </span><span style="color: #92caf4">width</span><span style="color: gray">="</span><span style="color: #c8c8c8">77</span><span style="color: gray">" </span><span style="color: #92caf4">height</span><span style="color: gray">="</span><span style="color: #c8c8c8">92</span><span style="color: gray">"/></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray">&#160;&#160;&#160; <</span><span style="color: #569cd6">SubTexture</span><span style="color: gray"> </span><span style="color: #92caf4">name</span><span style="color: gray">="</span><span style="color: #c8c8c8">mai_idle (2).png</span><span style="color: gray">" </span><span style="color: #92caf4">x</span><span style="color: gray">="</span><span style="color: #c8c8c8"></span><span style="color: gray">" </span><span style="color: #92caf4">y</span><span style="color: gray">="</span><span style="color: #c8c8c8">94</span><span style="color: gray">" </span><span style="color: #92caf4">width</span><span style="color: gray">="</span><span style="color: #c8c8c8">77</span><span style="color: gray">" </span><span style="color: #92caf4">height</span><span style="color: gray">="</span><span style="color: #c8c8c8">93</span><sp an style="color: gray">"/></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray">&#160;&#160;&#160; <</span><span style="color: #569cd6">SubTexture</span><span style="color: gray"> </span><span style="color: #92caf4">name</span><span style="color: gray">="</span><span style="color: #c8c8c8">mai_idle (3).png</span><span style="color: gray">" </span><span style="color: #92caf4">x</span><span style="color: gray">="</span><span style="color: #c8c8c8">157</span><span style="color: gray">" </span><span style="color: #92caf4">y</span><span style="color: gray">="</span><span style="color: #c8c8c8">98</span><span style="color: gray">" </span><span style="color: #92caf4">width</span><span style="color: gray">="</span><span style="color: #c8c8c8">76</span><span style="color: gray">" </span><span style="color: #92caf4">height</span><span style="color: gray">="</span><span style="color: #c8c8c8">95</span><span style="color: gray">"/></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray">&#160;&#160;&#160; <</span><span style="color: #569cd6">SubTexture</span><span style="color: gray"> </span><span style="color: #92caf4">name</span><span style="color: gray">="</span><span style="color: #c8c8c8">mai_idle (4).png</span><span style="color: gray">" </span><span style="color: #92caf4">x</span><span style="color: gray">="</span><span style="color: #c8c8c8">157</span><span style="color: gray">" </span><span style="color: #92caf4">y</span><span style="color: gray">="</span><span style="color: #c8c8c8"></span><span style="color: gray">" </span><span style="color: #92caf4">width</span><span style="color: gray">="</span><span style="color: #c8c8c8">76</span><span style="color: gray">" </span><span style="color: #92caf4">height</span><span style="color: gray">="</span><span style="color: #c8c8c8">96</span><span style="color: gray">"/></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray">&#160;&#160;&#160; <</span><span style="color: #569cd6">SubTexture</span><span style="color: gray"> </span><span style="color: #92caf4">name</span><span style="color: gray">="</span><span style="color: #c8c8c8">mai_idle (5).png</span><span style="color: gray">" </span><span style="color: #92caf4">x</span><span style="color: gray">="</span><span style="color: #c8c8c8">235</span><span style="color: gray">" </span><span style="color: #92caf4">y</span><span style="color: gray">="</span><span style="color: #c8c8c8">192</span><span style="color: gray">" </span><span style="color: #92caf4">width</span><span style="color: gray">="</span><span style="color: #c8c8c8">76</span><span style="color: gray">" </span><span style="color: #92caf4">height</span><span style="color: gray">="</span><span style="color: #c8c8c8">95</span><span style="color: gray">"/></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray">&#160;&#160;&#160; <</span><span style="color: #569cd6">SubTexture</span><span style="color: gray"> </span><span style="color: #92caf4">name</span><span style="color: gray">="</span><span style="color: #c8c8c8">mai_idle (6).png</span><span style="color: gray">" </span><span style="color: #92caf4">x</span><span style="color: gray">="</span><span style="color: #c8c8c8">235</span><span style="color: gray">" </span><span style="color: #92caf4">y</span><span style="color: gray">="</span><span style="color: #c8c8c8"></span><span style="color: gray">" </span><span style="color: #92caf4">width</span><span style="color: gray">="</span><span style="color: #c8c8c8">76</span><span style="color: gray">" </span><span style="color: #92caf4">height</span><span style="color: gray">="</span><span style="color: #c8c8c8">93</span><span style="color: gray">"/></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray">&#160;&#160;&#160; <</span><span style="color: #569cd6">SubTexture</span><span style="color: gray"> </span><span style="color: #92caf4">name</span><span style="color: gray">="</span><span style="color: #c8c8c8">mai_idle (7).png</span><span style="color: gray">" </span><span style="color: #92caf4">x</span><span style="color: gray">="</span><span style="color: #c8c8c8">79</span><span style="color: gray">" </span><span style="color: #92caf4">y</span><span style="color: gray">="</span><span style="color: #c8c8c8"></span><span style="color: gray">" </span><span style="color: #92caf4">width</span><span style="color: gray">="</span><span style="color: #c8c8c8">76</span><span style="color: gray">" </span><span style="color: #92caf4">height</span><span style="color: gray">="</span><span style="color: #c8c8c8">92</span><span style="color: gray">"/></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray">&#160;&#160;&#160; <</span><span style="color: #569cd6">SubTexture</span><span style="color: gray"> </span><span style="color: #92caf4">name</span><span style="color: gray">="</span><span style="color: #c8c8c8">mai_idle (8).png</span><span style="color: gray">" </span><span style="color: #92caf4">x</span><span style="color: gray">="</span><span style="color: #c8c8c8">79</span><span style="color: gray">" </span><span style="color: #92caf4">y</span><span style="color: gray">="</span><span style="color: #c8c8c8">192</span><span style="color: gray">" </span><span style="color: #92caf4">width</span><span style="color: gray">="</span><span style="color: #c8c8c8">76</span><span style="color: gray">" </span><span style="color: #92caf4">height</span><span style="color: gray">="</span><span style="color: #c8c8c8">93</span><span style="color: gray">"/></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray">&#160;&#160;&#160; <</span><span style="color: #569cd6">SubTexture</span><span style="color: gray"> </span><span style="color: #92caf4">name</span><span style="color: gray">="</span><span style="color: #c8c8c8">mai_idle (9).png</span><span style="color: gray">" </span><span style="color: #92caf4">x</span><span style="color: gray">="</span><span style="color: #c8c8c8">157</span><span style="color: gray">" </span><span style="color: #92caf4">y</span><span style="color: gray">="</span><span style="color: #c8c8c8">195</span><span style="color: gray">" </span><span style="color: #92caf4">width</span><span style="color: gray">="</span><span style="color: #c8c8c8">76</span><span style="color: gray">" </span><span style="color: #92caf4">height</span><span style="color: gray">="</span><span style="color: #c8c8c8">95</span><span style="color: gray">"/></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray">&#160;&#160;&#160; <</span><span style="color: #569cd6">SubTexture</span><span style="color: gray"> </span><span style="color: #92caf4">name</span><span style="color: gray">="</span><span style="color: #c8c8c8">mai_idle (10).png</span><span style="color: gray">" </span><span style="color: #92caf4">x</span><span style="color: gray">="</span><span style="color: #c8c8c8">79</span><span style="color: gray">" </span><span style="color: #92caf4">y</span><span style="color: gray">="</span><span style="color: #c8c8c8">94</span><span style="color: gray">" </span><span style="color: #92caf4">width</span><span style="color: gray">="</span><span style="color: #c8c8c8">76</span><span style="color: gray">" </span><span style="color: #92caf4">height</span><span style="color: gray">="</span><span style="color: #c8c8c8">96</span><span style="color: gray">"/></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray">&#160;&#160;&#160; <</span><span style="color: #569cd6">SubTexture</span><span style="color: gray"> </span><span style="color: #92caf4">name</span><span style="color: gray">="</span><span style="color: #c8c8c8">mai_idle (11).png</span><span style="color: gray">" </span><span style="color: #92caf4"
>x</span><span style="color: gray">="</span><span style="color: #c8c8c8">235</span><span style="color: gray">" </span><span style="color: #92caf4">y</span><span style="color: gray">="</span><span style="color: #c8c8c8">95</span><span style="color: gray">" </span><span style="color: #92caf4">width</span><span style="color: gray">="</span><span style="color: #c8c8c8">76</span><span style="color: gray">" </span><span style="color: #92caf4">height</span><span style="color: gray">="</span><span style="color: #c8c8c8">95</span><span style="color: gray">"/></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray"></</span><span style="color: #569cd6">TextureAtlas</span><span style="color: gray">></span>
  </p></p>
</div>

Básicamente se incluye el nombre del png original y su posición y tamaño dentro del png “global”.

Ahora usamos Assets Exporter para convertir el png grande a un asset en formato .wpk e incluímos dicho asset y el .xml en el proyecto de VS2012:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; display: inline; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_4DA559B8.png" width="198" height="175" />][7] 

Y con eso ya tendremos suficiente. Los 12 pngs originales no los necesitamos para nada.

**Animaciones 2D en WaveEngine**

Ahora el siguiente paso es modificar la definición de entidad para añadirle un componente nuevo de tipo Animation2D. Este componente, a pesar de su nombre, puede contener un conjunto de animaciones. De momento nosotros tendremos tan solo una (Mai en estado _idle_), pero podríamos tener varias animaciones distintas (andar, saltar, etc) usando un solo _sprite sheet_.

El código de definición de la entidad es como sigue:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">var</span> <span style="color: white">mai</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">Entity</span>(<span style="color: #d69d85">"Mai"</span>)<span style="color: #b4b4b4">.</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: white">AddComponent</span>(<span style="color: #569cd6">new</span> <span style="color: #4ec9b0">Transform2D</span>()
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">X</span> <span style="color: #b4b4b4">=</span> <span style="color: #b5cea8">50</span>,
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">Y</span> <span style="color: #b4b4b4">=</span> <span style="color: #4ec9b0">WaveServices</span><span style="color: #b4b4b4">.</span><span style="color: white">Platform</span><span style="color: #b4b4b4">.</span><span style="color: white">ScreenHeight</span> <span style="color: #b4b4b4">&#8211;</span> <span style="color: #b5cea8">46</span>,
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">Origin</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">Vector2</span>(<span style="color: #b5cea8">0.5f</span>, <span style="color: #b5cea8">1</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; })<span style="color: #b4b4b4">.</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: white">AddComponent</span>(<span style="color: #569cd6">new</span> <span style="color: #4ec9b0">Sprite</span>(<span style="color: #d69d85">"Content/mai.wpk"</span>))<span style="color: #b4b4b4">.</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: white">AddComponent</span>(<span style="color: #4ec9b0">Animation2D</span><span style="color: #b4b4b4">.</span><span style="color: white">Create</span><span style="color: #b4b4b4"><</span><span style="color: #4ec9b0">ShoeBoxXmlSpriteSheetLoader</span><span style="color: #b4b4b4">></span>(<span style="color: #d69d85">"Content/mai.xml"</span>)<span style="color: #b4b4b4">.</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">Add</span>(<span style="color: #d69d85">"Idle"</span>, <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">SpriteSheetAnimationSequence</span>() {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">First</span> <span style="color: #b4b4b4">=</span> <span style="color: #b5cea8">1</span>,
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">Length</span> <span style="color: #b4b4b4">=</span> <span style="color: #b5cea8">12</span>,
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">FramesPerSecond</span> <span style="color: #b4b4b4">=</span> <span style="color: #b5cea8">9</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; }))<span style="color: #b4b4b4">.</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: white">AddComponent</span>(<span style="color: #569cd6">new</span> <span style="color: #4ec9b0">AnimatedSpriteRenderer</span>(<span style="color: #4ec9b0">DefaultLayers</span><span style="color: #b4b4b4">.</span><span style="color: white">Alpha</span>));
  </p></p>
</div>

Las dos diferencias respecto el código del post anterior (además del uso de la fluent interface) son:

  1. Que se añade el componente Animation2D 
  2. El componente SpriteRenderer es modificado por el AnimatedSpriteRenderer 

Centrémonos en el primer punto: llamamos al método estático Animation2D.Create para crear una animación. A dicho método le tenemos que pasar un parámetro genérico que es una clase que va a ser la encargada de indicar donde, dentro del asset gráfico, está cada frame de la animación. En mi caso uso la clase ShoeBoxXmlSpriteSheetLoader. Esta clase me la he creado yo y lo que básicamente hace es leer el fichero XML generado por Shoebox y devolver un array de objetos Rectangle que indican la posición y tamaño de cada frame de la animación. El código es como sigue:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">ShoeBoxXmlSpriteSheetLoader</span> : <span style="color: #b8d7a3">ISpriteSheetLoader</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #4ec9b0">Rectangle</span>[] <span style="color: white">Parse</span>(<span style="color: #569cd6">string</span> <span style="color: white">path</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">doc</span> <span style="color: #b4b4b4">=</span> <span style="color: #4ec9b0">XDocument</span><span style="color: #b4b4b4">.</span><span style="color: white">Load</span>(<span style="color: white">path</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">descs</span> <span style="color: #b4b4b4">=</span> <span style="color: white">doc</span><span style="color: #b4b4b4">.</span><span style="color: white">Descendants</span>(<span style="color: #4ec9b0">XName</span><span style="color: #b4b4b4">.</span><span style="color: white">Get</span>(<span style="color: #d69d85">"SubTex<br /> ture"</span>))<span style="color: #b4b4b4">.</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">Select</span>(<span style="color: white">node</span> <span style="color: #b4b4b4">=></span> <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">Rectangle</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">X</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">int</span><span style="color: #b4b4b4">.</span><span style="color: white">Parse</span>(<span style="color: white">node</span><span style="color: #b4b4b4">.</span><span style="color: white">Attribute</span>(<span style="color: #d69d85">"x"</span>)<span style="color: #b4b4b4">.</span><span style="color: white">Value</span>),
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">Y</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">int</span><span style="color: #b4b4b4">.</span><span style="color: white">Parse</span>(<span style="color: white">node</span><span style="color: #b4b4b4">.</span><span style="color: white">Attribute</span>(<span style="color: #d69d85">"y"</span>)<span style="color: #b4b4b4">.</span><span style="color: white">Value</span>),
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">Width</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">int</span><span style="color: #b4b4b4">.</span><span style="color: white">Parse</span>(<span style="color: white">node</span><span style="color: #b4b4b4">.</span><span style="color: white">Attribute</span>(<span style="color: #d69d85">"width"</span>)<span style="color: #b4b4b4">.</span><span style="color: white">Value</span>),
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">Height</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">int</span><span style="color: #b4b4b4">.</span><span style="color: white">Parse</span>(<span style="color: white">node</span><span style="color: #b4b4b4">.</span><span style="color: white">Attribute</span>(<span style="color: #d69d85">"height"</span>)<span style="color: #b4b4b4">.</span><span style="color: white">Value</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; })<span style="color: #b4b4b4">.</span><span style="color: white">ToArray</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: white">descs</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Una vez tenemos el Animation2D creado debemos añadirle todas las animaciones que realmente contiene (recuerda que puede contener varias). En este caso tan solo contiene una, y por ello tenemos tan solo una llamada al método Add. A cada animación le damos un nombre, y luego un objeto SpriteSheetAnimationSecuence que determina cual es el frame inicial (dentro del Animation2D), cual es el final y cuantos frames deben renderizarse por segundo. En mi caso le coloco 9, y así toda la animación (que consta de 12 frames) tardará 1,3 segundos en realizarse lo que me pareció un valor razonable.

Finalmente el resto del código de la escena es parecido al del post anterior:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: white">EntityManager</span><span style="color: #b4b4b4">.</span><span style="color: white">Add</span>(<span style="color: white">mai</span>);
  </p>
  
  <p style="margin: 0px">
    <span style="color: white">mai</span><span style="color: #b4b4b4">.</span><span style="color: white">FindComponentOfType</span><span style="color: #b4b4b4"><</span><span style="color: #4ec9b0">Animation2D</span><span style="color: #b4b4b4">></span>()<span style="color: #b4b4b4">.</span><span style="color: white">Play</span>(<span style="color: #569cd6">true</span>);
  </p></p>
</div>

Añadimos la entidad a la escena y empezamos a reproducir la animación. El método Play reproduce la animación actual dentro del componente Animation2D (en nuestro caso solo hay una, si hay más de una se puede usar la propiedad CurrentAnimation del propio componente). El parámetro booleano que vale true es para indicar que la animación se vaya repitiendo en un bucle.

¡Y listos! Con esto ya tenemos a Mai en todo su esplendor: [http://screencast.com/t/eUwEVcSVn6xg][8]

_Let’s fight!_

Espero que os haya resultado interesante…

Un saludo!!!

 [1]: http://es.wikipedia.org/wiki/Mai_Shiranui
 [2]: http://blog.waveengine.net/2013/04/12/platform-game-sample/
 [3]: http://www.codeandweb.com/texturepacker
 [4]: http://renderhjs.net/shoebox/
 [5]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_5C50C744.png
 [6]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_3525E235.png
 [7]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_398C3D2F.png
 [8]: http://screencast.com/t/eUwEVcSVn6xg "http://screencast.com/t/eUwEVcSVn6xg"