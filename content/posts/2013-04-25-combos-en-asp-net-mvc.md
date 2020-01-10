---
title: Combos en ASP.NET MVC

author: eiximenis

date: 2013-04-25T17:21:48+00:00
geeks_url: /?p=1637
geeks_visits:
  - 10023
geeks_ms_views:
  - 8096
categories:
  - Uncategorized

---
Buenas! Una de las preguntas más referentes en ASP.NET MVC consiste en como crear combos, enlazarlas, etc, etc… La verdad es que la documentación sobre este punto es un poco difusa y dispersa así que intentaré en este post mostrar un poco las distintas formas que tenemos en ASP.NET MVC de crear combos.

Para ilustrar las distintas opciones partimos de una clase “Database” que simula un contexto de ORM. Es una clase que simplemente tiene dos listas (estáticas), una de ciudades (Cities) y otra de provincias (States). La definición de las clases City y State son:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">City</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">int</span> <span style="color: white">Id</span> { <span style="color: #569cd6">get</span>; <span style="color: #569cd6">set</span>; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">string</span> <span style="color: white">Name</span> { <span style="color: #569cd6">get</span>; <span style="color: #569cd6">set</span>; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">string</span> <span style="color: white">CP</span> { <span style="color: #569cd6">get</span>; <span style="color: #569cd6">set</span>; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">int</span> <span style="color: white">StateId</span> { <span style="color: #569cd6">get</span>; <span style="color: #569cd6">set</span>; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">State</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">int</span> <span style="color: white">Id</span> { <span style="color: #569cd6">get</span>; <span style="color: #569cd6">set</span>; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">string</span> <span style="color: white">Name</span> { <span style="color: #569cd6">get</span>; <span style="color: #569cd6">set</span>; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p></p>
</div>

**1. SelectListItem**

En HTML una combo (<select>) contiene una lista de opciones que siempre son clave y texto que se muestra (ambos alfanuméricos). Para representar esta información en ASP.NET MVC disponemos de la clase SelectListItem. SelectListItem nos permite almacenar la clave (Value) y el texto (Text), así como un valor booleano (Selected) que indica si este es el elemento seleccionado por defecto a la combo (se corresponde al atributo selected del tag <option>).

Una posible forma de usarlo sería así:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">var</span> <span style="color: white">items</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">List</span><span style="color: #b4b4b4"><</span><span style="color: #4ec9b0">SelectListItem</span><span style="color: #b4b4b4">></span>();
  </p>
  
  <p style="margin: 0px">
    <span style="color: white">items</span> <span style="color: #b4b4b4">=</span> <span style="color: #4ec9b0">Database</span><span style="color: #b4b4b4">.</span><span style="color: white">Cities</span><span style="color: #b4b4b4">.</span><span style="color: white">Select</span>(<span style="color: white">c</span> <span style="color: #b4b4b4">=></span> <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">SelectListItem</span>()
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">Text</span> <span style="color: #b4b4b4">=</span> <span style="color: white">c</span><span style="color: #b4b4b4">.</span><span style="color: white">Name</span>,
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">Value</span> <span style="color: #b4b4b4">=</span> <span style="color: white">c</span><span style="color: #b4b4b4">.</span><span style="color: white">Id</span><span style="color: #b4b4b4">.</span><span style="color: white">ToString</span>()
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; })<span style="color: #b4b4b4">.</span><span style="color: white">ToList</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    <span style="color: white">ViewBag</span><span style="color: #b4b4b4">.</span><span style="color: white">Cities</span> <span style="color: #b4b4b4">=</span> <span style="color: white">items</span>;
  </p>
  
  <p style="margin: 0px">
    <span style="color: #569cd6">return</span> <span style="color: white">View</span>();
  </p></p>
</div>

Obtenemos las ciudades y luego convertimos cada objeto “City” en un objeto SelectListItem. Finalmente guardamos esta lista de SelectListItem en el ViewBag.

Para mostrar esta combo basta con usar en la vista:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="background: #ffffb3; color: black">@</span><span style="color: white">Html</span><span style="color: #b4b4b4">.</span><span style="color: white">DropDownList</span>(<span style="color: #d69d85">"Cities"</span>)
  </p></p>
