---
title: ASP.NET MVC – ¿Por qué usar los helpers para formularios?

author: eiximenis

date: 2013-08-22T14:10:00+00:00
geeks_url: /?p=1650
geeks_visits:
  - 8501
geeks_ms_views:
  - 1646
categories:
  - Uncategorized

---
Hace nada mi compi <a href="http://twitter.com/jtorrecilla" target="_blank" rel="noopener noreferrer">Javier Torrecilla</a> (_Little Tower_ para los amigos) ha escrito <a href="http://geeks.ms/blogs/jtorrecilla/archive/2013/08/22/b-225-sico-mvc-los-helpers-en-mvc.aspx" target="_blank" rel="noopener noreferrer">un post sobre los helpers de ASP.NET MVC</a>.

En este post quiero centrarme en **por qué debes usar los helpers para formularios** de ASP.NET MVC. La respuesta “por qué están ahí” no vale. Hay muchas cosas que están por ahí y no deberían usarse salvo casos muy concretos (incluso cosas del .NET Framework).

Iremos como los New Kids on the Block, es decir paso a paso. Por supuesto si ya sabes todo lo que te aportan los helpers entonces este post no te aportará mucho, pero eres bienvenido a seguir leyendo por supuesto (y a comentar, claro) 🙂

**Pregunta: Como recibo en un controlador los datos mandados en un <form>?**

Esa es una pregunta típica que se hace cualquiera que empieza con ASP.NET MVC. Si vienes de ASP antiguo (el clásico, el de Interdev) o de otra tecnología como PHP, pues igual empiezas a buscar alguna propiedad llamada Form en la Request o algo así. ¡No busques! No lo hagas porque la encontrarás y entonces la usarás y te perderás uno de los elementos más poderosos de ASP.NET MVC: el model binder.

Para responder a esta pregunta vamos a usar esta vista:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    Da de alta una cerveza:
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray"><</span><span style="color: #569cd6">form</span> <span style="color: #9cdcfe">method</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"POST"</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; Nombre:
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">input</span> <span style="color: #9cdcfe">type</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"text"</span> <span style="color: #9cdcfe">name</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"Name"</span> <span style="color: gray">/></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">br</span> <span style="color: gray">/></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; Categoría:
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">input</span> <span style="color: #9cdcfe">type</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"text"</span> <span style="color: #9cdcfe">name</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"Category"</span> <span style="color: gray">/></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">br</span> <span style="color: gray">/></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">input</span> <span style="color: #9cdcfe">type</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"submit"</span> <span style="color: #9cdcfe">value</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"Enviar!"</span><span style="color: gray">/></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray"></</span><span style="color: #569cd6">form</span><span style="color: gray">></span>
  </p></p>
</div>

Todo HTML, que de moment no usamos Helpers 🙂

Veamos como **no** acceder a los valores de un formulario. Si vienes de PHP o Interdev probablemente habrás llegado a teclear algo como esto:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #4ec9b0">ActionResult</span> <span style="color: white">Index</span>()
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">if</span> (<span style="color: white">Request</span><span style="color: #b4b4b4">.</span><span style="color: white">Form</span>[<span style="color: #d69d85">"Name"</span>] <span style="color: #b4b4b4">!=</span> <span style="color: #569cd6">null</span> <span style="color: #b4b4b4">&&</span> <span style="color: white">Request</span><span style="color: #b4b4b4">.</span><span style="color: white">Form</span>[<span style="color: #d69d85">"Category"</span>] <span style="color: #b4b4b4">!=</span> <span style="color: #569cd6">null</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #608b4e">// Procesar alta</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: #569cd6">null</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">else</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: white">View</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Parece lógico ¿no? Buscamos si los datos de formulario existen, y si exiten es que venimos del POST y los procesamos. En caso contrario es que nos han hecho un GET por lo que devolvemos la vista.

