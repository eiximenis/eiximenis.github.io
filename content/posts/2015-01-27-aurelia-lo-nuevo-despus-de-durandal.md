---
title: Aurelia… lo nuevo después de Durandal
description: Aurelia… lo nuevo después de Durandal
author: eiximenis

date: 2015-01-27T11:57:41+00:00
geeks_url: /?p=1692
geeks_visits:
  - 1860
geeks_ms_views:
  - 2124
categories:
  - Uncategorized

---
Seguro que muchos de vosotros conocéis Durandal, la librería para crear aplicaciones SPA ideada por <a href="https://twitter.com/EisenbergEffect" target="_blank" rel="noopener noreferrer">Rob Eisenberg</a> autor también de otras librerías como Caliburn y con cierta obsesión por las espadas…

Hacé algún tiempo <a href="http://eisenbergeffect.bluespire.com/angular-and-durandal-converge/" target="_blank" rel="noopener noreferrer">Rob anunció que se había unido al equipo de Angular</a> para colaborar con el desarrollo de la versión 2.0 de dicha librería y que Angular y Durandal convergerían en una única librería. Eso generó muchas reacciones, algunas positivas, otras no tanto. A pesar de que Durandal tenía sus usuarios la verdad es que no había conseguido, ni mucho menos, la popularidad de Angular que con esas noticias parecía convertirse en el standard de facto para crear aplicaciones SPA (si es que ya no lo era).

Posteriormente <a href="http://eisenbergeffect.bluespire.com/leaving-angular/" target="_blank" rel="noopener noreferrer">Rob anunció que dejaba el equipo de Angular</a> debido a diferencias en como tenía que evolucionar la librería. Así que anunció que Durandal seguiría su camino y que evolucionaría de forma independiente a Angular y que en fin… todo seguía más o menos como antes. También prometía novedades sobre Durandal NextGen, la versión que tenía en mente y que nunca hubiese existido si Rob se hubiese quedado en el equipo de Angular.

Y finalmente <a href="http://eisenbergeffect.bluespire.com/introducing-aurelia/" target="_blank" rel="noopener noreferrer">Rob anuncia la aparición de Aurelia</a>, su nuevo framework para aplicaciones SPA. Ya… para seguir la tradición lo podría haber llamado Hrunting o Glamdring o ya puestos Tizona pero no… ha optado por Aurelia. Nuevo nombre para dejar claro que no se trata de una evolución de Durandal si no de una librería totalmente nueva, diseñada desde cero y pensada para ser moderna: está escrita integramente en ES6 aunque transpilada (vaya palabreja) y pollyfilizada (otra palabreja… lo siento por los de la RAE) para trabajar con los navegadores evergreen de hoy en día (un navegador evergreen es aquel del cual no importa su versión por estar siempre actualizado. Si te pregunto tu versión de Chrome o Firefox dudo que la sepas exactamente, esos son los navegadores evergreen).

A diferencia de Angular, Aurelia es más un conjunto de librerías que de forma conjunta forman un framework. Así se pueden usar las librerías sueltas para pequeñas tareas o bien utilizarlas todas en conjunto para crear aplicaciones SPAs. Eso tiene la ventaja de que uno solo paga por lo que usa y que la curva de aprendizaje es, a priori, más suave en tanto se pueden incorporar los conceptos de forma más independiente (en Angular todo está muy atado y es difícil explicar los conceptos de forma separada).

**Empezando con Aurelia**

