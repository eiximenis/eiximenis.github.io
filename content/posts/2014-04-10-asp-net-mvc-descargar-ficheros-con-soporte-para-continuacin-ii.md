---
title: ASP.NET MVC–Descargar ficheros con soporte para continuación – ii
author: eiximenis

date: 2014-04-10T11:11:31+00:00
geeks_url: /?p=1665
geeks_visits:
  - 985
geeks_ms_views:
  - 1084
categories:
  - Uncategorized

---
<a href="http://geeks.ms/blogs/etomas/archive/2014/04/10/asp-net-mvc-descargar-ficheros-con-soporte-para-continuaci-243-n-i.aspx" target="_blank" rel="noopener noreferrer">En el post anterior</a> vimos todo la parte teórica de HTTP que nos permite realizar descargas de ficheros, pausarlas y continuarlas. Pero ahora viene lo bueno… Vamos a implementar el soporte en **el servidor** para soportar dichas descargas.

Dado que nuestro amado FileStreamResult no soporta la cabecera Range, nos va a tocar a nostros hacer todo el trabajo. Pero, la verdad… no hay para tanto.

> **Nota importante: Todo, absolutamente todo el código que pongo en este blog está para que hagas con él lo que quieras. Pero del mismo modo, el código que hay en este blog NO ES CÓDIGO DE PRODUCCIÓN. En muchos casos no está suficientemente probado y en otros, para simplificar, no se tienen en cuenta todas las casuísticas o no hay apenas control de errores. Si quieres hacer copy/paste eres totalmente libre de hacerlo, pero revisa luego el código. Entiéndelo y hazlo tuyo.**

Bien, el primer paso va a ser crearnos un ActionResult nuevo, en este caso una clase llamada RangeFileActionResult y que herederá pues de ActionResult. En dicho ActionResult vamos a obtener el valor del campo Range y a parsearlo. Empecemos por el constructor:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:5a73e343-3f11-4295-8d14-dbbbb0bcd328" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> RangeFileActionResult(</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> filename, </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> contentType)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">_stream </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">FileStream</span><span style="background:#1e1e1e;color:#dcdcdc">(filename, </span><span style="background:#1e1e1e;color:#b8d7a3">FileMode</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Open, </span><span style="background:#1e1e1e;color:#b8d7a3">FileAccess</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Read);</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">_length </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> _stream</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Length;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">_contentType </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> contentType;</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">byte</span><span style="background:#1e1e1e;color:#dcdcdc">[] hash;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">using</span><span style="background:#1e1e1e;color:#dcdcdc"> (</span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> md5 </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">MD5</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Create())</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">hash </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> md5</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ComputeHash(_stream);</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">_etag </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Convert</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ToBase64String(hash);</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">_stream</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Seek(</span><span style="background:#1e1e1e;color:#b5cea8"></span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#b8d7a3">SeekOrigin</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Begin);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Siplemente nos guardamos el stream al fichero y calculamos el MD5. Vamos a usar el MD5 como ETag del fichero.

