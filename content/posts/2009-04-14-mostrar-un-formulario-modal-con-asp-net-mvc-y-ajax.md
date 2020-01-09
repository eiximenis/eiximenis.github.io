---
title: Mostrar un formulario modal con ASP.NET MVC y Ajax
author: eiximenis

date: 2009-04-14T14:11:28+00:00
geeks_url: /?p=1445
geeks_visits:
  - 14383
geeks_ms_views:
  - 4818
categories:
  - Uncategorized

---
¿Os gusta ASP.NET MVC? A mi personalmente me encanta… aunque está un poco _verde_, en el sentido que comparándolo con webforms hay varias cosas que _debes hacerte tu mismo_, el modelo de programación es simple y elegante… Gran parte del mérito lo tiene (además del uso del [patrón MVC][1] evidentemente), [jQuery][2] genial librería de Javascript donde las haya.

Hay mucha gente desarrollando en jQuery (al margen de que usen o no ASP.NET MVC) y dado lo bien que se entienden ASP.NET MVC y jQuery es muy fácil realizar tareas que antes eran un poco… complejas.

Yo me he encontrado con la necesidad de mostrar un pop-up (modal) en una aplicación ASP.NET MVC. Un par de búsquedas por google me han llevado a [SimpleModal][3], un genial plugin para jQuery que precisamente hace esto: mostrar formularios modales. En su página web hay varios ejemplos (en su caso él usa PHP).

Os cuento como he integrado SimpleModal en mi aplicación ASP.NET MVC por si a alguien le interesa… Esta ha sido _mi_ manera de hacerlo, no pretendo sentar cátedra porque hay muuuuuuucha gente que sabe más que yo (especialmente de jQuery).

En concreto la necesidad era mostrar un link, que al pulsarse desplegase un pop-up modal para que la gente pudiera darse de alta en la página.

La página que muestra el enlace (en mi caso Index.aspx) tiene el siguiente código ASP.NET:

<pre class="code"><span style="background: #ffee62">&lt;%</span><span style="color: blue">= </span>Ajax.PopupLink (<span style="color: #a31515">"Join the game"</span>, <span style="color: #a31515">"Signup"</span>,<span style="color: #a31515">"Account"</span>, <span style="color: #a31515">"popup"</span>) <span style="background: #ffee62">%&gt;<br /></span>and start playing!<br /><span style="color: blue">&lt;</span><span style="color: #a31515">div </span><span style="color: red">id</span><span style="color: blue">="popup" /&gt;</span></pre>

El método PopupLink es un método extensor de AjaxHelper:

<pre class="code"><span style="color: blue">public static string </span>PopupLink(<span style="color: blue">this </span><span style="color: #2b91af">AjaxHelper </span>helper, <span style="color: blue">string </span>linkText, <br /><span style="color: blue">string </span>actionName, <span style="color: blue">string </span>controllerName, <span style="color: blue">string </span>popupId)
{
    <span style="color: #2b91af">AjaxOptions </span>options = <span style="color: blue">new </span><span style="color: #2b91af">AjaxOptions</span>()
    {
        UpdateTargetId = popupId,
        OnComplete = <span style="color: #a31515">"show_popup"</span>,
        HttpMethod = <span style="color: #a31515">"GET"
    </span>};
    <span style="color: blue">string </span>link = helper.ActionLink(linkText, actionName, <br />       controllerName, options);
    <span style="color: blue">return </span>link;
}</pre>

Ok… no es un método muy configurable, pero a mi me va bien 🙂 Lo que hace es mostrar un enlace con el texto especificado y le establece unas opciones por defecto: Que la llamada sea via Ajax usando GET, que se llame a una función javascript “show_popup” al terminar y que se actualice el elemento DOM especificado (en este caso el último parámetro llamado ‘popup’). Los parámetros “actionName” y “controllerName” del método sirven para especificar que acción de que controlador debe devolver la vista parcial que contiene el popup. En mi caso la acción “Signup” del controllador “AccountController” que está definida tal y como sigue:

<pre class="code"><span style="color: blue">public </span><span style="color: #2b91af">ActionResult </span>Signup()
{
    <span style="color: blue">if </span>(Request.IsAjaxRequest())
    {
        <span style="color: blue">return </span>PartialView(<span style="color: #a31515">"SignupPopup"</span>);
    }
    <span style="color: blue">else return </span>RedirectToAction(<span style="color: #a31515">"Index"</span>, <span style="color: #a31515">"Home"</span>);
}</pre>

Como podeis ver me limito a devolver la vista parcial SignupPopup que es la que contiene el código HTML del popup. Cuando el usuario haga click en el enlace “Join the game” se llamará via Ajax a la acción Signup que devolverá la vista parcial “SignupPopup”, el código de la cual se incrustará dentro del div “popup”.

El código de la vista parcial en mi caso es muy simple:

<pre class="code"><span style="background: #ffee62">&lt;%</span><span style="color: blue">@ </span><span style="color: #a31515">Control </span><span style="color: red">Language</span><span style="color: blue">="C#" </span><span style="color: red">Inherits</span><span style="color: blue">="System.Web.Mvc.ViewUserControl" </span><span style="background: #ffee62">%&gt;
</span><span style="color: blue">&lt;</span><span style="color: #a31515">div</span><span style="color: blue">&gt;
    &lt;</span><span style="color: #a31515">h1</span><span style="color: blue">&gt;</span>Join the Game!<span style="color: blue">&lt;/</span><span style="color: #a31515">h1</span><span style="color: blue">&gt;
