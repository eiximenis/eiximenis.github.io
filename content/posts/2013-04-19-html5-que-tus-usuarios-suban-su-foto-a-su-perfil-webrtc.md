---
title: HTML5 – Que tus usuarios suban su foto a su perfil (WebRTC)
description: HTML5 – Que tus usuarios suban su foto a su perfil (WebRTC)
author: eiximenis

date: 2013-04-19T10:31:41+00:00
geeks_url: /?p=1636
geeks_visits:
  - 4565
geeks_ms_views:
  - 2390
categories:
  - Uncategorized

---
Venga, seguro que como la mitad de mortales tienes una idea de negocio que consiste en hacer una web y que te la compre Google (la otra mitad esperan que la compre Microsoft :p).

Si este es el caso, ya sabes que se trata de tener cuantos más usuarios mejor (ahí tienes el caso de Mammoth que han lanzado una campaña viral para que todos nos apuntemos allí aunque no tengamos ni idea de que va). Hoy en día cualquier web que se precie tiene un perfil donde el usuario puede subir una foto suya para que sea su avatar. Imagina la situación de que un usuario se registra a tu web y no tiene a mano ninguna foto suya para subir. ¿No estaría bien que se pudiese hacer una foto con la webcam del portátil y subirla directamente? Todo ello des de tu web, por supuesto.

Pues bien, eso es ni más ni menos lo que permite WebRTC. 😉

WebRTC significa Web Real Time Communications y es una de las futuras APIs de HTML5 que más darán que hablar. Hablo en futuro porque actualmente no son un estándard terminado: la <a href="http://www.w3.org/TR/webrtc/" target="_blank" rel="noopener noreferrer">especificación actual</a> es todavía un Working Draft. Eso significa que su soporte en navegadores es todavía muy escaso y en la mayoría de casos experimental. El código de este post ha sido probado en Chrome 26 y funciona. No he probado en otros navegadores, pero por lo que sé IE10 no soporta todavía WebRTC y por lo que he leído FF lo soporta a partir de su versión 20. Opera también parece que lo soporta pero no sé a partir de que versión. Al final no tengais ninguna duda de que IE terminará soportando WebRTC, pero como digo: no es un estándard terminado y su definición puede cambiar.

**Manos a la obra**

La implementación de WebRTC se basa básicamente en una función javascript llamada getUserMedia que está en el objeto navigator. De todos modos como ya digo el soporte puede ser experimental y así p.ej. en Chrome está función está prefijada y debe usarse webkitGetUserMedia. Cosas del desarrollo para web.

> **Nota:** Personalmente lo de los _vendor prefixes_ me parece una aberración. No sé, si una característica se soporta, se soporta y punto. No veo porque tenemos que andar prefijando cosas porque “están a medias” y tal. Al final el problema para el desarrollador es el mismo: debes acordarte de meter todos los prefijos que toquen. En CSS aún puede tener un pase pero… ¿en javascript? ¿De veras tenemos que prefijar una función javascript? A mi me parece que algo se nos está yendo de las manos, pero bueno los prefijos están ampliamente aceptados por el W3C así que supongo que será mejor tenerlos que no tenerlos.

La función getUserMedia permite obtener un stream de datos local. Resumiendo, con getUserMedia podemos obtener la imagen de la webcam o el sonido del microfono. Básicamente le pasamos dos o tres parámetros:

  1. Los streams locales que queremos capturar (p.ej audio y/o video). 
  2. La función de callback a invocar cuando la captura haya empezado 
  3. La función de callback en caso de error (opcional). 

Así para capturar la webcam bastaría con:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: white">navigator</span><span style="color: #b4b4b4">.</span><span style="color: white">getUserMedia</span><span style="color: #b4b4b4">({</span><span style="color: white">video</span><span style="color: #b4b4b4">:</span> <span style="color: #569cd6">true</span><span style="color: #b4b4b4">},</span> <span style="color: white">onSucessCallback, onFailCallback</span><span style="color: #b4b4b4">);</span>
  </p></p>
</div>

