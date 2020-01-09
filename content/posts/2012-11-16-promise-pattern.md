---
title: Promise Pattern
author: eiximenis

date: 2012-11-16T13:28:50+00:00
geeks_url: /?p=1616
geeks_visits:
  - 2843
geeks_ms_views:
  - 1357
categories:
  - Uncategorized

---
¡Buenas! 

Este post está “inspirado” <a href="https://twitter.com/0GiS0/status/268729356761849856" target="_blank" rel="noopener noreferrer">por un tweet de Gisela Torres</a>. Posteriormente ella misma <a href="http://www.returngis.net/2012/11/el-patron-promise-y-callback/" target="_blank" rel="noopener noreferrer">hizo un post en su blog sobre este mismo patrón</a> que os recomiendo leer.

Ultimamente está muy de moda, al menos en el mundo de Javascript, hablar del _Promise pattern_ (lo siento, pero llega un punto en que yo ya desisto de intentar encontrar traducciones para todo…). Ese patrón se asocia mucho con la realización de aplicaciones asíncronas, llegándose a ver como el mecanismo para la realización de este tipo de aplicaciones.

No. El promise pattern nada tiene que ver con la creación de aplicaciones asíncronas, aunque es en estas, por razones que ahora veremos, donde se puede explotar más. Pero es basicamente un patrón de organización de código. Tampoco es exclusivo de Javascript, por supuesto… De hecho, y para no ser clásicos, vamos a ver un ejemplo de su uso en C#.

