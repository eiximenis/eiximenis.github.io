---
title: Y el combate se decidió por KO (ii)

author: eiximenis

date: 2012-08-01T13:36:00+00:00
geeks_url: /?p=1607
geeks_visits:
  - 2420
geeks_ms_views:
  - 757
categories:
  - Uncategorized

---
Como indica el título del post, ese es el segundo post de la serie que he empezado sobre knockout. Honestamente no sé cuantos posts habrá ni donde me (nos) llevará, pero espero que os sea útil!

En el [post anterior (el primero)][1] vimos un poco que era knockout y como mostrar datos devueltos a partir de un servicio REST implementado con WebApi. Ahora toca ir un poco más allá...

**Formulario que te quiero formulario**

La web está llena de formularios. Están en todos los sitios: para login y password, para darse de alta en un sitio, para solicitar información, para reservar tus vacaciones a Playa Bávaro... Los formularios son el mecanismo más sencillo para enviar información desde el cliente (navegador) al servidor. Usualmente (aunque ello no es obligatorio) se envían via POST, es decir conteniendo sus datos en el cuerpo de la petición, en lugar de GET donde los datos deben ir en la querystring. La razón es que los datos de un formulario pueden ser abritrariamente largos y podríamos tener URLs demasiado largas. Aunque realmente la especificación de http no impone un límite a las URLs (<a href="http://www.faqs.org/rfcs/rfc2616.html" target="_blank" rel="noopener noreferrer">ver RFC2616</a> punto 3.2.1), también deja claro que no tienen porque ser &ldquo;arbitrariamente largas&rdquo;. Es decir, cada servidor puede establecer su tamaño máximo de URL y devolver un error 414 en caso de que la URL enviada por el user-agent sea demasiado larga. Y luego están los propios user-agents claro. Cada uno de ellos puede &ldquo;unilateralmente&rdquo; truncar la URL o rechazar peticiones si la URL excede cierto tamaño.

Cuando se envía un formulario via POST se usa por defecto el content-type <a href="http://www.w3.org/TR/html401/interact/forms.html#h-17.13.4.1" target="_blank" rel="noopener noreferrer">application/x-www-form-urlencoded</a> que básicamente significa codificar (en el cuerpo de la petición) todos los parámetros del formulario en la codificación clásica de <span style="font-family: 'Courier New'; color: #004080;">nombre=valor&nombre2=valor2&</span>...

Es una práctica (relativamente) habitual cuando se crean APIs REST que si alguien tiene que enviar datos a dicha API (sea a través de POST o PUT p.ej.) no codifique dichos datos usando el content-type _application/x-www-formurlencoded_ sino que codifique dichos datos usando algún otro formato (como p. ej. JSON). Esto es por desligarnos de &ldquo;los formularios&rdquo; (una API no trabaja con formularios, trabaja con datos) y también por coherencia: si mi API devuelve datos en JSON lo suyo es que los acepte también en este formato.

Bien, vamos a crear un formulario de edición. Pero esta vez no haremos un formulario normal, enviado via POST a través del content-type _application/x-www-form-urlencoded_, sino que vamos a enviar el contenido de este formulario _a través de un objeto JSON_ (content-type _application/json_)_&nbsp;_enviado por POST. Por supuesto dicho formulario será tratado por un servicio REST que crearemos con WebApi.

Primero lo haremos &ldquo;a la vieja usanza&rdquo; es decir usando jQuery para crear el objeto JSON y luego haremos que entre en escena knockout.

**A la vieja usanza...**

Primero vamos a crear el método de nuestra API que nos devuelva una cerveza según su ID. Así añadimos al controlador ApiBeersController el método GetById:

<div id="codeSnippetWrapper" style="overflow: auto; cursor: text; font-size: 8pt; font-family: 'Courier New', courier, monospace; direction: ltr; text-align: left; margin: 20px 0px 10px; line-height: 12pt; max-height: 200px; width: 97.5%; background-color: #f4f4f4; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> Beer GetById(<span style="color: #0000ff">int</span> id)<br />{<br />    <span style="color: #0000ff">return</span> <span style="color: #0000ff">new</span> Beer {Name = <span style="color: #006080">"Imperial Stout #"</span> + id, Abv = 10.0M, Ibu = 120};<br />}</pre>
</div>