</div>

El nombre usado “Cities” es el nombre del campo usado en el ViewBag y donde se encuentra la lista de SelectListItem.

La combo generada tendrá un atributo name llamado Cities y esto es importante a la hora de recibir el valor seleccionado de la combo. **Hay una gran confusión en esto.** Una combo envia UN SOLO ELEMENTO al controlador: El ID del elemento seleccionado.

Veamos como podemos recibir la ciudad seleccionada:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    [<span style="color: #4ec9b0">HttpPost</span>]
  </p>
  
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #4ec9b0">ActionResult</span> <span style="color: white">Index</span>(<span style="color: #569cd6">string</span> <span style="color: white">Cities</span>)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">id</span> <span style="color: #b4b4b4">=</span> <span style="color: #4ec9b0">Int32</span><span style="color: #b4b4b4">.</span><span style="color: white">Parse</span>(<span style="color: white">Cities</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #608b4e">// Recuperamos la ciudad ==> Consulta a BBDD</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">city</span> <span style="color: #b4b4b4">=</span> <span style="color: #4ec9b0">Database</span><span style="color: #b4b4b4">.</span><span style="color: white">Cities</span><span style="color: #b4b4b4">.</span><span style="color: white">FirstOrDefault</span>(<span style="color: white">c</span> <span style="color: #b4b4b4">=></span> <span style="color: white">c</span><span style="color: #b4b4b4">.</span><span style="color: white">Id</span> <span
 style="color: #b4b4b4">==</span> <span style="color: white">id</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #608b4e">// Operamos con la ciudad</span>
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Dos cosas a destacar:

  1. Lo que recibimos en Cities NO es la lista de ciudades. Es el valor de la propiedad Value del SelectListItem seleccionado (en mi caso el ID de la ciudad seleccionada). 
  2. El nombre del parámetro (Cities) es el mismo que el nombre del campo del ViewBag (y el mismo que pusimos en la llamada a Html.DropDownList). 

**2. IEnumerable**

Tener que convertir siempre nuestros datos (en este caso una lista de ciudades) a una lista de SelectListItem es muy pesado. Por suerte hay una clase SelectList que hace esto por nosotros. Basta con pasarle el IEnumerable que queremos, el nombre de la propiedad que es la clave y el nombre de la propiedad que contiene el texto a mostrar.

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #4ec9b0">ActionResult</span> <span style="color: white">Index</span>()
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">items</span> <span style="color: #b4b4b4">=</span> <span style="color: #4ec9b0">Database</span><span style="color: #b4b4b4">.</span><span style="color: white">Cities</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: white">ViewBag</span><span style="color: #b4b4b4">.</span><span style="color: white">Cities</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">SelectList</span>(<span style="color: white">items</span>, <span style="color: #d69d85">"Id"</span>, <span style="color: #d69d85">"Name"</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: white">View</span>();
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

En este punto ViewBag.Cities contiene una SelectList (que implemente IEnumerable<SelectListItem>) y el resto del código ya es exactamente el mismo que antes.

**3. Otros orígenes de datos**

Hasta ahora en la vista hemos usado Html.DropDownList pasándole tan solo una cadena (Cities). Esta cadena determina:

  * El nombre del atributo name del <select> generado. Que a su vez es el nombre del parámetro cuando recibimos los datos 
  * El nombre del campo del ViewBag que tiene los elementos. 

Si no queremos que estos dos valores sean iguales, podemos espcificarle a Html.DropDown donde está el IEnumerable<SelectListItem> que contiene los datos de la combo. Así en la vista podríamos utilizar:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="background: #ffffb3; color: black">@</span><span style="color: white">Html</span><span style="color: #b4b4b4">.</span><span style="color: white">DropDownList</span>(<span style="color: #d69d85">"selectedCity"</span>, <span style="color: white">ViewBag</span><span style="color: #b4b4b4">.</span><span style="color: white">Cities</span> <span style="color: #569cd6">as</span> <span style="color: #b8d7a3">IEnumerable</span><span style="color: #b4b4b4"><</span><span style="color: #4ec9b0">SelectListItem</span><span style="color: #b4b4b4">></span>)
  </p></p>