El promise pattern nos ayuda a organizar nuestro código en aquellos casos en que un método recibe como parámetro otro método (en el caso de C# un delegate). Este otro método (o delegate) es usualmente una función de callback, aunque no tiene exactamente porque.

Vamos a verlo con un ejemplo muy tonto. Imaginad una clase como la siguiente:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: blue">class</span> <span style="color: #2b91af">NoPromise</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">public</span> <span style="color: blue">void</span> Sumar (<span style="color: #2b91af">IEnumerable</span><<span style="color: blue">int</span>> data, <span style="color: #2b91af">Action</span><<span style="color: blue">int</span>> next )
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">var</span> result = 0;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">foreach</span> (<span style="color: blue">var</span> item <span style="color: blue">in</span> data) result += item;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; next(result);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">public</span> <span style="color: blue">void</span> Invertir(<span style="color: blue">int</span> value, <span style="color: #2b91af">Action</span><<span style="color: blue">int</span>> next)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">var</span> result = value * -1;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; next(result);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">public</span> <span style="color: blue">void</span> Dividir(<span style="color: blue">int</span> value, <span style="color: blue">float</span> divisor, <span style="color: #2b91af">Action</span><<span style="color: blue">float</span>> next)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">var</span> result = value/divisor;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; next(result);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Esta clase expone tres métodos Sumar, Dividir e Invertir. Ambos aceptan un segundo parámetro (un delegate) llamado next que le dice al método _que hacer con_ el resultado.

Así podemos invocar dichos métodos así:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    np.Sumar(list, x =>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; np.Invertir(x, y=>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; np.Dividir(y, 4.0f, z =>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #2b91af">Console</span>.WriteLine(<span style="color: #a31515">"El resultado es "</span> + z))));
  </p></p>
</div>

Aquí no tenemos asincronidad alguna, simplemente estamos encadenando métodos. Fijaos en como tenemos una “cascada” de funciones encadeandas. La llamada a la primera función (Sumar) tiene en su interior la llamada a Invertir, que en su interior tiene la llamada a Dividir que en su interior tiene el Console.WriteLine. Tenemos pues “una cascada de funciones dentro de funciones”.

Este problema es el que el promise pattern intenta solucionar… Veamos primero como sería el resultado de las llamadas usando este patrón:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: blue">var</span> prom1 = pd.Sumar(list);
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue">var</span> prom2 = prom1.Then(x => pd.Invertir(x));
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue">var</span> prom3 = prom2.Then(x => pd.Dividir(x, 4.0f));
  </p>
  
  <p style="margin: 0px">
    prom3.Then(x => <span style="color: #2b91af">Console</span>.WriteLine(<span style="color: #a31515">"El valor es "</span> + x));
  </p></p>
</div>

Si preferís podríais usar una sintaxis más compacta sin declarar las variables intermedias:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    pd.Sumar(list).
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; Then(x => pd.Invertir(x)).
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; Then(x => pd.Dividir(x, 4.0f)).
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; Then(x => <span style="color: #2b91af">Console</span>.WriteLine(<span style="color: #a31515">"final: "</span> + x));
  </p></p>
</div>

La idea es que el método Sumar **no** recibe como un parámetro el método a invocar a continuación si no que nos devuelve un objeto (el _promise_). A este objeto le podremos indicar (usando el método Then) cual es el siguiente método a invocar. Además los promises se encadenan entre ellos. Fijaos como hemos pasado de una llamada a un método (que dentro tenía varias llamadas más) a cuatro llamadas separadas. Eso mejora mucho la legibilidad del código que es el objetivo principal de este patrón.

Por si os interesa aquí tenéis la implementación que he usado del patrón. Primero la interfaz:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: blue">public</span> <span style="color: blue">interface</span> <span style="color: #2b91af">IPromise</span><T>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #2b91af">IPromise</span><TR> Then<TR>(<span style="color: #2b91af">Func</span><T, TR> action);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">void</span> Then(<span style="color: #2b91af">Action</span><T> action);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; T Value { <span style="color: blue">get</span>; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Y luego la clase que lo implementa:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: blue">class</span> <span style="color: #2b91af">Promise</span><T> : <span style="color: #2b91af">IPromise</span><T>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="mar
gin: 0px">
    &#160;&#160;&#160; <span style="color: blue">private</span> T _data;
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">public</span> Promise(T data)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; _data = data;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">public</span> <span style="color: #2b91af">IPromise</span><TR> Then<TR>(<span style="color: #2b91af">Func</span><T, TR> action)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">return</span> <span style="color: blue">new</span> <span style="color: #2b91af">Promise</span><TR>(action(_data));
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">public</span> T Value
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">get</span>{ <span style="color: blue">return</span> _data; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">public</span> <span style="color: blue">void</span> Then(<span style="color: #2b91af">Action</span><T> action)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; action(_data);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Finalmente necesitamos que el método Sumar (que es el iniciador de la cadena de promises) me devuelva un IPromise:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: blue">class</span> <span style="color: #2b91af">PromiseDemo</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">public</span> <span style="color: #2b91af">IPromise</span><<span style="color: blue">int</span>> Sumar(<span style="color: #2b91af">IEnumerable</span><<span style="color: blue">int</span>> data)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">var</span> result = 0;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">foreach</span> (<span style="color: blue">var</span> item <span style="color: blue">in</span> data) result += item;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">return</span> <span style="color: blue">new</span> <span style="color: #2b91af">Promise</span><<span style="color: blue">int</span>>(result);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">public</span> <span style="color: blue">int</span> Invertir(<span style="color: blue">int</span> value)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">return</span> value * -1;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">public</span> <span style="color: blue">float</span> Dividir(<span style="color: blue">int</span> value, <span style="color: blue">float</span> div)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">return</span> value / div;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

El resto de métodos son métodos “normales”.

**¿Y porqué (casi) todo el mundo asocia el promise pattern con asincronidad?**

Bueno… si os fijáis el promise pattern permite organizar mejor nuestro código en aquellos casos en que un método espera a otro método como parámetro. Y eso es un caso habitual en… las funciones asíncronas que, por norma general, esperan un parámetro adicional que es el callback.

Ahora imaginad la situación clásica de que el callback de una función asíncrona es a su vez una función asíncrona que espera otro callback que es a su vez otra función asíncrona con otro callback… Llegamos así a la cascada de callbacks dentro de callbacks que es justo el caso que soluciona este patrón. De ahí que la gente asocie promise pattern a asincronidad aunque no tengan nada que ver.

De hecho .NET, en la TPL tiene una implementación ya realizada del promise pattern… Os suena el método _<a href="http://msdn.microsoft.com/es-es/library/dd270696.aspx" target="_blank" rel="noopener noreferrer">ContinueWith</a>_ de la clase Task? Pues viene a ser una implementación del promise pattern (en este caso la propia clase Task actúa como promise).

**¿Y porque se habla tanto del promise pattern en Javascript?**

De hecho más que en javascript deberíamos decir en lenguajes funcionales donde las funciones son ciudadanos de primer orden. En C# hasta la aparición de las expresiones lambda y los delegados genéricos no podíamos implementar este patrón (de una forma cómoda). Por eso en lenguajes no funcionales no se habla mucho de este patrón ya que en estos lenguajes no puede pasarse “una función entera” como parámetro a una función.

Pero en Javascript si se puede, y la popularidad que está adquiriendo el lenguaje junto con muchos frameworks que tienen este tipo de llamadas hace que la gente empiece a hablar de este patrón.

Pero insisto: no tiene nada que ver con asincronidad (aunque ahí se use mucho). P.ej. el siguiente código jQuery no tiene nada de asíncrono pero sería un candidato perfecto a utilizarel promise pattern:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    $(document).ready(<span style="color: blue">function</span>() {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; $(<span style="color: #a31515">"#cmdOpen"</span>).click(<span style="color: blue">function</span>() {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; $(<span style="color: #a31515">"#divData"</span>).fadeIn(<span style="color: #a31515">"slow"</span>, <span style="color: blue">function</span>() {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; $(<span style="color: #a31515">"#divOther"</span>).show();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; });
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; });
  </p>
  
  <p style="margin: 0px">
    });
  </p></p>
</div>

¡Espero que este post sobre este patrón os haya parecido interesante!

Un saludo!