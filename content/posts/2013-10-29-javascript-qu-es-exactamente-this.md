---
title: JavaScript – ¿Qué es exactamente this?
description: JavaScript – ¿Qué es exactamente this?
author: eiximenis

date: 2013-10-29T18:00:44+00:00
geeks_url: /?p=1656
geeks_visits:
  - 3169
geeks_ms_views:
  - 717
categories:
  - Uncategorized

---
Si vienes de un lenguaje orientado a objetos “clásico” como C# o Java, tendrás claro el concepto de _this_: Se refiere al propio objeto. Dado que todos los objetos son instancias de una clase en concreto y el código se define _a nivel de clase_ el valor de this está claramente definido. Sólo leyendo el código puedes saber a que se refiere this en cada momento.

JavaScript también tiene la palabra _this_ pero su significado es un poco más “complejo” que el de C#.

Antes que nada mencionar que this en JavaScript NO es opcional. Siempre que quieras acceder a una propiedad de this, debes usarla. Si no usas this, JavaScript asume siempre que estás accediendo a una variable local (o global).

El valor de this en JavaScript es _parecido_ al de C#, pero a la vez es tan distinto que lo normal es que alguien que venga de C# termine sin entender nada. 

Empecemos diciendo que el valor de this _dentro de una función_ es el valor del objeto en el que se define esta función. Veamos un ejemplo:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:60f35295-cbfe-4c60-b567-d9cc10d9148f" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> persona </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    name</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">'edu'</span><span style="background:#1e1e1e;color:#b4b4b4">,</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    twitter</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">'eiximenis'</span><span style="background:#1e1e1e;color:#b4b4b4">,</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    twitterUrl</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">'http://twitter.com/'</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">+</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">twitter</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">persona</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">twitterUrl</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Este código lo que imprime por pantalla es… _http://twitter.com/undefined_

Pero… a ver: No valía _this_ el valor del objeto sobre el que estaba definido el código? No está _twitterUrl_ definido dentro del objeto _persona_? Qué narices ocurre?

Sencillo: Si relees mi definición de this, verás que empezaba diciendo “el valor de this _**dentro de una función…**_”. La propiedad twitterUrl no es una función, así que eso no se aplica. En este caso el valor de this es el contexto en el cual se esté ejecutando este código, que por defecto es el contexto global (window en los navegadores).

Para solucionarlo, debemos convertir twitterUrl a una función:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:18c49843-f968-4935-bddf-6fc39660e86d" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> persona </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    name</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">'edu'</span><span style="background:#1e1e1e;color:#b4b4b4">,</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    twitter</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">'eiximenis'</span><span style="background:#1e1e1e;color:#b4b4b4">,</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    twitterUrl</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">'http://twitter.com/'</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">+</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">twitter</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">persona</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">twitterUrl</span><span style="background:#1e1e1e;color:#b4b4b4">());</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Con este cambio, ahora si que el código imprime la respuesta esperada.

**Funciones constructoras**

Si leíste mi post sobre si [JavaScript era orientado a objetos][1], ya sabrás que llamo función constructora a aquella función que se usa para crear objetos que compartan un mismo prototipo. Las funciones construc
  
toras se invocan con new y es precisamente el uso de new lo que convierte a una función en constructora, no la función en sí.

El siguiente código también muestra el valor esperado:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:09fdc3f4-9008-44e7-8ba4-275e3e24b8c9" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #000080; color: #fff; font-family: Verdana, Tahoma, Arial, sans-serif; font-weight: bold; padding: 2px 5px">
      Code Snippet
    </div>
    
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> Persona </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">n</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#dcdcdc"> tw</span><span style="background:#1e1e1e;color:#b4b4b4">)</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">name</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> n</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">twitter </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> tw</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">twitterUrl </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc">  </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">'http://twitter.com/'</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">+</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">twitter</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> persona </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> Persona</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#d69d85">'edu'</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#d69d85">'eiximenis'</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">persona</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">twitterUrl</span><span style="background:#1e1e1e;color:#b4b4b4">());</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Insisto: **Es el uso de new el que hace que this sea el objeto que se está creando.** ¿Que pasaría si me olvidase el new al llamar a la función Persona?

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:cbdcfb70-0e50-4f66-b82f-ebe07826d6ca" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> foo </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> Persona</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#d69d85">'edu'</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">'eiximenis'</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">foo</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">name</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">name</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Ahora llamo a la función _Persona_ pero **sin usar new**. Ahora el valor de this dentro de _Persona_ **ya NO es el objeto que se está creando**. Ahora el valor de this dentro de persona es el valor del objeto dentro del cual la función Persona está definido… Como _Persona_ es una función global, pues ahora this es el contexto global.

