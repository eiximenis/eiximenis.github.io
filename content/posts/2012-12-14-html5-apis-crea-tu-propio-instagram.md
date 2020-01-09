---
title: HTML5 Apis – Crea tu propio Instagram
description: HTML5 Apis – Crea tu propio Instagram
author: eiximenis

date: 2012-12-14T14:41:00+00:00
geeks_url: /?p=1622
geeks_visits:
  - 2555
geeks_ms_views:
  - 1521
categories:
  - Uncategorized

---
Sigamos con la serie de posts sobre las APIs de HTML5. Ahora le toca al canvas, uno de los elementos más revolucionarios de HTML5. Yo siempre digo que si <video /> hirío a Flash, entonces <canvas /> lo mata definitivamente.

Que es el canvas? Pues dicho rápido y mal: Un nuevo elemento de HTML, que nos permite tener una superficie de dibujo. El canvas por si mismo **no** tiene una API asociada, en su lugar se obtiene un contexto de dibiujo sobre el canvas. Es dicho contexto el que nos proporciona una API para interaccionar con el canvas.

En la actualidad hay dos especificaciones de contextos distintas:

  * <a target="_blank" href="http://www.w3.org/TR/2dcontext/" rel="noopener noreferrer">2d</a>: Para operaciones de 2D. Contiene una API muy sencilla para por un lado dibujar en el canvas (líneas, círculos, cuadrados, etc) y por otra acceder directamente al contenido binario del canvas. 
  * <a target="_blank" href="http://www.khronos.org/registry/webgl/specs/latest/" rel="noopener noreferrer">webgl</a>: Para operaciones 3D. Ofrece una API basada en OpenGL ES 2.0. 

Actualmente todos los navegadores soportan el contexto 2d, y la mayoría soportan webgl. La excepción es IE y la razón principal (al margen de ciertas vulnerabilidades descubiertas en webgl) es que **no** es un estándard W3C.

**No** voy a explicar en este post como dibujar en el canvas, hay multitud de tutoriales para ello. En su lugar vamos a ver como combinar el canvas, File Api, un poco de drag and drop y acceso al contenido binario del canvas para crearnos nuestro propio Instagram 🙂

**Paso 1 &ndash; Declaración del <canvas />**

Bueno... eso no tiene apenas ningún secreto, basta con añadir un tag <canvas> y darle un tamaño:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: blue"><</span><span style="color: maroon">div</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue"><</span><span style="color: maroon">canvas</span> <span style="color: red">height</span><span style="color: blue">=&#8221;400&#8243;</span> <span style="color: red">width</span><span style="color: blue">=&#8221;600&#8243;</span> <span style="color: red">id</span><span style="color: blue">=&#8221;mc&#8221;</span> <span style="color: red">style</span><span style="color: blue">=&#8221;</span><span style="color: red">border</span>: <span style="color: blue">1px</span> <span style="color: blue">solid</span> <span style="color: blue">black&#8221;></</span><span style="color: maroon">canvas</span><span style="color: blue">></span>&nbsp;
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"></</span><span style="color: maroon">div</span><span style="color: blue">></span>
  </p>
</div>

**Paso 2 &ndash; Habilitar drag and drop**

Ahora lo que queremos es que el usuario pueda hacer drop de un fichero local sobre el canvas y que nosotros lo podamos procesar. Todavía no hemos visto la API nativa de drag and drop que incorpora HTML5 pero vamos a ver lo básico ahora para habilitarla. Debemos seguir dos pasos para que un elemento sea una zona donde se pueda hacer drop:

  1. Asociarnos a su evento _dragOver_ y allí hacer un _preventDefault_. Eso es porque por defecto todos los objetos del DOM heredan una implementación de dicho evento que significa &ldquo;no se puede hacer drop aquí&rdquo;. Al rededinir dicho evento quitamos este comportamiento por defecto y nuestro elemento del DOM pasa a ser un zona donde se puede hacer drop. El evento dragOver se dispara cuando estamos arrastrando algo y pasamos por encima del elemento. 
  2. Asociarnos a su evento _drop_ y allí recoger los datos que se hayan arrastrado. En el caso que nos ocupa (arrastre de ficheros locales hacia un elemento del DOM), el evento de drop tiene una propiedad dataTransfer que tiene una propiedad files que es una FileList (de File API), con la información de los ficheros que se hayan arrastrado. 

