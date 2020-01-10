---
title: 'ASP.NET WebApi: Subida de ficheros'

author: eiximenis

date: 2014-12-02T18:20:52+00:00
geeks_url: /?p=1687
geeks_visits:
  - 945
geeks_ms_views:
  - 1696
categories:
  - Uncategorized

---
Buenas! Vamos a ver en este post como podemos tratar la subida de ficheros en WebApi. 

En <a href="http://geeks.ms/blogs/etomas/archive/2010/09/08/subir-ficheros-al-servidor-en-asp-net-mvc.aspx" target="_blank" rel="noopener noreferrer">ASP.NET MVC la subida de ficheros</a> la gestiona un model binder para el tipo HttpFilePostedBase, por lo que basta con declarar un parámetro de este tipo de datos en el controlador y automáticamente recibimos el fichero subido.

En WebApi el enfoque es muy distinto: en el controlador no recibimos ningún parámetro con el contenido del fichero. En su lugar usamos la clase <a href="http://msdn.microsoft.com/es-es/library/system.net.http.multipartformdatastreamprovider%28v=vs.118%29.aspx" target="_blank" rel="noopener noreferrer">MultipartFormDataStreamProvider</a> para leer el fichero subido y guardarlo en disco (ambas cosas a la vez).

**Anatomía de una petición http con fichero subido**