De hecho el primer console.log dará error porque foo ahora vale undefined (¡la función Persona no devuelve nada!) y el segundo console.log mostrará “edu” porque la llamada a Persona ha creado la variable “name” en el contexto global.

Recuerda: No hay clases en JavaScript. Si quieres crear objetos a través de una función constructora asegúrate de usar siempre new. Si no obtendrás resultados imprevistos. Es el uso de new el que convierte una función en constructora.

**Call / Apply**

Vamos a ponernos un poco más ser
  
ios. Este ejemplo anterior muestra una característica fundamental de this: **Su valor depende de como se llame a una función.** Hemos visto que dentro de Persona el valor de this dependía de si llamábamos a la función con new o sin new.

Pero oye… ya que el valor de this dentro de una función depende de como esta sea llamada, porque no explotar al máximo esta capacidad y permitir que el valor de this sea el que el desarrollador quiera? Hasta ahora hemos visto que el valor de this podía ser:

  1. El objeto dentro del cual estaba definida la función
  2. Un objeto nuevo que se crea en aquel momento (al usar new).

Pues bien call y apply añaden un tercer escenario: el valor de this puede ser _cualquier objeto_ que el desarrollador desee… Veamos un ejemplo:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:872146c4-8675-4cc5-95ca-a5a5ed027460" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> Persona </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">n</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#dcdcdc"> tw</span><span style="background:#1e1e1e;color:#b4b4b4">)</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">name</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> n</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">twitter </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> tw</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">twitterUrl </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc">  </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">'http://twitter.com/'</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">+</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">twitter</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> p </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    blog</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#d69d85">'http://geeks.ms/blogs/etomas'</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">Persona</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">call</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">p</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">'edu'</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#d69d85">'eiximenis'</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">p</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">twitterUrl</span><span style="background:#1e1e1e;color:#b4b4b4">());</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">p</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">blog</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

La función Persona es exactamente la misma que teníamos antes. Luego definimos un objeto _p_ con una propiedad _blog_. Y en la línea 13 tenemos la clave: Usamos _call_ para llamar a la función Persona. La función call permite especificar el valor de this dentro de una función. El primer parámetro de call es el valor que tomará this, mientras que el resto de parámetros son los que se pasan a la función. 

Por lo tanto, como el primer parámetro de Persona.call es el objeto p, el valor de this dentro de Persona será el objeto p… por lo tanto las propiedades name, twitter y twitterUrl serán añadidas al objeto p.

Apply hace exactamente lo mismo que call, salvo que los parámetros que se mandan a la función se especifican en un array:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:f7336688-0289-4956-9c69-b99735ac8774" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">Persona</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">apply</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;co
lor:#dcdcdc">p</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">[</span><span style="background:#1e1e1e;color:#d69d85">'edu'</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#d69d85">'eiximenis'</span><span style="background:#1e1e1e;color:#b4b4b4">]);</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

¿P: ej. no te has preguntado nunca porque si te suscribes usando jQuery a un evento, puedes usar “this” dentro de la función gestora del evento para acceder al elemento del DOM que ha lanzado el evento? Pues ahí tienes la respuesta: Los que crearon jQuery conocen call y apply 😉

**El problema de la pérdida de this**

Este es un problema bastante molesto y difícil de detectar si no lo conoces y se entiende mejor con un ejemplo:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:a08b95c3-bd02-4c27-8372-03629c8f09e2" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> Persona </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">n</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#dcdcdc"> tw</span><span style="background:#1e1e1e;color:#b4b4b4">)</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">name</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> n</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">twitter </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> tw</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">twitterUrl </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc">  </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">'http://twitter.com/'</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">+</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">twitter</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        }</span><span style="background:#1e1e1e;color:#b4b4b4">());</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> p </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> Persona</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#d69d85">'edu'</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">'eiximenis'</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">p</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">twitterUrl</span><span style="background:#1e1e1e;color:#b4b4b4">());</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