Por lo tanto añadamos el mínimo código script para soportar el drag and drop:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: blue"><</span><span style="color: maroon">script</span> <span style="color: red">type</span><span style="color: blue">=&#8221;text/javascript&#8221;></span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> canvas = document.getElementById(<span style="color: #a31515">&#8220;mc&#8221;</span>);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; canvas.addEventListener(<span style="color: #a31515">&#8220;dragover&#8221;</span>, handleDragOver, <span style="color: blue">false</span>);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; canvas.addEventListener(<span style="color: #a31515">&#8220;drop&#8221;</span>, handleDrop, <span style="color: blue">false</span>);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">function</span> handleDragOver(e) {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; e.preventDefault();
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    &nbsp;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">function</span> handleDrop(e) {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> files = e.dataTransfer.files;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">if</span> (files.length > 0) {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> file = files[0];
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; procesarFichero(file);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    &nbsp;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; e.preventDefault();
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    &nbsp;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">function</span> procesarFichero(file) {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; alert(<span style="color: #a31515">&#8220;has arrastrado &#8220;</span> + file.name + <span style="color: #a31515">&#8220;->&#8221;</span> + file.type);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"></</span><span style="color: maroon">script</span><span style="color: blue">></span>
  </p>
</div>

Con esto ya tenemos el drag and drop habilitado para el canvas y podemos arrastrar y soltar en él, ficheros locales!

**Paso 3 &ndash; Leer la imagen y colocarla en el canvas**

De todas las funciones que nos ofrece el contexto 2d para el canvas, hay una que nos interesa ahora mismo que es drawImage. Esta función nos permite recoger el contenido de una imagen **ya existente** y colocarlo en el canvas. Por lo tanto _antes_ de usar drawImage necesitamos tener una imagen cargada. Ya vimos en el <a target="_blank" href="/blogs/etomas/archive/2012/12/05/html5-apis-file-api.aspx" rel="noopener noreferrer">post dedicado a File API</a> como leer un objeto File y asignarlo a una imagen: con el método ReadAsDataURL. Bien, pues lo hacemos:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: blue">function</span> procesarFichero(file) {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> fr = <span style="color: blue">new</span> FileReader();
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; fr.addEventListener(<span style="color: #a31515">&#8220;loadend&#8221;</span>, <span style="color: blue">function</span>(e) {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> datauri = e.target.result;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; loadCanvas(datauri);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; }, <span style="color: blue">false</span>);
  </p>
  
  <p style="margin: 0px">
    &nbsp;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; fr.readAsDataURL(file);
  </p>
  
  <p style="margin: 0px">
    }
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue">function</span> loadCanvas(datauri) {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; alert(datauri);
  </p>
  
  <p style="margin: 0px">
    }
  </p>
</div>

Ahora tan solo nos falta colocar la imagen en el canvas. Para ello, primero la tenemos que colocar en un objeto Imagen y luego sí, ya en el canvas:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: blue">function</span> loadCanvas(datauri) {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> img = <span style="color: blue">new</span> Image();
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; img.addEventListener(<span style="color: #a31515">&#8220;load&#8221;</span>, <span style="color: blue">function</span>(e) {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> ctx = canvas.getContext(<span style="color: #a31515">&#8220;2d&#8221;</span>);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ctx.drawImage(img, 0, 0);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; }, <span style="color: blue">false</span>);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; img.src = datauri;
  </p>
  
  <p style="margin: 0px">
    }
  </p>
</div>

