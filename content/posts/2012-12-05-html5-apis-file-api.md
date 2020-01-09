---
title: 'HTML5 Apis: File API'
description: 'HTML5 Apis: File API'
author: eiximenis

date: 2012-12-05T16:53:00+00:00
geeks_url: /?p=1620
geeks_visits:
  - 2193
geeks_ms_views:
  - 1598
categories:
  - Uncategorized

---
&iexcl;Muy buenas! Vamos a empezar una serie de posts (que como digo siempre, a ver donde nos llevan) sobre las APIs de HTML5, dado que hay muchas (algunas más conocidas que otras). La que veremos en este post es File API que dicho rápidamente nos permite leer ficheros locales usando javascript.

Si al leer que ahora podemos leer ficheros desde javascript se te han puesto los pelos como escarpias pensando en los posibles agujeros de seguridad, tranquilo: no hay forma alguna de leer un fichero a través de su ruta. Siempre se requiere que sea el usuario el que inicie la acción y _abra_ explícitamente el fichero.

**Leyendo datos de un fichero**

Lo primero que necesitamos para poder acceder al contenido de un fichero es que el usuario lo abra. ¿Y que mecanismo tenemos en HTML para permitir al usuario seleccionar uno o varios ficheros? Exacto! El feote <input type=&rdquo;file&rdquo; /> y eso es lo primero que necesitamos.

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 1</span>&nbsp;<span style="color: blue"><!</span><span style="color: maroon">DOCTYPE</span> <span style="color: red">html</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 2</span>&nbsp;
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 3</span>&nbsp;<span style="color: blue"><</span><span style="color: maroon">html</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 4</span>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue"><</span><span style="color: maroon">head</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 5</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue"><</span><span style="color: maroon">title</span><span style="color: blue">></span>title<span style="color: blue"></</span><span style="color: maroon">title</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 6</span>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue"></</span><span style="color: maroon">head</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 7</span>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue"><</span><span style="color: maroon">body</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 8</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue"><</span><span style="color: maroon">div</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 9</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Selecciona un fichero: <span style="color: blue"><</span><span style="color: maroon">br</span> <span style="color: blue">/></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 10</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue"><</span><span style="color: maroon">input</span> <span style="color: red">type</span><span style="color: blue">=&#8221;file&#8221;</span> <span style="color: red">value</span><span style="color: blue">=&#8221;Leer&#8221;</span> <span style="color: red">id</span><span style="color: blue">=&#8221;fichero&#8221;/></span>&nbsp;
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 11</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue"></</span><span style="color: maroon">div</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 12</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue"><</span><span style="color: maroon">script</span> <span style="color: red">type</span><span style="color: blue">=&#8221;text/javascript&#8221;></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 13</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">function</span> ficheroSeleccionado(evt) {
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 14</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> ficheros = evt.target.files;
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 15</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: green">// Tan solo procesaremos el primer fichero</span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 16</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> fichero = ficheros[0];
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 17</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 18</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; document.getElementById(<span style="color: #a31515">&#8220;fichero&#8221;</span>).addEventListener(<span style="color: #a31515">&#8216;change&#8217;</span>, ficheroSeleccionado, <span style="color: blue">false</span>);
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 19</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue"></</span><span style="color: maroon">script</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 20</span>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue"></</span><span style="color: maroon">body</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 21</span>&nbsp;<span style="color: blue"></</span><span style="color: maroon">html</span><span style="color: blue">></span>
  </p>
</div>

Si ejecuto eso mismo en Chrome, y pongo un breakpoint en la línea 17 veo lo siguiente:

[<img height="106" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_6D5A6340.png" alt="image" border="0" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" title="image" />][1]

Podemos ver como efectivamente la propiedad files del <input> me ha devuelto una FileList y el primer elemento es un File con la información del archivo seleccionado. &iexcl;Bien!

Una vez tenemos un objeto File ya podemos leerlo, para ello debemos instanciar un FileReader y llamar a algunos de sus métodos:

  * readAsBinaryString: para leerlo de forma binaria . Ese método devuelve una cadena con el contenido binario. Si te parece _raro_ que se reciba una cadena con el contenido en binario, decirte que a los que crearon la API les debió parecer lo mismo (léete la PD al final del post).
  * readAsText: para leerlo como texto 
  * readAsDataURL: para leerlo como &ldquo;data-url&rdquo; (eso es ideal para previews de imágenes). 

Vamos a ver un ejemplo rapidísimo: vamos a crear un visor hexadecimal. Para ello añadimos dos <div>s, uno de los cuales contendrá los códigos hexadecimales de cada byte del archivo y otro contendrá el carácter ASCII imprimible asociado:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 1</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue"><</span><span style="color: maroon">style</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 2</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: maroon">.terminal</span> {<span style="color: red">font-family</span>: <span style="color: blue">courier</span> <span style="color: blue">new</span>}
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 3</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: maroon">#raw</span> {<span style="color: red">float</span>: <span style="color: blue">left</span>}
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 4</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: maroon">#rawtext</span> {<span style="color: red">float</span>:<span style="color: blue">right</span>}
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 5</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue"></</span><span style="color: maroon">style</span><span style="color: blue">></span>
  </p>
</div>

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 1</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue"><</span><span style="color: maroon">div</span> <span style="color: red">id</span><span style="color: blue">=&#8221;raw&#8221;</span> <span style="color: red">class</span><span style="color: blue">=&#8221;terminal&#8221;></</span><span style="color: maroon">div</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 2</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue"><</span><span style="color: maroon">div</span> <span style="color: red">id</span><span style="color: blue">=&#8221;rawtext&#8221;</span> <span style="color: red">class</span><span style="color: blue">=&#8221;terminal&#8221;</span> <span style="color: blue">></</span><span style="color: maroon">div</span><span style="color: blue">></span>
  </p>
</div>

Y ahora usaremos readAsBinaryString para obtener el contenido binario del archivo.

Cuando usemos las funciones de FileReader debemos tener presente que esas NO devuelven ningún resultado, si no que nos lanzan un evento cuando la lectura ha finalizado, y entonces podemos consultar el valor de la propiedad result del FileReader. Para ello modificamos la función ficheroSeleccionado para que quede tal y como sigue:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 1</span>&nbsp;<span style="color: blue">function</span> ficheroSeleccionado(evt) {
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 2</span>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> ficheros = evt.target.files;
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 3</span>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: green">// Tan solo procesaremos el primer fichero</span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 4</span>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> fichero = ficheros[0];
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 5</span>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> reader = <span style="color: blue">new</span> FileReader();
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 6</span>&nbsp;
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 7</span>&nbsp;&nbsp;&nbsp;&nbsp; reader.onloadend = <span style="color: blue">function</span>(ievt) {
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 8</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">if</span> (ievt.target.readyState == FileReader.DONE) {
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 9</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; mostrarResultado(ievt.target.result);
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 10</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 11</span>&nbsp;&nbsp;&nbsp;&nbsp; };
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 12</span>&nbsp;
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 13</span>&nbsp;&nbsp;&nbsp;&nbsp; reader.readAsBinaryString(fichero);
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 14</span> }
  </p>
</div>

Fijaos que asignamos una función gestora al evento loadend, que nos lanza el FileReader para indicar que la lectura ha finalizado. En esta función gestora nos limitamos a asegurarnos que realmente la lectura no ha dado error y si es el caso llamamos a mostrarResultado que será la función que rellenerá los dos <div>s:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 1</span>&nbsp;<span style="color: blue">function</span> mostrarResultado(resultado) {
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 2</span>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> raw = document.getElementById(<span style="color: #a31515">&#8220;raw&#8221;</span>);
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 3</span>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> rawtext = document.getElementById(<span style="color: #a31515">&#8220;rawtext&#8221;</span>);
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 4</span>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> shtmlraw = <span style="color: #a31515">&#8220;&#8221;</span>;
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 5</span>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> shtmltext = <span style="color: #a31515">&#8220;&#8221;</span>;
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 6</span>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">for</span> (<span style="color: blue">var</span> idx = 0; idx < resultado.length; idx++) {
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 7</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> ascii = resultado.charCodeAt(idx);
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 8</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> codigo = ascii.toString(16);
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 9</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">if</span> (codigo.length < 2) codigo = <span style="color: #a31515">&#8220;0&#8221;</span> + codigo;
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 10</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; shtmlraw += codigo + <span style="color: #a31515">&#8220;&nbsp;&#8221;</span>;
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 11</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; shtmltext += ascii >= 32 && ascii <= 127 ? <span style="color: #a31515">&#8220;&#&#8221;</span> + ascii : <span style="color: #a31515">&#8220;.&#8221;</span>;
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 12</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">if</span> ((idx + 1) % 8 == 0 && (idx + 1) % 24 != 0) {
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 13</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; shtmlraw += <span style="color: #a31515">&#8220;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#8221;</span>;
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 14</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 15</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">if</span> ((idx+1) % 24 == 0) {
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 16</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; shtmlraw += <span style="color: #a31515">&#8220;<br />&#8221;</span>;
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 17</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; shtmltext += <span style="color: #a31515">&#8220;<br />&#8221;</span>;
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 18</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 19</span>&nbsp;&nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 20</span>&nbsp;
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 21</span>&nbsp;&nbsp;&nbsp;&nbsp; raw.innerHTML = shtmlraw;
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 22</span>&nbsp;&nbsp;&nbsp;&nbsp; rawtext.innerHTML = shtmltext;
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 23</span> }
  </p>
</div>

No hay mucho que contar sobre esa función, es javascript puro y duro 🙂 Un consejo por si alguna vez usáis la propiedad innerHTML para asignar código HTML a un elemento del DOM: es muuuuuuuuy lenta. Así que es mejor ir creando el código en una cadena y usar innerHTML una sola vez al final.

&iexcl;Y listos! Ya puedo seleccionar un fichero cualquiera y ver su contenido binario:

[<img height="132" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_300988AC.png" alt="image" border="0" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-top-width: 0px" title="image" />][2]

Ahhh... que nostalgia de aquellos tiempos de MS-DOS, con aquellos editores hexadecimales tan bonitos, eh??? Pues bueno, aquí tenéis una buena imitación, eso sí solo lectura, que File API no permite modificar en ningún caso el contenido de ningún archivo!

**Previsualizando imágenes**

Bueno... ya puestos vamos a añadir un poco de funcionalidad a nuestro editor hexadecimal. Si el archivo seleccionado es una imagen vamos a visualizarla como tal (junto con su volcado hexadecimal).

Y es que File API nos permite de forma muy fácil crear una previsualización de una imagen, gracias al método readAsDataURL. Ya [hablé hace tiempo en este blog de las data url][3] así que no me repetiré ahora, simplemente para resumir: el atributo src de un <img> puede ser una URI especial (que empieza por data:) y que tiene el contenido en Base64 de la imagen a mostrar.

Pues bien, eso es precisamente lo que nos devuelve readAsDataURL. Así para previsualizar una imagen basta con asignar su atributo src al valor leído por el FileReader (en el evento loadend por supuesto). Veamos como queda el código completo.

En este caso el código queda un poco más liado porque lo que debemos hacer ahora es:

  1. Si el fichero es una imagen debemos 
      1. Leerlo usando readAsDataURL y previsualizarlo 
      2. Leerlo de nuevo usando readAsBinaryString y previsualizarlo 
  2. Si el fichero NO es una imagen debemos 
      1. Leerlo usando readAsBinaryString y previsualizarlo 

Para ello modificamos primero la función ficheroSeleccionado para saber si el fichero es una imagen o no y actuar en consecuencia:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 1</span>&nbsp;<span style="color: blue">function</span> ficheroSeleccionado(evt) {
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 2</span>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> ficheros = evt.target.files;
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 3</span>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: green">// Tan solo procesaremos el primer fichero</span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 4</span>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> fichero = ficheros[0];
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 5</span>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> reader = <span style="color: blue">new</span> FileReader();
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 6</span>&nbsp;
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 7</span>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">if</span> (fichero.type.match(<span style="color: #a31515">&#8216;image.*&#8217;</span>)) {
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 8</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; leerImagen(reader, fichero);
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 9</span>&nbsp;&nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 10</span>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">else</span> {
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 11</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; leerBinario(reader, fichero);
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 12</span>&nbsp;&nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 13</span>&nbsp;
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp; 14</span> }
  </p>
</div>

El método leerBinario es simplemente el código de lectura que teniamos antes en ficheroSeleccionado:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 1</span>&nbsp;<span style="color: blue">function</span> leerBinario(reader, fichero) {
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 2</span>&nbsp;&nbsp;&nbsp;&nbsp; reader.onloadend = <span style="color: blue">function</span> (ievt) {
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 3</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">if</span> (ievt.target.readyState == FileReader.DONE) {
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 4</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; mostrarResultado(ievt.target.result);
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 5</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 6</span>&nbsp;&nbsp;&nbsp;&nbsp; };
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 7</span>&nbsp;
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 8</span>&nbsp;&nbsp;&nbsp;&nbsp; reader.readAsBinaryString(fichero);
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 9</span> }
  </p>
</div>

El método mostrarResultado por supuesto es el mismo de antes 😉

Vayamos ahora a por el método leerImagen. Este simplemente leerá usando readAsDataURL _y luego_ llamará a leerBinario:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 1</span>&nbsp;<span style="color: blue">function</span> leerImagen(reader, fichero) {
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 2</span>&nbsp;&nbsp;&nbsp;&nbsp; reader.onloadend = <span style="color: blue">function</span> (ievt) {
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 3</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">if</span> (ievt.target.readyState == FileReader.DONE) {
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 4</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; previsualizarImagen(ievt.target);
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 5</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; leerBinario(reader, fichero);
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 6</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 7</span>&nbsp;&nbsp;&nbsp;&nbsp; };
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 8</span>&nbsp;&nbsp;&nbsp;&nbsp; reader.readAsDataURL(fichero);
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 9</span> }
  </p>
</div>

Y para completar la ecuación tan solo nos queda el método previsualizarImagen:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 1</span>&nbsp;<span style="color: blue">function</span> previsualizarImagen(filereader) {
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 2</span>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> datauri = filereader.result;
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 3</span>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> image = document.getElementById(<span style="color: #a31515">&#8220;preview&#8221;</span>);
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 4</span>&nbsp;&nbsp;&nbsp;&nbsp; image.src = datauri;
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 5</span>&nbsp;&nbsp;&nbsp;&nbsp; image.style.visibility = <span style="color: #a31515">&#8220;visible&#8221;</span>;
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 6</span> }
  </p>
</div>

Fijaos que se limita a poner en el atributo _src_ de una imagen el valor leído por el FileReader usando readAsDataURL. Y eso es lo que vemos si seleccionamos una imagen:

[<img height="132" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_2588B48C.png" alt="image" border="0" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-top-width: 0px" title="image" />][4]

Fijaos como podemos ver la imagen, _sin_ que esa haya subido en ningún momento al servidor.

&iexcl;Y listos! Con eso ya hemos visto lo básico de File API. Hay más, pero mejor lo dejamos para otro post de esta serie 😉

**PD: Notas sobre soporte de los navegadores**.

El código de este post lo he probado con Chrome (a la hora de escribirlo era la versión 23.0, cuando le leáis a saber cual es) y con IE10. Ambos soportan File API (al igual que el resto de los navegadores importantes).

Peeeeeero el método _readAsBinaryString_ NO está soportado en IE10. Pero, que quede claro, **no es un incumplimiento del estándard**: El estandard de la W3C **no** define un método readAsBinaryString (lo podeis ver en [http://www.w3.org/TR/FileAPI/#dfn-filereader][5]). Existió tiempo atrás pero se marcó como obsoleto (y en la versión final ha desaparecido). Chrome lo mantiene (creo que FF también) pero IE10 es más estricto con el cumplimiento del estandard y no lo hace.

Eso es típico cuando usamos HTML5: algunas de esas APIs están vivas o se han terminado hace muy poco y a veces es fácil caer en métodos obsoletos o ya desparecidos. 

**PD2: Vale... muy bien ¿pero como acceder al contenido binario usando la última especificación del estandard?**

&iexcl;Que nadie se alarme! Es muy sencillo: debemos usar el método readAsArrayBuffer() en lugar de readAsBinaryString() que nos devuelve un <a target="_blank" href="https://developer.mozilla.org/en-US/docs/JavaScript_typed_arrays" rel="noopener noreferrer">ArrayBuffer</a> que contiene los valores. Veamoslo.

El cambio a leerBinario es trivial:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 1</span>&nbsp;<span style="color: blue">function</span> leerBinario(reader, fichero) {
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 2</span>&nbsp;&nbsp;&nbsp;&nbsp; reader.onloadend = <span style="color: blue">function</span> (ievt) {
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 3</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">if</span> (ievt.target.readyState == FileReader.DONE) {
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 4</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; mostrarResultado(ievt.target.result);
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 5</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 6</span>&nbsp;&nbsp;&nbsp;&nbsp; };
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 7</span>&nbsp;
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 8</span>&nbsp;&nbsp;&nbsp;&nbsp; reader.readAsArrayBuffer(fichero);
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 9</span> }
  </p>
</div>

Y será mostrarResultado el que ahora en lugar de recibir una cadena con el contenido en binario recibe un ArrayBuffer. El tema es que los ArrayBuffer no se pueden consultar directamente. En su lugar debe usarse una _vista_ por encima del ArrayBuffer. En nuestro caso dado que queremos leer byte a byte usaremos un Uint8Array.

El código de mostrarResultado es idéntico al que ya teníamos con la excepción del principio:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 1</span>&nbsp;<span style="color: blue">function</span> mostrarResultado(resultado) {
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 2</span>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> raw = document.getElementById(<span style="color: #a31515">&#8220;raw&#8221;</span>);
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 3</span>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> rawtext = document.getElementById(<span style="color: #a31515">&#8220;rawtext&#8221;</span>);
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 4</span>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> shtmlraw = <span style="color: #a31515">&#8220;&#8221;</span>;
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 5</span>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> shtmltext = <span style="color: #a31515">&#8220;&#8221;</span>;
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 6</span>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> view = <span style="color: blue">new</span> Uint8Array(resultado);
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 7</span>&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">for</span> (<span style="color: blue">var</span> idx = 0; idx < view.length; idx++) {
  </p>
  
  <p style="margin: 0px">
    <span style="color: teal">&nbsp;&nbsp;&nbsp; 8</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> ascii = view[idx];
  </p>
</div>

El resto del código es idéntico. Fijaos como creamos el Uint8Array por encima del ArrayBuffer devuelto y luego ya leemos byte a byte. Dado que ahora _view_ es un array ya de bytes y no una cadena como antes, ya no necesitamos usar charCodeAt() para obtener el código ascii (valor del byte) asociado.

Ahora sí que el código debería funcionar tanto en IE10, como en Chrome, como en FF como en el resto de navegadores importantes!

 [1]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_59AD79AC.png
 [2]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_1E799DE1.png
 [3]: /blogs/etomas/archive/2011/03/15/asp-net-mvc-previsualizar-im-225-genes-subidas-2.aspx
 [4]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_12B430E2.png
 [5]: http://www.w3.org/TR/FileAPI/#dfn-filereader "http://www.w3.org/TR/FileAPI/#dfn-filereader"