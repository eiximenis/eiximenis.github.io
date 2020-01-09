---
title: 'Y el combate se decició por KO (v): Filtrando colecciones'
description: 'Y el combate se decició por KO (v): Filtrando colecciones'
author: eiximenis

date: 2012-10-03T15:12:00+00:00
geeks_url: /?p=1611
geeks_visits:
  - 1401
geeks_ms_views:
  - 612
categories:
  - Uncategorized

---
Muy buenas! Tras un parón, volvemos a la carga con la serie que trata sobre knockoutjs. Recuerda que puedes ver <a target="_blank" href="/blogs/etomas/archive/tags/knockout/default.aspx" rel="noopener noreferrer">todos los artículos de esta serie</a>.

En este post vamos a ver como filtrar colecciones con knockout.

Como siempre comenzamos con una clase Beer y un controlador de WebApi que devuelva una colección de elementos Beer. La clase Beer es tal como sigue:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: blue">public</span> <span style="color: blue">class</span> <span style="color: #2b91af">Beer</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">public</span> <span style="color: blue">string</span> Name { <span style="color: blue">get</span>; <span style="color: blue">set</span>; }
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">public</span> <span style="color: blue">string</span> Brewery { <span style="color: blue">get</span>; <span style="color: blue">set</span>; }
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">public</span> <span style="color: blue">int</span> Ibu { <span style="color: blue">get</span>; <span style="color: blue">set</span>; }
  </p>
  
  <p style="margin: 0px">
    }
  </p>
</div>

Bien, ahora vamos a realizar una vista (Home/Index) que me muestre el listado de cervezas. Para ello creamos una vista como la que sigue:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: blue"><</span><span style="color: maroon">script</span> <span style="color: red">type</span><span style="color: blue">=&#8221;text/javascript&#8221;></span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp; $(document).ready(<span style="color: blue">function</span>() {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; $.getJSON(<span style="color: #a31515">&#8220;</span><span style="background: yellow">@</span>Url.RouteUrl(<span style="color: #a31515">&#8220;DefaultApi&#8221;</span>, <span style="color: blue">new</span> {httproute=<span style="color: #a31515">&#8220;&#8221;</span>, controller=<span style="color: #a31515">&#8220;Beers&#8221;</span>})<span style="color: #a31515">&#8220;</span>, <span style="color: blue">function</span>(data) {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; CreateViewModel(data);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; });
  </p>
  
  <p style="margin: 0px">
    &nbsp;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp; });
  </p>
  
  <p style="margin: 0px">
    &nbsp;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">function</span> CreateViewModel(data) {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> vm = { beers:data };
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; window.vm = vm;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ko.applyBindings(vm);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp; }
  </p>
  
  <p style="margin: 0px">
    &nbsp;<span style="color: blue"></</span><span style="color: maroon">script</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"><</span><span style="color: maroon">table</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue"><</span><span style="color: maroon">thead</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue"><</span><span style="color: maroon">th</span><span style="color: blue">></span>Nombre<span style="color: blue"></</span><span style="color: maroon">th</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue"><</span><span style="color: maroon">th</span><span style="color: blue">></span>Ibus<span style="color: blue"></</span><span style="color: maroon">th</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue"><</span><span style="color: maroon">th</span><span style="color: blue">></span>Cervecería<span style="color: blue"></</span><span style="color: maroon">th</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue"></</span><span style="color: maroon">thead</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue"><</span><span style="color: maroon">tbody</span> <span style="color: red">data-bind</span><span style="color: blue">=&#8221;foreach: beers&#8221;></span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue"><</span><span style="color: maroon">tr</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue"><</span><span style="color: maroon">td</span> <span style="color: red">data-bind</span><span style="color: blue">=&#8221;text: Name&#8221;></</span><span style="color: maroon">td</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue"><</span><span style="color: maroon">td</span> <span style="color: red">data-bind</span><span style="color: blue">=&#8221;text: Ibu&#8221;></</span><span style="color: maroon">td</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue"><</span><span style="color: maroon">td</span> <span style="color: red">data-bind</span><span style="color: blue">=&#8221;text: Brewery&#8221;></</span><span style="color: maroon">td</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue"></</span><span style="color: maroon">tr</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue"></</span><span style="color: maroon">tbody</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    <span style="color: blue"></</span><span style="color: maroon">table</span><span style="color: blue">></span>
  </p>
</div>

Nada nuevo hasta ahora, pero con eso ya vemos nuestra tabla de cervezas:

[<img height="184" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_7FC5B42D.png" alt="image" border="0" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" title="image" />][1]

Ahora vamos a jugar un poco con esos datos en cliente usando knockout. Para ello vamos a implementar unos filtros para filtrar los datos directamente desde cliente.

Así pues, vamos a introducir un texbox donde se introducirá el nombre de la cervecería y nos filtrará los datos. El código del textbox es:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: blue"><</span><span style="color: maroon">input</span> <span style="color: red">type</span><span style="color: blue">=&#8221;text&#8221;</span> <span style="color: red">data-bind</span><span style="color: blue">=&#8221;value: brewerySelected, valueUpdate: &#8216;afterkeydown'&#8221;</span> <span style="color: blue">/></span>
  </p>
