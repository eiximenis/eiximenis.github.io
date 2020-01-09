---
title: 'ASP.NET MVC: Autocompletado con enums'
author: eiximenis

date: 2013-08-19T17:45:34+00:00
geeks_url: /?p=1649
geeks_visits:
  - 1987
geeks_ms_views:
  - 1216
categories:
  - Uncategorized

---
La verdad es que el tema de los enums y ASP.NET MVC da para hablar bastante (<a href="http://geeks.ms/blogs/etomas/archive/2013/06/12/asp-net-mvc-tratando-con-enums.aspx" target="_blank" rel="noopener noreferrer">yo mismo hice un post hace ni mucho</a>). Pero hace algunos días mi buen amigo y <a href="http://foursessions.techdencias.net/enero.html" target="_blank" rel="noopener noreferrer">a veces rival</a>, <a href="https://twitter.com/Marc_Rubino" target="_blank" rel="noopener noreferrer">Marc Rubiño</a> publicó en su blog un interesante artículo sobre como <a href="http://mrubino.net/2013/08/08/truco-asp-net-enums/" target="_blank" rel="noopener noreferrer">crear combos que mostrasen valores de enums</a>.

En este post voy a mostrar una técnica parecida, pero a través de las _data list_, un concepto nuevo de HTML5 que como pasa muchas veces está recibiendo menos atención de la que merece. Y es que las data list nos dan una manera fácil y sencilla de tener cajas de texto que se autocompleten.

**Qué son las data lists de HTML5?**

Bueno, pues como ya debes deducir de su nombre las data list no es nada más que la posibilidad de definir un concepto nuevo, que es esto: una lista de datos. 

Una lista de datos se usa como _fuente de datos_ para un control que pueda tener una y lo bueno es que el viejo y a veces injustamente denostado <input type=”text”> entran en esta categoría. Honestamente no entiendo como el <select> no tiene soporte para data lists. Quiero suponer que hay alguna explicación, pero la verdad no la encuentro. Bueno, da igual… ¡al tajo!

La definición de una data list es muy simple:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: gray"><</span><span style="color: #569cd6">datalist</span> <span style="color: #9cdcfe">id</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"sexo"</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">option</span> <span style="color: #9cdcfe">value</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"N"</span><span style="color: gray">></span>Nada<span style="color: gray"></</span><span style="color: #569cd6">option</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">option</span> <span style="color: #9cdcfe">value</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"P"</span><span style="color: gray">></span>Poco<span style="color: gray"></</span><span style="color: #569cd6">option</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">option</span> <span style="color: #9cdcfe">value</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"M"</span><span style="color: gray">></span>Mucho<span style="color: gray"></</span><span style="color: #569cd6">option</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">option</span> <span style="color: #9cdcfe">value</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"D"</span><span style="color: gray">></span>Demasiado<span style="color: gray"></</span><span style="color: #569cd6">option</span><span style="color: gray">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: gray"></</span><span style="color: #569cd6">datalist</span><span style="color: gray">></span>
  </p></p>
</div>

Ahora en HTML5 podemos usar el atributo _list_ de la etiqueta <input> para asociar a un <input> (p. ej. una caja de texto) una lista de valores:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: gray"><</span><span style="color: #569cd6">input</span> <span style="color: #9cdcfe">type</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"text"</span> <span style="color: #9cdcfe">id</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"txtSexo"</span> <span style="color: #9cdcfe">list</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"sexo"</span> <span style="color: #9cdcfe">placeholder</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"Sexo"</span> <span style="color: #9cdcfe">autocomplete</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"on"</span> <span style="color: gray">/></span>
  </p></p>
</div>

Básicamente tan solo se trata de usar el atributo list con el valor del id de la data list a usar.

El atributo id es requerido por Chrome (si no se aplica al <input> Chrome ignora el atributo list). Este es el resultado en Chrome, Opera e IE10:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; display: inline; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_23837276.png" width="244" height="84" />][1] 

