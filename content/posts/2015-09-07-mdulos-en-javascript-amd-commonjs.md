---
title: Módulos en JavaScript… AMD, CommonJS

author: eiximenis

date: 2015-09-07T17:06:12+00:00
geeks_url: /?p=1700
geeks_visits:
  - 706
geeks_ms_views:
  - 3140
categories:
  - Uncategorized

---
Con las aplicaciones web cada vez con mayor carga de cliente, y el uso cada vez mayor de sistemas de _build_ en cliente como grunt o gulp, usar módulos para desarrollar en JavaScript es cada vez más habitual. Si todavía te descargas las librerías de sus páginas web y las incluyes una a una en tags <script/> es probable que este post te interese.

**¿Qué es un módulo?**

Llamamos módulo JavaScript a un código que de alguna manera es “auto contenido” y que expone una interfaz pública para ser usada. Esto no es realmente nuevo, el [patrón de módulo][1] ya hace bastantes años que se utiliza y no requiere más que algunos conocimientos de JavaScript para aplicarlo. El problema con los módulos en JavaScript no ha sido nunca el crearlos si no el de **cargarlos**. Puede parecer simple… de hecho, ¿no se trata solo de poner un tag <script />? Pues la realidad es que no, porque cargar un módulo implica que _antes_ deben estar cargadas sus **dependencias** y por lo tanto debemos tener un mecanismo para definir esas dependencias y otro mecaniso para cargarlas al tiempo que cargamos el módulo deseado.

Es ahí donde entran en juego los distintos estándares de módulos que tenemos. Nos permiten crear módulos JavaScript, declarar las dependencias (es decir indicar de qué módulos depende nuestro módulo e incorporar la funcionalidad del módulo del cual dependemos) y cargar determinados módulos. Hay dos estándares usados hoy en día: **CommonJS** y **AMD**.

**CommonJS**

CommonJS es un sistema de módulos **síncrono**: es decir la carga de módulos es un proceso síncrono que empieza por un módulo inicial. Al cargarse este módulo se cargaran todas sus dependencias (y las dependencias de las dependencias, y las dependencias de las dependencias de las dependencias… y así hasta cualquier nivel de profundidad). Una vez finalicen todas esas cargas, el módulo inicial está cargado y empieza a ejecutarse. Definir un módulo en formato CommonJS es muy sencillo:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:9e64f64a-1926-4377-a21c-c348dcd20de2" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> Complex = </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> (r, i) {</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">this</span><span style="background:#ffffff;color:#000000">.r = r </span><span style="background:#ffffff;color:#0000ff">instanceof</span><span style="background:#ffffff;color:#000000"> Complex ? r.r : r;</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">this</span><span style="background:#ffffff;color:#000000">.i = r </span><span style="background:#ffffff;color:#0000ff">instanceof</span><span style="background:#ffffff;color:#000000"> Complex ? r.i : (i || 0);</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">module.exports = Complex;</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Supón que este código está en un fichero Complex.js. Este código define un módulo que exporta una función (constructora) llamada Complex. Observa el uso de **module.exports** para indicar que es lo que exporta el módulo. Todo lo que no pertenezca al exports son variables (y funciones) privadas del módulo. Ahora podríamos declarar otro módulo que dependiese de este módulo:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:2a381c63-435a-4951-886c-d12159eddef3" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> Complex = require(</span><span style="background:#ffffff;color:#a31515">'./complex'</span><span style="background:#ffffff;color:#000000">);</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">addComplex = </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> (ca, cb) {</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">new</span><span style="background:#ffffff;color:#000000"> Complex(ca.r + cb.r, ca.i + cb.i);</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> math = {</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">add: </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> (a, b) {</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">if</span><span style="background:#ffffff;color:#000000"> (a </span><span style="background:#ffffff;color:#0000ff">instanceof</span><span style="background:#ffffff;color:#000000"> Complex || b </span><span style="background:#ffffff;color:#0000ff">instanceof</span><span style="background:#ffffff;color:#000000"> Complex) {</span>
        </li>
        <li>
                      <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> addComplex(</span><span style="background:#ffffff;color:#0000ff">new</span><span style="background:#ffffff;color:#000000"> Complex(a), </span><span style="background:#ffffff;color:#0000ff">new</span><span style="background:#ffffff;color:#000000"> Complex(b));</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> a + b;</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">module.exports = math;</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Este módulo (math.js) requiere el módulo complex.js (de ahí el uso de **require**), define un objeto math con un método y exporta dicho objeto. La función _addComplex_ es privada al módulo.