A destacar de este código anterior:

  1. El uso de getContext(&#8220;2d&#8221;) para obtener el contexto 2d. Ojo, que la cadena que se pasa a getContext es case-sensitive! 
  2. Creamos una imagen y le asignamos como src el resultado de readAsDataURL. 
  3. Usamos drawImage para dibujar la imagen en el canvas. Importante que esto lo tenemos que hacer dentro del evento _load_ de la imagen, para asegurarnos que esta estará cargada.

**Paso 4: Crear un filtro**

Vamos a crear un filtro para manipular nuestra imagen. En este ejemplo será muy sencillo: un botón que _invertirá_ los colores de la imagen.

Para implementar este filtro debemos acceder al contenido binario del canvas. Dicho contenido es muy sencillo: cada pixel del canvas ocupa 4 bytes (rojo, verde, azul y canal alfa), y está guardado por filas (es decir, primero todos los pixels de la primera fila de izquierda a derecha y así sucesivamente).

El método getImageData del contexto, nos devuelve dicho array rellenado con el contenido _actual_ del canvas. A dicho método se le pasa el rectángulo que queremos obtener (punto top-left, ancho y alto) y nos devolverá el array correspondiente. Cada píxel ocupará, insisto, 4 posiciones de dicho array (el array es de bytes).

Así me creo mi función de filtro, que simplemente invertirá los valores de r,g,b dejando el canal alfa tal cual estaba:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: blue">function</span> applyFilter() {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> ctx = canvas.getContext(<span style="color: #a31515">&#8220;2d&#8221;</span>);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">for</span> (<span style="color: blue">var</span> idx = 0; idx < imageData.data.length; idx+=4) {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> r = imageData.data[idx];
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> g = imageData.data[idx + 1];
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> b = imageData.data[idx + 2];
  </p>
  
  <p style="margin: 0px">
    &nbsp;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; imageData.data[idx] = 255 &#8211; r;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; imageData.data[idx + 1] = 255 &#8211; g;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; imageData.data[idx + 2] = 255 &#8211; b;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; ctx.putImageData(imageData, 0, 0);
  </p>
  
  <p style="margin: 0px">
    }
  </p>
</div>

Ahora tan solo me falta añadir un botón a la página y enlazar el click del botón con el método applyFilter.

**Paso 5: Enviar los datos del canvas al servidor**

Una vez hemos modificado el canvas nos puede interesar mandar la imagen que hay en el canvas hacia el servidor. Para ello no hay ninguna función específica, así que veremos como podemos hacerlo.

Hay un método <a target="_blank" href="http://www.whatwg.org/specs/web-apps/current-work/multipage/the-canvas-element.html#dom-canvas-toblob" rel="noopener noreferrer">toBlob</a>**** en el propio canvas que nos devuelve un Blob (es casi equivalente al File de File API que conocemos, de hecho File deriva de Blob), pero es muy nuevo y no está apenas implementado en ningún navegador, así que lo descartamos. Dicho método sería la manera más directa, ya que luego con un FormData podemos mandar directamente este Blob (como vimos en el <a target="_blank" href="/blogs/etomas/archive/2012/12/12/html5-apis-upload-de-ficheros-con-ajax-file-api-y-progress-api.aspx" rel="noopener noreferrer">post de uploads asíncronos</a>). Pero como digo, queda descartado.