Podemos ver las diferencias entre navegadores:

  * Chrome nos muestra el value a la izquierda y la descripción a la derecha 
  * Opera tan solo nos muestra el value 
  * IE10 tan solo nos muestra la descripción 

Sin duda el mejor mecanismo, en mi opinión, es el de Chrome, luego el de Opera y por último el de IE10. No he podido probar FF ni Safari por no tenerlos a mano en el momento de escribir el post.

Igual te sorprende que prefiera que el desplegable muestre N, P, M y D como hace Opera en lugar de Nada, Poco,… que muestra IE10. Eso es porque **el valor que se debe teclear en el textbox es el de value**. El valor de value es el que se almacena en el textbox, y por lo tanto es el que debe teclearse (a diferencia de la <select> que permite mostrar un valor y mandar otro el <input type=”file” /> no tiene esta opción).

Ten presente siempre que estamos usando un <input type=”text” /> por **lo que el usuario puede entrar lo que quiera**. No es como una <select> donde debe elegir un elemento de los que le indiquemos. Así pues los elementos de la data list son ¡más una sugerencia que una obligación!

**Un ejemplo.**

Veamos como podríamos usar esto con enums y MVC…

Lo primero es definirnos un enum:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">enum</span> <span style="color: #b8d7a3">Beers</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">Estrella_Damm</span>,
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">Voll_Damm</span>,
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">Moritz</span>,
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">Epidor</span>,
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">Mahou</span>,
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">Cruzcampo</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p></p>
</div>

Ahora vamos a extender la clase HtmlHelper con un método de extensión nuevo que
   