Ahora, el siguiente paso es implementar el método ExecuteResult, que es donde se procesa toda la petición:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:ac5a41e0-98c4-4edf-9fcb-bd4e9c206171" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">override</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">void</span><span style="background:#1e1e1e;color:#dcdcdc"> ExecuteResult(</span><span style="background:#1e1e1e;color:#4ec9b0">ControllerContext</span><span style="background:#1e1e1e;color:#dcdcdc"> context)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">using</span><span style="background:#1e1e1e;color:#dcdcdc"> (_stream)</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> ranges </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> ParseRangeHeader(context</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">HttpContext</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Request);</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="backgr
ound:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (ranges </span><span style="background:#1e1e1e;color:#b4b4b4">==</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">||</span><span style="background:#1e1e1e;color:#dcdcdc"> ranges</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Empty)</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">FileStreamResult</span><span style="background:#1e1e1e;color:#dcdcdc">(_stream, _contentType)</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ExecuteResult(context);</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (ranges</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Multiple)</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc">ProcessMultipleRanges(context, ranges);</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">else</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc">ProcessSingleRange(context, ranges</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Values</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Single());</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Básicamente:

  * Miramos el valor de la cabecera Range. Si no existe o no lo sabemos interpretar, creamos un FileStreamResult tradicional y lo ejecutamos. Es decir, nos saltamos todo lo de los rangos y devolvemos una descarga de fichero tradicional. 
  * En caso de que haya más de un rango vamos a generar la respuesta multipart, con el método ProcessMultipleRanges. 
  * En el caso de que haya un solo rango vamos a generar la respuesta con el método ProcessSingleRange. 

El método ParseRangeHeader se encarga de parsear el valor de la cabecera Range:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:163de03a-7ddd-4722-97d2-cb137e7b0bf2" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Ranges</span><span style="background:#1e1e1e;color:#dcdcdc"> ParseRangeHeader(</span><span style="background:#1e1e1e;color:#4ec9b0">HttpRequestBase</span><span style="background:#1e1e1e;color:#dcdcdc"> request)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> rangeHeader </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> request</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Headers[</span><span style="background:#1e1e1e;color:#d69d85">"Range"</span><span style="background:#1e1e1e;color:#dcdcdc">];</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">IsNullOrEmpty(rangeHeader)) </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">rangeHeader </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> rangeHeader</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Trim();</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (</span><span style="background:#1e1e1e;color:#b4b4b4">!</span><span style="background:#1e1e1e;color:#dcdcdc">rangeHeader</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">StartsWith(</span><span style="background:#1e1e1e;color:#d69d85">"bytes="</span><span style="background:#1e1e1e;color:#dcdcdc">))</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Ranges</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">InvalidUnit();</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Ranges</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">FromBytesUnit(rangeHeader</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Substring(</span><span style="background:#1e1e1e;color:#d69d85">"bytes="</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Length));</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Se apoya en unas clases llamadas Ranges (que es básicamente una coleccion de objetos Range) y Range que representa un rango (origen, destino). El método importante de Ranges es el FromBytesUnit que es el que realmente parsea una cadena (sin el prefijo bytes=):

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:41a887aa-ee72-4708-93c2-4e0796555df3" class="wlWriter
EditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  </p> 
  
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">static</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Ranges</span><span style="background:#1e1e1e;color:#dcdcdc"> FromBytesUnit(</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> value)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> tokens </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> value</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Split(</span><span style="background:#1e1e1e;color:#d69d85">','</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> ranges </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Ranges</span><span style="background:#1e1e1e;color:#dcdcdc">();</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">foreach</span><span style="background:#1e1e1e;color:#dcdcdc"> (</span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> token </span><span style="background:#1e1e1e;color:#569cd6">in</span><span style="background:#1e1e1e;color:#dcdcdc"> tokens)</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">ranges</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Add(</span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Range</span><span style="background:#1e1e1e;color:#dcdcdc">(token));</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> ranges;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Y la otra parte del trabajo la realiza el constructor de Range. Así si tenemos la cadena “0-100, 101-200” se llamará dos veces al constructor de Range pasándole primero la cadena “0-100” y luego “101-200”.

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:64669116-4a53-426f-b830-3d3827d7e8a0" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> Range(</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> value)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (</span><span style="background:#1e1e1e;color:#b4b4b4">!</span><span style="background:#1e1e1e;color:#dcdcdc">value</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Contains(</span><span style="background:#1e1e1e;color:#d69d85">"-"</span><span style="background:#1e1e1e;color:#dcdcdc">))</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">_from </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">long</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Parse(value);</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">_to </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">&#8211;</span><span style="background:#1e1e1e;color:#b5cea8">1</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">else</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (value</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">StartsWith(</span><span style="background:#1e1e1e;color:#d69d85">"-"</span><span style="background:#1e1e1e;color:#dcdcdc">))</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">_from </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">&#8211;</span><span style="background:#1e1e1e;color:#b5cea8">1</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">_to </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">long</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Parse(value</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Substring(</span><span style="background:#1e1e1e;color:#b5cea8">1</span><span style="background:#1e1e1e;color:#dcdcdc">));</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">else</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> idx </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> value</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">IndexOf(</span><span style="background:#1e1e1e;color:#d69d85">&#39<br /> ;-'</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">_from </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">long</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Parse(value</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Substring(</span><span style="background:#1e1e1e;color:#b5cea8"></span><span style="background:#1e1e1e;color:#dcdcdc">, idx));</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">_to </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> idx </span><span style="background:#1e1e1e;color:#b4b4b4">==</span><span style="background:#1e1e1e;color:#dcdcdc"> value</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Length </span><span style="background:#1e1e1e;color:#b4b4b4">&#8211;</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b5cea8">1</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">?</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">&#8211;</span><span style="background:#1e1e1e;color:#b5cea8">1</span><span style="background:#1e1e1e;color:#dcdcdc"> :</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">long</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Parse(value</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Substring(idx </span><span style="background:#1e1e1e;color:#b4b4b4">+</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b5cea8">1</span><span style="background:#1e1e1e;color:#dcdcdc">));</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

**Sirviendo rangos**

Empecemos por el caso más sencillo: Tenemos un solo rango. En este caso el método ProcessSingleRange toma el control:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:ac2eda0f-ee68-400c-a493-9941f18be71b" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">void</span><span style="background:#1e1e1e;color:#dcdcdc"> ProcessSingleRange(</span><span style="background:#1e1e1e;color:#4ec9b0">ControllerContext</span><span style="background:#1e1e1e;color:#dcdcdc"> context, </span><span style="background:#1e1e1e;color:#4ec9b0">Range</span><span style="background:#1e1e1e;color:#dcdcdc"> range)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> response </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> context</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">HttpContext</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Response;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">response</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">StatusCode </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b5cea8">206</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">response</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">AddHeader(</span><span style="background:#1e1e1e;color:#d69d85">"Content-Range"</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">ContentRange</span><span style="background:#1e1e1e;color:#dcdcdc">(range, _length)</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ToString());</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">response</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">AddHeader(</span><span style="background:#1e1e1e;color:#d69d85">"ETag"</span><span style="background:#1e1e1e;color:#dcdcdc">, _etag);</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">response</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ContentType </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> _contentType;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">FlushRangeDataInResponse(range, response);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Este método es muy sencillo. Establece el código de respuesta (206) y crea las cabeceras Content-Range, ETag y Content-Type. Luego llama al método FlushRangeDataInResponse. Dicho método va leyendo los bytes del fichero y los va escribiendo en el buffer de respuesta. Para evitar cargar todo el rango en memoria del servidor (un rango puede ser todo lo largo que desee el cliente), los datos se van leyendo en bloques de 1024 bytes y se van escribiendo en el buffer de salida:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:aabd1632-52e6-4f71-ad3d-6f3f89d86274" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">void</span><span style="background:#1e1e1e;color:#dcdcdc"> FlushRangeDataInResponse(</span><span style="background:#1e1e1e;color:#4ec9b0">Range</span><span style="background:#1e1e1e;color:#dcdcdc"> range, </span><span style="background:#1e1e1e;color:#4ec9b0">HttpResponseBase</span><span style="background:#1e1e1e;color:#dcdcdc"> response)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> creader </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc
"> </span><span style="background:#1e1e1e;color:#4ec9b0">ChunkReader</span><span style="background:#1e1e1e;color:#dcdcdc">(_stream);</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#4ec9b0">ChunkResult</span><span style="background:#1e1e1e;color:#dcdcdc"> result </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> startpos </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b5cea8"></span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">do</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">result </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> creader</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">GetBytesChunk(range, startpos);</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">startpos </span><span style="background:#1e1e1e;color:#b4b4b4">+=</span><span style="background:#1e1e1e;color:#dcdcdc"> result</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">BytesRead;</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (result</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">BytesRead </span><span style="background:#1e1e1e;color:#b4b4b4">></span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b5cea8"></span><span style="background:#1e1e1e;color:#dcdcdc">)</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc">response</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">OutputStream</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Write(result</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Data, </span><span style="background:#1e1e1e;color:#b5cea8"></span><span style="background:#1e1e1e;color:#dcdcdc">, result</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">BytesRead);</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">response</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Flush();</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">} </span><span style="background:#1e1e1e;color:#569cd6">while</span><span style="background:#1e1e1e;color:#dcdcdc"> (result</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">MoreToRead);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Aquí el que hace el trabajo de verdad es la clase ChunkReader. Esta clase es la que va leyendo un stream, por “trozos” y devuelve después de cada trozo si hay “más trozos por leer”:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:374cc3ab-af4a-47ec-9a7a-cd105290e853" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">ChunkReader</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">readonly</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Stream</span><span style="background:#1e1e1e;color:#dcdcdc"> _stream;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> ChunkReader(</span><span style="background:#1e1e1e;color:#4ec9b0">Stream</span><span style="background:#1e1e1e;color:#dcdcdc"> stream)</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">_stream </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> stream;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          &nbsp;
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">ChunkResult</span><span style="background:#1e1e1e;color:#dcdcdc"> GetBytesChunk(</span><span style="background:#1e1e1e;color:#4ec9b0">Range</span><span style="background:#1e1e1e;color:#dcdcdc"> range, </span><span style="background:#1e1e1e;color:#569cd6">int</span><span style="background:#1e1e1e;color:#dcdcdc"> startpos)</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> chunk </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">ChunkResult</span><span style="background:#1e1e1e;color:#dcdcdc">();</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> reader </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">BinaryReader</span><span style="background:#1e1e1e;color:#dcdcdc">(_stream);</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> remainingLen </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> range</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style
="background:#1e1e1e;color:#dcdcdc">Length </span><span style="background:#1e1e1e;color:#b4b4b4">!=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">&#8211;</span><span style="background:#1e1e1e;color:#b5cea8">1</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">?</span><span style="background:#1e1e1e;color:#dcdcdc"> range</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Length </span><span style="background:#1e1e1e;color:#b4b4b4">&#8211;</span><span style="background:#1e1e1e;color:#dcdcdc"> startpos : </span><span style="background:#1e1e1e;color:#b4b4b4">&#8211;</span><span style="background:#1e1e1e;color:#b5cea8">1</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (remainingLen </span><span style="background:#1e1e1e;color:#b4b4b4">==</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b5cea8"></span><span style="background:#1e1e1e;color:#dcdcdc">)</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">ChunkResult</span><span style="background:#1e1e1e;color:#dcdcdc">();</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
                          <span style="background:#1e1e1e;color:#dcdcdc"></span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> bytesWanted </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> remainingLen </span><span style="background:#1e1e1e;color:#b4b4b4">!=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">&#8211;</span><span style="background:#1e1e1e;color:#b5cea8">1</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">?</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Math</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Min(</span><span style="background:#1e1e1e;color:#b5cea8">1024</span><span style="background:#1e1e1e;color:#dcdcdc">, remainingLen) : </span><span style="background:#1e1e1e;color:#b5cea8">1024</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">reader</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">BaseStream</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Seek(range</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">FromBegin </span><span style="background:#1e1e1e;color:#b4b4b4">?</span><span style="background:#1e1e1e;color:#dcdcdc"> startpos : range</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">From </span><span style="background:#1e1e1e;color:#b4b4b4">+</span><span style="background:#1e1e1e;color:#dcdcdc"> startpos, </span><span style="background:#1e1e1e;color:#b8d7a3">SeekOrigin</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Begin);</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> buffer </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">byte</span><span style="background:#1e1e1e;color:#dcdcdc">[bytesWanted];</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">chunk</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">BytesRead </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> reader</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Read(buffer, </span><span style="background:#1e1e1e;color:#b5cea8"></span><span style="background:#1e1e1e;color:#dcdcdc">, (</span><span style="background:#1e1e1e;color:#569cd6">int</span><span style="background:#1e1e1e;color:#dcdcdc">)bytesWanted);</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">chunk</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Data </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> buffer;</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">chunk</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">MoreToRead </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> remainingLen </span><span style="background:#1e1e1e;color:#b4b4b4">!=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">&#8211;</span><span style="background:#1e1e1e;color:#b5cea8">1</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#b4b4b4">?</span><span style="background:#1e1e1e;color:#dcdcdc"> chunk</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">BytesRead </span><span style="background:#1e1e1e;color:#b4b4b4">!=</span><span style="background:#1e1e1e;color:#dcdcdc"> remainingLen</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc">: chunk</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">BytesRead </span><span style="background:#1e1e1e;color:#b4b4b4">!=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b5cea8"></span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> chunk;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

El objeto ChunkResult contiene los bytes leídos, el número real de bytes leídos y un booleano que indica si hay “más datos” que leer.

Vayamos ahora al soporte para múltiples rangos. La idea es exactamente la misma, salvo que entre cada rango hay que generar el _boundary_ correspondiente en la respuesta. Y eso es exactamente lo que hace el método ProcessMultipleRanges:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:e38dba35-69a0-4dc4-a11b-9e414203f32c" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace
; font-size: 10pt">
    </p> 
    
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">void</span><span style="background:#1e1e1e;color:#dcdcdc"> ProcessMultipleRanges(</span><span style="background:#1e1e1e;color:#4ec9b0">ControllerContext</span><span style="background:#1e1e1e;color:#dcdcdc"> context, </span><span style="background:#1e1e1e;color:#4ec9b0">Ranges</span><span style="background:#1e1e1e;color:#dcdcdc"> ranges)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> response </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> context</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">HttpContext</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Response;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">response</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">StatusCode </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b5cea8">206</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">response</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">AddHeader(</span><span style="background:#1e1e1e;color:#d69d85">"ETag"</span><span style="background:#1e1e1e;color:#dcdcdc">, _etag);</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">response</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">AddHeader(</span><span style="background:#1e1e1e;color:#d69d85">"Content-Type"</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#d69d85">"multipart/byteranges; boundary=THIS_STRING_SEPARATES"</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">foreach</span><span style="background:#1e1e1e;color:#dcdcdc"> (</span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> range </span><span style="background:#1e1e1e;color:#569cd6">in</span><span style="background:#1e1e1e;color:#dcdcdc"> ranges</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Values)</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">AddRangeInMultipartResponse(context, range);</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Primero añadimos los campos comunes (es decir el código de respuesta, el ETagm el content-type con el boundary). Y luego para cada rango llamamos al método AddRageInMultipartResponse, que simplemente coloca el _boundary,_ luego el content-range y el content-type correspondiente y finalmente volca los datos del rango en el buffer de respuesta:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:b3414ae9-17b6-41df-bbad-52c228dd49a5" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">void</span><span style="background:#1e1e1e;color:#dcdcdc"> AddRangeInMultipartResponse(</span><span style="background:#1e1e1e;color:#4ec9b0">ControllerContext</span><span style="background:#1e1e1e;color:#dcdcdc"> context, </span><span style="background:#1e1e1e;color:#4ec9b0">Range</span><span style="background:#1e1e1e;color:#dcdcdc"> range)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> response </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> context</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">HttpContext</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Response;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">response</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Write(</span><span style="background:#1e1e1e;color:#d69d85">"&#8211; THIS STRING SEPARATES&#92;x0D&#92;x0A"</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">response</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Write(</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Format(</span><span style="background:#1e1e1e;color:#d69d85">"Content-Type: {</span><span style="background:#1e1e1e;color:#80ff80">0}&#92;</span><span style="background:#1e1e1e;color:#d69d85">x0D&#92;x0A"</span><span style="background:#1e1e1e;color:#dcdcdc">, _contentType));</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> contentRange </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">ContentRange</span><span style="background:#1e1e1e;color:#dcdcdc">(range, _length);</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (contentRange</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">IsValid)</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">response</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Write(</span><span style="background:#1e1e1e;color:#d69d85">"Content-Range: "</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">+</span><span style="background:#1e1e1e;color:#dcdcdc"> contentRange </span><span style="background:#1e1e1e;color:#b4b4b4">+</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"&#92;x0D&#92;x0A&#92;x0D&#92;x0A"</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
              <span style="back
ground:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">FlushRangeDataInResponse(range, response);</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">response</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Write(</span><span style="background:#1e1e1e;color:#d69d85">"&#92;x0D&#92;x0A"</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

¡Y ya estamos! Algunos ejemplos de lo que vemos. La imagen de la izquierda contiene un solo rango y la de la derecha dos:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_495B638B.png" width="244" height="95" />][1][<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_4CFC0F19.png" width="244" height="125" />][2]

Con esto hemos visto como añadir soporte para rangos desde el servidor. Por supuesto no está del todo pulido, faltaría añadir el soporte para el If-Range pero bueno… los mimbres vendrían a ser eso.

> **Nota**: Si lo pruebas y colocas un valor de Range inválido (p. ej. Range: bytes=100) recibirás un HTTP 400. Este 400 es generado por IIS incluso antes de que la petición llegue a ASP.NET MVC.

Saludos!

**He subido todo el código del POST en un repositorio de GitHub: [https://github.com/eiximenis/PartialDownloads][3].** La aplicación web de demo simplemente tiene el siguiente código en Home/Index:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:caa7bdce-0ec3-42d4-a5d1-ff9cc63f3410" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">ActionResult</span><span style="background:#1e1e1e;color:#dcdcdc"> Index()</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">RangeFile(</span><span style="background:#1e1e1e;color:#d69d85">"~&#92;&#92;Content&#92;&#92;test.png"</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#d69d85">"image/png"</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Por lo tanto si navegas con un navegador a /Home/Index deberás ver o descargarte la imagen entera. Pero usando fiddler o cURL puedes generar una petición con rangos para ver como funciona.

Para usar cURL te basta con la opción &#8211;header:

curl &#8211;header "Range: bytes=0-100" http://localhost:39841/

Saludos!

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_4A3138C4.png
 [2]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_6B46C60F.png
 [3]: https://github.com/eiximenis/PartialDownloads "https://github.com/eiximenis/PartialDownloads"