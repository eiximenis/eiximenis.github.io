---
title: 'ASP.NET MVC y Ajax: fácil no, facilísimo (ii)'

author: eiximenis

date: 2009-06-30T15:50:00+00:00
geeks_url: /?p=1459
geeks_visits:
  - 24979
geeks_ms_views:
  - 11682
categories:
  - Uncategorized

---
Hola a todos! En este post (continuación del post anterior [ASP.NET MVC y Ajax][1]) voy a comentaros algunas cosillas más sobre ASP.NET MVC y ajax utilizando jQuery. En este caso vamos a ver como implementar un clásico de Ajax: las combos encadenadas.

<!--more-->

Vamos a hacer el manido ejemplo de continentes y paises: seleccionas un continente en la combo y via Ajax se rellena la combo de paises. Vereis que sencillo es crear combos encadenadas con ASP.NET MVC y jQuery.

Lo primero es crear una vista que tenga las dos combos que queramos (yo como siempre modifico la vista inicial (Views/Home/Index.aspx).

El código que he puesto es:

<pre class="code"><span style="color: blue">&lt;</span><span style="color: #a31515">h1</span><span style="color: blue">&gt;</span>Ajax y MVC... tan fácil como parece!<span style="color: blue">&lt;/</span><span style="color: #a31515">h1</span><span style="color: blue">&gt;
</span>Escoja continente y deje que el poder de Ajax le muestre algunos paises...
<span style="color: blue">&lt;</span><span style="color: #a31515">select </span><span style="color: red">id</span><span style="color: blue">="cmbContinente"&gt;
    &lt;</span><span style="color: #a31515">option </span><span style="color: red">value</span><span style="color: blue">="---" </span><span style="color: red">selected</span><span style="color: blue">="selected"&gt;</span>Escoja continente...<span style="color: blue">&lt;/</span><span style="color: #a31515">option</span><span style="color: blue">&gt;
    &lt;</span><span style="color: #a31515">option </span><span style="color: red">value</span><span style="color: blue">="eur"&gt;</span>Europa<span style="color: blue">&lt;/</span><span style="color: #a31515">option</span><span style="color: blue">&gt;
    &lt;</span><span style="color: #a31515">option </span><span style="color: red">value</span><span style="color: blue">="ame"&gt;</span>America<span style="color: blue">&lt;/</span><span style="color: #a31515">option</span><span style="color: blue">&gt;
    &lt;</span><span style="color: #a31515">option </span><span style="color: red">value</span><span style="color: blue">="asi"&gt;</span>Asia<span style="color: blue">&lt;/</span><span style="color: #a31515">option</span><span style="color: blue">&gt;
    &lt;</span><span style="color: #a31515">option </span><span style="color: red">value</span><span style="color: blue">="afr"&gt;</span>Africa<span style="color: blue">&lt;/</span><span style="color: #a31515">option</span><span style="color: blue">&gt;
&lt;/</span><span style="color: #a31515">select</span><span style="color: blue">&gt;
&lt;</span><span style="color: #a31515">select </span><span style="color: red">id</span><span style="color: blue">="cmbPaises"&gt;&lt;/</span><span style="color: #a31515">select</span><span style="color: blue">&gt;</span></pre>

[][2]

Excelente... ya tenemos las dos combos. Ahora empieza lo interesante: cuando se seleccione un elemento de la combo de continentes vamos&nbsp; a realizar una petición ajax al controlador, para que nos devuelva que países se corresponden al continente seleccionado...

Como siempre jQuery viene en nuestra ayuda:

<pre class="code"><span style="color: blue">&lt;</span><span style="color: #a31515">script </span><span style="color: red">type</span><span style="color: blue">="text/javascript"&gt;
    </span>$(document).ready(<span style="color: blue">function</span>() {
        $(<span style="color: #a31515">"#cmbContinente"</span>).change(<span style="color: blue">function</span>() {
            fillCombo(<span style="color: #a31515">"cmbPaises"</span>, $(<span style="color: #a31515">"#cmbContinente"</span>).val());
        });
    });
<span style="color: blue">&lt;/</span><span style="color: #a31515">script</span><span style="color: blue">&gt;</span></pre>

[][2]

Si visteis mi post anterior este código ya no os sonará tan extraño. Si no lo habeis visto, o bien no os acordáis ahí va un rápido resumen: con $(document) obtengo el manejador jQuery al documento actual, y llamo a su método ready. El método ready se ejecuta cuando el documento está &ldquo;listo&rdquo; (esto es todo el árbol DOM inicializado), y como parámetro espera una función. En mi caso le paso una función _anónima_ con el código a ejecutar.

Lo que hace la función anónima es muy sencillo. Recordad que en jQuery la expresión $(&ldquo;#xx&rdquo;) obtiene un manejador jQuery al elemento DOM cuyo id sea &ldquo;xx&rdquo;. Así pues obtengo el manejador jQuery de la combo cuyo id es &ldquo;cmbContinente&rdquo; y me suscribo a su evento change. Cuando se usa jQuery generalmente no nos suscribimos a los eventos de los elementos DOM usando la sintaxis clásica onXXX, sinó que usamos las propias funciones de jQuery. En mi caso uso la función change() que se correspondería el evento onChange. La razón de actuar así es que jQuery intenta ofrecer un modelo de eventos **unificado** entre distintos navegadores. Si veis el código, simplemente se le pasa al método change una función _anónima_ con el código a ejecutar cuando se lance el evento change: una llamada a una función fillCombo&nbsp; que definiremos luego. A esta fillCombo le pasamos el id de la combo que vamos a actualizar y el valor seleccionado de la combo de continentes. Fijaos que de nuevo usamos jQuery para saber el valor seleccionado de la combo: obtenemos un manejador jQuery a la combo y llamamos al método val().

Bien, vamos ahora a ver el código de la función fillCombo. Esta función coje dos parámetros _updateId_ y _value_ (el id de la combo a rellenar y el valor seleccionado de la primera combo):

<pre class="code"><span style="color: blue">&lt;</span><span style="color: #a31515">script </span><span style="color: red">type</span><span style="color: blue">="text/javascript"&gt;
    function </span>fillCombo(updateId, value) {
        $.getJSON(<span style="color: #a31515">"&lt;%= Url.Action("</span>PaisesPorContinente<span style="color: #a31515">") %&gt;" <br />            </span>+ <span style="color: #a31515">"/" </span>+ value,
            <span style="color: blue">function</span>(data) {
                $(<span style="color: #a31515">"#"</span>+updateId).empty();
                $.each(data, <span style="color: blue">function</span>(i, item) {
                    $(<span style="color: #a31515">"#"</span>+updateId).append(<span style="color: #a31515">"&lt;option id='" <br />                       </span>+ item.IdPais +<span style="color: #a31515">"'&gt;" </span>+ item.Nombre <br />                       + <span style="color: #a31515">"&lt;/option&gt;"</span>);
                });
            });
    }
<span style="color: blue">&lt;/</span><span style="color: #a31515">script</span><span style="color: blue">&gt;</span></pre>

[][2]

Veamos que hace exactamente fillCombo:

  * Usa la función getJSON de jQuery. Esta función realiza una llamada a la URL indicada (en mi caso una llamada a la acción _PaisesPorContinente_), usando AJAX y espera que dicha función devuelva un resultado en formato JSON. Además parsea el resultado JSON y lo convierte en un objeto javascript. Para los que no sepáis que es JSON os diré que es un lenguaje para representar objetos (tal y como puede ser XML) pero mucho más compacto, y además más &ldquo;compatible&rdquo; con javascript. Para más información pasaros por [json.org][3]. 
  * La función getJSON admite dos parámetros: el primero es la URL y el segundo una función a ejecutar cuando se reciba el resultado... Para variar le pasamos una función anónima 🙂 
  * Y la función anónima qué hace? 
      * Pues bueno, primero vacía la combo indicada (llama al método empty() que lo que hace es vaciar el árbol DOM de un determinado elemento, por lo que aplicado a un <select> le elimina todas sus <option>). 
      * Despues usa el método [each de jQuery][4] para iterar sobre la respuesta (asume que lo que se recibe es un array JSON). El método each recibe dos parámetros: el primero es la variable sobre la que iteramos y el segundo la función a ejecutar por cada elemento. En mi caso _otra_ función anónima que usa el método _append_ para añadir una etiqueta <option> a la combo (accediendo a las propiedades IdPais y Nombre de cada elemento del array JSON). 

Bueno... hemos terminado la parte de cliente... veamos que debemos hacer en el servidor.

En primer lugar debemos crear la acción _PaisesPorContinente_ en nuestro controlador: Una acción es un método público del controlador, y el nombre del método define el nombre de la acción, aunque esto se puede modificar con el uso del atributo [ActionNameAttribute][5]:

<pre class="code">[<span style="color: #2b91af">ActionName</span>(<span style="color: #a31515">"PaisesPorContinente"</span>)]
<span style="color: blue">public </span><span style="color: #2b91af">ActionResult </span>GetPaisesPorContinente(<span style="color: blue">string </span>id)
{
    <span style="color: blue">var  </span>paises = <span style="color: blue">new </span><span style="color: #2b91af">PaisesModel</span>().GetPaisesPorContinente(id);
    <span style="color: blue">return new </span><span style="color: #2b91af">JsonResult</span>() { Data = paises };
}</pre>

[][2]

Accedemos al modelo y le pedimos que nos devuelva los paises del continente según el id indicado. Luego debemos devolver el resultado de la petición, que ahora **no** es código html, sinó código JSON. Por suerte la gente del framework MVC sabe que JSON se usa constantemente en AJAX, así que han implementado un serializador para convertir objetos de .NET en objetos JSON. Para devolver un objeto serializado en JSON simplemente devolved un JsonResult con la propiedad Data establecida al objeto que quereis devolver... y listos.

El código de mi modelo es **muy simple**:

<pre class="code"><span style="color: blue">public class </span><span style="color: #2b91af">Pais
</span>{
    <span style="color: blue">public string </span>Continente { <span style="color: blue">get</span>; <span style="color: blue">set</span>; }
    <span style="color: blue">public string </span>Nombre { <span style="color: blue">get</span>; <span style="color: blue">set</span>; }
    <span style="color: blue">public string </span>IdPais { <span style="color: blue">get</span>; <span style="color: blue">set</span>; }
}
<span style="color: blue">public class </span><span style="color: #2b91af">PaisesModel
</span>{
    <span style="color: blue">private </span><span style="color: #2b91af">List</span>&lt;<span style="color: #2b91af">Pais</span>&gt; paises;
    <span style="color: blue">public </span>PaisesModel()
    {
        <span style="color: blue">this</span>.paises = <span style="color: blue">new </span><span style="color: #2b91af">List</span>&lt;<span style="color: #2b91af">Pais</span>&gt;();
        <span style="color: blue">this</span>.paises.Add(<span style="color: blue">new </span><span style="color: #2b91af">Pais</span>()
        {
            Continente = <span style="color: #a31515">"eur"</span>,
            IdPais = <span style="color: #a31515">"es"</span>,
            Nombre = <span style="color: #a31515">"España"
        </span>});
        <span style="color: green">// Más y más paises añadidos...
</span>    }
    <span style="color: blue">public </span><span style="color: #2b91af">IEnumerable</span>&lt;<span style="color: #2b91af">Pais</span>&gt; GetPaisesPorContinente<br />    (<span style="color: blue">string </span>continente)
    {
        <span style="color: blue">return this</span>.paises.FindAll<br />            (x =&gt; x.Continente == continente);
    }
}</pre>

[][2][][2]

Simplemente devuelve una lista de objetos de la clase Pais. Fijaos como la clase Pais tiene las propiedades IdPais y Nombre que usábamos dentro de fillCombo! Y ya hemos terminado! Si quereis ver el proyecto completo, os lo podeis descargar de [**aquí**][6].

Qué... fácil, no??? 

Un saludo a todos! 😉

pd: Por cierto, si vais a trabajar con combos y jQuery os recomiendo que le echeis un vistazo al [plugin de jQuery para selectboxes][7], que añade funciones específicas para trabajar con selects (sí, sí... jQuery puede extenderse con plugins que le añaden todavía más funcionalidades!).

 [1]: /blogs/etomas/archive/2009/06/26/asp-net-mvc-y-ajax-f-225-cil-no-facil-237-simo.aspx
 [2]: http://11011.net/software/vspaste
 [3]: http://www.json.org/
 [4]: http://docs.jquery.com/Utilities/jQuery.each
 [5]: http://msdn.microsoft.com/en-us/library/system.web.mvc.actionnameattribute.aspx
 [6]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas.30062009/AjaxDemoCombosEncadenadas.zip
 [7]: http://plugins.jquery.com/node/27/release