Para no andar jugando con los prefijos es preferible hacer algo como:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: white">navigator</span><span style="color: #b4b4b4">.</span><span style="color: white">getUserMedia</span> <span style="color: #b4b4b4">=</span> <span style="color: white">navigator</span><span style="color: #b4b4b4">.</span><span style="color: white">getUserMedia</span> <span style="color: #b4b4b4">||</span> <span style="color: white">navigator</span><span style="color: #b4b4b4">.</span><span style="color: white">webkitGetUserMedia</span> <span style="color: #b4b4b4">|| </span><span style="color: white">navigator</span><span style="color: #b4b4b4">.</span><span style="color: white">mozGetUserMedia</span> <span style="color: #b4b4b4">||</span> <span style="color: white">navigator</span><span style="color: #b4b4b4">.</span><span style="color: white">msGetUserMedia</span><span style="color: #b4b4b4">;</span>
  </p></p>
</div>

Así a medida que los navegadores vayan incorporando la función (con o sin prefijo) pues ya la tendremos disponible.

Vale… capturamos un stream de video, pero lo suyo es poderlo mostrar ¿no? En HTML5 tenemos una etiqueta que nos permite mostrar vídeos (<video />). ¿No sería genial que la pudiesemos utilizar? Pues, por suerte, podemos. Para ello nos tenemos que apoyar en _otra_ API de HTML5: window.URL

Con los métodos de window.URL podemos crear URLs “ficticias” que apunten a “objetos” (técnicamente Blobs) que están vivos dentro del documento. La idea viene a ser la siguiente: con getUserMedia capturamos un stream de video. Para mostrar videos tenemos la etiqueta <video />. Pero la etiqueta <video /> espera la URL del vídeo. Pues con window.URL vamos a poder crear esta URL.

El código es muy simple:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: gray"><!</span><span style="color: #569cd6">DOCTYPE</span> <span style="color: #9cdcfe">html</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray"><</span><span style="color: #569cd6">html</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray"><</span><span style="color: #569cd6">head</span><span style="color: gray">></</span><span style="color: #569cd6">head</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray"><</span><span style="color: #569cd6">body</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">video</span> <span style="color: #9cdcfe">autoplay</span><span style="color: gray">></<span style="color: #569cd6">video</span>></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">script</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">onErrorCallback</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">function</span> <span style="color: #b4b4b4">(</span><span style="color: white">e</span><span style="color: #b4b4b4">)</span> <span style="color: #b4b4b4">{</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">console</span><span style="color: #b4b4b4">.</span><span style="color: white">log</span><span style="color: #b4b4b4">(</span><span style="color: #d69d85">&#8216;Error!&#8217;</span><span style="color: #b4b4b4">,</span> <span style="color: white">e</span><span style="color: #b4b4b4">);</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #b4b4b4">};</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">navigator</span><span style="color: #b4b4b4">.</span><span style="color: white">getUserMedia</span> <span style="color: #b4b4b4">=</span> <span style="color: white">navigator</span><span
 style="color: #b4b4b4">.</span><span style="color: white">getUserMedia</span> <span style="color: #b4b4b4">||</span> <span style="color: white">navigator</span><span style="color: #b4b4b4">.</span><span style="color: white">webkitGetUserMedia</span> <span style="color: #b4b4b4">|| </span><span style="color: white">navigator</span><span style="color: #b4b4b4">.</span><span style="color: white">mozGetUserMedia</span> <span style="color: #b4b4b4">||</span> <span style="color: white">navigator</span><span style="color: #b4b4b4">.</span><span style="color: white">msGetUserMedia</span><span style="color: #b4b4b4">;</span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: #b4b4b4"></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">navigator</span><span style="color: #b4b4b4">.</span><span style="color: white">getUserMedia</span><span style="color: #b4b4b4">({</span> <span style="color: white">video</span><span style="color: #b4b4b4">:</span> <span style="color: #569cd6">true</span><span style="color: #b4b4b4">},</span> <span style="color: #569cd6">function</span> <span style="color: #b4b4b4">(</span><span style="color: white">localMediaStream</span><span style="color: #b4b4b4">)</span> <span style="color: #b4b4b4">{</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">video</span> <span style="color: #b4b4b4">=</span> <span style="color: white">document</span><span style="color: #b4b4b4">.</span><span style="color: white">querySelector</span><span style="color: #b4b4b4">(</span><span style="color: #d69d85">&#8216;video&#8217;</span><span style="color: #b4b4b4">);</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">video</span><span style="color: #b4b4b4">.</span><span style="color: white">src</span> <span style="color: #b4b4b4">=</span> <span style="color: white">window</span><span style="color: #b4b4b4">.</span><span style="color: white">URL</span><span style="color: #b4b4b4">.</span><span style="color: white">createObjectURL</span><span style="color: #b4b4b4">(</span><span style="color: white">localMediaStream</span><span style="color: #b4b4b4">);</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #b4b4b4">},</span> <span style="color: white">onErrorCallback</span><span style="color: #b4b4b4">);</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"></</span><span style="color: #569cd6">script</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray"></</span><span style="color: #569cd6">body</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray"></</span><span style="color: #569cd6">html</span><span style="color: gray">></span>
  </p></p>