</div>

Aquí enlazamos la propiedad value del textbox con la propiedad &ldquo;bewerySelected&rdquo; del viewmodel (que deberemos crear) y además le indicamos **cuando debe modificarse el valor de la propiedad del viewmodel.** El valor de valueUpdate es el nombre del evento que knockout usará para modificar la propiedad del viewmodel. En este caso le indicamos &lsquo;afterkeydown&rsquo; para que nos actualice la propiedad del viewmodel cada vez que pulsemos una tecla. Si no ponemos valueUpdate el evento usado es &lsquo;change&rsquo; (en el caso de los textboxs se lanza cuando pierden el foco).

Ahora añadimos esta propiedad a nuestrov viewmodel:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: blue">var</span> vm = {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; beers: data,
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; brewerySelected : ko.observable(<span style="color: #a31515">&#8220;&#8221;</span>)
  </p>
  
  <p style="margin: 0px">
    };
  </p>
</div>

Es importante destacar que brewerySelected **se declara como un observable**. Pero &iexcl;ojo! eso NO es para que desde el textbox pueda modificarse dicha propiedad (eso viene de serie). Necesitamos que sea un observable porque vamos a necesitar _que la propia knockout sepa cuando se ha modificado dicha propiedad_. La razón es que vamos a crear un observable calculado que depende de dicha propiedad y nos interesa que knockout refresque el valor del observable calculado cuando el valor de brewerySelected cambie. De ahí que necesitemos que sea un observable.

¿Y cual va a ser el valor calculado? Pues la lista de cervezas cuya cervecería empieza por el valor que haya en brewerySelected:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    vm.filteredByBrewery = ko.computed(<span style="color: blue">function</span>() {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">var</span> filter = <span style="color: blue">this</span>.brewerySelected().toLowerCase();
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">if</span> (!filter) <span style="color: blue">return</span> <span style="color: blue">this</span>.beers;
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; <span style="color: blue">return</span> ko.utils.arrayFilter(<span style="color: blue">this</span>.beers, <span style="color: blue">function</span> (item) {
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span style="color: blue">return</span> ko.utils.stringStartsWith(item.Brewery.toLowerCase(), filter);
  </p>
  
  <p style="margin: 0px">
    &nbsp;&nbsp;&nbsp; });
  </p>
  
  <p style="margin: 0px">
    }, vm);
  </p>
</div>

Fijaos que usamos ko.computed (que ya vimos) para definir el observable calculado. En este caso dicho observable es la colección de elementos cuya propiedad Brewery empieza por el valor que haya en brewerySelected. Para implementarla hacemos uso de dos métodos propios de knockout:

  1. ko.utils.arrayFilter: Filtra los elementos de un array por aquellos que cumplan el predicado que se le pasa. Dicho predicado es una función que se llama por todos elementos del array y debe devolver true o false según el elemento deba ser incluído o no. 
  2. ko.utils.stringStartsWith: Devuelve si una cadena empieza por otra indicada. 

Y casi listos! Tan solo nos queda hacer una pequeña modificación, que es enlazar el <tbody> a la propiedad filteredByBrewery (en lugar de a la propiedad beers):

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: blue"><</span><span style="color: maroon">tbody</span> <span style="color: red">data-bind</span><span style="color: blue">=&#8221;foreach: filteredByBrewery&#8221;></span>
  </p>
</div>

&iexcl;Y listos, podemos ver que a medida que vayamos escribiendo la lista se va filtrando!

[<img height="181" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_11A4D8AC.png" alt="image" border="0" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; border-left: 0px; display: inline; padding-right: 0px" title="image" />][2][<img height="151" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_47A957B9.png" alt="image" border="0" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" title="image" />][3]

La clave es, recordad, en declarar selectedBrewery como observable. Eso hace que cuando se modifique knockout recalcule el observable calculado filteredByBrewery. Y este último al ser un observable cuando es recalculado forza una modificación de la UI.

Ahora a lo mejor alguien se está haciendo la pregunta: ¿Como sabe knockout que debe recalcular el observable calculado filterByBrewery al modificarse el observable selectedBrewery? Pues simple y llanamente porque _desde el código de la función de filterByBrewery se llama a selectedBrewery_. A mi personalmente que sea capaz de llegar a ese nivel me parece brutal! Si no te lo crees prueba de añadir otro observable, enlázalo a un cuadro de texto y observa que al modificar este segundo observable el observable calculado filterByBrewery no es recalculado (puedes comprobarlo poniendo un breakpoint con cualquiera de las herramientas de debug de javascript). Es decir knockout sabe de que observables depende cada observable calculado y así solo recalcularlos cuando es necesario... &iexcl;Simplemente fantástico!

Os dejo el código fuente del proyecto en mi skydrive: [https://skydrive.live.com/redir?resid=6521C259E9B1BEC6!314][4] (es el fichero KoDemoV.zip).

Saludos!

 [1]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_47EE4EA8.png
 [2]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_3AACE6F7.png
 [3]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_617D8DE8.png
 [4]: https://skydrive.live.com/redir?resid=6521C259E9B1BEC6!314 "https://skydrive.live.com/redir?resid=6521C259E9B1BEC6!314"