---
title: Recaptcha, ASP.NET MVC, SimpleModal y un poco de Ajax…
author: eiximenis

date: 2009-04-21T10:11:00+00:00
geeks_url: /?p=1447
geeks_visits:
  - 3484
geeks_ms_views:
  - 1106
categories:
  - Uncategorized

---
Hola família!

En los dos últimos posts ([http://geeks.ms/blogs/etomas/archive/2009/04/14/mostrar-un-formulario-modal-con-asp-net-mvc-y-ajax.aspx][1] y [http://geeks.ms/blogs/etomas/archive/2009/04/15/mostrar-un-formulario-modal-con-asp-net-mvc-y-ajax-ii.aspx][2]) comenté como he usado SimpleModal, junto con ASP.NET MVC para mostrar un formulario modal y comunicarlo via AJAX con nuestro controlador.

En mi caso, este formulario era el formulario de registro... y para evitar los spammers (tal y como diría cierto ministro, _yo no digo que haya que prohibir el spam, pero yo lo prohibiría_ :p) decidí usar un captcha.

**Paso 1: El captcha**

Nevagando por esas webs de dios, llegué a [Recaptcha][3] un sitio donde ofrecen de forma totalmente gratuita un servicio de captchas bastante interesante... Empecé a ver como integrar Recaptcha en ASP.NET MVC... la API no es muy complicada y hay gente que ha hecho utilidades para casi todos lenguajes de servidor (como se puede ver en la [página de recursos de Recaptcha][4]). Aunque no aparece en esta página de recursos, googleando un poco más vi que [Andrew Wilinksi][5] ya había hecho una API para recaptcha usando ASP.NET MVC Beta 1: [RecaptchaMvc][6]. En su blog anuncia que es para la Beta 1 de ASP.NET MVC, así que me descargue el código fuente desde [http://recaptchamvc.codeplex.com/SourceControl/ListDownloadableCommits.aspx][7] (el changeset 27773) y lo compilé contra la versión final de ASP.NET MVC.

El resultado es que **NO** compila, debido al uso de la interfaz IValueProvider y la clase DefaultValueProvider, que existían en la Beta y que fueron eliminadas. La solución es bastante simple, en el fichero controllerextensions.cs, cambiar las referencias a IValueProvider por IDictionary<string, ValueProviderResult>. Al final del post adjunto el fichero modificado para que compile contra la versión final. Este es el único fichero que debe ser modificado.

RecaptchaMvc incorpora métodos de extensión para HtmlHelper, de forma que poner un Captcha es tan sencillo como:

<pre class="code"><span style="background: #ffee62">&lt;%</span><span style="color: blue">= </span>Html.Recaptcha(<span style="color: blue">this</span>.Model) <span style="background: #ffee62">%&gt;</span></pre>

[][8]

Entiendendo que estamos en una vista tipada cuyo tipo de modelo es IRecaptchaModelState, una interfaz proporcionada por RecaptchaMvc, que también proporciona la clase RecaptchaModelState que implementa la interfaz. Dicha clase espera nuestra clave pública de Recaptcha.

**Paso 2: El formulario modal...**

Y este post se habría terminado ya, si en mi caso yo no estuviese mostrando el captcha en un formulario modal via SimpleModa. A ver, a grandes rasgos el problema es que la llamada a Html.Recaptcha, genera un tag <script>. Si mostramos un formulario usando Ajax, lo que hacermos _básicamente_ es rellenar un <div> via javascript con cierto contenido html. En este caso, los tags <script> son ignorados, por lo que el captcha no se ve... 

La solución? Una vez esté cargado el formulario modal, llamar via Ajax a recaptcha y mostrar entonces el captcha. Por suerte esto es posible porque la gente de recaptcha ofrecen una API Ajax, porque si no... buf! En la [página de la API cliente de recaptcha][9] se describe como funciona la API de recaptcha si queremos usar Ajax. Hay también un ejemplo y la verdad es que es bastante sencillo... De hecho son dos pasos muy sencillos:

  1. Incluir el fichero de script recaptcha_ajax.js 
  2. Llamar a la función Recaptcha.Create. Esta función espera la clave pública, el ID del objeto DOM a rellenar con el captcha y un objeto con varias propiedades adicionales para personalizar el captcha. 

Por lo tanto, lo que he hecho ha sido:

En la vista principal (la que muestra el popup cuando se pulsa un enlace), que en mi caso se llama Home/Index.aspx, he añadido el tag <script>:

<pre class="code"><span style="color: blue">&lt;</span><span style="color: #a31515">script </span><span style="color: red">type</span><span style="color: blue">="text/javascript" </span><span style="color: red">src</span><span style="color: blue">="http://api.recaptcha.net/js/recaptcha_ajax.js"&gt;&lt;/</span><span style="color: #a31515">script</span><span style="color: blue">&gt;</span></pre>

[][8]

Y luego he definido una función showRecaptcha, que llama Recaptcha.Create:

<pre>&lt;script type="text/javascript"&gt;
    function showRecaptcha() {<br />        Recaptcha.create("clave_publica_de_recaptcha", <br />        "div_recaptcha", {<br />          theme: "red",<br />          tabindex: 0
        });
    }
&lt;/script&gt;</pre>

Finalmente, llamo a esta función en cuanto se ha mostrado el formulario modal. Para ello uso el callback onOpen de SimpleModal. Si mirais mi post anterior, vereis que ya usaba este callback para añadir un manejador de eventos javascript para ir comprobando (via AJAX) si el login del usuario estaba libre o no, sin necesidad de hacer submit de todo el popup. En mi caso tenía una función popup_open, y ha sido en ella que he añadido el código para mostrar el captcha:

<pre class="code"><span style="color: blue">function </span>popup_open(dialog) {
    dialog.overlay.fadeIn(<span style="color: #a31515">'slow'</span>, <span style="color: blue">function</span>() {
        dialog.container.fadeIn(<span style="color: #a31515">'slow'</span>, <span style="color: blue">function</span>() {
            dialog.data.fadeIn(<span style="color: #a31515">'slow'</span>, <span style="color: blue">function</span>() {
                showRecaptcha();<br />                <span style="color: #008000;">// resto de código...</span>
            });
        });
    });
}</pre>

[][8]

**Paso 3: Comprobar el captcha**

Esto, usando RecaptchaMvc es muy fácil. En mi caso, cuando el usuario pulsa el botón &ldquo;Enviar&rdquo; de mi popup, se hace un POST a una URL gestionada por la acción Signup del controlador Account, cuyo código es:

<pre class="code">[<span style="color: #2b91af">AcceptVerbs</span>(<span style="color: #2b91af">HttpVerbs</span>.Post)]
<span style="color: blue">public </span><span style="color: #2b91af">ActionResult </span>Signup(<span style="color: #2b91af">FormCollection </span>values)
{
    <span style="color: blue">string </span>privateKey = <span style="color: blue">this</span>.GetRecaptchaPrivateKey();
    <span style="color: blue">var </span>response = <span style="color: blue">this</span>.TryValidateRecaptcha
        (privateKey, values.ToValueProvider());
    <span style="color: blue">if </span>(response.IsValid) {
        <span style="color: green">// Recaptcha ha sido Ok
    </span>}
    <span style="color: blue">else </span>{
        <span style="color: green">// Recapcha ha ido mal
    </span>}
}</pre>

[][8]

Muy simple: Obtenemos la clave privada de Recaptcha (el método GetRecaptchaPrivateKey) es un método extensor mío, que obtiene la clave privada que guardo en el web.config. Luego simplemente llamamos a TryValidateRecaptcha (método extensor que incorpora RecapthcaMvc) y con esto ya sabemos si el captcha es correcto o no!

Y aquí teneis una captura de pantalla con todo funcionando:

[<img height="199" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_56275DE1.png" alt="image" border="0" title="image" style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" />][10] 

(Sí, sí... el estilo es muy... ASP.NET MVC :p :p :p)

Saludos!

Enlace del fichero [controllerextensions.cs de RecaptchaMvc preparado para ASP.NET MVC versión final][11].

 [1]: /blogs/etomas/archive/2009/04/14/mostrar-un-formulario-modal-con-asp-net-mvc-y-ajax.aspx
 [2]: /blogs/etomas/archive/2009/04/15/mostrar-un-formulario-modal-con-asp-net-mvc-y-ajax-ii.aspx
 [3]: http://recaptcha.net/
 [4]: http://recaptcha.net/resources.html
 [5]: http://weblogs.asp.net/awilinsk/
 [6]: http://weblogs.asp.net/awilinsk/archive/2008/12/09/recaptchamvc.aspx
 [7]: http://recaptchamvc.codeplex.com/SourceControl/ListDownloadableCommits.aspx "http://recaptchamvc.codeplex.com/SourceControl/ListDownloadableCommits.aspx"
 [8]: http://11011.net/software/vspaste
 [9]: http://recaptcha.net/apidocs/captcha/client.html
 [10]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_1DE3C567.png
 [11]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas.21042009/ControllerExtensions.cs.txt