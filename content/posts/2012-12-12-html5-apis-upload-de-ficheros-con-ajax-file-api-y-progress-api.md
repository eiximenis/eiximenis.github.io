---
title: HTML5 Apis – Upload de ficheros con Ajax, File Api y Progress Api

author: eiximenis

date: 2012-12-12T17:13:39+00:00
geeks_url: /?p=1621
geeks_visits:
  - 5104
geeks_ms_views:
  - 2283
categories:
  - Uncategorized

---
¡Buenas! En el post anterior <a href="http://geeks.ms/blogs/etomas/archive/2012/12/05/html5-apis-file-api.aspx" target="_blank" rel="noopener noreferrer">vimos el funcionamiento de File Api</a> y como leer ficheros locales en servidor. En este post vamos a seguir usando File Api pero lo vamos a combinar con XMLHttpRequest y progress Api para ver como podemos hacer uploads de ficheros al servidor de forma fácil y asíncrona.

Para empezar vamos a montar la página:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: blue"><!</span><span style="color: maroon">DOCTYPE</span> <span style="color: red">html</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"><</span><span style="color: maroon">html</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"><</span><span style="color: maroon">head</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">meta</span> <span style="color: red">name</span><span style="color: blue">="viewport"</span> <span style="color: red">content</span><span style="color: blue">="width=device-width"</span> <span style="color: blue">/></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">title</span><span style="color: blue">></span>Index<span style="color: blue"></</span><span style="color: maroon">title</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"></</span><span style="color: maroon">head</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"><</span><span style="color: maroon">body</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">div</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; Selecciona fichero.
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">input</span> <span style="color: red">type</span><span style="color: blue">="file"</span> <span style="color: red">id</span><span style="color: blue">="file"</span> <span style="color: red">value</span><span style="color: blue">="enviar"</span> <span style="color: blue">/></span> <span style="color: blue"><</span><span style="color: maroon">br</span> <span style="color: blue">/></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">progress</span> <span style="color: red">id</span><span style="color: blue">="progress"></</span><span style="color: maroon">progress</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue"></</span><span style="color: maroon">div</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"></</span><span style="color: maroon">body</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"></</span><span style="color: maroon">html</span><span style="color: blue">></span>
  </p></p>
</div>

Un simple input file y un progress para poder mostrar el progreso.

Ahora el siguiente punto es usar XMLHttpRequest para hacer la petición y enviar el fichero de forma asíncrona. Para ello nos vamos a aprovechar de la característica de que el XMLHttpRequest puede enviar un objeto <a href="http://www.w3.org/TR/FileAPI/#dfn-file" target="_blank" rel="noopener noreferrer">File</a>. Para ello usamos <a href="http://dvcs.w3.org/hg/xhr/raw-file/tip/Overview.html#interface-formdata" target="_blank" rel="noopener noreferrer">FormData</a> que nos simplifica el código. Usando formData nos basta con una llamada a append para añadir el fichero. No solo esto, al usar FormData XMLHttpRequest usará el content-type correcto (“multipart/form-data”) al realizar la petición, sin que lo tengamos que establecer nosotros. Fijaos como després de abrir la conexión (xhr.open) nos limitamos a mandar el FormData que es quien contiene los datos 🙂

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: blue"><</span><span style="color: maroon">script</span> <span style="color: red">type</span><span style="color: blue">="text/javascript"></span>&#160;&#160;&#160;&#160;&#160;&#160;&#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">function</span> ficheroSeleccionado(e) {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">if</span> (e.target.files.length > 0) {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; subirFichero(e.target.files[0]);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">function</span> subirFichero(file) {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">var</span> xhr = <span style="color: blue">new</span> XMLHttpRequest();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">var</span> formData = <span style="color: blue">new</span> FormData();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; formData.append(<span style="color: #a31515">"file"</span>, file);
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; xhr.addEventListener(<span style="color: #a31515">"error"</span>, <span style="color: blue">function</span>(e) {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; alert(<span style="color: #a31515">"Error subiendo el archivo."</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">var</span> progress = document.getElementById(<span style="color: #a31515">"progress"</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; progress.value = 0;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; }, <span style="color: blue">false</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; xhr.addEventListener(<span style="color: #a31515">"load"</span>, <span style="color: blue">function</span>(e) {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; alert(<span style="color: #a31515">"fichero subido: "</span> + e.target.status + <span style="color: #a31515">"->"</span> + e.target.statusText);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; });
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; xhr.open(<span style="color: #a31515">&#8216;POST&#8217;</span>, <span style="color: #a31515">&#8216;</span><span style="background: yellow">@</span>Url.Action(<span style="color: #a31515">"Index"</span>)<span style="color: #a31515">&#8216;</span>, <span style="color: blue">true</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; xhr.send(formData);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; document.getElementById(<span style="color: #a31515">"file"</span>).addEventListener(<span style="color: #a31515">"change"</span>, ficheroSeleccionado,<br /> <span style="color: blue">false</span>);
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"></</span><span style="color: maroon">script</span><span style="color: blue">></span>
  </p></p>
</div>

> **Nota:** El objeto File se añade con el nombre “_file_” al FormData porque ese es el nombre del parámetro en la acción de destino del controlador (he usado ASP.NET MVC).

Con esto ya tenemos un file uploader con Ajax… ¿Qué sencillo, no? Pues, tal y como diría cierto conejo pesado, ¡no se vayan todavía, aún hay más!

**Progress API**

El objeto XMLHttpRequest define una propiedad llamada _upload_ que devuelve el <a href="http://www.w3.org/TR/XMLHttpRequest/#xmlhttprequestupload" target="_blank" rel="noopener noreferrer">XMLHttpRequestUpload</a> asociado. Pues bien dicho objeto define un evento, llamado “progress” que se va lanzando cada cierto tiempo donde _nos informa del estado de la petición_. Es decir, que ahora podemos saber cada cierto tiempo el total de bytes que se tienen que enviar y el total ya enviados. Y con ello podemos mostrar una progress bar (en mi caso he usado directamente un <progress>):

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    xhr.upload.addEventListener(<span style="color: #a31515">"progress"</span>, <span style="color: blue">function</span>(e) {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">if</span> (e.lengthComputable) {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">var</span> progress = document.getElementById(<span style="color: #a31515">"progress"</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; progress.max = e.total;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; progress.value = e.loaded;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }, <span style="color: blue">false</span>);
  </p></p>
</div>

Y lisos, es así de sencillo! Con ello ya tenemos un file uploader, que funciona por Ajax y que además con una progress nos va mostrando como va la subida de ficheros al servidor.

¿Fácil no? 🙂

PD: Si vas a probar el código en local recuerda que .NET limita el tamaño de la petición por defecto (sobre unos 4MB creo). Si vas a probar con archivos grandes usa <httpRuntime maxRequestLength=”xxx” /> para establecer el tamaño máximo de petición.

> PD2: Y recuerda que aún cuando uses httpRuntime, hay otro límite que aplica IIS (maxAllowedContentLength): [http://msdn.microsoft.com/en-us/library/ms689462%28VS.90%29.aspx][1]

 [1]: http://msdn.microsoft.com/en-us/library/ms689462%28VS.90%29.aspx "http://msdn.microsoft.com/en-us/library/ms689462%28VS.90%29.aspx"