Antes de nada veamos como es una petición HTTP en la que se suba un fichero. Para ello he creado un HTML como el siguiente:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:4f15de38-9fbe-4079-b9ff-80a22bd59008" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">form</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">method</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"post"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">enctype</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"multipart/form-data"</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">File: </span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">input</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">type</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"file"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">name</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"aFile"</span><span style="background:#1e1e1e;color:#808080">><</span><span style="background:#1e1e1e;color:#569cd6">br</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#808080">/></span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">File: </span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">input</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">type</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"file"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">name</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"aFile"</span><span style="background:#1e1e1e;color:#808080">><</span><span style="background:#1e1e1e;color:#569cd6">br</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#808080">/></span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">input</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">type</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"submit"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">value</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"Submit"</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">form</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Selecciono dos ficheros cualesquiera y capturo la petición generada con fiddler. El resultado (eliminando todo lo que no nos importa) es el siguiente:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:74755595-1e25-4f1b-ae3d-229c3153941a" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">Content-Type: multipart/form-data; boundary=&#8212;-WebKitFormBoundaryQoYjfxGXTHG6DESL</span>
        </li>
        <li style="background: #111111">
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">&#8212;&#8212;WebKitFormBoundaryQoYjfxGXTHG6DESL</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">Content-Disposition: form-data; name="aFile"; filename="jsio.png"</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">Content-Type: image/png</span>
        </li>
        <li style="background: #111111">
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">Contenido binario del fichero</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">&#8212;&#8212;WebKitFormBoundaryQoYjfxGXTHG6DESL</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">Content-Disposition: form-data; name="aFile"; filename="logo_mvp.png"</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">Content-Type: image/png</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">Contenido binario del fichero</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">&#8212;&#8212;WebKitFormBoundaryQoYjfxGXTHG6DESL&#8211;</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Básicamente:

  * El Content-Type debe ser multipart/form-data
  * El Content-Type debe especificar una boundary. La boundary es un cadena que se usa para separar cada valor de la petición (tanto los ficheros como los valores enviados por formdata si los hubiese).
  * Para cada valor:
  * Se coloca el boundary precedido de &#8212;
  * Si es un fichero.
  * se coloca un content-disposition que indica (entre otras cosas) el nombre del fichero
  * El conteido binario del fichero

  * Si no es un fichero (p. ej. es el un formdata que viene de un <input type=text>
  * se coloca un content-disposition que indica el nombre del parámetro
  * Se coloca su valor

  * Finalmente se coloca la boundary para finalizar la petición

<s trong>Enviar peticiones usando HttpClient</strong>

Conociendo como es una petición de subida de ficheros, crearla usando HttpClient es muy simple. El siguiente código sube un fichero:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:cfbe61c6-f1c6-45d4-9360-490209c84498" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> requestContent </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">MultipartFormDataContent</span><span style="background:#1e1e1e;color:#dcdcdc">();</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> imageContent </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">StreamContent</span><span style="background:#1e1e1e;color:#dcdcdc">(stream);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">imageContent</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Headers</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ContentType </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">MediaTypeHeaderValue</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Parse(</span><span style="background:#1e1e1e;color:#d69d85">"image/png"</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">requestContent</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Add(imageContent, </span><span style="background:#1e1e1e;color:#d69d85">"image"</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Format(</span><span style="background:#1e1e1e;color:#d69d85">"{</span><span style="background:#1e1e1e;color:#80ff80">0:00}</span><span style="background:#1e1e1e;color:#d69d85">.png"</span><span style="background:#1e1e1e;color:#dcdcdc">, idx));</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

La variable stream es un Stream para acceder al fichero, mientras que la variable idx es un entero que en este caso se usa para dar nombre al fichero subdido (01.png, 02.png, …).

Si capturamos con fiddler como es la petición generada por este código vemos que es como sigue:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:72640d2d-5d7e-4359-89e6-c3f64d971fed" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">POST http://localtest.me:2706/Upload/Photo/568b8c05-aab8-46db-8cbc-aec2a96dec18/2 HTTP/1.1</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">Content-Type: multipart/form-data; boundary="c609aabb-3872-4d04-a69d-72024c9325a5"</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">&#8211;c609aabb-3872-4d04-a69d-72024c9325a5</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">Content-Type: image/png</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">Content-Disposition: form-data; name=image; filename=02.png; filename*=utf-8''02.png</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Podemos observar como se ha generado un boundary para nosotros (realmente el valor del boundary no se usa, es solo para separar los campos) y como se genera un Content-Disposition. Es pues una petición equivalente a usar un <input type=”file” /> (cuyo atributo name fuese “image”).

**Recibir el fichero en WebApi**

Para recibir el fichero subido, necesitamos una acción de un controlador WebApi y usar un MultipartFormDataStreamProvider para **guardar** el fichero en disco:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:bbac539f-8c8f-4e59-ba54-e7fc33de39d7" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> streamProvider </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">MultipartFileStreamProvider</span><span style="background:#1e1e1e;color:#dcdcdc">(uploadFolder);</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">await</span><span style="background:#1e1e1e;color:#dcdcdc"> Request</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Content</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ReadAsMultipartAsync(streamProvider);</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Este código **ya guarda el fichero en el disco**. La carpeta usada es la especificada en la variable uploadFolder. De hecho **si hubiese varios ficheros subidos a la vez, este código los guarda todos**.

En mi caso he enviado una petición con un Content-Disposition cuyo nombre de fichero es 02.png, así que lo suyo sería esperar que en la carpeta especificada por uploadFolder hubiese este fichero. Pero **no vais a encontrar ningún fichero llamado así. Por diseño WebApi ignora el valor de Content-Disposition** (por temas de seguridad). En su lugar os vais a encontrar con un fichero (o varios) llamados BodyPart y un guid:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blo
gs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_67BB1D71.png" width="309" height="110" />][1]

Por suerte para hacer que WebApi tenga en cuenta el valor del campo Content-Disposition y guarde el fichero con el nombre especificado basta con heredar de MultipartFormDataStreamProvider y reimplementar el método GetLocalFileName:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:51eaf7ac-4816-4a31-a0be-cb2e574b49d1" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">MultipartFormDataContentDispositionStreamProvider</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#4ec9b0">MultipartFormDataStreamProvider</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> MultipartFormDataContentDispositionStreamProvider(</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> rootPath) : </span><span style="background:#1e1e1e;color:#569cd6">base</span><span style="background:#1e1e1e;color:#dcdcdc">(rootPath)</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> MultipartFormDataContentDispositionStreamProvider(</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> rootPath, </span><span style="background:#1e1e1e;color:#569cd6">int</span><span style="background:#1e1e1e;color:#dcdcdc"> bufferSize) : </span><span style="background:#1e1e1e;color:#569cd6">base</span><span style="background:#1e1e1e;color:#dcdcdc">(rootPath, bufferSize)</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">override</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> GetLocalFileName(</span><span style="background:#1e1e1e;color:#4ec9b0">HttpContentHeaders</span><span style="background:#1e1e1e;color:#dcdcdc"> headers)</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (headers</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ContentDisposition </span><span style="background:#1e1e1e;color:#b4b4b4">!=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc">)</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> headers</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ContentDisposition</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">FileName;</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">base</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">GetLocalFileName(headers);</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Ahora en el controlador instanciamos un objeto MultipartFormDataContentDispositionStreamProvider en lugar del MultipartFormDataStreamProvider y ahora ya se nos guardarán los ficheros con los nombres especificados. Ojo, recuerda que WebApi no hace eso por defecto por temas de seguridad, así que si implementas esta solución valida los nombres de fichero que te envía el cliente.

¡Y ya está! La verdad es que el modelo de WebApi es radicalmente distinto al de ASP.NET MVC pero igual de sencillo y efectivo 😉

Saludos!

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_34C8FC59.png