Finalmente podemos crear un tercer módulo (main.js) que use esos módulos para sumar tanto números reales como complejos. Este va a ser nuestro módulo inicial:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:a1a36503-f58a-42c6-ac56-4517f7d30a65" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000
; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    </p> 
    
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> Complex = require(</span><span style="background:#ffffff;color:#a31515">'./complex'</span><span style="background:#ffffff;color:#000000">);</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> math = require(</span><span style="background:#ffffff;color:#a31515">'./math'</span><span style="background:#ffffff;color:#000000">);</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">console.log(math.add(40, 2));</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> c1 = </span><span style="background:#ffffff;color:#0000ff">new</span><span style="background:#ffffff;color:#000000"> Complex(40, 3);</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">console.log(math.add(c1, 2));</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Si ejecutamos el siguiente código mediante nodejs vemos como todo funciona correctamente:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; margin: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_0AAA960D.png" width="244" height="182" />][2]

Nodejs soporta módulos CommonJS de forma nativa, pero… ¿qué pasa con el navegador? Pues que necesitamos soporte de alguna herramienta externa. Una de las más conocidas es <a href="http://browserify.org/" target="_blank" rel="noopener noreferrer">browserify</a>**** que se instala como un paquete de node. Browserify es, a la vez, una herramienta de línea de comandos y un módulo CommonJS que podemos integrar con grunt o gulp. Si usamos la herrramienta de línea de comandos, se puede usar el comando _**browserify main.js > bundle.js**_ para crear un fichero (bundle.js) que contenga el código de main.js y de todos sus módulos requeridos. Este fichero es el que usaría con un tag script.

Lo bueno de browserify es que solo debo indicarle el fichero inicial (en mi caso main.js). Él se encarga de ver los módulos necesarios y empaquetarlos todos juntos en un fichero. Usar browserify como herramienta de línea de comandos es posible (para ello basta con que lo tengas instalado como módulo global, es decir _npm install –g browserify_), pero no es lo más cómodo: lo suyo es tenerlo integrado dentro de nuestro script de build que tengamos con gulp o grunt. Por suerte browserify es también un módulo CommonJS por lo que podemos usarlo dentro de nuestro script de build. P. ej. el siguiente código muestra como usarlo mediante gulp:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:28b170c8-b9f3-4bda-b701-47de5001829e" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> gulp = require(</span><span style="background:#ffffff;color:#a31515">'gulp'</span><span style="background:#ffffff;color:#000000">);</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> browserify = require(</span><span style="background:#ffffff;color:#a31515">'browserify'</span><span style="background:#ffffff;color:#000000">);</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> source = require(</span><span style="background:#ffffff;color:#a31515">'vinyl-source-stream'</span><span style="background:#ffffff;color:#000000">);</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">gulp.task(</span><span style="background:#ffffff;color:#a31515">'browserify'</span><span style="background:#ffffff;color:#000000">, </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> () {</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">browserify(</span><span style="background:#ffffff;color:#a31515">'./main.js'</span><span style="background:#ffffff;color:#000000">)</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000">.bundle()</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000">.pipe(source(</span><span style="background:#ffffff;color:#a31515">'bundle.js'</span><span style="background:#ffffff;color:#000000">))</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000">.pipe(gulp.dest(</span><span style="background:#ffffff;color:#a31515">'./scripts'</span><span style="background:#ffffff;color:#000000">));</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">});</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Con esto basta con que ejecutes “gulp broswerify” para que browserify te deje en scripts un fichero bundle.js con el resultado de browserify. El único requisito es tener instalado (además de gulp y broswerify, obviamente) un módulo llamado <a href="https://github.com/hughsk/vinyl-source-stream" target="_blank" rel="noopener noreferrer">vinyl-source-stream</a> que se usa para pasar de streams basados en texto (los que usa browserify)&#160; a streams de gulp.

Muchas de las librerías JavaScript (incluyendo incluso jQuery) tienen versión CommonJS lo que ayuda mucho a organizar tu código. Por supuesto se puede configurar browserify para que genere source maps o que aplique más transformaciones al código (p. ej. convertir código JSX de React en código JavaScript).

**AMD**

AMD es otra especificación de módulos JavaScript, cuya principal diferencia con CommonJS es que es asíncrona (AMD significa Asynchronous Module Definition). La implementación más conocida para navegadores de AMD es <a href="http://requirejs.org/" target="_blank" rel="noopener noreferrer">requirejs</a>. Al ser asíncrona permite escenarios **con carga de módulos bajo demanda** (es decir cargar un módulo sólo si se va a usar), lo que puede ser interesante en según que aplicaciones.

Si usas requirejs no necesitas nada más: no es necesario que uses ninguna herramienta de línea de comandos o que crees tareas de grunt o gulp. Dado que requirejs implementa AMD va a ir cargando los módulos JavaScript de forma asíncrona. No tienes por qué crear un bundle con todos ellos.