Las instrucciones para empezar están todas en [http://aurelia.io/get-started.html][1]. Básicamente antes se tiene que instalar gulp y jspm. El primero se usará como sistema de build y el segundo se usa como gestor de paquetes. Esas dos herramientas no forman parte de Aurelia como tal y podrías usar grunt o bower si lo prefieres (asumiendo que los paquetes de Aurelia estén subidos en bower que no lo sé), pero todos los ejemplos de la <a href="http://aurelia.io/" target="_blank" rel="noopener noreferrer">página web de Aurelia</a> están en glup y jspm.

Veamos lo mínimo necesario para usar Aurelia. Para ello partimos de un directorio vacío y ejecutamos jspm install aurelia-bootstrapper lo que nos instalará un montón de dependencias.

Ahora vamos a necesitar crear **tres** ficheros:

  * El bootstrapper o lanzador de nuestra aplicación
  * La vista
  * El viewmodel

Vemos pues que Aurelia usa el patrón MVVM y en efecto tiene enlace bidireccional entre la vista y el viewmodel asociado. Veamos cada uno de esos componentes.

El lanzador (index.html) es el encargado de cargar aurelia y lanzar la vista inicial:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:ba2f88b8-be99-4710-a286-adad1153194b" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">!doctype</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">html</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">html</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">head</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">head</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">body</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">aurelia-app</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">script</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">src</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"jspm_packages/system.js"</span><span style="background:#1e1e1e;color:#808080">></</span><span style="background:#1e1e1e;color:#569cd6">script</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">script</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">src</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"config.js"</span><span style="background:#1e1e1e;color:#808080">></</span><span style="background:#1e1e1e;color:#569cd6">script</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">script</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">System</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">baseUrl </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">'myapp'</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">System</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">import</span><span style="background:#1e1e1e;color:#b4b4b4"><br /> (</span><span style="background:#1e1e1e;color:#d69d85">'aurelia-bootstrapper'</span><span style="background:#1e1e1e;color:#b4b4b4">).</span><span style="background:#1e1e1e;color:#dcdcdc">catch</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">console</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">error</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">bind</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">console</span><span style="background:#1e1e1e;color:#b4b4b4">));</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">script</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">body</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">html</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

La línea clave aquí es la que establece System.baseUrl a “myapp”. Eso le indica a Aurelia que cargue la vista myapp/app.html y el viewmodel asociado myapp/app.js.