Ahora toca crear la vista para edición. Dicha vista será retornada por el controlador Beers:

<div id="codeSnippetWrapper" style="overflow: auto; cursor: text; font-size: 8pt; font-family: 'Courier New', courier, monospace; direction: ltr; text-align: left; margin: 20px 0px 10px; line-height: 12pt; max-height: 200px; width: 97.5%; background-color: #f4f4f4; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> ActionResult Edit(<span style="color: #0000ff">int</span>? id)<br />{<br />    ViewBag.BeerId = id ?? 0;<br />    <span style="color: #0000ff">return</span> View();<br />}</pre>
</div>

Bien, vayamos a lo que importa, el código de la vista:

<div id="codeSnippetWrapper" style="overflow: auto; cursor: text; font-size: 8pt; font-family: 'Courier New', courier, monospace; direction: ltr; text-align: left; margin: 20px 0px 10px; line-height: 12pt; max-height: 200px; width: 97.5%; background-color: #f4f4f4; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;">@{<br />    ViewBag.Title = <span style="color: #006080">"Edit Beer "</span> + ViewBag.BeerId;<br />}<br /><br />&lt;script type=<span style="color: #006080">"text/javascript"</span>&gt;<br />    $(document).ready(<span style="color: #0000ff">function</span> () {<br />        <span style="color: #0000ff">var</span> uri = <span style="color: #006080">"@Url.RouteUrl("</span>DefaultApi<span style="color: #006080">", new {httproute="</span><span style="color: #006080">", controller="</span>ApiBeers<span style="color: #006080">", id=ViewBag.BeerId})"</span>;<br />        $.getJSON(uri, <span style="color: #0000ff">function</span>(data) {<br />            $(<span style="color: #006080">"#Name"</span>).val(data.Name);<br />            $(<span style="color: #006080">"#Abv"</span>).val(data.Abv);<br />            $(<span style="color: #006080">"#Ibu"</span>).val(data.Ibu);<br />        });<br /><br />        $(<span style="color: #006080">"#frm"</span>).submit(<span style="color: #0000ff">function</span>(evt) {<br />            <span style="color: #008000">// Crea el objeto JSON a partir de los datos del formulario</span><br />            <span style="color: #0000ff">var</span> beer = { };<br />            beer.Name = $(<span style="color: #006080">"#Name"</span>).val();<br />            beer.Abv = $(<span style="color: #006080">"#Abv"</span>).val();<br />            beer.Ibu = $(<span style="color: #006080">"#Ibu"</span>).val();<br />            <span style="color: #0000ff">var</span> jsonBeer = JSON.stringify(beer);<br />            <span style="color: #008000">// Enviamos el objeto json</span><br />            <span style="color: #0000ff">var</span> uriEdit = <span style="color: #006080">"@Url.RouteUrl("</span>DefaultApi<span style="color: #006080">", new {httproute="</span><span style="color: #006080">", controller="</span>ApiBeers<span style="color: #006080">"})"</span>;<br />            $.ajax({<br />                url: uriEdit,<br />                dataType: <span style="color: #006080">'json'</span>,<br />                contentType: <span style="color: #006080">'application/json'</span>,<br />                type: <span style="color: #006080">'post'</span>,<br />                data: jsonBeer<br />            }).done(<span style="color: #0000ff">function</span>(data) {<br />                alert(<span style="color: #006080">"Beer "</span> + data.Id + <span style="color: #006080">" has been edited "</span> + data.Status);<br />            });<br />                <br />            evt.preventDefault();<br />        });<br />    });<br /><br />&lt;/script&gt;<br /><br />&lt;h2&gt;Edit a beer! ;-)&lt;/h2&gt;<br /><br />&lt;form method=<span style="color: #006080">"POST"</span> id=<span style="color: #006080">"frm"</span>&gt;<br />&lt;label <span style="color: #0000ff">for</span>=<span style="color: #006080">"Name"</span>&gt;Name:&lt;/label&gt;<br />&lt;input type=<span style="color: #006080">"text"</span> name=<span style="color: #006080">"Name"</span> id=<span style="color: #006080">"Name"</span> /&gt;<br />&lt;br /&gt;<br /><br />&lt;label <span style="color: #0000ff">for</span>=<span style="color: #006080">"Name"</span>&gt;Abv:&lt;/label&gt;<br />&lt;input type=<span style="color: #006080">"text"</span> name=<span style="color: #006080">"Abv"</span> id=<span style="color: #006080">"Abv"</span> /&gt;<br />&lt;br /&gt;<br /><br />&lt;label <span style="color: #0000ff">for</span>=<span style="color: #006080">"Name"</span>&gt;Ibu&lt;/label&gt;<br />&lt;input type=<span style="color: #006080">"text"</span> name=<span style="color: #006080">"Name"</span> id=<span style="color: #006080">"Ibu"</span> /&gt;<br />&lt;br /&gt;<br /><br />&lt;input type=<span style="color: #006080">"submit"</span> value=<span style="color: #006080">"edit!"</span> /&gt;<br />&lt;/form&gt;</pre>
</div>

Si lo analizamos vemos que hace lo siguiente:

  1. Nada más cargarse la página, llama a la API al método de obtener una cerveza usando $.getJSON. Una vez recibe la cerveza rellena los campos del formulario. 
  2. Se suscribe al evento &ldquo;submit&rdquo; del formulario. En dicho evento: 
      1. Crea un objeto beer y lo rellena con los valores del formulario 
      2. Crea la cadena en formato json de dicho objeto (usando JSON.stringify). 
      3. Finalmente usa $.ajax para enviar la petición POST hacia la API de editar cervezas pasándole la cerveza a editar. 
          1. Una vez la petición ajax se haya realizado muestra un alert con la información devuelta por el servidotr (que a su vez es otro objeto JSON). 

Veamos la información que se manda a través de la red:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; display: inline; border-top-width: 0px" border="0" alt="image" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_661C456C.png" width="521" height="155" />][2]&nbsp;

Vemos tres peticiones:

  1. GET a /beers/edit/10. Es la que termina llamando la acción Edit del BeersController y devuelve la vista. 
  2. GET a /api/apibeers/10. Es la que termina llamando a la acción GetById del ApiBeersController 
  3. POST a /api/apibeers. Es la que termina llamando a la acción Post del ApiBeersController. 

La siguiente imágen muestra la respuesta de la segunda petición, que son los datos de la cerveza en JSON que envía el servidor:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; display: inline; border-top-width: 0px" border="0" alt="image" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_7C08486D.png" width="495" height="144" />][3]

Y esta otra imagen muestra los datos enviados en la tercera petición (el $.ajax). Fijaos como el &ldquo;request payload&rdquo; es un json y como el content-type es application/json:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; display: inline; border-top-width: 0px" border="0" alt="image" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_212AB43C.png" width="491" height="138" />][4] 

Y la respuesta recibida del servidor es el objeto JSON que devuelve la acción Post:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; display: inline; border-top-width: 0px" border="0" alt="image" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_1048A0FD.png" width="493" height="128" />][5] 

