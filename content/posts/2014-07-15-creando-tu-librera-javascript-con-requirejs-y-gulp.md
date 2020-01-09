---
title: Creando tu librería JavaScript con requirejs y gulp
description: Creando tu librería JavaScript con requirejs y gulp
author: eiximenis

date: 2014-07-15T11:14:00+00:00
geeks_url: /?p=1677
geeks_visits:
  - 2075
geeks_ms_views:
  - 1747
categories:
  - Uncategorized

---
Si desarrollas aplicaciones web con alta carga de código en cliente, es posible que tengas que desarrollarte tus propias funciones JavaScript. Incluso tu propia librería o framework si es el caso.

Una opción es abrir tu editor de texto favorito, crear un archivo .js y empezar a teclear. Total, jQuery viene en un solo archivo .js, ¿verdad? Antes de hacer esto, párate a pensar un poco: ¿a qué nunca colocarías todo el código c# de tu proyecto en un solo fichero .cs? Tenerlo en varios ficheros permite localizar el código más rápidamente, tenerlo mejor organizado y evitar conflictos cuando se trabaja en equipo. Pues bien, eso mismo aplica a JavaScript.

Por supuesto los desarrolladores de jQuery (y de cualquier otra librería decente) no trabajan en un solo fichero. Tienen su código distribuido en ficheros separados que se combinan al final para crear la librería. Vamos a ver como podemos hacer esto de forma sencilla.

Lo primero que tenemos que hacer es precisamente definir los **módulos** de nuestra librería. Básicamente un módulo es un fichero .js, que define un conjunto de funciones que son accesibles a todos los clientes que usen el módulo. El código de dichos ficheros se organiza siguiendo un patrón que se conoce precisamente como patrón de módulo (<a target="_blank" href="/blogs/etomas/archive/2011/03/28/html-js-module-pattern.aspx" rel="noopener noreferrer">revealing module pattern</a> o alguna de sus variantes).

El problema está que en JavaScript no hay (de momento) ninguna manera de establecer las dependencias entre módulos: es decir, de indicar que el módulo A, depende del módulo B (lo que quiere decir que el módulo B debe estar cargad antes que el módulo A). Es ahí donde entra una <a target="_blank" href="http://requirejs.org/" rel="noopener noreferrer">librería llamada requirejs</a>. Dicha librería permite especificar las dependencias entre módulos. Veamos un ejemplo de módulo con soporte para require:

<div style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px" class="wlWriterEditableSmartContent" id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:05c85aaa-bfc5-4a07-84ea-d06186db11e1">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;" start="1">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">define</span><span style="background:#1e1e1e;color:#b4b4b4">([],</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li style="background: #111111">
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> rnd </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b5cea8">42</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li style="background: #111111">
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li style="background: #111111">
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> isOdd </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">i</span><span style="background:#1e1e1e;color:#b4b4b4">)</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">i </span><span style="background:#1e1e1e;color:#b4b4b4">%</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b5cea8">2</span><span style="background:#1e1e1e;color:#b4b4b4">)</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">!=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b5cea8"></span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li style="background: #111111">
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li style="background: #111111">
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">rnd</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> rnd</span><span style="background:#1e1e1e;color:#b4b4b4">,</span>
        </li>
        <li style="background: #111111">
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">isOdd</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> isOdd</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li style="background: #111111">
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Lo importante ahí es la funcion **define**. Dicha función está definida en requirejs y suele tomar dos parámetros:

  1. Un array con **las dependencias del módulo**.
  2. Una función con el código del módulo. Dicha función puede recibir como parámetro los módulos especificados en las dependencias.

El módulo define dos funciones (rnd y isOdd) y luego **las exporta**. Exportarlas significa que las hace visibles al resto de módulos que dependan de este. La forma de exportar es devolviendo un resultado: todo lo que se devuelve, es exportado.

Ahora vamos a declarar otro módulo (llamémosle main.js) que hará uso de este módulo que hemos definido:

<div style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px" class="wlWriterEditableSmartContent" id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:b2a65d20-c946-414a-817b-3857531b18f8">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;" start="1">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">define</span><span style="background:#1e1e1e;color:#b4b4b4">([</span><span style="background:#1e1e1e;color:#d69d85">&#8216;math/rnd&#8217;</span><span style="background:#1e1e1e;color:#b4b4b4">],</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">math</span><span style="background:#1e1e1e;color:#b4b4b4">)</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li style="background: #111111">
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> addOnlyIfOdd </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">a</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#dcdcdc"> b</span><span style="background:#1e1e1e;color:#b4b4b4">)</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> result </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b5cea8"></span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li style="background: #111111">
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">math</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">isOdd</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">a</span><span style="background:#1e1e1e;color:#b4b4b4">))</span><span style="background:#1e1e1e;color:#dcdcdc"> result </span><span style="background:#1e1e1e;color:#b4b4b4">+=</span><span style="background:#1e1e1e;color:#dcdcdc"> a</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">math</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">isOdd</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">b</span><span style="background:#1e1e1e;color:#b4b4b4">))</span><span style="background:#1e1e1e;color:#dcdcdc"> result </span><span style="background:#1e1e1e;color:#b4b4b4">+=</span><span style="background:#1e1e1e;color:#dcdcdc"> b</span>
        </li>
        <li style="background: #111111">
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> result</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li style="background: #111111">
          &nbsp;
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li style="background: #111111">
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">addOnlyIfOdd</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> addOnlyIfOdd</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Es lo mismo que antes con la diferencia de que ahora el array de dependencias contiene el módulo (math/rnd). Este es nuestro módulo anterior. **El nombre de un módulo es el nombre del fichero js tal cual (incluyendo la ruta)** (existen los llamados módulos AMD donde esto no tiene porque ser así, pero tampoco es relevante ahora). Por lo tanto, en este código, el módulo anterior lo habíamos guardado en math/rnd.js Dentro de este módulo podemos usar todo lo que el módulo anterior exportaba a través del parámetro math.

¿Como usaríamos eso desde una página web?

Lo primero es cargar requirejs desde la página web y usar data-main para indicarle a require el módulo &ldquo;inicial&rdquo;:

<div style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px" class="wlWriterEditableSmartContent" id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:5020f150-990b-4fc8-bbee-48ba82831c5f">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;" start="1">
        <li>
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">!DOCTYPE</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">html</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">head</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">title</span><span style="background:#1e1e1e;color:#808080">></span><span style="background:#1e1e1e;color:#dcdcdc">Demo</span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">title</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li style="background: #111111">
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">script</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">data-main</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">&#8220;./main.js&#8221;</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">src</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">&#8220;http://requirejs.org/docs/release/2.1.14/minified/require.js&#8221;</span><span style="background:#1e1e1e;color:#808080">></</span><span style="background:#1e1e1e;color:#569cd6">script</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">head</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">body</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">body</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
      </ol>
    </div>
  </div>
</div>

Si ejecutamos este fichero html y miramos la pestaña Network veremos como no solo se carga el fichero requirejs si no que tanto main.js como también math/rnd.js se cargan: require ha cargado tanto el módulo inicial como todas sus dependencias.

Esto nos permite dividir nuestro código JavaScript con la tranquilidad de que luego los scripts se cargarán en el orden correcto, gracias a requirejs.

Vale, pero estarás diciendo que jQuery viene en un solo js único y para cargar jQuery no debes usar require para nada. Y tienes razón. La realidad es que la gente de jQuery (y tantos otros frameworks) tienen su código dividido en módulos pero luego **los unen todos en el orden correcto en un solo js**. Y eso lo hacen cuando &ldquo;construyen&rdquo; jQuery. Vamos a ver un ejemplo usando <a target="_blank" href="http://gulpjs.com/" rel="noopener noreferrer">gulp</a> (en jQuery se usa grunt que es similar).

Gulp es un **sistema de build basado en JavaScript**. Se basa en <a target="_blank" href="http://nodejs.org/" rel="noopener noreferrer">node</a> (así que es necesario tener node instalado). Pero teniendo node instalado, instalar gulp es seguir dos pasos:

  1. Ejecutar npm install &ndash;g gulp. Eso instala gulp a nivel &ldquo;global&rdquo;. Debe ejecutarse una sola vez.
  2. Ejecutar npm install &#8211;save-dev gulp. Eso debe ejecutarse desde el directorio donde tenemos el código fuente de nuestra librería. Eso instala gulp a nivel local y debe hacerse una vez por cada librería instalada.

El tercer paso es crear un fichero javascript, llamado gulpfile.js que será el que tenga las distintas tareas para generar nuestro proyecto JavaScript.

Para el ejemplo vamos a suponer que tenemos nuestro fichero main.js y el fichero math/rnd.js dentro de una subcarpeta src. El fichero gulpfile.js está en la carpeta que contiene a /src. Es decir tenemos una estructura tal que así:

[<img height="177" width="140" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_11B73C68.png" alt="image" border="0" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; border-left: 0px; display: inline; padding-right: 0px" title="image" />][1]

Ahora vamos a crear una tarea en gulp para combinar todos los módulos de nuestro proyecto en un .js final. Para ello tenemos que instalar requirejs como módulo de node mediante npm install &#8211;save-dev requirejs (desde el directorio de nuestro proyecto).

Finalmente podemos crear una tarea, llamada &ldquo;make&rdquo; en nuestro gulpfile:

<div style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px" class="wlWriterEditableSmartContent" id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:586bc996-c5c2-4042-86a1-fb21af83358f">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;" start="1">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> gulp </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> require</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#d69d85">&#8216;gulp&#8217;</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> requirejs </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> require</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#d69d85">&#8216;requirejs&#8217;</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">gulp</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">task</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#d69d85">&#8216;make&#8217;</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li style="background: #111111">
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> config </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">baseUrl</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">&#8216;src&#8217;</span><span style="background:#1e1e1e;color:#b4b4b4">,</span>
        </li>
        <li style="background: #111111">
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">name</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">&#8216;main&#8217;</span><span style="background:#1e1e1e;color:#b4b4b4">,</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">out</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">&#8216;dist/demo.js&#8217;</span><span style="background:#1e1e1e;color:#b4b4b4">,</span>
        </li>
        <li style="background: #111111">
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">findNestedDependencies</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">true</span><span style="background:#1e1e1e;color:#b4b4b4">,</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">skipSemiColonInsertion</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">true</span><span style="background:#1e1e1e;color:#b4b4b4">,</span>
        </li>
        <li style="background: #111111">
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">optimize</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">&#8216;none&#8217;</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li style="background: #111111">
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">requirejs</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">optimize</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">config</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">buildResponse</span><span style="background:#1e1e1e;color:#b4b4b4">)</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">buildResponse</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li style="background: #111111">
          &nbsp;&nbsp;&nbsp;&nbsp;<span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
      </ol>
    </div>
  </div>
</div>

Cargamos los módulos de node que vamos a usar (gulp y require) y luego usamos requirejs.optimize para cargar el módulo incial, junto con todas sus dependencias y generar un solo .js. Esto es precisamente lo que hace la llamada a requirejs.optimize. El primer parámetro es la configuración. <a target="_blank" href="https://github.com/jrburke/r.js/blob/master/build/example.build.js" rel="noopener noreferrer">Existen muchas opciones de configuración</a>, siendo las más importantes:

  1. baseUrl: Directorio base donde están los módulos
  2. name: Módulo inicial
  3. out: Fichero de salida
  4. optimize: Si debe minimificar el archivo de salida o no.

Una vez hemos modificado el fichero gulpfile.js ya podemos ejecutarlo, mediante **gulp make**. Esto invocará la tarea &lsquo;make&rsquo; del fichero gulpfile:

[<img height="121" width="504" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_45EB65AE.png" alt="image" border="0" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" title="image" />][2]

Una vez ejecutado en el directorio dist tendremos un fichero (demo.js) con todos nuestros módulos combinados en el orden correcto. Este sería el fichero que distribuiríamos y que se usaría desde el navegador.

Eso nos permite tener nuestro código javascript bien separado y organizado, con la tranquilidad de que siempre podemos generar la versión unificada &ldquo;final&rdquo; usando gulp.

Por supuesto gulp sirve para mucho más que combinar los módulos. Puede minimificar salidas, pasar tests unitarios, realizar optimizaciones adicionales, tareas propias... en fin, cualquier cosa que uno esperaría de un sistema de builds.

&iexcl;Un saludo!

 [1]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_1D4CF9A7.png
 [2]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_0D40BBA1.png