Pero bueno, tampoco es ningún drama. La solución pasa por crearnos el Blob a mano y rellenarlo con los datos del canvas... Veamos:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: blue">function</span> uploadCanvas() {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> ctx = canvas.getContext(<span style="color: #a31515">&#8220;2d&#8221;</span>);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> xhr = <span style="color: blue">new</span> XMLHttpRequest();
  </p>
  
  <p style="margin: 0px">
    &nbsp;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> arrayBuffer = <span style="color: blue">new</span> ArrayBuffer(imageData.data.length);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> view = <span style="color: blue">new</span> Uint8Array(arrayBuffer);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">for</span> (<span style="color: blue">var</span> idx = 0; idx < imageData.data.length; idx++) {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; view[idx] = imageData.data[idx];
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> blob = <span style="color: blue">new</span> Blob([view]);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> formData = <span style="color: blue">new</span> FormData();
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; formData.append(<span style="color: #a31515">&#8220;canvas&#8221;</span>, blob);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; formData.append(<span style="color: #a31515">&#8220;w&#8221;</span>, canvas.width.toString());
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; formData.append(<span style="color: #a31515">&#8220;h&#8221;</span>, canvas.height.toString());
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; xhr.open(<span style="color: #a31515">&#8220;POST&#8221;</span>, <span style="color: #a31515">&#8220;</span><span style="background: yellow">@</span>Url.Action(<span style="color: #a31515">&#8220;Index&#8221;</span>, <span style="color: #a31515">&#8220;Instagram&#8221;</span>)<span style="color: #a31515">&#8220;</span>, <span style="color: blue">true</span>);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; xhr.send(formData);
  </p>
  
  <p style="margin: 0px">
    }
  </p>
</div>

Para crear un Blob debemos crear antes un ArrayBuffer con los datos de dicho Blob, y para acceder (y rellenar) un ArrayBuffer debemos hacerlo a través de un Uint8Array. De ahí que el código es un poco liadillo.

Luego, además de los datos binarios del canvas, mandamos el ancho y el alto, para tenerlos en servidor.

Ahora nos falta la parte de servidor:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    [<span style="color: #2b91af">HttpPost</span>]
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue">public</span> <span style="color: #2b91af">ActionResult</span> Index(<span style="color: #2b91af">HttpPostedFileBase</span> canvas, <span style="color: blue">int</span> w, <span style="color: blue">int</span> h)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> stream = canvas.InputStream;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> bmp = <span style="color: blue">new</span> <span style="color: #2b91af">Bitmap</span>(w, h);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">int</span> r, g, b, a;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">for</span> (<span style="color: blue">var</span> row = 0; row < h; row++)
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">for</span> (<span style="color: blue">var</span> col = 0; col < w; col++)
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; r = stream.ReadByte();
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; g = stream.ReadByte();
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; b = stream.ReadByte();
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; a = stream.ReadByte();
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; bmp.SetPixel(col, row, <span style="color: #2b91af">Color</span>.FromArgb(a, r, g, b));
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; bmp.Save(<span style="color: #a31515">&#8220;d:\filename.png&#8221;</span>, <span style="color: #2b91af">ImageFormat</span>.Png);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">return</span> <span style="color: blue">new</span> <span style="color: #2b91af">HttpStatusCodeResult</span>(200);
  </p>
  
  <p style="margin: 0px">
    }
  </p>
</div>

¿Sencillo no? Creamos un bitmap, decodificamos los datos binarios del canvas y &ldquo;pintamos&rdquo; el Bitmap. Luego lo guardamos y listo 😉

**Paso 6: Guardar en el cliente**

Para finalizar esta maravilla que estamos creando vamos a añadir un botón para guardar la imagen modificada en el cliente, sin necesidad de subirla al servidor.

Para ello nos vamos a aprovechar de un método existente en el canvas llamado toDataURL, que como supondrás nos convierte el contenido del canvas a una data URL:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: blue">function</span> saveCanvas() {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> datauri = canvas.toDataURL(<span style="color: #a31515">&#8220;image/png&#8221;</span>);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; window.open(datauri, <span style="color: #a31515">&#8220;preview&#8221;</span>, <span style="color: #a31515">&#8220;width=&#8221;</span> + (canvas.width + 20) +<span style="color: #a31515">&#8220;,height=&#8221;</span> + (canvas.height + 20));
  </p>
  
  <p style="margin: 0px">
    }
  </p>
</div>

Esta función la tengo enlazada al evento click de un botón y lo que hace es simplemente mostrar la imagen en un popup y así el usuario puede hacer &#8220;guardar como&#8221; y listos.

Existe una API definida (<a target="_blank" href="http://www.w3.org/TR/file-writer-api/" rel="noopener noreferrer">FileWriter</a>) para escribir ficheros desde javascript pero su soporte es hoy en día tan minoritario que no tiene sentido plantearse el usarla. Otra alternativa, pero también minoritaria (sólo funciona con Chrome) sería usar un tag <a /> dinámico con el <a target="_blank" href="http://www.whatwg.org/specs/web-apps/current-work/multipage/links.html#attr-hyperlink-download" rel="noopener noreferrer">atributo download</a> y forzar el click con javascript.

Saludos!