El ejemplo es un pelín forzado pero ilustra la problemática de la pérdida de this.

Primero, que sepas que este código imprime **http://twitter.com/undefined** en lugar de http://twitter.com/eiximenis como sería de esperar. Si entiendes porque entenderás la problemática de pérdida de this.

La propiedad twitterUrl sigue siendo una función, así que… ¿porque ahora this.twitter no tiene valor? Pues porque ahora estamos accediendo a this dentro de **una función que está dentro de una función**. Fíjate que ahora twitterUrl es una función que **define internamente a otra función anónima, la invoca y devuelve su resultado**. La función interna NO está definida dentro del mismo objeto en el cual se define la propiedad twitterUrl y por lo tanto el valor de _this_ dentro de la función interna NO es el propio objeto…&#160; De hecho en este caso this es el contexto global. Y claro el contexto global no tiene definida ninguna propiedad _twitter_.

Como digo el ejemplo está muy forzado porque nadie haría eso… pero tener una función dentro de otra función es muy habitual en el caso de callbacks, así que te terminarás encontrando con este caso. Y cuando te encuentres con él… te acordarás de mi! 😛

Hay dos soluciones para esto:

  1. Hacer que el valor de this dentro de la función interna sea el correcto (a través de call/apply). Algunas librerías de javascript se encargan de ello con las funciones de callback que gestionan ellos.
  2. Guardar el valor de this en una variable.

Veamos como quedaría el código para el primer caso:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:e798c426-51d9-4387-bc4b-5eea63216c13" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding
-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  </p> 
  
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> Persona </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">n</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#dcdcdc"> tw</span><span style="background:#1e1e1e;color:#b4b4b4">)</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">name</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> n</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">twitter </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> tw</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">twitterUrl </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc">  </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">'http://twitter.com/'</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">+</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">twitter</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        }</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">apply</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">));</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> p </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> Persona</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#d69d85">'edu'</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">'eiximenis'</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">p</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">twitterUrl</span><span style="background:#1e1e1e;color:#b4b4b4">());</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Llamadme friki, pero a mi esto me encanta… Como puedes ver la única diferencia que hay respecto al código anterior es que ahora invocamos a la función anónima interna usando apply. Da igual que la función sea anónima y la defina justo en este lugar: puedo usar apply igualmente… Una maravilla. Si vienes de C# igual te cuesta entender esto, pero ten presente que en JavaScript las funciones son ciudadanas de primer orden. No necesitamos artificios raros como los delegates para poder pasar funciones como parámetro. Y las funciones tienen un prototipo común (de hecho apply está definido en el prototipo de Function).

El código para la segunda opción (guardar el valor de this) sería el siguiente:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:f2a8739b-51af-4cf8-a873-d14c43f3e8d1" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> Persona </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">n</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#dcdcdc"> tw</span><span style="background:#1e1e1e;color:#b4b4b4">)</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">name</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> n</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">twitter </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> tw</span><span style="background:#1e1e1e;color:#b4b4b
4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">twitterUrl </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> self </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">            </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">'http://twitter.com/'</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">+</span><span style="background:#1e1e1e;color:#dcdcdc"> self</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">twitter</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        }</span><span style="background:#1e1e1e;color:#b4b4b4">());</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> p </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> Persona</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#d69d85">'edu'</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">'eiximenis'</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">p</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">twitterUrl</span><span style="background:#1e1e1e;color:#b4b4b4">());</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

El código sigue siendo muy parecido con la salvedad de que ahora en la línea 5 guardamos el valor de this en una variable llamada self. En la línea 5 el valor de this es el objeto que queremos (pues estamos dentro de la funcion twitterUrl y no dentro de la función interna). Y dentro de la función interna (línea 7) no usamos this si no la variable anterior (self). Es una convención usar los nombres that o self para esta variable que guarda el valor de this.

