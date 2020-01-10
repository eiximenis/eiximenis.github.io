---
title: 'JavaScript: Limitar el tiempo de ejecución de una promise'

author: eiximenis

date: 2019-01-31T15:45:13+00:00
geeks_url: /?p=2257
geeks_ms_views:
  - 577
categories:
  - javascript

---
Este es un post cortito y motivado porque hace algunos días en el curso de [JavaScript avanzado][1] que tengo, junto al maestro [José Manuel Alarcón][2], en CampusMVP un alumno preguntó más o menos eso: como se podía tener un _timeout_ en una promise, de forma que esa fallase automáticamente si se superaba un determinado tiempo.
  
<!--more-->


  
No es mi intención hablar de promises, solo comentar que una promise es tan asíncrona, como asíncrono sea su código: es decir si dentro de la promise no usamos alguna api realmente asíncrona (como _fetch_, _XMLHttpRequest_, la API de IndexedDb u otras) no tendremos asincronía real.
  
Bien, si la API que usamos dentro de la promise ofrece algún mecanismo de timeout, lo ideal es usarlo, pero si no es el caso (como curiosamente ocurre con _fetch_) hay un mecanismo muy sencillo para conseguirlo: usar [Promise.race][3]. Esta función toma un iterable de promises y devuelva otra promise que se resuelve/rechaza tan buen punto una de las promises del iterable se resuelva o se rechace.
  
Por lo tanto el mecanismo es muy sencillo. Esta función toma una promise y devuelve otra promise con un timeout:

<pre class="EnlighterJSRAW" data-enlighter-language="js">const TimeoutPromise = (pr, timeout) =&gt;
  Promise.race([pr, new Promise((_, rej) =&gt;
    setTimeout(rej, timeout)
  )]);</pre>

Su uso es muy sencillo:

<pre class="EnlighterJSRAW" data-enlighter-language="null">let pr = fetch('http://slowwly.robertomurray.co.uk/delay/8000/url/http://www.google.co.uk', {mode: 'no-cors'});
let tpr = TimeoutPromise(pr, 500)
  .then(() =&gt; console.log('fetch done'))
  .catch(() =&gt;console.log('timeout cancelled'));</pre>

La promise _pr_ es una promise que tarda 8 segundos en resolverse (es lo que tarda en cargar esa página), pero si ejecutas ese código verás que al cabo de 500ms la promise tpr es rechazada.
  
Más fácil imposible, ¿verdad?
  
Bien, nada es perfecto: este método **cuando se rechaza la promise por el _timeout_, el resto de promises internas siguen ejecutándose**. Es decir, en nuestro caso, _tpr_ es rechazada al cabo de 500ms, pero la promise _pr_ se sigue ejecutando durante los 8s (hasta que se completa la petición de red). Por lo tanto **este mecanismo no aborta promises. Se limita a envolverlas con una promise que, esta sí, se resuelve/rechaza en un tiempo máximo.**
  
Lo ideal **sería poder cancelar la promise** pero no ofrece ECMAScript un mecanismo para cancelar promises, [**aunque el TC39 está trabajando en ello**][4].
  
**&#8220;Cancelando&#8221; promises**
  
[Hay otras técnicas][5], pero si para ti es vital el poder &#8220;cancelar promises&#8221; (entre comillas) aquí tienes un código muy sencillo que **permite interrumpir una promise en cada uno de sus pasos**. Cada uno de esos pasos debe ser a su vez una promise:

<pre class="EnlighterJSRAW" data-enlighter-language="js">const token = function() {
  return {
    cancel: function () {this.isCancelled = true},
    isCancelled: false
    }
}
async function  MultistepPromise (iterf, token)
{
  for (let f of iterf) {
    await f();
    if (token.isCancelled)  {
      throw 'promise cancelled';
    }
  }
}
</pre>

Bueno, como puedes ver el código es trivial: simplemente itera por el iterable de promises si por alguna razón el token se cancela.
  
Su uso es, de nuevo, muy sencillo:

<pre class="EnlighterJSRAW" data-enlighter-language="js">const func = () =&gt; fetch(
  'http://slowwly.robertomurray.co.uk/delay/3000/url/http://www.google.co.uk',
  {mode: 'no-cors'});
const tok = token();
const pr = MultistepPromise([func, func, func, func, func, func], tok)
  .then(() =&gt; console.log('all acceoted'))
  .catch((e) =&gt; console.log('some error', e));</pre>

Si una vez pr se está ejecutando, llamas a _tok.cancel()_, entonces **pr se cancelará una vez se haya finalizado el paso correspondiente **(a medio paso no se puede cancelar).
  
Observa que MultistepPromise no espera un array de promises, si no simplemente un iterable de promises, por lo que puedes usar otras técnicas tales como un generador para pasarle las promises a ejecutar.
  
Por supuesto, puedes combinar ambas técnicas: es decir, hacer una MultistepPromise, donde cada uno de sus pasos sea TimeoutPromise. De este modo te aseguras que si uno de los pasos excede el tiempo, todo el resto de pasos se cancelan (no se ejecutan).

 [1]: https://www.campusmvp.es/catalogo/Product-Programaci%C3%B3n-avanzada-con-JavaScript-y-ECMAScript_206.aspx
 [2]: https://twitter.com/jm_alarcon
 [3]: https://developer.mozilla.org/es/docs/Web/JavaScript/Referencia/Objetos_globales/Promise/race
 [4]: https://github.com/tc39/proposal-cancellation
 [5]: https://medium.com/@benlesh/promise-cancellation-is-dead-long-live-promise-cancellation-c6601f1f5082