---
title: Cifrado en WebApi
author: eiximenis

date: 2014-07-01T13:27:29+00:00
geeks_url: /?p=1674
geeks_visits:
  - 1256
geeks_ms_views:
  - 2208
categories:
  - Uncategorized

---
Muy buenas! El otro día recibí el siguiente correo (a través del formulario de contacto del blog): 

> _Estoy desarrollando una serie de web apis para transacciones con tarjetas de crédito. Mi problema es que no encuentro la forma de cifrar los datos sensibles como el numero de tarjeta de crédito. con un web service esta claro como hacerlos pero no encuentro la forma en una web api. que me recomendas?_

Es un tema interesante y que da para mucho pero a ver si podemos **dar cuatro pinceladas**…

Antes de nada lo dicho en este post **trata de evitar que alguien que use un sniffer para analizar el tráfico de red pueda ver los datos confidenciales que enviamos**. Este post no pretende ni puede ser una solución completa al problema, ya que hay muchas casuísticas que se deben tratar y muchos tipos de ataque a los que debemos responder. Honestamente **la solución más sencilla pasa por usar HTTPS**. Si usas HTTPS delegas toda la seguridad en el canal. Sin duda es lo más sencillo. **No hay que hacer nada especial para poder usar HTTPS en WebApi.** Si usas IIS Express para desarrollar (lo habitual con VS) habilitar el soporte para HTTPS está a un click de distancia:

[<img title="SNAGHTML5d216779" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="SNAGHTML5d216779" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/SNAGHTML5d216779_5F00_thumb_5F00_500C358C.png" width="244" height="235" />][1]

Al seleccionar SSL Enabled VS te indica tanto la URL http como la https (con otro puerto claro). Para que todo funcione IIS Express genera un certificado SSL de desarrollo y ya tienes HTTPS habilitado. Por supuesto cuando despliegues en producción deberás desplegar el certificado SSL de producción en IIS. 

**Conceptos básicos de cifrado**

Si no quieres usar IIS entonces debes encriptar los datos en el cliente y desencriptarlos en el servidor. Eso indica que debes especificar que método de encriptación se usa… hay muchos pero a grandes rasgos se dividen en dos grandes grupos:

  1. De clave simétrica
  2. De clave asimétrica

En los métodos de clave simétrica, cliente y servidor **comparten una clave.** Dicha clave es usada para generar texto cifrado a partir de texto plano… y viceversa. Es decir alguien que conozca la clave **puede tanto enviar mensajes cifrados y descifrarlos**.

En los métodos de clave asimétrica cada uno de los actores tiene un _par de claves_. Ambas claves sirven tanto para cifrar como para descifrar, pero con una particularidad: **lo cifrado con una clave debe ser descifrado con la otra** y viceversa.

Los métodos asimétricos tienen _a priori_ una seguridad mayor que el método simétrico. Imagina a alguien llamado Alice que quiera enviar un mensaje a Bob. Con un método simétrico:

  1. Alice cifraría el mensaje con la clave K de encriptación
  2. Bob usaría la misma clave K para descifrarlo
  3. Si Bob quiere enviar una respuesta cifrada a Alice la cifraría usando la misma clave K y Alice lo descifraría usando K.

El punto débil aquí es **el intercambio de claves**. Si queremos que un mensaje de Alice a Bob solo pueda ser leído por Bob debemos asegurarnos que la clave K solo sea conocida por Alice y por Bob. Porque si dicha clave K llega a un tercero este podrá leer el mensaje.

Para evitar este punto entran los sistemas asimétricos:

  1. Bob tiene su par de claves Kpu y Kpr. Bob publica la clave Kpu en su web pero se guarda bajo llave la clave Kpr.
  2. Alice quiere mandarle un mensaje cifrado a Bob. Para ello lo cifra con **la clave Kpu de Bob** (que está en su web).
  3. Bob recibe el mensaje y lo descifra usando su propia clave Kpr (que tiene guardada bajo llave).
  4. Si Bob quiere responder a Alice **puede cifrar su respuesta usando la clave Kpu de Alice** (que también está en la web de Alice). Alice puede descifrar el texto usando su propia clave Kpr.

No hay intercambio de claves. Par enviar algo a Bob se cifra con su clave pública (la Kpu de Bob) y tan solo Bob lo puede descifrar con su Kpr (su clave privada). Los métodos asimétricos son más lentos (tanto para cifrar como para descifrar) que los asimétricos y tienen sus propios problemas: existen ataques sobre la Kpu para intentar averiguar la Kpr asociada, ya que es obvio que tienen alguna relación. Algunos algoritmos asimétricos han sido rotos gracias a que se han encontrado relaciones más o menos obvias entre ambas claves que permiten deducir (o acotar suficientemente el ámbito de búsqueda para permitir un ataque por fuerza bruta) la clave privada al saber la pública.

**Cifrando y descifrando datos**

Bueno… veamos como podemos implementar un cifrado asimétrico usando WebApi. Nos centramos solo en el cifrado de datos (no entraremos en temas de firma digital aunque los principios sean muy parecidos). El algoritmo que usaremos será RSA (pero vamos, hay otros) a través de la clase <a href="http://msdn.microsoft.com/en-us/library/system.security.cryptography.rsacryptoserviceprovider.aspx" target="_blank" rel="noopener noreferrer">RSACryptoServerProvider</a>.

La propia clase se encarga de crear el par de claves necesario y luego pueden usarse los métodos ToXmlString() y FromXmlString() para guardar dichas claves o bien incorporar dichas claves en un RSACryptoServerProvider:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:f03f66f0-aab1-43c6-8409-f7f73ab5074b" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">static</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">void</span><span style="background:#1e1e1e;color:#dcdcdc"> Main(</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc">[] args)</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> rsa </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">RSACryptoServiceProvider</span><span style="background:#1e1e1e;color:#dcdcdc">();</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> both </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> rsa</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ToXmlString(</span><span style="background:#1e1e1e;color:#569cd6">true</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color
:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> pub </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> rsa</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ToXmlString(</span><span style="background:#1e1e1e;color:#569cd6">false</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#4ec9b0">File</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">WriteAllText(</span><span style="background:#1e1e1e;color:#d69d85">"both.xml"</span><span style="background:#1e1e1e;color:#dcdcdc">, both);</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#4ec9b0">File</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">WriteAllText(</span><span style="background:#1e1e1e;color:#d69d85">"pub.xml"</span><span style="background:#1e1e1e;color:#dcdcdc">, pub);</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Dicho código crea dos archivos xml (both.xml y pub.xml). El primero contiene el par de claves (pública y privada) mientras que el segundo contiene tan solo la clase pública. **Por supuesto hay mejores maneras de guardar las claves** pero para este post eso será suficiente.

Ahora vamos a crear una aplicación ASP.NET WebApi.

He generado un par de claves y las he guardado en una clase (he copiado el contenido entero de both.xml en una propiedad CryptoKeys.Both y lo mismo para CryptoKeys.Pub):

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:987c527d-b4a1-47fd-ad5c-3be5cd33b44f" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">CryptoKeys</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">internal</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">const</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> Both </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"[CONTENIDO ENTERO DEL FICHERO BOTH.XML]"</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">internal</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">const</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> Pub </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"[CONTENIDO ENTERO DEL FICHERO PUB.XML]"</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Vale… en un mundo real esto NO estaría hardcodeado pero… ¡estamos en el mundo ideal de los blogs!

Ahora creamos un controlador WebApi para que nos devuelva la clave pública:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:d4c0b0a2-849e-48c8-9a19-72c44e474996" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">KeyController</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#4ec9b0">ApiController</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#57a64a">// GET api/values</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> Get()</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">CryptoKeys</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Pub;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Una llamada GET a /api/Key nos permite obtener la clave pública del servidor:

[<img title="SNAGHTML5d500782" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="SNAGHTML5d500782" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/SNAGHTML5d500782_5F00_thumb_5F00_06E91A84.png" width="504" height="186" />][2]

Vale… vamos a ver el **código del cliente**.

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:bdc30538-8bb8-4d87-82cf-a929fbb70736" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">static</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e
;color:#569cd6">void</span><span style="background:#1e1e1e;color:#dcdcdc"> Main(</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc">[] args)</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">DoEverythingAsync()</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Wait();</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">static</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">async</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Task</span><span style="background:#1e1e1e;color:#dcdcdc"> DoEverythingAsync()</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> kpu </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#57a64a">// Obtenemos clave del servidor</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">using</span><span style="background:#1e1e1e;color:#dcdcdc"> (</span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> client </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">HttpClient</span><span style="background:#1e1e1e;color:#dcdcdc">())</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">client</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">DefaultRequestHeaders</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Accept</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ParseAdd(</span><span style="background:#1e1e1e;color:#d69d85">"application/json"</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> data </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">await</span><span style="background:#1e1e1e;color:#dcdcdc"> client</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">GetAsync(</span><span style="background:#1e1e1e;color:#d69d85">"http://localhost:25986/api/key"</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">kpu </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">await</span><span style="background:#1e1e1e;color:#dcdcdc"> data</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Content</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ReadAsStringAsync();</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">kpu </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">JsonConvert</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">DeserializeObject</span><span style="background:#1e1e1e;color:#b4b4b4"><</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#b4b4b4">></span><span style="background:#1e1e1e;color:#dcdcdc">(kpu);</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> encrypted </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> EncryptData(kpu, </span><span style="background:#1e1e1e;color:#d69d85">"PRIVATE DATA TO SEND"</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">await</span><span style="background:#1e1e1e;color:#dcdcdc"> SendEncryptedDataAsync(encrypted);</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Básicamente obtenemos la clave pública (llamando a /api/Key del servidor), encriptamos unos datos y los enviamos. El código de EncryptData es:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:cbbb3ddc-f8d1-4d76-866e-c27e141fff79" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">static</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">byte</span><span style="background:#1e1e1e;color:#dcdcdc">[] EncryptData(</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> kpu, </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> data)</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> rsa </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">RSACryptoServiceProvider</span><span style="background:#1e1e1e;color:#dcdcdc">();</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">rsa</span><span style="background:#1e1e1e;color:#b4b
4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">FromXmlString(kpu);</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> bytes </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">byte</span><span style="background:#1e1e1e;color:#dcdcdc">[data</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Length </span><span style="background:#1e1e1e;color:#b4b4b4">*</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">sizeof</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#569cd6">char</span><span style="background:#1e1e1e;color:#dcdcdc">)];</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#4ec9b0">Buffer</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">BlockCopy(data</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ToCharArray(), </span><span style="background:#1e1e1e;color:#b5cea8"></span><span style="background:#1e1e1e;color:#dcdcdc">, bytes, </span><span style="background:#1e1e1e;color:#b5cea8"></span><span style="background:#1e1e1e;color:#dcdcdc">, bytes</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Length);</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> encrypted </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> rsa</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Encrypt(bytes, </span><span style="background:#1e1e1e;color:#569cd6">true</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> encrypted;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Simplemente usamos el método FromXmlString para inicializar el RSACryptoServiceProvider con la clave pública obtenida previamente. Luego llamamos a Encrypt para encriptar los datos.

Finalmente el código de SendEncryptedDataAsync:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:13cffa4d-70d6-43dd-b796-611532ab3169" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">static</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">async</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Task</span><span style="background:#1e1e1e;color:#dcdcdc"> SendEncryptedDataAsync(</span><span style="background:#1e1e1e;color:#569cd6">byte</span><span style="background:#1e1e1e;color:#dcdcdc">[] encrypted)</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">using</span><span style="background:#1e1e1e;color:#dcdcdc"> (</span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> client </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">HttpClient</span><span style="background:#1e1e1e;color:#dcdcdc">())</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> content </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">FormUrlEncodedContent</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc">[] </span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">KeyValuePair</span><span style="background:#1e1e1e;color:#b4b4b4"><</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#b4b4b4">></span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#d69d85">"CC"</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#4ec9b0">Convert</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ToBase64String(encrypted)),</span>
        </li>
        <li style="background: #111111">
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">KeyValuePair</span><span style="background:#1e1e1e;color:#b4b4b4"><</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#b4b4b4">></span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#d69d85">"Name"</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#d69d85">"edu"</span><span style="background:#1e1e1e;color:#dcdcdc">),</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">});</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> result </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">await</span><span style="background:#1e1e1e;color:#dcdcdc"> client</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="back
ground:#1e1e1e;color:#dcdcdc">PostAsync(</span><span style="background:#1e1e1e;color:#d69d85">"http://localhost:25986/api/Test"</span><span style="background:#1e1e1e;color:#dcdcdc">, content);</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#4ec9b0">Console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">WriteLine(</span><span style="background:#1e1e1e;color:#d69d85">"Status Code: "</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">+</span><span style="background:#1e1e1e;color:#dcdcdc"> result</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">StatusCode);</span>
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

Enviamos dos campos en el POST, uno llamado CC (el encriptado) y otro llamado Name que NO está encriptado.

Hecho esto, en el servidor nos creamos un controlador de prueba (Test):

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:a77bdadf-5b03-42f6-9279-ec0259165818" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">TestController</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#4ec9b0">ApiController</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">void</span><span style="background:#1e1e1e;color:#dcdcdc"> Post(</span><span style="background:#1e1e1e;color:#4ec9b0">CreditCardInfo</span><span style="background:#1e1e1e;color:#dcdcdc"> card)</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li style="background: #111111">
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">CreditCardInfo</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> Name { </span><span style="background:#1e1e1e;color:#569cd6">get</span><span style="background:#1e1e1e;color:#dcdcdc">; </span><span style="background:#1e1e1e;color:#569cd6">set</span><span style="background:#1e1e1e;color:#dcdcdc">; }</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> CC { </span><span style="background:#1e1e1e;color:#569cd6">get</span><span style="background:#1e1e1e;color:#dcdcdc">; </span><span style="background:#1e1e1e;color:#569cd6">set</span><span style="background:#1e1e1e;color:#dcdcdc">; }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Si ahora levantamos la aplicación WebApi y ejecutamos nuestra aplicación de consola cliente vemos que los datos llegan al servidor:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_020666C8.png" width="504" height="68" />][3]

Por supuesto el servidor tiene que descifrarlos… Veamos como:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:7f37fccb-5db8-4533-8324-9e720ad0f300" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">void</span><span style="background:#1e1e1e;color:#dcdcdc"> Post(</span><span style="background:#1e1e1e;color:#4ec9b0">CreditCardInfo</span><span style="background:#1e1e1e;color:#dcdcdc"> card)</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> rsa </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">RSACryptoServiceProvider</span><span style="background:#1e1e1e;color:#dcdcdc">();</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">rsa</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">FromXmlString(</span><span style="background:#1e1e1e;color:#4ec9b0">CryptoKeys</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Both);</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> bytes </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Convert</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">FromBase64String(card</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">CC);</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background
:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> decrypted </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> rsa</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Decrypt(bytes, </span><span style="background:#1e1e1e;color:#569cd6">true</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> chars </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">char</span><span style="background:#1e1e1e;color:#dcdcdc">[decrypted</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Length </span><span style="background:#1e1e1e;color:#b4b4b4">/</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">sizeof</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#569cd6">char</span><span style="background:#1e1e1e;color:#dcdcdc">)];</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#4ec9b0">Buffer</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">BlockCopy(decrypted, </span><span style="background:#1e1e1e;color:#b5cea8"></span><span style="background:#1e1e1e;color:#dcdcdc">, chars, </span><span style="background:#1e1e1e;color:#b5cea8"></span><span style="background:#1e1e1e;color:#dcdcdc">, decrypted</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Length);</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> decryptedString </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc">(chars);</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

¡Al final en la variable decryptedString estarán los datos descifrados!

**Un poco más de integración con WebApi**

Tener que colocar el código de descifrado continuamente no es muy agradecido. Por suerte podemos integrarnos dentro de WebApi de forma relativamente simple.

WebApi usa media type formatters para pasar los datos que vienen **en el cuerpo de la petición http** a un parámetro del controlador. En nuestro caso mandamos dos datos (Name y CC). Para saber que media type formatter usar WebApi debe saber en qué formato están los datos en la petición y para ello usa la cabecera content-type. El valor normal de content-type cuando enviamos datos vía POST es application/x-www-form-urlencoded (si enviasemos un JSON usaríamos application/json). WebApi viene con soporte nativo para application/x-www-form-urlencoded a través de dos media type formatters:

  * FormUrlEncodedMediaTypeFormatter: Se usa si el controlador recibe un JToken o un FormCollection
  * JQueryMvcFormUrlEncodedFormatter: Se usa si el controlador recibe un objeto como parámetro. Deriva del anterior.

En nuestro caso el controlador recibe un CreditCardInfo por lo que será JQueryMvcFormUrlEncodedFormatter el que procese los datos de la petición, y cree el CreditCardInfo que recibe el controlador en el método POST.

Así la solución pasa por **derivar de JQueryMvcFormUrlEncodedFormatter y añadir en la clase derivada el código de descifrado**. Veamos como hacerlo de forma sencilla. Primero creamos la clase CypheredFomUrlEncodedMediaTypeFormatter:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:f806e4e7-3f53-4a1b-8b63-d97cc0058e1b" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">CypheredFormUrlEncodedMediaTypeFormatter</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#4ec9b0">JQueryMvcFormUrlEncodedFormatter</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">override</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Task</span><span style="background:#1e1e1e;color:#b4b4b4"><</span><span style="background:#1e1e1e;color:#569cd6">object</span><span style="background:#1e1e1e;color:#b4b4b4">></span><span style="background:#1e1e1e;color:#dcdcdc"> ReadFromStreamAsync(</span><span style="background:#1e1e1e;color:#4ec9b0">Type</span><span style="background:#1e1e1e;color:#dcdcdc"> type, </span><span style="background:#1e1e1e;color:#4ec9b0">Stream</span><span style="background:#1e1e1e;color:#dcdcdc"> readStream, </span><span style="background:#1e1e1e;color:#4ec9b0">HttpContent</span><span style="background:#1e1e1e;color:#dcdcdc"> content,</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#b8d7a3">IFormatterLogger</span><span style="background:#1e1e1e;color:#dcdcdc"> formatterLogger)</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ReadFromStreamAsyncCore(type, readStream, content, formatterLogger);</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Sobreescribimos el método ReadFromStreamAsync. Este método debe leer los datos del cuerpo de la petición, crear el objeto del tipo type indicado y pasar los datos de la petición al objeto creado.

Todo el código lo tendremos en el método ReadFromStreamAsyncCore de la propia clase:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:23c9f7fe-e4af-43af-8cdf-287bc1d9ecd6" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Mon
ospace; font-size: 10pt">
    </p> 
    
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">async</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Task</span><span style="background:#1e1e1e;color:#b4b4b4"><</span><span style="background:#1e1e1e;color:#569cd6">object</span><span style="background:#1e1e1e;color:#b4b4b4">></span><span style="background:#1e1e1e;color:#dcdcdc"> ReadFromStreamAsyncCore(</span><span style="background:#1e1e1e;color:#4ec9b0">Type</span><span style="background:#1e1e1e;color:#dcdcdc"> type, </span><span style="background:#1e1e1e;color:#4ec9b0">Stream</span><span style="background:#1e1e1e;color:#dcdcdc"> readStream, </span><span style="background:#1e1e1e;color:#4ec9b0">HttpContent</span><span style="background:#1e1e1e;color:#dcdcdc"> content, </span><span style="background:#1e1e1e;color:#b8d7a3">IFormatterLogger</span><span style="background:#1e1e1e;color:#dcdcdc"> formatterLogger)</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> obj </span><span style="background:#1e1e1e;color:#b4b4b4">=</span>
        </li>
        <li style="background: #111111">
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">await</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">base</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ReadFromStreamAsync(</span><span style="background:#1e1e1e;color:#569cd6">typeof</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#4ec9b0">FormDataCollection</span><span style="background:#1e1e1e;color:#dcdcdc">), readStream, content, formatterLogger) </span><span style="background:#1e1e1e;color:#569cd6">as</span>
        </li>
        <li>
                          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#4ec9b0">FormDataCollection</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> binded </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> obj</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ReadAs(type);</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">foreach</span><span style="background:#1e1e1e;color:#dcdcdc"> (</span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> property </span><span style="background:#1e1e1e;color:#569cd6">in</span><span style="background:#1e1e1e;color:#dcdcdc"> type</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">GetProperties()</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Where(p </span><span style="background:#1e1e1e;color:#b4b4b4">=></span><span style="background:#1e1e1e;color:#dcdcdc"> p</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">PropertyType </span><span style="background:#1e1e1e;color:#b4b4b4">==</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">typeof</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc">)))</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> attr </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> property</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">GetCustomAttribute</span><span style="background:#1e1e1e;color:#b4b4b4"><</span><span style="background:#1e1e1e;color:#4ec9b0">CypheredDataAttribute</span><span style="background:#1e1e1e;color:#b4b4b4">></span><span style="background:#1e1e1e;color:#dcdcdc">();</span>
        </li>
        <li style="background: #111111">
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">if</span><span style="background:#1e1e1e;color:#dcdcdc"> (attr </span><span style="background:#1e1e1e;color:#b4b4b4">!=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">null</span><span style="background:#1e1e1e;color:#dcdcdc">)</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li style="background: #111111">
                          <span style="background:#1e1e1e;color:#dcdcdc">property</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">SetValue(binded, Decrypt(property</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">GetValue(binded) </span><span style="background:#1e1e1e;color:#569cd6">as</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc">));</span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> binded;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

El método ReadFromStreamAsyncCore hace lo siguiente:

  1. Usamos el método (de la clase base) ReadFromStreamAsync pasándole como parámetro un FormDataCollection. Con eso obtenemos un FormDataCollection (básicamente un diccionario clave, valor) con los datos del cuerpo de la petición. Eso nos funciona porque derivamos de FormUrlEncodedMediaTypeFormatter que tiene soporte para FormDataCollection.
  2. Usamos el método (de extensión) ReadAs de FormDataCollection que crea un objeto del tipo indicado a partir de los datos de un FormDataCollection. Es decir, hace el binding de propiedades.
  3. Ahora iteramos sobre todas las propiedades **de tipo cadena** y miramos si alguna tiene el atributo CypheredDataAttribute aplicado.
  1. Si lo tiene desciframos el valor de dicha propiedad mediante una llamada a Decrypt.

El código del método Decrypt es básicamente el mismo que teníamos antes en el controlador:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:536c04da-8685-4756-8297-8a8fde39bf27" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-fam
ily: 'Courier New', Courier, Monospace; font-size: 10pt">
    </p> 
    
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">private</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> Decrypt(</span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> encryptedBase64)</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> rsa </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">RSACryptoServiceProvider</span><span style="background:#1e1e1e;color:#dcdcdc">();</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">rsa</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">FromXmlString(</span><span style="background:#1e1e1e;color:#4ec9b0">CryptoKeys</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Both);</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> bytes </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Convert</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">FromBase64String(encryptedBase64);</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> decrypted </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> rsa</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Decrypt(bytes, </span><span style="background:#1e1e1e;color:#569cd6">true</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> chars </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">char</span><span style="background:#1e1e1e;color:#dcdcdc">[decrypted</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Length </span><span style="background:#1e1e1e;color:#b4b4b4">/</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">sizeof</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#569cd6">char</span><span style="background:#1e1e1e;color:#dcdcdc">)];</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#4ec9b0">Buffer</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">BlockCopy(decrypted, </span><span style="background:#1e1e1e;color:#b5cea8"></span><span style="background:#1e1e1e;color:#dcdcdc">, chars, </span><span style="background:#1e1e1e;color:#b5cea8"></span><span style="background:#1e1e1e;color:#dcdcdc">, decrypted</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Length);</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> decryptedString </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc">(chars);</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> decryptedString;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Con esto ya tenemos un media type formatter que descifrará automáticamente todas aquellas propiedades string decoradas con [CypheredData]. El código de CypheredDataAttribute es trivial:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:5ec51f3f-c6c8-4743-8702-86ac57166ad7" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">[</span><span style="background:#1e1e1e;color:#4ec9b0">AttributeUsage</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#b8d7a3">AttributeTargets</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Property)]</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">CypheredDataAttribute</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#4ec9b0">Attribute</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Ahora nuestra clase CreditCardInfo nos queda de la siguiente manera:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:b81345fc-7c58-4f1e-90b8-413ae9e91a31" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">CreditCardInfo</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span st
yle="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> Name { </span><span style="background:#1e1e1e;color:#569cd6">get</span><span style="background:#1e1e1e;color:#dcdcdc">; </span><span style="background:#1e1e1e;color:#569cd6">set</span><span style="background:#1e1e1e;color:#dcdcdc">; }</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">[</span><span style="background:#1e1e1e;color:#4ec9b0">CypheredData</span><span style="background:#1e1e1e;color:#dcdcdc">]</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> CC { </span><span style="background:#1e1e1e;color:#569cd6">get</span><span style="background:#1e1e1e;color:#dcdcdc">; </span><span style="background:#1e1e1e;color:#569cd6">set</span><span style="background:#1e1e1e;color:#dcdcdc">; }</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

¡Un último detalle! Tenemos que agregar nuestro MediaTypeFormatter en la configuración de webapi:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:9a171b57-0173-465c-8e7f-d0c3a7fe0c62" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">config</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Formatters</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Remove(</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">config</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Formatters</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Single(f </span><span style="background:#1e1e1e;color:#b4b4b4">=></span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">typeof</span><span style="background:#1e1e1e;color:#dcdcdc"> (</span><span style="background:#1e1e1e;color:#4ec9b0">JQueryMvcFormUrlEncodedFormatter</span><span style="background:#1e1e1e;color:#dcdcdc">) </span><span style="background:#1e1e1e;color:#b4b4b4">==</span><span style="background:#1e1e1e;color:#dcdcdc"> f</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">GetType()));</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">config</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Formatters</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Add(</span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">CypheredFormUrlEncodedMediaTypeFormatter</span><span style="background:#1e1e1e;color:#dcdcdc">());</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Y lo mejor: ¡en el controlador no tenemos que hacer nada para tener los datos descifrados!

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_7D23B30B.png" width="504" height="104" />][4]

¡Listos! Hemos visto como podemos enviar datos cifrados y como podemos integrarnos un poco dentro de webapi para que el descifrado sea sencillo. No hemos cubierto todos los casos posibles, pero espero que os haya dado algunas ideas de como implementarlo!

Saludos!

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/SNAGHTML5d216779_5F00_4A2F0142.png
 [2]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/SNAGHTML5d500782_5F00_4476784D.png
 [3]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_7B535D44.png
 [4]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_488356D0.png