llamaremos TextBoxEnumerableFor:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">static</span> <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">HtmlHelperEnumExtensions</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">static</span> <span style="color: #b8d7a3">IHtmlString</span> <span style="color: white">TextBoxEnumerableFor</span><span style="color: #b4b4b4"><</span><span style="color: white">TModel</span>, <span style="color: white">TProperty</span><span style="color: #b4b4b4">></span>(<span style="color: #569cd6">this</span> <span style="color: #4ec9b0">HtmlHelper</span><span style="color: #b4b4b4"><</span>TModel<span style="color: #b4b4b4">></span> <span style="color: white">helper</span>, <span style="color: #4ec9b0">Type</span> <span style="color: white">enumerationType</span>, <span style="color: #4ec9b0">Expression</span><span style="color: #b4b4b4"><</span><span style="color: #4ec9b0">Func</span><span style="color: #b4b4b4"><</span>TModel, TProperty<span style="color: #b4b4b4">>></span> <span style="color: white">expression</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">httpContext</span> <span style="color: #b4b4b4">=</span> <span style="color: white">helper</span><span style="color: #b4b4b4">.</span><span style="color: white">ViewContext</span><span style="color: #b4b4b4">.</span><span style="color: white">HttpContext</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">html</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">StringBuilder</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">if</span> (<span style="color: white">httpContext</span><span style="color: #b4b4b4">.</span><span style="color: white">Items</span>[<span style="color: white">enumerationType</span><span style="color: #b4b4b4">.</span><span style="color: white">FullName</span>] <span style="color: #b4b4b4">==</span> <span style="color: #569cd6">null</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">httpContext</span><span style="color: #b4b4b4">.</span><span style="color: white">Items</span><span style="color: #b4b4b4">.</span><span style="color: white">Add</span>(<span style="color: white">enumerationType</span><span style="color: #b4b4b4">.</span><span style="color: white">FullName</span>, <span style="color: white">GenerateDataList</span>(<span style="color: white">enumerationType</span>, <span style="color: white">html</span>));
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">textbox</span> <span style="color: #b4b4b4">=</span> <span style="color: white">helper</span><span style="color: #b4b4b4">.</span><span style="color: white">TextBoxFor</span>(<span style="color: white">expression</span>, <span style="color: #569cd6">new</span> {<span style="color: white">list</span> <span style="color: #b4b4b4">=</span> <span style="color: white">enumerationType</span><span style="color: #b4b4b4">.</span><span style="color: white">FullName</span>});
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">html</span><span style="color: #b4b4b4">.</span><span style="color: white">AppendLine</span>(<span style="color: white">textbox</span><span style="color: #b4b4b4">.</span><span style="color: white">ToString</span>());
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: #4ec9b0">MvcHtmlString</span><span style="color: #b4b4b4">.</span><span style="color: white">Create</span>(<span style="color: white">html</span><span style="color: #b4b4b4">.</span><span style="color: white">ToString</span>());
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">private</span> <span style="color: #569cd6">static</span> <span style="color: #569cd6">string</span> <span style="color: white">GenerateDataList</span>(<span style="color: #4ec9b0">Type</span> <span style="color: white">enumerationType</span>, <span style="color: #4ec9b0">StringBuilder</span> <span style="color: white">html</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">id</span> <span style="color: #b4b4b4">=</span> <span style="color: white">enumerationType</span><span style="color: #b4b4b4">.</span><span style="color: white">FullName</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">html</span><span style="color: #b4b4b4">.</span><span style="color: white">AppendFormat</span>(<span style="color: #d69d85">@"<datalist id=""</span><span style="color: #80ff80">{0}</span><span style="color: #d69d85">"">"</span>, <span style="color: white">id</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">html</span><span style="color: #b4b4b4">.</span><span style="color: white">AppendLine</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">foreach</span> (<span style="color: #569cd6">var</span> <span style="color: white">item</span> <span style="color: #569cd6">in</span> <span style="color: #4ec9b0">Enum</span><span style="color: #b4b4b4">.</span><span style="color: white">GetNames</span>(<span style="color: white">enumerationType</span>))
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">html</span><span style="color: #b4b4b4">.</span><span style="color: white">AppendFormat</span>(<span style="color: #d69d85">@"<option></span><span style="color: #80ff80">{0}</span><span style="color: #d69d85"></option>"</span>, <span style="color: white">item</span><span style="color: #b4b4b4">.</span><span style="color: white">Replace</span>(<span style="color: #d69d85">&#8216;_&#8217;</span>, <span style="color: #d69d85">&#8216; &#8216;</span>));
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">html</span><span style="color: #b4b4b4">.</span><span style="color: white">AppendLine</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">html</span><span style="color: #b4b4b4">.</span><span style="color: white">AppendLine</s pan>(<span style="color: #d69d85">"</datalist>"</span>);</p> 
    
    <p style="margin: 0px">
      &#160;
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: white">id</span>;
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160; }
    </p></p></div> 
    
    <p>
      Aspectos a comentar de este código:
    </p>
    
    <ol>
      <li>
        Uso HttpContext.Items para “hacer un seguimiento” de si ya se ha creado una datalist. Esto es para no definir en el mismo HTML dos veces la misma datalist si se generan dos (o más) cajas de texto vinculadas al mismo enum.
      </li>
      <li>
        Sustituyo los guiones bajos (_) por un espacio. Eso debería tenerse en cuenta luego al recibir los datos.
      </li>
    </ol>
    
    <p>
      Este código es muy sencillo y realmente debería sobrecargarse el método para dar más opciones (p. ej. especificar atributos html adicionales que ahora no es posible). Pero como ejemplo, creo que sirve.
    </p>
    
    <p>
      Su uso es muy sencillo:
    </p>
    
    <div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
      <p style="margin: 0px">
        <span style="background: #ffffb3; color: black">@</span><span style="color: #569cd6">using</span> <span style="color: white">MvcApplication1</span>
      </p>
      
      <p style="margin: 0px">
        <span style="background: #ffffb3; color: black">@</span><span style="color: #569cd6">using</span> <span style="color: white">MvcApplication1</span><span style="color: #b4b4b4">.</span><span style="color: white">Models</span>
      </p>
      
      <p style="margin: 0px">
        <span style="background: #ffffb3; color: black">@model </span><span style="color: white">MvcApplication1</span><span style="color: #b4b4b4">.</span><span style="color: white">Models</span><span style="color: #b4b4b4">.</span><span style="color: #4ec9b0">BeerModel</span>
      </p>
      
      <p style="margin: 0px">
        &#160;
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
        Entra el nombre de tu cerveza preferida aquí:
      </p>
      
      <p style="margin: 0px">
        &#160;
      </p>
      
      <p style="margin: 0px">
        <span style="background: #ffffb3; color: black">@</span><span style="color: white">Html</span><span style="color: #b4b4b4">.</span><span style="color: white">TextBoxEnumerableFor</span>(<span style="color: #569cd6">typeof</span>(<span style="color: #b8d7a3">Beers</span>), <span style="color: white">m</span><span style="color: #b4b4b4">=></span><span style="color: white">m</span><span style="color: #b4b4b4">.</span><span style="color: white">BeerName</span>)
      </p></p>
    </div>
    
    <p>
      BeerName es una propiedad (string) del modelo BeerModel. Y este es el HTML generado:
    </p>
    
    <pre class="csharpcode">Entra el nombre de tu cerveza preferida aquí:
<span class="kwrd">&lt;</span><span class="html">datalist</span> <span class="attr">id</span><span class="kwrd">="MvcApplication1.Models.Beers"</span><span class="kwrd">&gt;</span>
<span class="kwrd">&lt;</span><span class="html">option</span><span class="kwrd">&gt;</span>Estrella Damm<span class="kwrd">&lt;/</span><span class="html">option</span><span class="kwrd">&gt;</span>
<span class="kwrd">&lt;</span><span class="html">option</span><span class="kwrd">&gt;</span>Voll Damm<span class="kwrd">&lt;/</span><span class="html">option</span><span class="kwrd">&gt;</span>
<span class="kwrd">&lt;</span><span class="html">option</span><span class="kwrd">&gt;</span>Moritz<span class="kwrd">&lt;/</span><span class="html">option</span><span class="kwrd">&gt;</span>
<span class="kwrd">&lt;</span><span class="html">option</span><span class="kwrd">&gt;</span>Epidor<span class="kwrd">&lt;/</span><span class="html">option</span><span class="kwrd">&gt;</span>
<span class="kwrd">&lt;</span><span class="html">option</span><span class="kwrd">&gt;</span>Mahou<span class="kwrd">&lt;/</span><span class="html">option</span><span class="kwrd">&gt;</span>
<span class="kwrd">&lt;</span><span class="html">option</span><span class="kwrd">&gt;</span>Cruzcampo<span class="kwrd">&lt;/</span><span class="html">option</span><span class="kwrd">&gt;</span>
<span class="kwrd">&lt;/</span><span class="html">datalist</span><span class="kwrd">&gt;</span>
<span class="kwrd">&lt;</span><span class="html">input</span> <span class="attr">id</span><span class="kwrd">="BeerName"</span>
    <span class="attr">list</span><span class="kwrd">="MvcApplication1.Models.Beers"</span>
    <span class="attr">name</span><span class="kwrd">="BeerName"</span>
    <span class="attr">type</span><span class="kwrd">="text"</span> <span class="attr">value</span><span class="kwrd">=""</span> <span class="kwrd">/&gt;</span></pre>
    
    <p>
      No generamos atributo value, en este caso dicho atributo toma el valor del propio texto del option.
    </p>
    
    <p>
      Y un pantallazo de como se ve en Opera:
    </p>
    
    <p>
      <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_54A2AD16.png"><img title="image" style="border-top: 0px; border-right: 0px; border-bottom: 0px; border-left: 0px; display: inline" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_5285AE4D.png" width="244" height="107" /></a>
    </p>
    
    <p>
      Puedes ver como se usa los elementos del enum para mostrar una sugerencia de autocompletado del textbox. Pero recuerda: el usuario <strong>puede entrar el valor que quiera</strong>.
    </p></p> 
    
    <p>
      Como digo, el código se puede mejorar mucho, pero como idea y ejemplo creo que es suficiente.
    </p>
    
    <p>
      Un saludo!
    </p>

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_7ED25AFE.png