</div>

Bueno… Si ejecutais este código veréis algo como:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_17DE55B7.png" width="244" height="206" />][1]

Por supuesto no vereis a este tipo en pantalla, seguramente el que aparezca sea más feo, pero bueno eso son cosas que pasan 😛 😛 😛 😛

La función getUserMedia pide permisos. Es decir el usuario debe confirmar que da acceso a la webcam en este caso:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_7DF9F9BA.png" width="504" height="94" />][2]

Vale… ya estamos mostrando el vídeo de la webcam. Pero el objetivo era que el usuario pudiese subir una foto de su perfil a nuestra web, ¿recordáis?

Bueno, para ello acude en nuestra ayuda otro de los nuevos elementos de HTML5: el <canvas />. Como ya sabréis la mayoría el canvas de HTML5 es un espacio dentro del documento para dibujar gráficos en 2D o en 3D.

Pues bien, la idea es volcar el frame actual del video al canvas. Y por suerte nos basta con llamar al método drawImage del contexto 2D del canvas. Sí, tan simple como esto.

Veamos el código:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: gray"><!</span><span style="color: #569cd6">DOCTYPE</span> <span style="color: #9cdcfe">html</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray"><</span><span style="color: #569cd6">html</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray"><</span><span style="color: #569cd6">head</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">style</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #d7ba7d">video</span> {<span style="color: #9cdcfe">width</span>: <span style="color: #c8c8c8">300px</span>; <span style="color: #9cdcfe">height</span>: <span style="color: #c8c8c8">300px</span>;}
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"></</span><span style="color: #569cd6">style</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray"></</span><span style="color: #569cd6">head</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray"><</span><span style="color: #569cd6">body</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">video</span> <span style="color: #9cdcfe">autoplay</span><span style="color: gray">></</span><span style="color: #569cd6">video</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">input</span> <span style="color: #9cdcfe">type</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"button"</span> <span style="color: #9cdcfe">value</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"snaphsot!"</span> <span style="color: #9cdcfe">id</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"snap"</span> <span style="color: gray">/></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">canvas</span><span style="color: gray">></</span><span style="color: #569cd6">canvas</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">script</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">navigator</span><span style="color: #b4b4b4">.</span><span style="color: white">getUserMedia</span> <span style="color: #b4b4b4">=</span> <span style="color: white">navigator</span><span style="color: #b4b4b4">.</span><span style="color: white">getUser<br /> Media</span> <span style="color: #b4b4b4">||</span> <span style="color: white">navigator</span><span style="color: #b4b4b4">.</span><span style="color: white">webkitGetUserMedia</span> <span style="color: #b4b4b4">|| </span><span style="color: white">navigator</span><span style="color: #b4b4b4">.</span><span style="color: white">mozGetUserMedia</span> <span style="color: #b4b4b4">||</span> <span style="color: white">navigator</span><span style="color: #b4b4b4">.</span><span style="color: white">msGetUserMedia</span><span style="color: #b4b4b4">;</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #608b4e">// Definición de variables globales</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">localUserMedia</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">null</span><span style="color: #b4b4b4">;</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">canvas</span> <span style="color: #b4b4b4">=</span> <span style="color: white">document</span><span style="color: #b4b4b4">.</span><span style="color: white">querySelector</span><span style="color: #b4b4b4">(</span><span style="color: #d69d85">&#8216;canvas&#8217;</span><span style="color: #b4b4b4">);</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">video</span> <span style="color: #b4b4b4">=</span> <span style="color: white">document</span><span style="color: #b4b4b4">.</span><span style="color: white">querySelector</span><span style="color: #b4b4b4">(</span><span style="color: #d69d85">&#8216;video&#8217;</span><span style="color: #b4b4b4">);</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">ctx</span> <span style="color: #b4b4b4">=</span> <span style="color: white">canvas</span><span style="color: #b4b4b4">.</span><span style="color: white">getContext</span><span style="color: #b4b4b4">(</span><span style="color: #d69d85">&#8216;2d&#8217;</span><span style="color: #b4b4b4">);</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #608b4e">// Cuando cargamos el vídeo guardamos la relación</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #608b4e">// de aspecto y ajustamos el tamaño del canvas</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #608b4e">// para que se mantenga</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">video</span><span style="color: #b4b4b4">.</span><span style="color: white">addEventListener</span><span style="color: #b4b4b4">(</span><span style="color: #d69d85">&#8216;loadedmetadata&#8217;</span><span style="color: #b4b4b4">,</span> <span style="color: #569cd6">function</span> <span style="color: #b4b4b4">(</span><span style="color: white">e</span><span style="color: #b4b4b4">)</span> <span style="color: #b4b4b4">{</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">relation</span> <span style="color: #b4b4b4">=</span>&#160; <span style="color: white">e</span><span style="color: #b4b4b4">.</span><span style="color: white">target</span><span style="color: #b4b4b4">.</span><span style="color: white">videoWidth</span><span style="color: #b4b4b4">/</span><span style="color: white">e</span><span style="color: #b4b4b4">.</span><span style="color: white">target</span><span style="color: #b4b4b4">.</span><span style="color: white">videoHeight</span><span style="color: #b4b4b4">;</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">canvas</span><span style="color: #b4b4b4">.</span><span style="color: white">width</span> <span style="color: #b4b4b4">=</span> <span style="color: #b5cea8">300</span><span style="color: #b4b4b4">;</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">canvas</span><span style="color: #b4b4b4">.</span><span style="color: white">height</span> <span style="color: #b4b4b4">=</span> <span style="color: #b5cea8">300</span><span style="color: #b4b4b4">/</span><span style="color: white">relation</span><span style="color: #b4b4b4">;</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #b4b4b4">},</span> <span style="color: #569cd6">false</span><span style="color: #b4b4b4">);</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #608b4e">// Capturamos el frame actual del video</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">document</span><span style="color: #b4b4b4">.</span><span style="color: white">getElementById</span><span style="color: #b4b4b4">(</span><span style="color: #d69d85">&#8216;snap&#8217;</span><span style="color: #b4b4b4">).</span><span style="color: white">addEventListener</span><span style="color: #b4b4b4">(</span><span style="color: #d69d85">&#8216;click&#8217;</span><span style="color: #b4b4b4">,</span> <span style="color: #569cd6">function</span> <span style="color: #b4b4b4">(</span><span style="color: white">e</span><span style="color: #b4b4b4">)</span> <span style="color: #b4b4b4">{</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">if</span> <span style="color: #b4b4b4">(</span><span style="color: white">localUserMedia</span><span style="color: #b4b4b4">)</span> <span style="color: #b4b4b4">{</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">ctx</span><span style="color: #b4b4b4">.</span><span style="color: white">drawImage</span><span style="color: #b4b4b4">(</span><span style="color: white">video</span><span style="color: #b4b4b4">,</span> <span style="color: #b5cea8"></span><span style="color: #b4b4b4">,</span> <span style="color: #b5cea8"></span><span style="color: #b4b4b4">,</span> <span style="color: white">canvas</span><span style="color: #b4b4b4">.</span><span style="color: white">width</span><span style="color: #b4b4b4">,</span> <span style="color: white">canvas</span><span style="color: #b4b4b4">.</span><span style="color: white">height</span><span style="color: #b4b4b4">);</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #b4b4b4">}</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #b4b4b4">},</span> <span style="color: #569cd6">false</span><span style="color: #b4b4b4">);</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #608b4e">// Callback de error de getUserMedia</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">onErrorCallback</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">function</span> <span style="color: #b4b4b4">(</span><span style="color: white">e</span><span style="color: #b4b4b4">)</span> <span style="color: #b4b4b4">{</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">console</span><span style="color: #b4b4b4">.</span><span style="color: white">log</span><span style="color: #b4b4b4">(</span><span style="color: #d69d85">&#8216;Error!&#8217;</span><span style="color: #b4b4b4">,</span> <span style="color: white">e</span><span style="color: #b4b4b4">);</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #b4b4b4">};</span>
  </p
