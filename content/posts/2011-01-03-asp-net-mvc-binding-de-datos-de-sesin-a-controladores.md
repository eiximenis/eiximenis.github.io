---
title: 'ASP.NET MVC: Binding de datos de sesión a controladores'
author: eiximenis

date: 2011-01-03T11:08:00+00:00
geeks_url: /?p=1549
geeks_visits:
  - 2539
geeks_ms_views:
  - 1124
categories:
  - Uncategorized

---
Muy buenas! Que tal el fin de año? Empachados con turrones, polvorones y demás? En fin, vamos a inaugurar el 2011 y que mejor manera que hacerlo que con un post! 😉

En realidad hubiese querido que este post fuese el último del año anterior, pero no puede publicarlo antes por problemas logísticos. La idea del post surge <a target="_blank" href="http://twitter.com/luisruizpavon/status/20062211649052672" rel="noopener noreferrer">de un tweet que publicó Luis Ruiz Pavón.</a> Su pregunta era que tal acceder a la sesión desde un Model Binder para poner datos a disposición de los controladores. <a target="_blank" href="http://twitter.com/eiximenis/status/20062646694846464" rel="noopener noreferrer">Mi respuesta fue que yo usaría un value provider</a>, y así llegamos a este post.

Para conseguir binding de los datos de la sesión a los parámetros de un controlador no es necesario crear ningún Model Binder. En MVC2 se introdujo un concepto nuevo (del que ya he hablado varias veces por aquí) que se llama ValueProvider y que es el encargado de _acceder donde están los datos y ponerlos a disposición de los Model Binders_. Si ignoramos los value providers y hacemos un model binder que acceda a la sesión, entonces realmente nuestro model binder hace dos cosas:

  1. Ir donde están los datos (la sesión) y recogerlos
  2. Enlazar los datos con los parámetros de los controladores

Según la arquitectura de ASP.NET MVC los model binders sólo se encargan de lo segundo, y son los value providers quienes se encargan de lo primero. Así, pues, tened presente la regla:

  1. Si lo que queréis canviar es **cómo** se enlazan los datos (vengan de donde vengan) a los controladores: cread un model binder
  2. Si lo que queréis es **modificar de dónde** se obtienen los datos que se enlazan a los controladores: usad un value provider.

En nuestro caso, tal y como se enlazan los valores a los controladores ya nos va bien (el DefaultModelBinder es realmente bueno en su tarea), sólo que queremos que si un dato está en la sesión se coja de allí: necesitamos un value provider nuevo.

**Factoría de value providers**

En ASP.NET MVC los value providers se crean siempre mediante una factoría y lo que realmente registramos en el runtime son esas factorías. En cada petición ASP.NET MVC le pide a las distintas factorías que creen los value providers necesarios.

Así pues lo primero va a ser crear nuestra factoría, que devolverá objetos de _nuestro_ value provider vinculado a sesión:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff;">class</span> SessionValueProviderFactory : ValueProviderFactory<br />{<br />    <span style="color: #0000ff;">public</span> <span style="color: #0000ff;">override</span> IValueProvider GetValueProvider(ControllerContext controllerContext)<br />    {<br />        <span style="color: #0000ff;">return</span> <span style="color: #0000ff;">new</span> SessionValueProvider(controllerContext.HttpContext.Session);<br />    }<br />}</pre>
</div>

Simplemente debemos derivar de ValueProviderFactory y en el método GetValueProvider devolver una instancia de nuestro value provider. En mi caso devuelvo una instancia del SessionValueProvider y le paso la sesión en el constructor.

Debemos registrar esa factoría de value providers en el runtime de ASP.NET MVC. Para ello en el Global.asax basta con meter la siguiente línea (usualmente en el Application_Start):

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">ValueProviderFactories.Factories.Add(<span style="color: #0000ff;">new</span> SessionValueProviderFactory());</pre>
</div>

**El value provider**

Crear un value provider es &ldquo;tan sencillo&rdquo; como implementar la interfaz IValueProvider. De todos modos debemos conocer un poco como funcionan los model binders a los que debemos proporcionar los valores.

Los value providers vienen a ser como un &ldquo;diccionario enorme&rdquo; que los model binders consultan cuando quieren obtener un dato. Debemos saber cómo (&ldquo;con que clave&rdquo;) nos va a pedir el model binder los datos y como debemos dárselos. En un post mío de hace tiempo <a target="_blank" href="/blogs/etomas/archive/2010/05/10/asp-net-mvc-el-defaultmodelbinder.aspx" rel="noopener noreferrer">ya comenté como funciona el DefaultModelBinder</a>, y allí cuento también la interacción entre el DefaultModelBiner y los value providers. 