La sintaxis para definir un módulo AMD es un poco más “liosa” que la sintaxis de CommonJS, pero tampoco mucho más. Empecemos por ver como sería el módulo AMD para definir el tipo Complex:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:c2c5e479-fda0-4439-9565-b6e35f45fc82" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#000000">define([], </span><span style="background:#ffffff;colo
r:#0000ff">function</span><span style="background:#ffffff;color:#000000"> () {</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">console.log(</span><span style="background:#ffffff;color:#a31515">'complex loaded...'</span><span style="background:#ffffff;color:#000000">);</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> Complex = </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> (r, i) {</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">this</span><span style="background:#ffffff;color:#000000">.r = r </span><span style="background:#ffffff;color:#0000ff">instanceof</span><span style="background:#ffffff;color:#000000"> Complex ? r.r : r;</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">this</span><span style="background:#ffffff;color:#000000">.i = r </span><span style="background:#ffffff;color:#0000ff">instanceof</span><span style="background:#ffffff;color:#000000"> Complex ? r.i : (i || 0);</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> Complex;</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">});</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Los módulos AMD empiezan con una llamada a define, que acepta básicamente dos parámetros: un array con las dependencias del módulo (el equivalente al require de CommonJS) y luego una función con el código del&#160; módulo. Esa función devuelve lo que el módulo exporta (es decir, el return de la función equivale al module.exports de CommonJS). El módulo que define Complex no depende de nadie, así que el array está vacío. No ocurre lo mismo con el modulo math (fichero math_amd.js):

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:870300c4-7947-4a41-ad46-748ec3d007a9" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#000000">define([</span><span style="background:#ffffff;color:#a31515">'complex_amd'</span><span style="background:#ffffff;color:#000000">], </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> (Complex) {</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">addComplex = </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> (ca, cb) {</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#0000ff">new</span><span style="background:#ffffff;color:#000000"> Complex(ca.r + cb.r, ca.i + cb.i);</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> math = {</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000">add: </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> (a, b) {</span>
        </li>
        <li>
                      <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">if</span><span style="background:#ffffff;color:#000000"> (a </span><span style="background:#ffffff;color:#0000ff">instanceof</span><span style="background:#ffffff;color:#000000"> Complex || b </span><span style="background:#ffffff;color:#0000ff">instanceof</span><span style="background:#ffffff;color:#000000"> Complex) {</span>
        </li>
        <li>
                          <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> addComplex(</span><span style="background:#ffffff;color:#0000ff">new</span><span style="background:#ffffff;color:#000000"> Complex(a), </span><span style="background:#ffffff;color:#0000ff">new</span><span style="background:#ffffff;color:#000000"> Complex(b));</span>
        </li>
        <li>
                      <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
                      <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> a + b;</span>
        </li>
        <li>
                  <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">}</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">return</span><span style="background:#ffffff;color:#000000"> math;</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">});</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Observa ahora como el módulo depende del módulo complex\_amd. Eso significa que al cargarse este módulo, el módulo complex\_amd (fichero complex_amd.js) debe estar cargado. Si no lo está requirejs lo cargará asincronamente, y cuando esta carga haya finalizado invocará la función que define el módulo. Observa ahora que la función tiene un parámetro. Este parámetro se corresponde **con lo que exporta (devuelve) el módulo complex_amd del cual dependíamos**. Básicamente, por cada elemento (dependencia) del array tendremos un parámetro en la función. Eso se ve todavía más claro en el modulo main\_amd.js quie depende tanto de complex\_amd como de math_amd:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:adc74b07-8191-4359-b4c9-2d3be1d8a081" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#000000">define([</span><span style="background:#ffffff;color:#a31515">'complex_amd'</span><span style="background:#ffffff;color:#000000">, </span><span style="background:#ffffff;color:#a31515">'math_amd'</span><span style="background:#ffffff;color:#000000">], </span><span style="background:#ffffff;color:#0000ff">function</span><span style="background:#ffffff;color:#000000"> (Complex, math) {</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">console.log(math.add(40, 2));</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff">var</span><span style="background:#ffffff;color:#000000"> c1 = </span><span style="background:#ffffff;color:#0000ff">new</span><span style="background:#ffffff;color:#000000"> Complex(40, 3);</span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000">console.log(math.add(c1, 2));</span>
        </li>
        <li>
          <span style="background:#ffffff;color:#000000">});</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Observa como hay dos parámetros en el array de dependencias y por lo tanto la función del módulo recibe dos parámetros. El array indica
  
las dependencias y el parámetro de la función permite acceder a ellas.

Finalmente tan solo nos queda tener un html que cargue primero requirejs y una vez haya terminado, indique a requirejs que cargue el modulo main_amd.js y lo ejecute. Al cargar este módulo, requirejs cargará asíncronamente todas las dependencias. El código del fichero HTML es trivial:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:36c75005-72dd-44b9-bb97-8c4a18189fd8" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #ffffff; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffff;color:#0000ff"><</span><span style="background:#ffffff;color:#800000">!DOCTYPE</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#ff0000">html</span><span style="background:#ffffff;color:#0000ff">></span>
        </li>
        <li>
          <span style="background:#ffffff;color:#0000ff"><</span><span style="background:#ffffff;color:#800000">html</span><span style="background:#ffffff;color:#0000ff">></span>
        </li>
        <li>
          <span style="background:#ffffff;color:#0000ff"><</span><span style="background:#ffffff;color:#800000">head</span><span style="background:#ffffff;color:#0000ff">></span>
        </li>
        <li>
              <span style="background:#ffffff;color:#000000"></span><span style="background:#ffffff;color:#0000ff"><</span><span style="background:#ffffff;color:#800000">title</span><span style="background:#ffffff;color:#0000ff">></</span><span style="background:#ffffff;color:#800000">title</span><span style="background:#ffffff;color:#0000ff">></span>
        </li>
        <li>
          <span style="background:#ffffff;color:#0000ff"></</span><span style="background:#ffffff;color:#800000">head</span><span style="background:#ffffff;color:#0000ff">></span>
        </li>
        <li>
          <span style="background:#ffffff;color:#0000ff"><</span><span style="background:#ffffff;color:#800000">script</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#ff0000">data-main</span><span style="background:#ffffff;color:#0000ff">="main_amd"</span><span style="background:#ffffff;color:#000000"> </span><span style="background:#ffffff;color:#ff0000">src</span><span style="background:#ffffff;color:#0000ff">="bower_components/requirejs/require.js"></</span><span style="background:#ffffff;color:#800000">script</span><span style="background:#ffffff;color:#0000ff">></span>
        </li>
        <li>
          <span style="background:#ffffff;color:#0000ff"><</span><span style="background:#ffffff;color:#800000">body</span><span style="background:#ffffff;color:#0000ff">></span>
        </li>
        <li>
          <span style="background:#ffffff;color:#0000ff"></</span><span style="background:#ffffff;color:#800000">body</span><span style="background:#ffffff;color:#0000ff">></span>
        </li>
        <li>
          <span style="background:#ffffff;color:#0000ff"></</span><span style="background:#ffffff;color:#800000">html</span><span style="background:#ffffff;color:#0000ff">></span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

El escenario “cargar requirejs y una vez haya terminado empieza a cargar el módulo indicado” es tan común que requirejs lo soporta a través del atributo data-main del tag script. Podemos ver en la pestaña network del navegador como realmente se cargan los tres módulos por separado:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_65ACD593.png" width="644" height="182" />][3]

**¿Cuál usar?**   
La verdad es que AMD se acerca mucho más a la filosofía de la web que CommonJS. La carga asíncrona y on-demand es mucho más natural en la web que la carga síncrona que tiene CommonJS. Lo que ocurre es que actualmente solemos siempre crear un bundle de todos nuestros JavaScript, porque sabemos que es más rápido descargarse un solo fichero de 100Ks que 10 ficheros de 10Ks cada uno. Seguro que todos habéis oído que una de las normas básicas de optimizar una página web consiste en minimizar la descarga de ficheros. Los bundles de JavaScript, de CSS, los sprite-sheets y el uso de data-uris van todos por ese camino: cargar un fichero más grande antes que varios de pequeños. Si seguimos esa tónica perdemos la característica de carga on-demand y asíncrona de AMD (porque _antes_ de ejecutar la aplicación hemos tenido que generar ese bundle). La verdad es que cargar los scripts on-demand no es algo que requieran la mayoría de aplicaciones (un escenario sería casos en que una aplicación quiere cargar scripts distintos en función de ciertos datos de ejecución, p. ej. scripts distintos por usuario).

Así parece que, actualmente, no haya una diferencia sustancial entre usar CommonJS y AMD si al final terminamos en un bundle. La cuestión puede reducirse a gustos personales o cantidad de módulos existentes en cada formato (a pesar de que es posible, con poco trabajo, usar módulos CommonJS bajo AMD) pero **HTTP2 puede cambiar eso**. HTTP2 convierte los bundles en no necesarios, ya que mejora el soporte para varias conexiones… bajo ese nueva prisma AMD parece ser una mejor opción que CommonJS. Pero HTTP2 es todavía muy nuevo, así que hasta todos los navegadores y servidores web lo soporten va a pasar algún tiempo… y cuando esté establecido, quizá y solo quizá, la duda de si usar CommonJS o AMD dejará de tenir sentido porque los módulos nativos de ES6 habrán tomado el relevo.

Saludos!

 [1]: http://geeks.ms/blogs/etomas/archive/2011/03/28/html-js-module-pattern.aspx
 [2]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_187A4B57.png
 [3]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_51278615.png