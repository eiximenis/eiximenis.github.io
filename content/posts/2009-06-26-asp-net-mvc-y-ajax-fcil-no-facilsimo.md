---
title: 'ASP.NET MVC y Ajax: fácil no… facilísimo :)'
author: eiximenis

date: 2009-06-26T22:09:00+00:00
geeks_url: /?p=1458
geeks_visits:
  - 16550
geeks_ms_views:
  - 7798
categories:
  - Uncategorized

---
Hola a todos amigos! 😉

El comentario de Gabriel en este post de mi blog ([http://geeks.ms/blogs/etomas/archive/2009/04/02/asp-net-mvc-controles-chart-y-ajax.aspx][1]) me ha motivado a escribir la siguiente entrada.

&Eacute;l preguntaba sobre si los controles Ajax de ASP.NET, como p.ej. UpdatePanel se podían usar bajo el framework MVC. No conozco mucho los controles de la Ajax Library porque personalmente no me interesan demasiado, aunque apuesto que la mayoría usan viewstate así que me imagino que no deben poder usarse bajo MVC...

... por otro lado sobreentiendo que la duda de Gabriel, va un poco más allá y quiere saber _como_ usar Ajax con el framework MVC. Pues para verlo **muy someramente** aquí va esta entrada.

El soporte para Ajax del framework MVC no es que sea espectacular: las vistas tienen una propiedad _Ajax_ que permite acceder al objeto [AjaxHelper][2], que contiene varios métodos para permitir usar Ajax en el framework MVC. P.ej. si quieres poner un link que al pulsarlo cargue el div cuyo id sea &#8220;ajaxpanel&rdquo; puedes hacer:

<pre class="code"><span style="background: yellow">&lt;%</span><span style="color: blue">=</span>Ajax.ActionLink(<span style="color: #a31515">"Pulsa aquí"</span>, <span style="color: #a31515">"view1"</span>, <span style="color: blue">new </span><span style="color: #2b91af">AjaxOptions</span>() <br />{ UpdateTargetId=<span style="color: #a31515">"ajaxpanel"</span>}) <span style="background: yellow">%&gt;</span></pre>

[][3]

Al pulsar sobre el enlace, el framework MVC ejecutará la acción &ldquo;view1&rdquo; (del controlador actual) y la vista que esta acción devuelva será incrustada dentro del elemento DOM cuyo id sea _ajaxpanel_.

No está mal, pero tiene dos pegas:

  1. De serie no nos viene nada más: si queremos usar imágenes que sean enlaces Ajax, o botones, o una combo, debemos hacerlo nosotros.
  2. Usa la librería propia de Ajax de MS (MicrosoftAjax.js y MicrosoftMvcAjax.js)... No tengo nada personal en contra de esta librería, pero no veo muy claro a que viene: ¿teniendo jQuery, para que reinventar la rueda?

Yo personalmente uso sólo [jQuery][4] para mi soporte Ajax... ¿Por qué? Pues porqué me da **todo** lo que necesito... y más. Además de que es cross-browser y no sólo ofrece temas para Ajax, sino para muchas otras cosas. Si estás desarrollando web y no usas jQuery cuando empieces te preguntarás como has podido estar sin ella todo este tiempo.

Usar jQuery implica que no usamos las funciones del AjaxHelper, pero bueno no importa demasiado: rehacerlas usando jQuery no cuesta nada... ah! Y además jQuery **ya viene** con el framework MVC (en la carpeta /scripts).

En esta demo, veremos como hacer en un momento una vista con dos botones: al pulsar cada uno de ellos se cargará via Ajax una vista y se incrustará dentro de un div.

**1. Inicio: Modificación de la vista master**

Creamos una nueva aplicación ASP.NET MVC. Ello nos creará la aplicación inicial, con varias vistas (login, home, registro) para empezar a trabajar. Vamos a modificar directamente la vista inicial (Views/Home/Index.aspx).

Lo primero a hacer es incluir la carga de jQuery en nuestra vista. En mi caso modifico la vista master, para que jQuery esté incluida &ldquo;de serie&rdquo; en **todas** mis vistas... yo la uso a mansalva, y creedme: vais a terminar haciendo lo mismo 🙂

La vista master está en Views/Shared/Site.Master. Si la abrís vereis que tiene un poco de código. Podeis obviarlo, simplemente añadid una etiqueta <script> dentro del <head>:

<pre class="code"><span style="color: blue">&lt;</span><span style="color: maroon">head </span><span style="color: red">runat</span><span style="color: blue">="server"&gt;
    &lt;</span><span style="color: maroon">title</span><span style="color: blue">&gt;&lt;</span><span style="color: maroon">asp</span><span style="color: blue">:</span><span style="color: maroon">ContentPlaceHolder </span><span style="color: red">ID</span><span style="color: blue">="TitleContent" </span><span style="color: red">runat</span><span style="color: blue">="server" /&gt;<br />    &lt;/</span><span style="color: maroon">title</span><span style="color: blue">&gt;
    &lt;</span><span style="color: maroon">link </span><span style="color: red">href</span><span style="color: blue">="../../Content/Site.css" <br />     </span><span style="color: red">rel</span><span style="color: blue">="stylesheet" </span><span style="color: red">type</span><span style="color: blue">="text/css" /&gt;
    </span><span style="background: yellow">&lt;%</span><span style="color: #006400">-- Incluimos jQuery --</span><span style="background: yellow">%&gt;
</span>    <strong><span style="color: blue">&lt;</span><span style="color: maroon">script </span><span style="color: red">type</span><span style="color: blue">="text/javascript" </span><span style="color: red">src</span><span style="color: blue">="../../Scripts/jquery-1.3.2.js"&gt;<br />    &lt;/</span><span style="color: maroon">script</span><span style="color: blue">&gt;</span></strong><span style="color: blue">
&lt;/</span><span style="color: maroon">head</span><span style="color: blue">&gt;</span></pre>

[][3]

(En este caso incluyo jquery-1.3.2.js que es la versión que viene junto con el framework MVC).

**2. Modificación de la vista inicial**

Como comenté en el punto anterior vamos a trabajar modificando directamente la vista inicial (Views/Home/Index.aspx). Para ello vamos a añadir simplemente dos botones y un <div> vacío que será nuestro contenedor ajax:

<pre class="code"><span style="background: yellow">&lt;%</span><span style="color: blue">@ </span><span style="color: maroon">Page </span><span style="color: red">Language</span><span style="color: blue">="C#" </span><span style="color: red">MasterPageFile</span><span style="color: blue">="~/Views/Shared/Site.Master" <br /></span><span style="color: red">Inherits</span><span style="color: blue">="System.Web.Mvc.ViewPage" </span><span style="background: yellow">%&gt;
</span><span style="color: blue">&lt;</span><span style="color: maroon">asp</span><span style="color: blue">:</span><span style="color: maroon">Content </span><span style="color: red">ID</span><span style="color: blue">="indexTitle" </span><span style="color: red">ContentPlaceHolderID</span><span style="color: blue">="TitleContent" <br /></span><span style="color: red">runat</span><span style="color: blue">="server"&gt;
    </span>Home Page
<span style="color: blue">&lt;/</span><span style="color: maroon">asp</span><span style="color: blue">:</span><span style="color: maroon">Content</span><span style="color: blue">&gt;
&lt;</span><span style="color: maroon">asp</span><span style="color: blue">:</span><span style="color: maroon">Content </span><span style="color: red">ID</span><span style="color: blue">="indexContent" </span><span style="color: red">ContentPlaceHolderID</span><span style="color: blue">="MainContent" </span><span style="color: red">runat</span><span style="color: blue">="server"&gt;
</span><span style="color: blue">    &lt;</span><span style="color: maroon">h2</span><span style="color: blue">&gt;</span>Pulsa los botones para refrescar el div usando ajax<span style="color: blue">&lt;/</span><span style="color: maroon">h2</span><span style="color: blue">&gt;
    &lt;</span><span style="color: maroon">input </span><span style="color: red">type</span><span style="color: blue">="button" </span><span style="color: red">id</span><span style="color: blue">="view1" </span><span style="color: red">value</span><span style="color: blue">="Vista 1" /&gt;
    &lt;</span><span style="color: maroon">input </span><span style="color: red">type</span><span style="color: blue">="button" </span><span style="color: red">id</span><span style="color: blue">="view2"  </span><span style="color: red">value</span><span style="color: blue">="Vista 2"/&gt;
    </span><span style="background: yellow">&lt;%</span><span style="color: #006400">-- Este es el div que vamos a modificar via ajax  --</span><span style="background: yellow">%&gt;
</span>    <span style="color: blue">&lt;</span><span style="color: maroon">div </span><span style="color: red">id</span><span style="color: blue">="ajaxpanel" </span><span style="color: red">style</span><span style="color: blue">="</span><span style="color: red">border-color</span><span style="color: blue">:red; </span><span style="color: red">border-style</span><span style="color: blue">:solid; <br /></span><span style="color: red">border-width</span><span style="color: blue">:thin"&gt;&lt;/</span><span style="color: maroon">div</span><span style="color: blue">&gt;
&lt;/</span><span style="color: maroon">asp</span><span style="color: blue">:</span><span style="color: maroon">Content</span><span style="color: blue">&gt;</span></pre>

[][3]

He puesto un border rojo al div para que vea (sí, reconozco que el diseño no es mi fuerte :p).

**2.1 Crear las llamadas Ajax al pulsar los botones**

Ahora vamos a añadir código javascript para que cuando se pulsen los botones se hagan las llamadas ajax... Si ya estás corriendo a meter un onClick en cada <input> tranquilo: bienvenido al mundo de jQuery 🙂

Este es el código javascript que puedes colocar en tu página (justo **antes** del tag <h2>, _dentro_ del asp:Content _IndexContent_):

<pre class="code"><span style="color: blue">&lt;</span><span style="color: maroon">script </span><span style="color: red">type</span><span style="color: blue">="text/javascript"&gt;
    </span>$(document).ready(<span style="color: blue">function</span>() {
        $(<span style="color: maroon">"#view1"</span>).click(<span style="color: blue">function</span>() {
            $(<span style="color: maroon">"#ajaxpanel"</span>).load(<span style="color: maroon">"</span><span style="background: yellow; color: maroon">&lt;%</span><span style="color: maroon">=Url.Action("</span>View1<span style="color: maroon">") </span><span style="background: yellow; color: maroon">%&gt;</span><span style="color: maroon">"</span>);
        });
        $(<span style="color: maroon">"#view2"</span>).click(<span style="color: blue">function</span>() {
            $(<span style="color: maroon">"#ajaxpanel"</span>).load(<span style="color: maroon">"</span><span style="background: yellow; color: maroon">&lt;%</span><span style="color: maroon">= Url.Action("</span>View2<span style="color: maroon">") </span><span style="background: yellow; color: maroon">%&gt;</span><span style="color: maroon">"</span>);
        });
    });
<span style="color: blue">&lt;/</span><span style="color: maroon">script</span><span style="color: blue">&gt;</span></pre>

[][3]

Vamos a comentar rápidamente este código... aunque es muy simple muestra dos conceptos clave de jQuery: los selectores y la potencia de sus funciones incorporadas.

El $ es el simbolo &ldquo;jQuery&rdquo; por excelencia. $(document) me devuelve un manejador al documento actual. La función ready() espera como parámetro _otra_ función que se ejecutará cuando el documento esté cargado y todos los objetos DOM existan. 

En mi caso le paso a la función ready _una función anónima_ que hace lo que a mi me interesa que se haga cuando el documento esté cargado: crear funciones gestoras de los click de los botones para que se llame via Ajax a otras URLs.

El código $(<span style="color: maroon">&#8220;#view1&#8221;</span>) es un **selector**: los selectores son una de las claves de jQuery, ya que permiten obtener un manejador jQuery a uno **o varios** objetos DOM para realizar tareas con ellos. Aquí estoy usando uno de los más simples, el # que devuelve un manejador al objeto DOM cuyo ID sea la cadena que hay después de #. Así $(&ldquo;#view1&rdquo;) me devuelve un manejador al objeto DOM cuyo id sea &ldquo;view1&rdquo;, que en mi caso es el primer boton.

Una vez tengo un manejador de jQuery puedo hacer barbaridad de cosas con él (que afectarán al objeto (u objetos) DOM a los que apunte dicho manejador). En mi caso llamo a la función **click** que espera como parámetro otra función. La función click lo que hace es ejecutar la función que se pase como parámetro cuando se lance el evento _click_ del elemento DOM subyacente... Y que le paso como parámetro a la función click? Pues _otra_ función anónima con el código a ejecutar cuando se lance el evento. ¿Y qué codigo es este? Pues usar un selector para obtener un manejador al objeto cuyo ID sea &ldquo;ajaxpanel&rdquo; (que es el <div>) y llamar el método load. El método load usa Ajax para cargar la URL especificada y incrustar el resultado _dentro_ del DOM del manejador usado. Es decir, en nuestro caso dentro del div.

Ya casi estamos...

**3. Crear las acciones en el controlador**

En ASP.NET MVC las URLs se mapean a acciones de los controladores, no a archivos físicos .aspx. Si os fijais en el codigo de la vista se ha usado <%= Url.Action(&ldquo;xxx&rdquo;) %> para pasar la URL al método load de jQuery. Esta llamada a URL.Action obtiene la URL de la acción indicada del controlador actual. En mi caso he llamado a las acciones &ldquo;View1&rdquo; y &ldquo;View2&rdquo;.

Dado que estamos en una vista gestionada por el controlador _Home_, debemos crear las acciones en el controlador HomeController. La forma más simple de crear una acción es definir un método **público** en dicho controlador. Así, pues definimos los métodos View1 y View2 en HomeController (que está dentro de la carpeta _Controllers_):

<pre class="code"><span style="color: blue">public </span><span style="color: #2b91af">ActionResult </span>View1()
{
    <span style="color: blue">return </span>PartialView(<span style="color: #a31515">"View1"</span>);
}
<span style="color: blue">public </span><span style="color: #2b91af">ActionResult </span>View2()
{
    <span style="color: blue">return </span>PartialView(<span style="color: #a31515">"View2"</span>);
}</pre>

[][3]

En este caso las dos acciones son bastante tontas: el controlador no hace nada salvo retornar dos vistas, llamadas _View1_ y _View2_. El método _PartialView_ se usa cuando lo que se va a devolver es una _vista parcial_, esto es una vista que no es un HTML completo sino una parte.

Solo nos queda una pequeña cosa... crear las dos vistas!

**4. Crear las dos vistas parciales**

Vamos a crear las dos vistas que necesitamos. Dado que estamos creando vistas del controlador _Home_, debemos crearlas dentro de _Views/Home_. Para ello hacemos click con el botón derecho en la carpeta _Home_ en el Solution Explorer y seleccionamos la opción Add->View. Esto nos despliega el cuadro de nueva vista de ASP.NET MVC.

[<img height="244" width="229" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_5CD55DE6.png" alt="image" border="0" title="image" style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" />][5] 

Damos nombre a la vista (View1) y **marcamos la checkbox** de &ldquo;Create a partial view&rdquo;. Con ello nos va a crear un archivo .ascx, en lugar de un .aspx (que sería una página completa).

Yo he añadido el siguiente código a mi vista View1:

<pre class="code"><span style="background: yellow">&lt;%</span><span style="color: blue">@ </span><span style="color: maroon">Control </span><span style="color: red">Language</span><span style="color: blue">="C#" </span><span style="color: red">Inherits</span><span style="color: blue">="System.Web.Mvc.ViewUserControl" </span><span style="background: yellow">%&gt;
</span><span style="color: blue">&lt;</span><span style="color: maroon">h1</span><span style="color: blue">&gt;</span>Esta es la vista1<span style="color: blue">&lt;/</span><span style="color: maroon">h1</span><span style="color: blue">&gt;</span></pre>

[][3]

Ya veis... poca cosa, no?

Repetid el proceso para crear la View2 y... **ya habeis terminado**.

Ejecutad el proyecto y os aparecerá la página principal, con los dos botones. Pulsad uno u otro y observad como el div se rellena via Ajax con el código de cada vista.

En la última imagen os dejo una captura con el firebug donde se ve la última petición ajax:

[<img height="100" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_007E5FB4.png" alt="image" border="0" title="image" style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" />][6] 

Esto ha sido una **muy breve** introducción... pero espero que os haya servido para que os entre el gusanillo de ASP.NET MVC!!! 😉

Saludos!

 [1]: /blogs/etomas/archive/2009/04/02/asp-net-mvc-controles-chart-y-ajax.aspx "http://geeks.ms/blogs/etomas/archive/2009/04/02/asp-net-mvc-controles-chart-y-ajax.aspx"
 [2]: http://msdn.microsoft.com/en-us/library/system.web.mvc.ajaxhelper.aspx
 [3]: http://11011.net/software/vspaste
 [4]: http://jquery.com/
 [5]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_32B5D5CB.png
 [6]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_011A4269.png