Empecemos por ver el viewmodel **que es una clase pura de JavaScript**:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:19af09c5-f11c-447e-b03f-97c85d8ff4a7" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">export</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> Welcome{</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">constructor</span><span style="background:#1e1e1e;color:#b4b4b4">(){</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">name </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">'Eduard Toms'</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">twitter </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">'@eiximenis'</span><span style="background:#1e1e1e;color:#b4b4b4">;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li style="background: #111111">
          &nbsp;
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">get twitterUri</span><span style="background:#1e1e1e;color:#b4b4b4">()</span><span style="background:#1e1e1e;color:#dcdcdc"> {</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> `http</span><span style="background:#1e1e1e;color:#b4b4b4">:</span><span style="background:#1e1e1e;color:#57a64a">//twitter.com/${this.twitter}`;</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li style="background: #111111">
          &nbsp;
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">welcome</span><span style="background:#1e1e1e;color:#b4b4b4">(){</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">alert</span><span style="background:#1e1e1e;color:#b4b4b4">(</span><span style="background:#1e1e1e;color:#dcdcdc">`Welcome ${</span><span style="background:#1e1e1e;color:#569cd6">this</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">name}</span><span style="background:#1e1e1e;color:#b4b4b4">!</span><span style="background:#1e1e1e;color:#dcdcdc">`</span><span style="background:#1e1e1e;color:#b4b4b4">);</span>
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

Aunque quizá no te lo parezca **eso es JavaScript puro**, concretamente EcmaScript 6, que tiene entre otras características la posibilidad de definir módulos y clases. En este caso el el fichero myapp/app.js es un módulo que se limita a exportar una sola clase llamada Welcome.

Observa también que las propiedades twitterUri() y welcome() usan cadenas pero con el carácter \` (tilde abierta) y no con la comilla simple (ya, el coloreado de código falla totalmente :p). Esa <a href="https://developer.mozilla.org/es/docs/Web/JavaScript/Reference/template_strings" target="_blank" rel="noopener noreferrer"><strong>es otra característica de ES6 llamada string interoplation</strong></a>_._ Sí… ES6 viene cargado con un montón de novedades.

Finalmente nos queda la vista. Las vistas son pedazos de html **encapsulados dentro de una etiqueta template:**

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:853d3f85-bb3e-4cea-827b-a08073c41ed8" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">template</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">section</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">h2</span><span style="background:#1e1e1e;color:#808080">></span><span style="background:#1e1e1e;color:#dcdcdc">${</span><span style="background:#1e1e1e;color:#ff00ff">heading}</span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">h2</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">form</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">role</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"<br /> form"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">submit.delegate</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"welcome()"</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">div</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">class</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"form-group"</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li style="background: #111111">
                          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">label</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">for</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"fn"</span><span style="background:#1e1e1e;color:#808080">></span><span style="background:#1e1e1e;color:#dcdcdc">First Name</span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">label</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
                          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">input</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">type</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"text"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">value.bind</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"name"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">class</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"form-control"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">id</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"fn"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#808080">/></span>
        </li>
        <li style="background: #111111">
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">div</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">div</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">class</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"form-group"</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li style="background: #111111">
                          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">label</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">for</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"ln"</span><span style="background:#1e1e1e;color:#808080">></span><span style="background:#1e1e1e;color:#dcdcdc">Twitter</span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">label</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
                          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">input</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">type</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"text"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">value.bind</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"twitter"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">class</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"form-control"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">id</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"ln"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#808080">/></span>
        </li>
        <li style="background: #111111">
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">div</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">div</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">class</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"form-group"</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li style="background: #111111">
                          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">label</span><span style="background:#1e1e1e;color:#808080">></span><span style="background:#1e1e1e;color:#dcdcdc">Hello ${</span><span style="background:#1e1e1e;color:#ff00ff">name}</span><span style="background:#1e1e1e;color:#dcdcdc">. Check your twitter account (</span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">a</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">href.bind</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"twitterUri"</span><span style="background:#1e1e1e;color:#808080">></span><span style="background:#1e1e1e;color:#dcdcdc">${</span><span style="background:#1e1e1e;color:#ff00ff">twitter}</span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">a</span><span style="background:#1e1e1e;color:#808080">></span><span style="background:#1e1e1e;color:#dcdcdc">)?</span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">label</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
                          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;colo
r:#569cd6">p</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">class</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"help-block"</span><span style="background:#1e1e1e;color:#808080">></span><span style="background:#1e1e1e;color:#dcdcdc">${</span><span style="background:#1e1e1e;color:#ff00ff">fullName}</span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">p</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li style="background: #111111">
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">div</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
                      <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">button</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">type</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"submit"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">class</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"btn btn-default"</span><span style="background:#1e1e1e;color:#808080">></span><span style="background:#1e1e1e;color:#dcdcdc">Submit</span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">button</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">form</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">section</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">template</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Puedes observar como los enlaces en texto se definen con la sintaxis ${propiedad} (estos enlaces son obviamente unidireccionales). Por otro lado cuando enlazas una propiedad (tal como value o href) se usa la sintaxis propiedad.bind=”propiedad” (p. ej. value.bind o href.bind).

Si ejecutas index.html eso en un navegador que soporte ES6 (p. ej. Chrome) deberás ver algo parecido a:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_349DE2EC.png" width="244" height="139" />][2]

(El alert aparece al pulsar el botón de submit).

Por supuesto si el navegador que tienes no soporta ES6 **<a href="https://6to5.org/" target="_blank" rel="noopener noreferrer">siempre puedes usar 6to5</a>** para transpilar (es decir convertir xD) el código ES6 usado al ES5 soportado hoy en día para la mayoría de navegadores. Aquí es donde entra gulp en el proceso.&#160; Otra opción es usar **directamente ES5** si prefieres. Pero bueno… veremos esto en otro post 🙂

Y bueno… tampoco me ha dado tiempo para ver mucho más de Aurelia. Por lo poco que he visto la idea en general es parecida a Angular o Ember en tanto que usa el patrón MVVM (el viewmodel de Aurelia se corresponde aproximadamente al controlador de Angular) y que soporta enlaces bidireccionales.

Os animo por supuesto a que le echéis un vistazo, al menos para conocerla brevemente… veremos si gana popularidad y se convierte en una alternativa a Angular o se queda en el intento. 🙂

Saludos!

 [1]: http://aurelia.io/get-started.html "http://aurelia.io/get-started.html"
 [2]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_451653E7.png