Bien, ahora vamos a mantener toda la infraestructura de servidor (no vamos a tocar ningún controlador) pero vamos a usar knockout en la vista.

**Desplegamos el poder de knockout**

En el post anterior vimos como usar data-bind para enlazar una propiedad de nuestro viewmodel a un elemento del DOM. Ahora básicamente vamos a hacer lo mismo. Tendremos un viewmodel con 3 propiedades (Name, Abv, Ibu) y lo enlazaremos a los 3 controles del formulario:

<div id="codeSnippetWrapper" style="overflow: auto; cursor: text; font-size: 8pt; font-family: 'Courier New', courier, monospace; direction: ltr; text-align: left; margin: 20px 0px 10px; line-height: 12pt; max-height: 200px; width: 97.5%; background-color: #f4f4f4; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #0000ff">&lt;</span><span style="color: #800000">form</span> <span style="color: #ff0000">method</span><span style="color: #0000ff">="POST"</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="frm"</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">label</span> <span style="color: #ff0000">for</span><span style="color: #0000ff">="Name"</span><span style="color: #0000ff">&gt;</span>Name:<span style="color: #0000ff">&lt;/</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="Name"</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="Name"</span> <span style="color: #ff0000">data-bind</span><span style="color: #0000ff">="value: Name"</span> <span style="color: #0000ff">/&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">br</span> <span style="color: #0000ff">/&gt;</span><br /><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">label</span> <span style="color: #ff0000">for</span><span style="color: #0000ff">="Name"</span><span style="color: #0000ff">&gt;</span>Abv:<span style="color: #0000ff">&lt;/</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="Abv"</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="Abv"</span> <span style="color: #ff0000">data-bind</span><span style="color: #0000ff">="value: Abv"</span> <span style="color: #0000ff">/&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">br</span> <span style="color: #0000ff">/&gt;</span><br /><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">label</span> <span style="color: #ff0000">for</span><span style="color: #0000ff">="Name"</span><span style="color: #0000ff">&gt;</span>Ibu<span style="color: #0000ff">&lt;/</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="Name"</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="Ibu"</span> <span style="color: #ff0000">data-bind</span><span style="color: #0000ff">="value: Ibu"</span> <span style="color: #0000ff">/&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">br</span> <span style="color: #0000ff">/&gt;</span><br /><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="submit"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="edit!"</span> <span style="color: #0000ff">/&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">form</span><span style="color: #0000ff">&gt;</span></pre>
</div>