Vale… olvida este código. **Así no se hacen las cosas en ASP.NET MVC**. Primero la lógica del GET (mostrar una vista) y del POST (procesar una alta) están mezclados y eso no es buena señal (¿conoces el <a href="http://en.wikipedia.org/wiki/Single_responsibility_principle" target="_blank" rel="noopener noreferrer">SRP</a>)? Por suerte ASP.NET MVC nos permite separar la lógica del GET de la del POST definiendo dos métodos y decorando el que gestiona el POST con el atributo [HttpPost].

Tener la lógica separada sería un poco mejor y de hecho hay por Internet código parecido al siguiente:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #4ec9b0">ActionResult</span> <span style="color: white">Index</span>()
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: white">View</span>();
  </p>
  
  <p style="margin: 0px">
    }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    [<span style="color: #4ec9b0">HttpPost</span>]
  </p>
  
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #4ec9b0">ActionResult</span> <span style="color: white">Index</span>(<span style="color: #4ec9b0">FormCollection</span> <span style="color: white">form</span>)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">if</span> (<span style="color: white">form</span>[<span style="color: #d69d85">"Name"</span>] <span style="color: #b4b4b4">!=</span> <span style="color: #569cd6">null</span> <span style="color: #b4b4b4">&&</span> <span style="color: white">form</span>[<span style="color: #d69d85">"Category"</span>] <span style="color: #b4b4b4">!=</span> <span style="color: #569cd6">null</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #608b4e">// Procesar alta</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd
6">return</span> <span style="color: #569cd6">null</span>;
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

La clase FormCollection que aparece es una clase propia de ASP.NET MVC que tiene la misma información que Request.Form. Este código funciona y de hecho verás código así por Internet (se ve de todo en este mundo) pero la realidad es que hay muy pocas razones para usar FormCollection hoy en día. ASP.NET MVC tiene un concepto mucho más poderoso, un amigo al que debes conocer y entender: el **model binder**.

Ya he hablado en este blog sobre el model binder, así que ahora solo introduciré la regla más básica: Si tu acción tiene un parámetro cuyo nombre es idéntico al de un atributo name de un campo de un formulario, el model binder enlazará el valor del campo al parámetro automáticamente.

Es decir, el código anterior nos puede quedar como:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    [<span style="color: #4ec9b0">HttpPost</span>]
  </p>
  
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #4ec9b0">ActionResult</span> <span style="color: white">Index</span>(<span style="color: #569cd6">string</span> <span style="color: white">name</span>, <span style="color: #569cd6">string</span> <span style="color: white">category</span>)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #608b4e">// procesar alta</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: #569cd6">null</span>;
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Esa es la regla básica del model binder. Estoy seguro que esto te parece precioso pero a la vez es posible que una inquietud atormente tu nueva felicidad: si tengo un formulario con 20 campos… se deben declarar 20 parámetros en la acción?