</div>

Con esto le estamos diciendo que nos genere un <select> cuyo atributo name valga “selectedCity” y que los datos están en ViewBag.Cities.

Ahora cuando recibimos los datos debemos tener presente que el parámetro ya NO se llama Cities, si no selectedCity:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    [<span style="color: #4ec9b0">HttpPost</span>]
  </p>
  
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #4ec9b0">ActionResult</span> <span style="color: white">Index</span>(<span style="color: #569cd6">string</span> <span style="color: white">selectedCity</span>)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">id</span> <span style="color: #b4b4b4">=</span> <span style="color: #4ec9b0">Int32</span><span style="color: #b4b4b4">.</span><span style="color: white">Parse</span>(<span style="color: white">selectedCity</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #608b4e">// Recuperamos la ciudad ==> Consulta a BBDD</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">city</span> <span style="color: #b4b4b4">=</span> <span style="color: #4ec9b0">Database</span><span style="color: #b4b4b4">.</span><span style="color: white">Cities</span><span style="color: #b4b4b4">.</span><span style="color: white">FirstOrDefault</span>(<span style="color: white">c</span> <span style="color: #b4b4b4">=></span> <span style="color: white">c</span><span style="color: #b4b4b4">.</span><span style="color: white">Id</span> <span style="color: #b4b4b4">==</span> <span style="color: white">id</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #608b4e">// Operamos con la ciudad</span>
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

**4. HtmlDropDownListFor**

Este helper lía un poco porque tendemos a compararlo con el resto de helpers similares. Así, si yo hago Html.TextboxFor(m=>m.Name) me va a generar un Textbox vinculado a la propiedad Name del ViewModel de la vista. HtmlDropDownListFor también espera una propiedad del modelo pero NO es la propiedad que tiene los elementos, si no donde dejará el valor del elemento seleccionado.