Fijaos que ahora usamos data-bind=&rdquo;value: xxx&rdquo; para indicar que lo que estamos enlazando es el valor de la propiedad &ldquo;value&rdquo; del objeto del DOM al valor de la propiedad xxx de nuestro viewmodel.

Ahora ha llegado el momento de crear nuestro viewmodel. Pero &iexcl;ey! recuerda que esto es un formulario de edición, eso significa que desde los controles podremos editar nuestro viewmodel. Así pues necesitamos un enlace bidireccional. Por supuesto knockout tiene soporte para estos enlaces **de forma automática**. Es decir, no haría falta que hicieramos nada especial (crear las propiedades en el viewmodel y listos).

**Observables**

Vamos a introducir un concepto nuevo: los observables. Si vienes del mundo de Silverlight o WPF debes entender que un observable de Knockout viene a ser lo mismo que una propiedad que implemente INotifyPropertyChanged, es decir la UI se enterará de los cambios de dicha propiedad. Insisto en que para que una propiedad del viewmodel sea _editable_&nbsp;a partir de un control **no**&nbsp;es necesario que sea un observable. Un observable es solo para que la UI se entere de las modificaciones del viewmodel.

Este es el código para crear nuestro viewmodel a partir de los datos devueltos por la acción GetById de ApiBeersController:

<div id="codeSnippetWrapper" style="overflow: auto; cursor: text; font-size: 8pt; font-family: 'Courier New', courier, monospace; direction: ltr; text-align: left; margin: 20px 0px 10px; line-height: 12pt; max-height: 200px; width: 97.5%; background-color: #f4f4f4; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #0000ff">var</span> uri = <span style="color: #006080">"@Url.RouteUrl("</span>DefaultApi<span style="color: #006080">", new {httproute="</span><span style="color: #006080">", controller="</span>ApiBeers<span style="color: #006080">", id=ViewBag.BeerId})"</span>;<br />$.getJSON(uri, <span style="color: #0000ff">function</span>(data) {<br />    vm = {<br />        Name  : ko.observable(data.Name),<br />        Ibu  : ko.observable(data.Ibu),<br />        Abv : ko.observable(data.Abv)<br />    };<br />    ko.applyBindings(vm);<br />});</pre>
</div>

Fijaos en el uso de ko.observable para crear una propiedad observable (el parámetro es el valor inicial). Realmente en este momento Name, Ibu y Abv ya no son variables miembro del viewmodel. Ahora son funciones:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; display: inline; border-top-width: 0px" border="0" alt="image" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_0A2675C4.png" width="513" height="72" />][6] 