Por suerte el equipo de ASP.NET MVC se dio cuenta de ello y así surge la segunda (y última que veremos en este post) regla del model binder. Ojo, que ahí va: Si tu accíón recibe como parámetro, un objeto de una clase que tiene una propiedad cuyo nombre es igual al de un atributo name de un campo del formulario, el model binder enlazará automáticamente el valor de este campo a la propiedad correspondiente del objeto recibido como parámetro. Quizá te parezca un poco rebuscado, pero lo que significa es que puedo tener una clase como la que sigue:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">BeerModel</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">string</span> <span style="color: white">Name</span> { <span style="color: #569cd6">get</span>; <span style="color: #569cd6">set</span>; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">string</span> <span style="color: white">Category</span> { <span style="color: #569cd6">get</span>; <span style="color: #569cd6">set</span>; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Y ahora en mi controlador recibir los datos simplemente usando un parámetro de tipo BeerModel:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    [<span style="color: #4ec9b0">HttpPost</span>]
  </p>
  
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #4ec9b0">ActionResult</span> <span style="color: white">Index</span>(<span style="color: #4ec9b0">BeerModel</span> <span style="color: white">beer</span>)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #608b4e">// procesar alta</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: #569cd6">null</span>;
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

El model binder se encarga de crear el objeto beer y de rellenarlo con los parámetros del formulario. ¿Genial, no?

**Pregunta: Vale, pero esto que tiene que ver con los helpers?**

De momento nada, pero ahora veamos que ocurre si no los ponemos. Imagina que alguien me da de alta una cerveza pero hace submit del formulario sin entrar el nombre que es obligatorio. Evidentemente yo tengo que mostrarle de nuevo la vista de dar de alta una cerveza. Una forma **no muy correcta de hacerlo** es la siguiente:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    [<span style="color: #4ec9b0">HttpPost</span>]
  </p>
  
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #4ec9b0">ActionResult</span> <span style="color: white">Index</span>(<span style="color: #4ec9b0">BeerModel</span> <span style="color: white">beer</span>)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">if</span> (<span style="color: #569cd6">string</span><span style="color: #b4b4b4">.</span><span style="color: white">IsNullOrEmpty</span>(<span style="color: white">beer</span><span style="color: #b4b4b4">.</span><span style="color: white">Name</span>))
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: white">View</span>(<span style="color: white">beer</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #608b4e">// procesar alta</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: #569cd6">null</span>;
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Si el nombre de la cerveza está vacío devuelvo la vista como resultado, pero le paso la propia cerveza. ¿Para qué? Pues porque ahora tengo que meter código para recuperar el estado de los controles. Es decir, debo rellenar el atributo value de los dos <input> con los valores que me vienen de beer, ya que si no el usuario recibiría… ¡una vista completamente vacía!. Si vienes de Webforms este es un buen punto para empezar a odiar ASP.NET MVC, pero si lo sobrepasas luego, estoy seguro, lo empezarás a amar irremediablemente.

En resumen nuestra vista ahora queda algo como así:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="background: #ffffb3; color: black">@model </span><span style="color: white">MvcApplication1</span><span style="color: #b4b4b4">.</span><span style="color: white">Models</span><span style="color: #b4b4b4">.</span><span style="color: #4ec9b0">BeerModel</span>
  </p>
  
  <p style="margin: 0px">
    <span style="background: #ffffb3; color: black">@{</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: white">ViewBag</span><span style="color: #b4b4b4">.</span><span style="color: white">Title</span> <span style="color: #b4b4b4">=</span> <span style="color: #d69d85">"Index"</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">nameValue</span> <span style="color: #b4b4b4">=</span> <span style="color: white">Model</span> <span style="color: #b4b4b4">!=</span> <span style="color: #569cd6">null</span> <span style="color: #b4b4b4">?</span> <span style="color: white">Model</span><span style="color: #b4b4b4">.</span><span style="color: white">Name</span> : <span style="color: #569cd6">null</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">categoryValue</span> <span style="color: #b4b4b4">=</span> <span style="color: white">Model</span> <span style="color: #b4b4b4">!=<br /> </span> <span style="color: #569cd6">null</span> <span style="color: #b4b4b4">?</span> <span style="color: white">Model</span><span style="color: #b4b4b4">.</span><span style="color: white">Category</span> : <span style="color: #569cd6">null</span>;
  </p>
  
  <p style="margin: 0px">
    <span style="background: #ffffb3; color: black">}</span>
  </p>
  
  <p style="margin: 0px">
    Da de alta una cerveza:
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray"><</span><span style="color: #569cd6">form</span> <span style="color: #9cdcfe">method</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"POST"</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; Nombre:
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">input</span> <span style="color: #9cdcfe">type</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"text"</span> <span style="color: #9cdcfe">name</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"Name"</span> <span style="color: #9cdcfe">value</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"</span><span style="background: #ffffb3; color: black">@</span><span style="color: white">nameValue</span><span style="color: #c8c8c8">"</span><span style="color: gray">/></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">br</span> <span style="color: gray">/></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; Categoría:
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">input</span> <span style="color: #9cdcfe">type</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"text"</span> <span style="color: #9cdcfe">name</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"Category"</span> <span style="color: #9cdcfe">value</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"</span><span style="background: #ffffb3; color: black">@</span><span style="color: white">categoryValue</span><span style="color: #c8c8c8">"</span><span style="color: gray">/></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">br</span> <span style="color: gray">/></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">input</span> <span style="color: #9cdcfe">type</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"submit"</span> <span style="color: #9cdcfe">value</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"Enviar!"</span><span style="color: gray">/></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray"></</span><span style="color: #569cd6">form</span><span style="color: gray">></span>
  </p></p>
</div>

Aquí me estoy aprovechando de una característica de Razor que es que si igualamos un atributo de un elemento HTML a null, entonces Razor ni nos renderiza el atributo. Es decir si nameValue o categoryValue vale null, el atributo value ni aparece en el HTML generado.

Vale… ahora imagínate un formulario con 20 campos y ya puedes empezar a llorar.

Y esta es la primera razón por la que usar helpers. Si modificamos la vista para usar los helpers:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="background: #ffffb3; color: black">@model </span><span style="color: white">MvcApplication1</span><span style="color: #b4b4b4">.</span><span style="color: white">Models</span><span style="color: #b4b4b4">.</span><span style="color: #4ec9b0">BeerModel</span>
  </p>
  
  <p style="margin: 0px">
    <span style="background: #ffffb3; color: black">@{</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: white">ViewBag</span><span style="color: #b4b4b4">.</span><span style="color: white">Title</span> <span style="color: #b4b4b4">=</span> <span style="color: #d69d85">"Index"</span>;
  </p>
  
  <p style="margin: 0px">
    <span style="background: #ffffb3; color: black">}</span>
  </p>
  
  <p style="margin: 0px">
    Da de alta una cerveza:
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray"><</span><span style="color: #569cd6">form</span> <span style="color: #9cdcfe">method</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"POST"</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; Nombre:
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="background: #ffffb3; color: black">@</span><span style="color: white">Html</span><span style="color: #b4b4b4">.</span><span style="color: white">TextBoxFor</span>(<span style="color: white">m</span><span style="color: #b4b4b4">=></span><span style="color: white">m</span><span style="color: #b4b4b4">.</span><span style="color: white">Name</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">br</span> <span style="color: gray">/></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; Categoría:
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="background: #ffffb3; color: black">@</span><span style="color: white">Html</span><span style="color: #b4b4b4">.</span><span style="color: white">TextBoxFor</span>(<span style="color: white">m</span><span style="color: #b4b4b4">=></span><span style="color: white">m</span><span style="color: #b4b4b4">.</span><span style="color: white">Category</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">br</span> <span style="color: gray">/></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">input</span> <span style="color: #9cdcfe">type</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"submit"</span> <span style="color: #9cdcfe">value</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"Enviar!"</span><span style="color: gray">/></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray"></</span><span style="color: #569cd6">form</span><span style="color: gray">></span>
  </p></p>
</div>

Automáticamente los controles del formulario recuperan su estado automáticamente. ¡Hey! El que venía de Webforms… sigues ahí? 😉

**Pregunta: Y como muestro que campo es el que ha producido el error?**

Jejejeee… muy buena pregunta. Hemos redirigido de nuevo al usuario porque se ha dejado de introducir el nombre de la cerveza, pero estaría bien que le informáramos de ello. Antes he dicho que el código del controlador no era muy correcto. Y no lo es porque no usa un objeto de ASP.NET MVC llamado ModelState. El ModelState guarda entre otras cosas todos los errores que se producen al validar los datos por parte del controlador.

Así que lo suyo es introducir el error en el ModelState cuando lo detectamos:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    [<span style="color: #4ec9b0">HttpPost</span>]
  </p>
  
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #4ec9b0">ActionResult</span> <span style="color: white">Index</span>(<span style="color: #4ec9b0">BeerModel</span> <span style="color: white">beer</span>)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">if</span> (<span style="color: #569cd6">string</span><span style="color: #b4b4b4">.</span><span style="color: white">IsNullOrEmpty</span>(<span style="color: white">beer</span><span style="color: #b4b4b4">.</span><span style="color: white">Name</sp an>))</p> 
    
    <p style="margin: 0px">
      &#160;&#160;&#160; {
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">ModelState</span><span style="color: #b4b4b4">.</span><span style="color: white">AddModelError</span>(<span style="color: #d69d85">"Name"</span>, <span style="color: #d69d85">"¿Donde has visto una cerveza sin nombre?"</span>);
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: white">View</span>(<span style="color: white">beer</span>);
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160; }
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160; <span style="color: #608b4e">// procesar alta</span>
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: #569cd6">null</span>;
    </p>
    
    <p style="margin: 0px">
      }
    </p></p></div> 
    
    <p>
      El método AddModelError añade un error a la propiedad indicada (el segundo parámetro es el mensaje de error). Ahora si ejecutamos el proyecto y envías el formulario dejando el campo nombre vacío, el código HTML generado de vuelta será algo como:
    </p>
    
    <div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
      <p style="margin: 0px">
        <span style="color: gray"><</span><span style="color: #569cd6">form</span> <span style="color: #9cdcfe">method</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"POST"</span><span style="color: gray">></span>
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; Nombre:
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">input</span> <span style="color: #9cdcfe">class</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"input-validation-error"</span> <span style="color: #9cdcfe">id</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"Name"</span> <span style="color: #9cdcfe">name</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"Name"</span> <span style="color: #9cdcfe">type</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"text"</span> <span style="color: #9cdcfe">value</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">""</span> <span style="color: gray">/></span>
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">br</span> <span style="color: gray">/></span>
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; Categoría:
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">input</span> <span style="color: #9cdcfe">id</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"Category"</span> <span style="color: #9cdcfe">name</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"Category"</span> <span style="color: #9cdcfe">type</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"text"</span> <span style="color: #9cdcfe">value</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"sdsd"</span> <span style="color: gray">/></span>
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">br</span> <span style="color: gray">/></span>
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">input</span> <span style="color: #9cdcfe">type</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"submit"</span> <span style="color: #9cdcfe">value</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"Enviar!"</span><span style="color: gray">/></span>
      </p>
      
      <p style="margin: 0px">
        <span style="color: gray"></</span><span style="color: #569cd6">form</span><span style="color: gray">></span>
      </p></p>
    </div>
    
    <p>
      Fíjate que el <input> generado para tener el nombre de la cerveza ahora incluye una clase llamada “input-validation-error”. Esta clase la añade el helper cuando ve que hay un error asociado en el ModelState.
    </p>
    
    <p>
      Luego ya por CSS es cosa tuya personalizar esta clase para que haga algo útil (p. ej. mostrar el campo en rojo). Lo importante es que el helper la añade automáticamente sin que tu tengas que hacer nada.
    </p>
    
    <p>
      Ahora imagínate sin los helpers… para un formulario con 20 campos, tener que consultar para cada uno si existe un error en el ModelState… buf, sería para morirse.
    </p>
    
    <p>
      Y es por esto, básicamente, que debemos utilizar los helpers!
    </p>
    
    <p>
      Un saludo!
    </p>
    
    <p>
      PD: Ojo, hay <strong>mejores</strong> maneras de validar la entrada de datos en ASP.NET MVC. Basta decir que el código de validación no debería estar en el controlador y que ASP.NET MVC ofrece dos mecanismos built-in que son DataAnnotations y la interfaz IValidatableObject. No hemos visto nada de esto porque no es el objetivo de este post.
    </p>