Mucha gente se confunde y se cree que la propiedad que pasamos a Html.DropDownListFor es la propiedad que contiene los valores a mostrar. Imaginemos que tenemos el siguiente ViewModel:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">ComboCitiesViewModel</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #b8d7a3">IEnumerable</span><span style="color: #b4b4b4"><</span><span style="color: #4ec9b0">City</span><span style="color: #b4b4b4">></span> <span style="color: white">Cities</span> { <span style="color: #569cd6">get</span>; <span style="color: #569cd6">set</span>; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">int</span> <span style="color: white">SelectedCity</span> { <span style="color: #569cd6">get</span>; <span style="color: #569cd6">set</span>; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

La acción Index nos queda ahora de la siguiente forma:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #4ec9b0">ActionResult</span> <span style="color: white">Index</span>()
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">items</span> <span style="color: #b4b4b4">=</span> <span style="color: #4ec9b0">Database</span><span style="color: #b4b4b4">.</span><span style="color: white">Cities</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">vm</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">ComboCitiesViewModel</span>();
  </p>
  
  <p st
yle="margin: 0px">
    &#160;&#160;&#160; <span style="color: white">vm</span><span style="color: #b4b4b4">.</span><span style="color: white">Cities</span> <span style="color: #b4b4b4">=</span> <span style="color: white">items</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: white">View</span>(<span style="color: white">vm</span>);
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Para usar Html.DropDownListFor podemos hacerlo tal y como sigue:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="background: #ffffb3; color: black">@</span><span style="color: white">Html</span><span style="color: #b4b4b4">.</span><span style="color: white">DropDownListFor</span>(<span style="color: white">m</span><span style="color: #b4b4b4">=></span><span style="color: white">m</span><span style="color: #b4b4b4">.</span><span style="color: white">SelectedCity</span>, <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">SelectList</span>(<span style="color: white">Model</span><span style="color: #b4b4b4">.</span><span style="color: white">Cities</span>, <span style="color: #d69d85">"Id"</span>, <span style="color: #d69d85">"Name"</span>))
  </p></p>
</div>

Le paso DOS parámetros a Html.DropDownListFor:

  1. La propiedad del ViewModel que contendrá el valor seleccionado 
  2. El IEnumerable<SelectListItem> con los datos. Fijaos que en este caso dado que mi ViewModel contiene un IEnumerable<City> uso la clase SelectList que hemos visto antes para hacer la conversión. 

Para recibir los datos puedo declarar la siguiente acción:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    [<span style="color: #4ec9b0">HttpPost</span>]
  </p>
  
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #4ec9b0">ActionResult</span> <span style="color: white">Index</span>(<span style="color: #4ec9b0">ComboCitiesViewModel</span> <span style="color: white">info</span>)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">id</span> <span style="color: #b4b4b4">=</span> <span style="color: white">info</span><span style="color: #b4b4b4">.</span><span style="color: white">SelectedCity</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #608b4e">// Recuperamos la ciudad ==> Consulta a BBDD</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">city</span> <span style="color: #b4b4b4">=</span> <span style="color: #4ec9b0">Database</span><span style="color: #b4b4b4">.</span><span style="color: white">Cities</span><span style="color: #b4b4b4">.</span><span style="color: white">FirstOrDefault</span>(<span style="color: white">c</span> <span style="color: #b4b4b4">=></span> <span style="color: white">c</span><span style="color: #b4b4b4">.</span><span style="color: white">Id</span> <span style="color: #b4b4b4">==</span> <span style="color: white">id</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #608b4e">// Operamos con la ciudad</span>
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Y aquí es donde hay otro punto de confusión: en info NO VAS A RECIBIR la lista de ciudades. Es decir la propiedad Cities va a ser null:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_7D262225.png" width="504" height="70" />][1]

¿Y eso? Pues bueno, simple y llanamente nadie manda estos valores de vuelta para el controlador. La lista de ciudades NO está en la petición POST que hace el navegador y por lo tanto el controlador no la recibe. 

De hecho, podríamos modificar el parámetro ComboCitiesViewModel para que fuese un string llamado SelectedCity y funcionaría igual.

**5. Combos encadenadas**

Lo que vamos a ver es una implementación de combos encadenadas pero SIN ajax. Es decir, seleccionas provincia, envías la petición y te aparecen las ciudades. En una web “actual” seguramente se haría via Ajax, pero hacerlo de la “manera antigua” nos permitirá terminar de ver como funcionan las combos.

Antes que nada modificamos el viewmodel:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">ComboCitiesViewModel</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #b8d7a3">IEnumerable</span><span style="color: #b4b4b4"><</span><span style="color: #4ec9b0">City</span><span style="color: #b4b4b4">></span> <span style="color: white">Cities</span> { <span style="color: #569cd6">get</span>; <span style="color: #569cd6">set</span>; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">int</span> <span style="color: white">SelectedCity</span> { <span style="color: #569cd6">get</span>; <span style="color: #569cd6">set</span>; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #b8d7a3">IEnumerable</span><span style="color: #b4b4b4"><</span><span style="color: #4ec9b0">State</span><span style="color: #b4b4b4">></span> <span style="color: white">States</span> { <span style="color: #569cd6">get</span>; <span style="color: #569cd6">set</span>; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">int</span> <span style="color: white">SelectedState</span> { <span style="color: #569cd6">get</span>; <span style="color: #569cd6">set</span>; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

La acción Index que envía la página inicial la modificamos para que rellene States:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #4ec9b0">ActionResult</span> <span style="color: white">Index</span>()
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">items</span> <span style="color: #b4b4b4">=</span> <span style="color: #4ec9b0">Database</span><span style="color: #b4b4b4">.</span><span style="color: white">States</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">vm</span> <span style="color: #b4b4b4">=</span> <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">ComboCitiesViewModel</span>();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: white">vm</span><span style="color: #b4b4b4">.</span><span style="color: white">States</span> <span style="color: #b4b4b4">=</span> <span style="color: white">items</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: white">View</span>(<span style="color: white">vm</span>);
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

La vista nos quedará de la siguiente forma:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="background: #ffffb3; color: black">@model </span><span style="color: white">MvcCombos</s pan><span style="color: #b4b4b4">.</span><span style="color: white">Models</span><span style="color: #b4b4b4">.</span><span style="color: #4ec9b0">ComboCitiesViewModel</span></p> 
    
    <p style="margin: 0px">
      &#160;
    </p>
    
    <p style="margin: 0px">
      <span style="background: #ffffb3; color: black">@</span><span style="color: #569cd6">using</span> (<span style="color: white">Html</span><span style="color: #b4b4b4">.</span><span style="color: white">BeginForm</span>()) {
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">label</span> <span style="color: #9cdcfe">for</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"Cities"</span><span style="color: gray">></span>Ciudad:<span style="color: gray"></</span><span style="color: #569cd6">label</span><span style="color: gray">></span>
    </p>
    
    <p style="margin: 0px">
      &#160;
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="background: #ffffb3; color: black">@</span><span style="color: white">Html</span><span style="color: #b4b4b4">.</span><span style="color: white">DropDownListFor</span>(<span style="color: white">m</span><span style="color: #b4b4b4">=></span><span style="color: white">m</span><span style="color: #b4b4b4">.</span><span style="color: white">SelectedState</span>, <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">SelectList</span>(<span style="color: white">Model</span><span style="color: #b4b4b4">.</span><span style="color: white">States</span>, <span style="color: #d69d85">"Id"</span>, <span style="color: #d69d85">"Name"</span>))
    </p>
    
    <p style="margin: 0px">
      &#160;
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160; <span style="color: #569cd6">if</span> (<span style="color: white">Model</span><span style="color: #b4b4b4">.</span><span style="color: white">SelectedState</span> <span style="color: #b4b4b4">!=</span> <span style="color: #b5cea8"></span>)
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160; {
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="background: #ffffb3; color: black">@</span><span style="color: white">Html</span><span style="color: #b4b4b4">.</span><span style="color: white">DropDownListFor</span>(<span style="color: white">m</span><span style="color: #b4b4b4">=></span><span style="color: white">m</span><span style="color: #b4b4b4">.</span><span style="color: white">SelectedCity</span>, <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">SelectList</span>(<span style="color: white">Model</span><span style="color: #b4b4b4">.</span><span style="color: white">Cities</span>, <span style="color: #d69d85">"Id"</span>, <span style="color: #d69d85">"Name"</span>))
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160; }
    </p>
    
    <p style="margin: 0px">
      &#160;
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">input</span> <span style="color: #9cdcfe">type</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"submit"</span> <span style="color: #9cdcfe">value</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"enviar"</span> <span style="color: gray">/></span>
    </p>
    
    <p style="margin: 0px">
      }
    </p></p></div> 
    
    <p>
      Es importante entender lo que hacemos en la vista:
    </p>
    
    <ol>
      <li>
        Si NO hay provincia seleccionada (Model.SelectedState vale 0) entonces mostramos la primera combo para seleccionar estado.
      </li>
      <li>
        Si hay estado seleccionado entonces generamos la combo para seleccionar la ciudad.
      </li>
    </ol>
    
    <blockquote>
      <p>
        <strong>Nota:</strong> Este código tiene algunos problemas, como p.ej. que ocurre si el usuario selecciona una provincia, envía el formulario, y cuando aparece de nuevo la vista con las dos combos modifica la provincia seleccionada? En una aplicación real deberías, al menos, deshabilitar la combo de estados cuando ya haya estado seleccionado.
      </p>
    </blockquote>
    
    <p>
      Y finalmente ahora la acción que recibe los resultados debe gestionar que será llamada dos veces (una para seleccionar el estado, la segunda con estado y ciudad):
    </p>
    
    <div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
      <p style="margin: 0px">
        [<span style="color: #4ec9b0">HttpPost</span>]
      </p>
      
      <p style="margin: 0px">
        <span style="color: #569cd6">public</span> <span style="color: #4ec9b0">ActionResult</span> <span style="color: white">Index</span>(<span style="color: #4ec9b0">ComboCitiesViewModel</span> <span style="color: white">info</span>)
      </p>
      
      <p style="margin: 0px">
        {
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: #569cd6">if</span> (<span style="color: white">info</span><span style="color: #b4b4b4">.</span><span style="color: white">SelectedState</span> <span style="color: #b4b4b4">!=</span> <span style="color: #b5cea8"></span> <span style="color: #b4b4b4">&&</span> <span style="color: white">info</span><span style="color: #b4b4b4">.</span><span style="color: white">SelectedCity</span> <span style="color: #b4b4b4">==</span> <span style="color: #b5cea8"></span>)
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; {
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">info</span><span style="color: #b4b4b4">.</span><span style="color: white">States</span> <span style="color: #b4b4b4">=</span> <span style="color: #4ec9b0">Database</span><span style="color: #b4b4b4">.</span><span style="color: white">States</span>;
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">info</span><span style="color: #b4b4b4">.</span><span style="color: white">Cities</span> <span style="color: #b4b4b4">=</span> <span style="color: #4ec9b0">Database</span><span style="color: #b4b4b4">.</span><span style="color: white">Cities</span><span style="color: #b4b4b4">.</span><span style="color: white">Where</span>(<span style="color: white">c</span> <span style="color: #b4b4b4">=></span> <span style="color: white">c</span><span style="color: #b4b4b4">.</span><span style="color: white">StateId</span> <span style="color: #b4b4b4">==</span> <span style="color: white">info</span><span style="color: #b4b4b4">.</span><span style="color: white">SelectedState</span>);
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: white">View</span>(<span style="color: white">info</span>);
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; }
      </p>
      
      <p style="margin: 0px">
        &#160;
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">id</span> <span style="color: #b4b4b4">=</span> <span style="color: white">info</span><span style="color: #b4b4b4">.</span><span style="color: white">SelectedCity</span>;
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: #608b4e">// Recuperamos la ciudad ==> Consulta a BBDD</span>
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">city</span> <span style="color: #b4b4b4">=</span> <span style="color: #4ec9b0">Database</span><span style="color: #b4b4b4">.</span><span style="color: white">Cities</span><span style="color: #b4b4b4">.</span><span style="color: white">FirstOrDefault</span>(<span style="color: white">c</span> <span style="color: #b4b4b4">=></span> <span style="color: white">c</span><span style="color: #b4b4b4">.</span><span style="color: white">Id</span> <span style="color: #b4b4b4">==</span> <span style="color: white">id</span>);
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: #608b4e">// Operamos con la ciudad</span>
      </p>
      
      <p style="margin: 0px">
        }
      </p></p>
    </div>
    
    <p>
      Fíjate en un par de cosas:
    </p>
    
    <ol>
      <li>
        Debemos volver a cargar todos las provincias dentro del viewmodel. Si no cuando la vista intente renderizar la combo de provincias dará error.
      </li>
      <li>
        En las ciudades seleccionamos tan solo aquellas que son de la provin<br /> cia que el usuario ha seleccionado.
      </li>
    </ol>
    
    <p>
      Insisto, en una vista “real” la segunda vez no mostraríamos la combo de provincias, quizá mostraríamos el nombre de la ciudad seleccionada. Veamos como podríamos hacerlo.
    </p>
    
    <p>
      Por un lado podemos modificar el viewmodel:
    </p>
    
    <div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
      <p style="margin: 0px">
        <span style="color: #569cd6">public</span> <span style="color: #569cd6">class</span> <span style="color: #4ec9b0">ComboCitiesViewModel</span>
      </p>
      
      <p style="margin: 0px">
        {
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #b8d7a3">IEnumerable</span><span style="color: #b4b4b4"><</span><span style="color: #4ec9b0">City</span><span style="color: #b4b4b4">></span> <span style="color: white">Cities</span> { <span style="color: #569cd6">get</span>; <span style="color: #569cd6">set</span>; }
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">int</span> <span style="color: white">SelectedCity</span> { <span style="color: #569cd6">get</span>; <span style="color: #569cd6">set</span>; }
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #b8d7a3">IEnumerable</span><span style="color: #b4b4b4"><</span><span style="color: #4ec9b0">State</span><span style="color: #b4b4b4">></span> <span style="color: white">States</span> { <span style="color: #569cd6">get</span>; <span style="color: #569cd6">set</span>; }
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">int</span> <span style="color: white">SelectedState</span> { <span style="color: #569cd6">get</span>; <span style="color: #569cd6">set</span>; }
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: #569cd6">public</span> <span style="color: #569cd6">string</span> <span style="color: white">SelectedStateName</span> { <span style="color: #569cd6">get</span>; <span style="color: #569cd6">set</span>; }
      </p>
      
      <p style="margin: 0px">
        }
      </p></p>
    </div>
    
    <p>
      Añadimos la propiedad para guardar el nombre de la provincia seleccionada. Y en la vista usamos esta propiedad o Html.DropDownList en función de si hay o no provincia seleccionada:
    </p>
    
    <div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
      <p style="margin: 0px">
        <span style="background: #ffffb3; color: black">@model </span><span style="color: white">MvcCombos</span><span style="color: #b4b4b4">.</span><span style="color: white">Models</span><span style="color: #b4b4b4">.</span><span style="color: #4ec9b0">ComboCitiesViewModel</span>
      </p>
      
      <p style="margin: 0px">
        &#160;
      </p>
      
      <p style="margin: 0px">
        <span style="background: #ffffb3; color: black">@</span><span style="color: #569cd6">using</span> (<span style="color: white">Html</span><span style="color: #b4b4b4">.</span><span style="color: white">BeginForm</span>()) {
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">label</span> <span style="color: #9cdcfe">for</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"Cities"</span><span style="color: gray">></span>Ciudad:<span style="color: gray"></</span><span style="color: #569cd6">label</span><span style="color: gray">></span>
      </p>
      
      <p style="margin: 0px">
        &#160;
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: #569cd6">if</span> (<span style="color: white">Model</span><span style="color: #b4b4b4">.</span><span style="color: white">SelectedState</span> <span style="color: #b4b4b4">==</span> <span style="color: #b5cea8"></span>)
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; {
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="background: #ffffb3; color: black">@</span><span style="color: white">Html</span><span style="color: #b4b4b4">.</span><span style="color: white">DropDownListFor</span>(<span style="color: white">m</span> <span style="color: #b4b4b4">=></span> <span style="color: white">m</span><span style="color: #b4b4b4">.</span><span style="color: white">SelectedState</span>, <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">SelectList</span>(<span style="color: white">Model</span><span style="color: #b4b4b4">.</span><span style="color: white">States</span>, <span style="color: #d69d85">"Id"</span>, <span style="color: #d69d85">"Name"</span>))
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
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="background: #ffffb3; color: black"><text></span>Provincia <span style="background: #ffffb3; color: black">@</span><span style="color: white">Model</span><span style="color: #b4b4b4">.</span><span style="color: white">SelectedStateName</span> <span style="background: #ffffb3; color: black"></text></span>
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="background: #ffffb3; color: black">@</span><span style="color: white">Html</span><span style="color: #b4b4b4">.</span><span style="color: white">DropDownListFor</span>(<span style="color: white">m</span><span style="color: #b4b4b4">=></span><span style="color: white">m</span><span style="color: #b4b4b4">.</span><span style="color: white">SelectedCity</span>, <span style="color: #569cd6">new</span> <span style="color: #4ec9b0">SelectList</span>(<span style="color: white">Model</span><span style="color: #b4b4b4">.</span><span style="color: white">Cities</span>, <span style="color: #d69d85">"Id"</span>, <span style="color: #d69d85">"Name"</span>))
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; }
      </p>
      
      <p style="margin: 0px">
        &#160;
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: gray"><</span><span style="color: #569cd6">input</span> <span style="color: #9cdcfe">type</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"submit"</span> <span style="color: #9cdcfe">value</span><span style="color: #b4b4b4">=</span><span style="color: #c8c8c8">"enviar"</span> <span style="color: gray">/></span>
      </p>
      
      <p style="margin: 0px">
        }
      </p></p>
    </div>
    
    <p>
      Finalmente en la acción que recibe los datos nos ahorramos de rellenar de nuevo las provincias (ya que la vista ya no las usará la segunda vez) y ponemos el nombre de la provincia seleccionada (fíjate que hemos de ir a buscarlo a la BBDD ya que solo tenemos el ID):
    </p>
    
    <div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
      <p style="margin: 0px">
        [<span style="color: #4ec9b0">HttpPost</span>]
      </p>
      
      <p style="margin: 0px">
        <span style="color: #569cd6">public</span> <span style="color: #4ec9b0">ActionResult</span> <span style="color: white">Index</span>(<span style="color: #4ec9b0">ComboCitiesViewModel</span> <span style="color: white">info</span>)
      </p>
      
      <p style="margin: 0px">
        {
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: #569cd6">if</span> (<span style="color: white">info</span><span style="color: #b4b4b4">.</span><span style="color: white">SelectedState</span> <span style="color: #b4b4b4">!=</span> <span style="color: #b5cea8"></span> <span style="color: #b4b4b4">&&</span> <span style="color: white">info</span><span style="color: #b4b4b4">.</span><span style="color: white">SelectedCity</span> <span style="color: #b4b4b4">==</span> <span style="color: #b5cea8"></span>)
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; {
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">info</span><span style="color: #b4b4b4">.</span><span style="color: white">SelectedStateName</span> <span style="color: #b4b4b4">=</span> <span style="color: #4ec9b0">Database</span><span style="color: #b4b4b4">.</span
><span style="color: white">States</span><span style="color: #b4b4b4">.</span><span style="color: white">Single</span>(<span style="color: white">s</span> <span style="color: #b4b4b4">=></span> <span style="color: white">s</span><span style="color: #b4b4b4">.</span><span style="color: white">Id</span> <span style="color: #b4b4b4">==</span> <span style="color: white">info</span><span style="color: #b4b4b4">.</span><span style="color: white">SelectedState</span>)<span style="color: #b4b4b4">.</span><span style="color: white">Name</span>;
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: white">info</span><span style="color: #b4b4b4">.</span><span style="color: white">Cities</span> <span style="color: #b4b4b4">=</span> <span style="color: #4ec9b0">Database</span><span style="color: #b4b4b4">.</span><span style="color: white">Cities</span><span style="color: #b4b4b4">.</span><span style="color: white">Where</span>(<span style="color: white">c</span> <span style="color: #b4b4b4">=></span> <span style="color: white">c</span><span style="color: #b4b4b4">.</span><span style="color: white">StateId</span> <span style="color: #b4b4b4">==</span> <span style="color: white">info</span><span style="color: #b4b4b4">.</span><span style="color: white">SelectedState</span>);
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: white">View</span>(<span style="color: white">info</span>);
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; }
      </p>
      
      <p style="margin: 0px">
        &#160;
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">id</span> <span style="color: #b4b4b4">=</span> <span style="color: white">info</span><span style="color: #b4b4b4">.</span><span style="color: white">SelectedCity</span>;
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: #608b4e">// Recuperamos la ciudad ==> Consulta a BBDD</span>
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: #569cd6">var</span> <span style="color: white">city</span> <span style="color: #b4b4b4">=</span> <span style="color: #4ec9b0">Database</span><span style="color: #b4b4b4">.</span><span style="color: white">Cities</span><span style="color: #b4b4b4">.</span><span style="color: white">FirstOrDefault</span>(<span style="color: white">c</span> <span style="color: #b4b4b4">=></span> <span style="color: white">c</span><span style="color: #b4b4b4">.</span><span style="color: white">Id</span> <span style="color: #b4b4b4">==</span> <span style="color: white">id</span>);
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: #608b4e">// Operamos con la ciudad</span>
      </p>
      
      <p style="margin: 0px">
        }
      </p></p>
    </div>
    
    <p>
      Con esta aproximación:
    </p>
    
    <ol>
      <li>
        La primera vez la vista mostrará la combo de provincias y el controlador recibirá en SelectedState el id de la provincia seleccionada.
      </li>
      <li>
        La segunda vez la vista mostrará un texto con el nombre de la provincia y la combo de ciudades y el controlador recibirá en SelectedCity el ID de la ciudad seleccionada. Esta segunda vez el controlador NO recibirá SelectedState ya que no se envía. En nuestro caso no es necesario ya que lo podemos sacar de la ciudad. Si fuese necesario deberíamos usar un campo Hidden.
      </li>
    </ol>
    
    <p>
      Bueno… creo que esto más o menos es todo. ¡Espero que este post os ayude a resolver las dudas que podáis tener con las combos en ASP.NET MVC!
    </p>
    
    <p>
      ¡Un saludo!
    </p>

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_4CDF4D6F.png