></p> 
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #608b4e">// Capturamos el video con getUserMedia y lo</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #608b4e">// mandamos a un vídeo</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">navigator</span><span style="color: #b4b4b4">.</span><span style="color: white">getUserMedia</span><span style="color: #b4b4b4">({</span> <span style="color: white">video</span><span style="color: #b4b4b4">:</span> <span style="color: #569cd6">true</span> <span style="color: #b4b4b4">},</span> <span style="color: #569cd6">function</span> <span style="color: #b4b4b4">(</span><span style="color: white">localMediaStream</span><span style="color: #b4b4b4">)</span> <span style="color: #b4b4b4">{</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">video</span> <span style="color: #b4b4b4">=</span> <span style="color: white">document</span><span style="color: #b4b4b4">.</span><span style="color: white">querySelector</span><span style="color: #b4b4b4">(</span><span style="color: #d69d85">&#8216;video&#8217;</span><span style="color: #b4b4b4">);</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">video</span><span style="color: #b4b4b4">.</span><span style="color: white">src</span> <span style="color: #b4b4b4">=</span> <span style="color: white">window</span><span style="color: #b4b4b4">.</span><span style="color: white">URL</span><span style="color: #b4b4b4">.</span><span style="color: white">createObjectURL</span><span style="color: #b4b4b4">(</span><span style="color: white">localMediaStream</span><span style="color: #b4b4b4">);</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">localUserMedia</span> <span style="color: #b4b4b4">=</span> <span style="color: white">localMediaStream</span><span style="color: #b4b4b4">;</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #b4b4b4">},</span> <span style="color: white">onErrorCallback</span><span style="color: #b4b4b4">);</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"></</span><span style="color: #569cd6">script</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray"></</span><span style="color: #569cd6">body</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray"></</span><span style="color: #569cd6">html</span><span style="color: gray">></span>
  </p></p>