</span><span style="color: blue">    </span><span style="background: #ffee62">&lt;%</span> <span style="color: blue">using </span>(Html.BeginForm(<span style="color: #a31515">"Signup"</span>, <span style="color: #a31515">"Account"</span>, <span style="color: #2b91af">FormMethod</span>.Post)) { <span style="background: #ffee62">%&gt;
</span>        <span style="color: blue">&lt;</span><span style="color: #a31515">label </span><span style="color: red">for</span><span style="color: blue">="nick"&gt;</span>*Nick Name:<span style="color: blue">&lt;/</span><span style="color: #a31515">label</span><span style="color: blue">&gt;
        &lt;</span><span style="color: #a31515">input </span><span style="color: red">type</span><span style="color: blue">="text" </span><span style="color: red">id</span><span style="color: blue">="nick" </span><span style="color: red">name</span><span style="color: blue">="nick" </span><span style="color: red">tabindex</span><span style="color: blue">="1001" /&gt;
        &lt;</span><span style="color: #a31515">br </span><span style="color: blue">/&gt;
        &lt;</span><span style="color: #a31515">label </span><span style="color: red">for</span><span style="color: blue">="email"&gt;</span>*Email:<span style="color: blue">&lt;/</span><span style="color: #a31515">label</span><span style="color: blue">&gt;
        &lt;</span><span style="color: #a31515">input </span><span style="color: red">type</span><span style="color: blue">="text" </span><span style="color: red">id</span><span style="color: blue">="email" </span><span style="color: red">name</span><span style="color: blue">="email" </span><span style="color: red">tabindex</span><span style="color: blue">="1002" /&gt;
        &lt;</span><span style="color: #a31515">br </span><span style="color: blue">/&gt;
        &lt;</span><span style="color: #a31515">label</span><span style="color: blue">&gt;</span>A email for validate your account will be sent at <br />        email address you specified.<span style="color: blue">&lt;/</span><span style="color: #a31515">label</span><span style="color: blue">&gt;
        &lt;</span><span style="color: #a31515">br </span><span style="color: blue">/&gt;
        &lt;</span><span style="color: #a31515">button </span><span style="color: red">type</span><span style="color: blue">="submit"  </span><span style="color: red">tabindex</span><span style="color: blue">="1006"&gt;</span>Send<span style="color: blue">&lt;/</span><span style="color: #a31515">button</span><span style="color: blue">&gt;
        &lt;</span><span style="color: #a31515">button </span><span style="color: red">type</span><span style="color: blue">="reset"  </span><span style="color: red">tabindex</span><span style="color: blue">="1007"&gt;</span>Cancel<span style="color: blue">&lt;/</span><span style="color: #a31515">button</span><span style="color: blue">&gt;
        &lt;</span><span style="color: #a31515">br</span><span style="color: blue">/&gt;
    </span><span style="background: #ffee62">&lt;%</span> } <span style="background: #ffee62">%&gt;
</span><span style="color: blue">&lt;/</span><span style="color: #a31515">div</span><span style="color: blue">&gt;  </span></pre>

[][4]

Basicamente tenemos un formulario con dos campos: nick y email. Cuando hagamos un submit del formulario (via POST) se llamará a la acción Signup del controlador AccountController, acción que está definida como sigue:

<pre class="code">[<span style="color: #2b91af">AcceptVerbs</span>(<span style="color: #2b91af">HttpVerbs</span>.Post)]
<span style="color: blue">public </span><span style="color: #2b91af">ActionResult </span>Signup(<span style="color: blue">string </span>nick, <span style="color: blue">string </span>email)
{
    <span style="color: green">// Codigo para dar de alta el nuevo usuario...
    // Mostramos la vista de Bienvenida
    </span><span style="color: blue">return </span>View();
}</pre>

[][4][][4]

No mucha cosa… El controlador da de alta el usuario y finalmente muestra una vista de bienvenida.

Finalmente en la página Index.aspx, debemos tener el método javascript show_popup, que será el encargado de mostrar el popup usando SimpleModal:

<pre class="code"><span style="color: blue">&lt;</span><span style="color: #a31515">script </span><span style="color: red">type</span><span style="color: blue">="text/javascript"&gt;
    function </span>show_popup() {
        $(<span style="color: #a31515">"#popup"</span>).modal();
    }
<span style="color: blue">&lt;/</span><span style="color: #a31515">script</span><span style="color: blue">&gt;    </span></pre>

El código es muy simple: accedemos al elemento div con id=”popup” que hemos rellenado con el contenido de la vista parcial, y usamos el método modal() que define SimpleModal para mostrar este div como un formulario modal…

… y listos!

Simple y sencillo… en otro post mostraré como comunicar nuestro formulario via Ajax con nuestros controladores (p.ej. para poder validar datos en servidor sin necesidad de hacer submit del formulario).

 [1]: http://msdn.microsoft.com/en-us/library/ms978748.aspx
 [2]: http://jquery.com/
 [3]: http://www.ericmmartin.com/projects/simplemodal/
 [4]: http://11011.net/software/vspaste