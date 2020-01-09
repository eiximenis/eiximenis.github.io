---
title: Enviar un array (JS) a un controlador MVC5
description: Enviar un array (JS) a un controlador MVC5
author: eiximenis

date: 2015-07-15T16:11:53+00:00
geeks_url: /?p=1698
geeks_visits:
  - 732
geeks_ms_views:
  - 3684
categories:
  - Uncategorized

---
Buenas!

En los foros de MSDN aparece <a href="https://social.msdn.microsoft.com/Forums/es-ES/776bf7b9-ee2f-408f-af94-4ab75ec028bd/enviar-array-long-a-controller-con-ajax?forum=aspnetmvces" target="_blank" rel="noopener noreferrer">la pregunta sobre como enviar un array JS a un controlador de MVC</a>. La verdad es que hay varias maneras de hacerlo… veamos dos de ellas, ambas muy sencillas.

En todos los casos el controlador tiene el siguiente método:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:8125b5c3-29d5-4112-ab2c-140576d58185" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #000000; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">ActionResult</span><span style="background:#1e1e1e;color:#dcdcdc"> Index(</span><span style="background:#1e1e1e;color:#569cd6">long</span><span style="background:#1e1e1e;color:#dcdcdc">[] data)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> View();</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

****

**Opción 1 – Mandar el array como un JSON**

Esta es la aproximación que se intenta en el post. Basta el siguiente código en el cliente:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:2e14e5ef-5966-49e9-9785-20168eb41589" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #000000; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> uri </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">'</span><span style="background:#ffffb3;color:#000000">@</span><span style="background:#1e1e1e;color:#dcdcdc">Url</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Action(</span><span style="background:#1e1e1e;color:#d69d85">"Index"</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#d69d85">"Home"</span><span style="background:#1e1e1e;color:#dcdcdc">)</span><span style="background:#1e1e1e;color:#d69d85">'</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> arr </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">[</span><span style="background:#1e1e1e;color:#b5cea8">4</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#b5cea8">8</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#b5cea8">15</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#b5cea8">16</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#b5cea8">23</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#b5cea8">42</span><span style="background:#1e1e1e;color:#b4b4b4">];</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">$</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ajax</span><span style="background:#1e1e1e;color:#b4b4b4">({</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">url</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> uri</span><span style="background:#1e1e1e;color:#b4b4b4">,</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">data</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> JSON</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">stringify</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">arr</span><span style="background:#1e1e1e;color:#b4b4b4">),</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">type</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">'POST'</span><span style="background:#1e1e1e;color:#b4b4b4">,</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">contentType</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">'application/json'</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Y el array se recibe sin problema alguno en el controlador. En el caso que el controlador declarase el parámetro como IEnumerable<long> funcionaría igual.

**Opción 2 – Mandar el array como x-www-form-urlencoded**

Esto, con jQuery es un poco más complejo. El siguiente código:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:251a800b-0ab6-4485-9b61-f279ce20d9ee" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #000000; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> arr </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">[</span><span style="background:#1e1e1e;color:#b5cea8">4</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#b5cea8">8</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#b5cea8">15</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#b5cea8">16</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#b5cea8">23</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#b5cea8">42</span><span style="background:#1e1e1e;color:#b4b4b4">];</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">$</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ajax</span><span style="backgro
und:#1e1e1e;color:#b4b4b4">({</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">url</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> uri</span><span style="background:#1e1e1e;color:#b4b4b4">,</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">data</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> arr</span><span style="background:#1e1e1e;color:#b4b4b4">,</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">type</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">'POST'</span><span style="background:#1e1e1e;color:#b4b4b4">,</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Genera una petición incorrecta, debido como jQuery serializa los parámetros. Lo que jQuery manda en el cuerpo de la petición es:

> _undefined=&undefined=&undefined=&undefined=&undefined=&undefined=_

Si en la llamada a $.ajax colocamos el parámetro processData a false, entonces jQuery nos genera lo siguiente en el cuerpo de la petición:

> _4,8,15,16,23,42_

Aunque ahora si que están los datos, MVC5 no es capaz de procesar los datos en este formato, y recibiremos _null_ en el parámetro.

La solución es bastante sencilla: debemos convertir el array en un objeto, cuya propiedad sea el array:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:0207398c-b79d-4be4-92cf-2c38645e9ef5" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #000000; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> arr </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">[</span><span style="background:#1e1e1e;color:#b5cea8">4</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#b5cea8">8</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#b5cea8">15</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#b5cea8">16</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#b5cea8">23</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#b5cea8">42</span><span style="background:#1e1e1e;color:#b4b4b4">];</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">$</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ajax</span><span style="background:#1e1e1e;color:#b4b4b4">({</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">url</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> uri</span><span style="background:#1e1e1e;color:#b4b4b4">,</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">data</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span><span style="background:#1e1e1e;color:#dcdcdc"> data</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> arr }</span><span style="background:#1e1e1e;color:#b4b4b4">,</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">type</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">'POST'</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Ahora bien, es **muy importante** que el nombre de la propiedad (_data_) sea el mismo que el nombre del parámetro en el controlador, ya que el _model binder_ de MVC nos enlazará por nombre.

En este caso lo que se envia en el cuerpo de la petición que se genera es lo siguiente:

> _data%5B%5D=4&data%5B%5D=8&data%5B%5D=15&data%5B%5D=16&data%5B%5D=23&data%5B%5D=42_

Parece un poco críptico, pero ten presente que %5B es la codificación del carácter [ y %5D se corresponde al carácter ]. Así que realmente lo que se envía es:

> _data[]=4&data[]=8&data[]=15&data[]=16&data[]=23&data[]=42_

El _model binder_ de MVC es capaz de entender esto sin ningún problema. Al igual que antes sigue funcionando si el controlador recibe un IEnumerable<long> en lugar de un long[] (eso sí, que se llame “data” igual!).

Saludos!