</div>

He puesto comentarios en el código para que sea más fácil de seguir. Pero la clave está en que ahora al pulsar el botón de “snapshot” utilizamos drawImage para volcar el contenido del frame actual del vídeo al canvas.

Una vez tenemos la imagen en un canvas ya tenemos vía libre! Por un lado podríamos utilizar el propio canvas para permitir que el usuario manipule la imagen y luego subirla via Ajax (ver mi post <a href="http://geeks.ms/blogs/etomas/archive/2012/12/14/html5-apis-crea-tu-propio-instagram.aspx" target="_blank" rel="noopener noreferrer">Crea tu propio Instagram</a>) o bien podemos directamente volcar el contenido del canvas en una imagen (utilizando toDataURL del propio canvas).

> No voy a poner en este post como subir los datos del canvas al servidor porque sería repetir lo que puse en el post de _Crea tu propio Instagram_ (Paso 5 del post).

En fin… hemos visto como gracias a WebRTC podemos (podremos) hacer algo que hasta hace poco parecía fuera totalmente de las posibilidades de la web: el acceso a dispositivos locales tales como micro y webcam. Y WebRTC no se queda ahí! Según la especificación será posible montar meetings on-line utilizando plataforma 100% web! Pero esta parte está todavía en una fase muy experimental e inicial de implementación!

Un saludo!

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_27E1D447.png
 [2]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_38556F9C.png