En fin, que todo ese rollete es para comentaros que muchas veces en lugar de implementar IValueProvider desde cero, es mucho mejor derivar de alguna de las clases que ya hay hechas, y hay una en concreto que nos viene al pelo<a target="_blank" href="http://msdn.microsoft.com/es-es/library/ee703471.aspx" rel="noopener noreferrer">: DictionaryValueProvider<TValue></a>. Esta clase implementa un value provider cuya fuente de datos es un diccionario cuyos valores son de tipo TValue. Y que es la sesión en el fondo sinó un gran diccionario cuyos valores son de tipo object?

Así pues creamos nuestra clase que derive de DictionaryValueProvider y lo único que tenemos que hacer es pasarle a nuestra clase base el diccionario que debe usar, que construimos a partir de la sesión:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff;">class</span> SessionValueProvider : DictionaryValueProvider&lt;<span style="color: #0000ff;">object</span>&gt;<br />{<br />    <span style="color: #0000ff;">public</span> SessionValueProvider(HttpSessionStateBase session)<br />        : <span style="color: #0000ff;">base</span>(CreateDictionary(session), CultureInfo.CurrentCulture)<br />    {<br /><br />    }<br /><br />    <span style="color: #0000ff;">private</span> <span style="color: #0000ff;">static</span> IDictionary&lt;<span style="color: #0000ff;">string</span>, <span style="color: #0000ff;">object</span>&gt; CreateDictionary(HttpSessionStateBase session)<br />    {<br />        var entries = session.Keys.Cast&lt;<span style="color: #0000ff;">string</span>&gt;().ToDictionary(key =&gt; key, key =&gt; session[key]);<br />        <span style="color: #0000ff;">return</span> entries;<br />    }<br />}</pre>
</div>

Trivial no? El método CreateDictionary simplemente crea un IDictionary a partir de los datos de la sesión.

**Y listos!** Hemos terminado, No necesitamos hacer nada más para que el binding de objetos de sesión funcione. El requisito para que el binding se efectúe es el de siempre: que el nombre del parámetro en la acción del controlador tenga el mismo nombre que la clave de sesión:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff;">public</span> <span style="color: #0000ff;">class</span> SessionController : Controller<br />{<br />    <span style="color: #0000ff;">public</span> ActionResult Put()<br />    {<br />        Session[<span style="color: #006080;">"ufo"</span>] = <span style="color: #006080;">"String en sessión"</span>;<br />        Session[<span style="color: #006080;">"complex"</span>] = <span style="color: #0000ff;">new</span> Foo()<br />                                 {<br />                                     Cadena = <span style="color: #006080;">"Una cadena en objeto complejo"</span>,<br />                                     Entero = 100,<br />                                     Lista = <span style="color: #0000ff;">new</span> List&lt;<span style="color: #0000ff;">int</span>&gt;() {1, 1, 3, 5, 8, 13}<br />                                 };<br />        <span style="color: #0000ff;">return</span> View();<br />    }<br />    <span style="color: #0000ff;">public</span> ActionResult Get(<span style="color: #0000ff;">string</span> ufo)<br />    {<br />        ViewData[<span style="color: #006080;">"data"</span>] = ufo;<br />        <span style="color: #0000ff;">return</span> View();<br />    }<br /><br />    <span style="color: #0000ff;">public</span> ActionResult GetClass(Foo complex)<br />    {<br />        <span style="color: #0000ff;">return</span> View(complex);<br />    }<br />}</pre>
</div>

En este controlador la acción Put coloca dos datos en la sesión: una cadena con clave &ldquo;ufo&rdquo; y un objeto de una clase llamada Foo, con clave &ldquo;complex&rdquo;. Las dos acciones restantes (Get y GetClass) usan el binding y obtienen los datos de la sesión.

**Ventajas de usar el binding para obtener los valores**

La ventaja principal de usar el binding para obtener los valores en lugar de acceder a la sesión directamente es que desacopla el código de las acciones de la sesión de HTTP. En definitiva, si quiero probar si la acción Get funciona correctamente, p.ej. usando un unit test, me basta con pasarle una cadena cualquiera. Si en el código de la acción accediese a la sesión debería tener acceso a la sesión (desde la prueba) y rellenarla.

Espero que os sea útil!

Un saludo!

PD: Teneis un proyecto en VS2010 con MVC2 con el código de dicho post en mi skydrive: <http://cid-6521c259e9b1bec6.office.live.com/self.aspx/BurbujasNet/ZipsPosts/MvcSessionBinding.zip>