Para enviar los datos en el servidor no hay mucha diferencia respecto al caso anterior:

<div id="codeSnippetWrapper" style="overflow: auto; cursor: text; font-size: 8pt; font-family: 'Courier New', courier, monospace; direction: ltr; text-align: left; margin: 20px 0px 10px; line-height: 12pt; max-height: 200px; width: 97.5%; background-color: #f4f4f4; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;">$(<span style="color: #006080">"#frm"</span>).submit(<span style="color: #0000ff">function</span>(evt) {<br />           <span style="color: #008000">// Crea el objeto JSON a partir de los datos del formulario</span><br />           <span style="color: #0000ff">var</span> jsonBeer = JSON.stringify({<br />               Name : vm.Name(),<br />               Ibu : vm.Ibu(),<br />               Abv : vm.Abv()<br />           });<br />           <span style="color: #008000">// Enviamos el objeto json</span><br />           <span style="color: #0000ff">var</span> uriEdit = <span style="color: #006080">"@Url.RouteUrl("</span>DefaultApi<span style="color: #006080">", new {httproute="</span><span style="color: #006080">", controller="</span>ApiBeers<span style="color: #006080">"})"</span>;<br />           $.ajax({<br />               url: uriEdit,<br />               dataType: <span style="color: #006080">'json'</span>,<br />               contentType: <span style="color: #006080">'application/json'</span>,<br />               type: <span style="color: #006080">'post'</span>,<br />               data: jsonBeer<br />           }).done(<span style="color: #0000ff">function</span>(data) {<br />               alert(<span style="color: #006080">"Beer "</span> + data.Id + <span style="color: #006080">" has been edited "</span> + data.Status);<br />           });<br />           evt.preventDefault();<br />       });</pre>
</div>

Creamos la representación en JSON del viewmodel. Fijaos que dado que Name, Ibu y Abv son realmente funciones, no puedo serializar el viewmodel directamente a JSON, en su lugar creo un objeto intermedio, con las propiedades Name, Ibu y Abv invocando a las funciones del viewmodel para obtener el valor actual. Y el resto ya es el mismo código de antes.

Aunque a priori no haya mucha diferencia en cuanto a &ldquo;la cantidad&rdquo; de código, hay una diferencia conceptual enorme. En el caso en que usábamos jQuery el código javascript estaba muy atado al DOM. Los datos con los que trabajábamos estaban dispersos (en #Name, en #Abv, ...). Ahora en este segundo caso el código javascript es totalmente ignorante del DOM y trabaja tan solo con el viewmodel. Separación de responsabilidades.

**¿Y ese submit?**

Incluso ahora aunque estemos &ldquo;aislados&rdquo; del DOM, seguimos gestionando una parte de nuestro código con eventos vinculados a éste: en concreto estamos suscritos al evento &ldquo;submit&rdquo; de un objeto del DOM (el formulario).

¿Nos puede ayudar knockout aquí? Pues claro 🙂

El patrón MVVM también recomienda que el código de interacción esté en el viewmodel. Es decir evitar tener código que no sea exclusivo de presentación en las vistas. En el mundo WPF y Silverlight eso se traduce en evitar el code-behind y enlazar los controles del XAML con comandos que se ejecutan en el viewmodel. Pues knockout nos ofrece algo parecido: &iexcl;Podemos enlazar eventos del DOM a acciones de nuestro viewmodel!

Para empezar le indicamos al formulario que hay un enlace a través del evento submit:

<div id="codeSnippetWrapper" style="overflow: auto; cursor: text; font-size: 8pt; font-family: 'Courier New', courier, monospace; direction: ltr; text-align: left; margin: 20px 0px 10px; line-height: 12pt; max-height: 200px; width: 97.5%; background-color: #f4f4f4; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;"><span style="color: #0000ff">&lt;</span><span style="color: #800000">form</span> <span style="color: #ff0000">method</span><span style="color: #0000ff">="POST"</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="frm"</span> <span style="color: #ff0000">data-bind</span><span style="color: #0000ff">="submit: editBeer"</span><span style="color: #0000ff">&gt;</span></pre>
</div>

Fijaos de nuevo en el uso de data-bind. Ahora enlazamos el evento submit con la función editBeer del viewmodel. Esto es una de las cosas que más me gustan de Knockout: la sintaxis para establecer todos los bindings es muy simple (hemos establecido bindings de texto, bindings bidireccionables y ahora nos enlazamos a una función con la misma sintaxis).

¿Y como es esta función editBeer? Pues bueno es una función de nuestro viewmodel que tiene básicamente el mismo código que teníamos antes dentro del evento &ldquo;submit&rdquo; que gestionábamos con jQuery:

<div id="codeSnippetWrapper" style="overflow: auto; cursor: text; font-size: 8pt; font-family: 'Courier New', courier, monospace; direction: ltr; text-align: left; margin: 20px 0px 10px; line-height: 12pt; max-height: 200px; width: 97.5%; background-color: #f4f4f4; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="overflow: visible; font-size: 8pt; font-family: 'Courier New', courier, monospace; color: black; direction: ltr; text-align: left; margin: 0em; line-height: 12pt; width: 100%; background-color: #f4f4f4; border-style: none; padding: 0px;">vm = {<br />    Name  : ko.observable(data.Name),<br />    Ibu  : ko.observable(data.Ibu),<br />    Abv : ko.observable(data.Abv),<br />    editBeer : <span style="color: #0000ff">function</span>() {<br />        <span style="color: #0000ff">var</span> jsonBeer = JSON.stringify({<br />            Name : <span style="color: #0000ff">this</span>.Name(),<br />            Ibu : <span style="color: #0000ff">this</span>.Ibu(),<br />            Abv : <span style="color: #0000ff">this</span>.Abv()<br />        });<br />        <span style="color: #008000">// Enviamos el objeto json</span><br />        <span style="color: #0000ff">var</span> uriEdit = <span style="color: #006080">"@Url.RouteUrl("</span>DefaultApi<span style="color: #006080">", new {httproute="</span><span style="color: #006080">", controller="</span>ApiBeers<span style="color: #006080">"})"</span>;<br />        $.ajax({<br />            url: uriEdit,<br />            dataType: <span style="color: #006080">'json'</span>,<br />            contentType: <span style="color: #006080">'application/json'</span>,<br />            type: <span style="color: #006080">'post'</span>,<br />            data: jsonBeer<br />        }).done(<span style="color: #0000ff">function</span>(data) {<br />            alert(<span style="color: #006080">"Beer "</span> + data.Id + <span style="color: #006080">" has been edited "</span> + data.Status);<br />        });<br />    }<br />};</pre>
</div>

&iexcl;Fantástico! Ahora ya NO tenemos nada fuera de nuestro viewmodel. Hemos establecido una separación total entre la vista (el HTML) y el viewmodel (el objeto vm) que mantiene no solo los valores que tiene la vista, si no también gestiona los comandos que la vista puede realizar.

Puede seguir pareciendo que en cuanto a &ldquo;cantidad&rdquo; de código javascript no hemos mejorado mucho respecto a la versión inicial que no usaba knockout, pero es indudable que a nivel de claridad y mantenibilidad nuestro código es ahora mucho, mucho, mucho mejor.

Espero que os haya resultado interesante.

&iexcl;En el siguiente post seguiremos explorando las maravillas de knockout!

**PD:** He dejado el código (VS2012 RC) del proyecto (KoDemo1) que se corresponde a este post. En la carpeta Views/Beers vereis que hay una Edit.cshtml y otra Edit-no-ko.cshtml. La segunda contiene el código sin usar knockout.

<iframe height="120" src="https://skydrive.live.com/embed?cid=6521C259E9B1BEC6&resid=6521C259E9B1BEC6%21288&authkey=AIpTC_DNQqm8saU" frameborder="0" width="98" scrolling="no"></iframe>

 [1]: /blogs/etomas/archive/2012/07/31/y-el-combate-se-decidi-243-por-ko-i.aspx
 [2]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_784060D9.png
 [3]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_4C4A9FEE.png
 [4]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_54D08FE9.png
 [5]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_7986C8C2.png
 [6]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_7B59AF3A.png