**Bind**

Bueno… vamos a ponernos ya en plan superlativo. Si creías que ya lo habíamos visto todo sobre JavaScript y this, todavía nos queda una sorpresa más: La función bind.

Bind es muy similar a call o apply, pero en lugar de devolver el resultado de la función que se llama, devuelve… otra función. ¿Y qué es esta otra función? Pues es la función original pero con el valor this por defecto establecido siempre al valor del parámetro pasado a bind.

Veamos un ejemplo:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:865ca727-6aac-4d87-9703-f3d356a9e456" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> Persona </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">n</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#dcdcdc"> tw</span><span style="background:#1e1e1e;color:#b4b4b4">)</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">name</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> n</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">twitter </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> tw</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#569cd6">function</span><span style="background:#1e1e1e;color:#dcdcdc"> getTwitterUri</span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">'http://twitter.com/'</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b4b4b4">+</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">twitter</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> p </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="back
ground:#1e1e1e;color:#dcdcdc"> Persona</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#d69d85">'edu'</span><span style="background:#1e1e1e;color:#b4b4b4">,</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">'eiximenis'</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">getTwitterUri</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">apply</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">p</span><span style="background:#1e1e1e;color:#b4b4b4">));</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li>
          <span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> ftw </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> getTwitterUri</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">bind</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">p</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">ftw</span><span style="background:#1e1e1e;color:#b4b4b4">());</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

¿Cual es la salida de este código? ¡Exacto! Se imprime dos veces ‘http://twitter.com/eiximenis’:

  1. La primera vez es porque usamos apply para invocar a la función getTwitterUri con el objeto p, por lo que el valor de this dentro de getTwitterUri es el objeto p.
  2. La segunda es porque invocamos la funcion ftw. La función ftw es devuelta por la llamada a getTwitterUri.bind por lo que al invocar la función ftw el valor de this es el propio objeto p.

Ahora añadamos las siguientes dos líneas más de código:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:42c8195c-d28a-47bb-acb0-7fcd66d41a53" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">getTwitterUri</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">apply</span><span style="background:#1e1e1e;color:#b4b4b4">({</span><span style="background:#1e1e1e;color:#dcdcdc"> twitter</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">'msdev_es'</span><span style="background:#1e1e1e;color:#dcdcdc"> }</span><span style="background:#1e1e1e;color:#b4b4b4">));</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">log</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">ftw</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">apply</span><span style="background:#1e1e1e;color:#b4b4b4">({</span><span style="background:#1e1e1e;color:#dcdcdc"> twitter</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">'msdev_es'</span><span style="background:#1e1e1e;color:#dcdcdc"> }</span><span style="background:#1e1e1e;color:#b4b4b4">));</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

¿Qué crees que imprimen esas dos líneas de código adicionales?

Si has respondido que ambas van a imprimir ‘http://twitter.com/msdev_es’ déjame darte una buena noticia y una mala:

  * La buena es que has entendido como funcionan call y apply! 🙂
  * La mala es que la respuesta no es correcta 😛

La primera línea SI que imprime ‘http://twitter.com/msdev\_es’, ya que llamamos a getTwitterUri usando apply y pasándole un objeto cuya propiedad twitter es ‘msdev\_es’…

… Pero la segunda línea NO imprime ‘http://twitter.com/msdev_es’. Debería hacerlo si ftw fuese una funcion “normal”. Pero ftw es una función obtenida a través de bind, por lo que **sigue conservando su valor de this** con independencia de call y apply.

Así que ya ves… Para saber el valor de this que usa una función ya ni te basta con mirar si es invocada usando call y apply: debes además saber si esta función ha sido generada a través de bind o no 😛

En fin… Creo que más o menos con esto cubrimos todos los aspectos de this en JavaScript. Como puedes ver, aunque su significado es _parecido_ al de C# realmente es tan distinto que lo más normal es que si asumes que el this de JavaScript funciona como el de C# termines por no entender nada y maldecir a JavaScript… 😉

Un saludo! 😉

 [1]: http://geeks.ms/blogs/etomas/archive/2013/10/25/191-es-javascript-orientado-a-objetos.aspx