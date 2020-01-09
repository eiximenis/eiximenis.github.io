---
title: 'Facebook Connect (iv): Que todo el mundo sepa lo que has hecho!'
description: 'Facebook Connect (iv): Que todo el mundo sepa lo que has hecho!'
author: eiximenis

date: 2010-02-02T12:27:00+00:00
geeks_url: /?p=1492
geeks_visits:
  - 2526
geeks_ms_views:
  - 790
categories:
  - Uncategorized

---
Se comenta que las redes sociales dan fama, mujeres y dinero aunque no necesariamente en este orden...

En los tres primeros posts sobre facebook connect vimos como <a target="_blank" href="/blogs/etomas/archive/2009/12/10/facebook-connect-si-est-225-s-en-facebook-bienvenido-a-mi-web.aspx" rel="noopener noreferrer">permitir al usuario que hiciera login con su cuenta de facebook</a>, como <a target="_blank" href="/blogs/etomas/archive/2009/12/14/facebook-connect-ii-adi-243-s-amigo-adi-243-s-o-como-hacer-el-logout.aspx" rel="noopener noreferrer">implementar el logout</a> y como <a target="_blank" href="/blogs/etomas/archive/2009/12/15/facebook-connect-iii-eh-que-esta-parte-de-mi-web-s-243-lo-es-para-usuarios-de-facebook-asp-net-mvc.aspx" rel="noopener noreferrer">crear zonas &ldquo;privadas&rdquo; de nuestra web sólo para usuarios de facebook</a>.

Hoy vamos a ir un paso más allá: vamos a ver como podemos publicar mensajes en el muro del usuario de facebook autenticado. De esta manera sus amigos verán los logros que nuestro usuario consigue en nuestra web y más importante aún: verán nuestra web! Si el mensaje es _sugerente_ podremos conseguir un buen puñado de visitas a nuestra web! Todos ganamos! El usuario consigue fama y mujeres, y nosotros... dinero. Lo que decía al principio 🙂

**1. Montando toda la infrastructura...**

Aprovechando que ha salido ya ASP.NET MVC 2 RC, este es un buen momento para empezar a usarlo, no? 😉

<a target="_blank" href="http://www.microsoft.com/downloads/details.aspx?FamilyID=3b537c55-0948-4e6a-bf8c-aa1a78878da0&displaylang=en" rel="noopener noreferrer">Descargaos ASP.NET MVC 2 RC</a> e instaláoslo. Una novedad es que ahora (a diferencia de MVC 1) podemos crear una aplicación MVC vacía... parece una chorrada, pero cuando se hacen demos se agradece no ir por ahí borrando el controlador Account y todas sus vistas asociadas! 😉

Vamos a hacer un proyecto super simple: un botón para que usuario pueda entrar sus credenciales de facebook, y una vez las haya entrado, le mostraremos una página donde podrá entrar un texto que será publicado en el perfil del usuario **previa aceptación suya**.

Dadle a &ldquo;New project&rdquo; &ndash;> &ldquo;ASP.NET MVC 2 Empty Web Application&rdquo;. Esto os creará un proyecto vacío, pero con la estructura de ASP.NET MVC.

Recordad que para que connect os funcione debéis tener creada vuestra aplicación facebook y el fichero xd_receiver.htm debe estar en vuestra aplicación (tal y como se cuenta en el primer post de esta serie). Ah, y no os olvideis de añadir una referencia al <a target="_blank" href="http://www.codeplex.com/FacebookToolkit/" rel="noopener noreferrer">Facebook Developer Toolkit</a> (facebook.dll).

**2. Definiendo la aplicación**

Nuestra aplicación va a ser simple: un botón de facebook connect para que el usuario se auntentique. Una vez autenticado vamos a pedirle que entre un texto que será publicado en su muro (recordad: previa aceptación).

Para ello vamos a tener un controlador (vamos a ser originales y vamos a llamarlo Home) con tres vistas: la vista inicial (Index) desde donde el usuario podrá autenticarse con su cuenta de facebook, otra vista (Welcome) donde redireccionaremos a los usuarios autenticados para que entren un texto que se publicará en su muro y finalmente la vista Publish que será la que publicará el elemento en el muro del usuario.

**3. El controlador Home**

Vamos a empezar por el controlador. Vamos a tener dos acciones: Index que se limitará a mostrar la vista inicial y Welcome que mostrará la vista con el campo de texto o bien lo publicará en facebook, dependiendo de si se entra via GET o via POST.

La acción Index es muy simple:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> ActionResult Index()<br />{<br />    ConnectSession cs = <span style="color: #0000ff">this</span>.GetConnectSession();<br />    <span style="color: #0000ff">if</span> (cs.IsConnected())<br />    {<br />        <span style="color: #0000ff">return</span> RedirectToAction(<span style="color: #006080">"Welcome"</span>);<br />    }<br />    <span style="color: #0000ff">return</span> View();<br />}</pre>
  
  <p>
    </div> 
    
    <p>
      El método GetConnectSession es un método extensor de Controller que me devuelve una ConnectSession (clase que pertence al <a target="_blank" href="http://www.codeplex.com/FacebookToolkit/" rel="noopener noreferrer">Facebook Developer Toolkit</a>), que está definido así:
    </p>
    
    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">class</span> ControllerExtensions<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> ConnectSession GetConnectSession(<span style="color: #0000ff">this</span> Controller self)<br />    {<br />        <span style="color: #0000ff">string</span> appKey = ConfigurationManager.AppSettings[<span style="color: #006080">"ApiKey"</span>];<br />        <span style="color: #0000ff">string</span> secretKey = ConfigurationManager.AppSettings[<span style="color: #006080">"Secret"</span>];<br />        <span style="color: #0000ff">return</span> <span style="color: #0000ff">new</span> ConnectSession(appKey, secretKey);<br />    }<br />}</pre>
      
      <p>
        </div> 
        
        <p>
          Efectivamente en mi web.config tengo mi clave API y mi clave secreta como elementos dentro de <appSettings />
        </p>
        
        <p>
          La acción de Welcome es muy parecida a la acción Index, salvo por la diferencia de que en Index mirábamos si el usuario ya estaba autenticado para redirigirlo a Welcome y aquí haremos al revés: si no está autenticado lo mandaremos para Index:
        </p>
        
        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> ActionResult Welcome()<br />{<br />    ConnectSession cs = <span style="color: #0000ff">this</span>.GetConnectSession();<br />    <span style="color: #0000ff">if</span> (!cs.IsConnected())<br />    {<br />        <span style="color: #0000ff">return</span> RedirectToAction(<span style="color: #006080">"Index"</span>);<br />    }<br /><br />    <span style="color: #0000ff">return</span> View();<br />}</pre>
          
          <p>
            </div> 
            
            <p>
              <strong>4. Las vistas...</strong>
            </p>
            
            <p>
              Necesitamos (de momento) dos vistas: Home/Index.aspx y Home/Welcome.aspx. La primera debe contener simplemente un botón de facebook connect, renderizado usando XFBML. No pongo el código aquí, ya que es idéntico al del primer post de esta serie y luego tenéis el adjunto con todo el código...
            </p>
            
            <p>
              Me interesa más que veais la vista Welcome. Esta vista debe renderizar un cuadro de texto para que el usuario pueda entrar el texto que luego debemos publicar en el muro. El código de la vista Welcome es:
            </p>
            
            <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
              <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #006080">&lt;</span>%@ Page Title="" Language="C#" MasterPageFile="~/Views/Shared/Site<span style="color: #cc6633">.Master</span>" Inherits="System<span style="color: #cc6633">.Web</span><span style="color: #cc6633">.Mvc</span><span style="color: #cc6633">.ViewPage</span><span style="color: #006080">&lt;</span>WallPublish<span style="color: #cc6633">.Models</span><span style="color: #cc6633">.WallMessageModel</span><span style="color: #006080">&gt;</span>" %<span style="color: #006080">&gt;</span><br /><br /><span style="color: #006080">&lt;</span>asp:Content ID="Content1" ContentPlaceHolderID="TitleContent" runat="server"<span style="color: #006080">&gt;</span><br />    Welcome<br /><span style="color: #006080">&lt;</span>/asp:Content<span style="color: #006080">&gt;</span><br /><br /><span style="color: #006080">&lt;</span>asp:Content ID="Content2" ContentPlaceHolderID="MainContent" runat="server"<span style="color: #006080">&gt;</span><br /><br />    <span style="color: #006080">&lt;</span><span style="color: #0000ff">h2</span><span style="color: #006080">&gt;</span>Welcome<span style="color: #006080">&lt;</span>/<span style="color: #0000ff">h2</span><span style="color: #006080">&gt;</span><br />    <span style="color: #006080">&lt;</span><span style="color: #0000ff">table</span><span style="color: #006080">&gt;</span><br />    <span style="color: #006080">&lt;</span><span style="color: #0000ff">tr</span><span style="color: #006080">&gt;</span><br />    <span style="color: #006080">&lt;</span>% using (Html<span style="color: #cc6633">.BeginForm</span>()) { %<span style="color: #006080">&gt;</span><br />        <span style="color: #006080">&lt;</span><span style="color: #0000ff">td</span><span style="color: #006080">&gt;</span><br />        <span style="color: #006080">&lt;</span>%= Html<span style="color: #cc6633">.LabelFor</span>(x=<span style="color: #006080">&gt;</span>x<span style="color: #cc6633">.Text</span>) %<span style="color: #006080">&gt;</span><br />        <span style="color: #006080">&lt;</span>/<span style="color: #0000ff">td</span><span style="color: #006080">&gt;</span><br />        <span style="color: #006080">&lt;</span><span style="color: #0000ff">td</span><span style="color: #006080">&gt;</span><br />        <span style="color: #006080">&lt;</span>%= Html<span style="color: #cc6633">.EditorFor</span>(x=<span style="color: #006080">&gt;</span>x<span style="color: #cc6633">.Text</span>) %<span style="color: #006080">&gt;</span><br />        <span style="color: #006080">&lt;</span>%= Html<span style="color: #cc6633">.ValidationMessageFor</span>(x=<span style="color: #006080">&gt;</span>x<span style="color: #cc6633">.Text</span>) %<span style="color: #006080">&gt;</span><br />        <span style="color: #006080">&lt;</span>/<span style="color: #0000ff">td</span><span style="color: #006080">&gt;</span><br />    <span style="color: #006080">&lt;</span>% } %<span style="color: #006080">&gt;</span><br />    <span style="color: #006080">&lt;</span>/<span style="color: #0000ff">tr</span><span style="color: #006080">&gt;</span><br />    <span style="color: #006080">&lt;</span>/<span style="color: #0000ff">table</span><span style="color: #006080">&gt;</span><br /><br /><span style="color: #006080">&lt;</span>/asp:Content<span style="color: #006080">&gt;</span></pre>
              
              <p>
                </div> 
                
                <p>
                  Lo más interesante aquí es el uso de templated helpers para renderizar un editor para la propiedad Text del modelo WallMessageModel. El código Html.EditorFor(x=>x.Text) es quien renderiza un <em>editor</em> para la propiedad Text del modelo. En mi caso esta propiedad es de tipo string, por lo que se va a renderizar un textbox. También es interesante observar el uso de Html.ValidationMessageFor que va a permitir poner mensajes de error en caso de que alguno de los datos entrados sea incorrecto.
                </p>
                
                <p>
                  Ah, el modelo WallMessageModel es super simple: solo una propiedad Text de tipo string. La peculiaridad es que vamos a usar <a target="_blank" href="http://msdn.microsoft.com/en-us/library/dd901590(VS.95).aspx" rel="noopener noreferrer">Data Annotations</a> para evitar que el texto que se entre sea una cadena vacía:
                </p>
                
                <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">public class WallMessageModel<br />{<br />    [Required(ErrorMessage="Text es obligatorio")]<br />    public string Text <span style="color: #006080">{ get;</span> <span style="color: #006080">set;</span> }<br />}</pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      El uso de [Required] en la propiedad Text, convierte esta en obligatoria.
                    </p>
                    
                    <p>
                      Ok, ya tenemos la vista que renderiza un formulario (Html.BeginForm). Por defecto el formulario se envía via POST a la misma URL de la vista (Home/Welcome en nuestro caso), por lo tanto vamos a necesitar una nueva acción Welcome en el controlador Home pero que trate peticiones POST:
                    </p>
                    
                    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">[HttpPost()]<br /><span style="color: #0000ff">public</span> ActionResult Welcome(WallMessageModel data)<br />{<br />    <span style="color: #0000ff">if</span> (ModelState.IsValid)<br />    {<br />        <span style="color: #0000ff">return</span> View(<span style="color: #006080">"Publish"</span>, data);<br />    }<br />    <span style="color: #0000ff">else</span><br />    {<br />        <span style="color: #0000ff">return</span> View(data);<br />    }<br />}</pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          Fijaos en tres cosas:
                        </p>
                        
                        <ol>
                          <li>
                            El uso de HttpPost hace que esta acción trate solamente peticiones via POST.
                          </li>
                          <li>
                            El parámetro de la acción es de tipo WallMessageModel. ASP.NET MVC es capaz de crear un objeto de tipo WallMessageModel siempre y cuando la petición POST contenga todos los datos para crearlo. En nuestro caso dado que hemos metido en el formulario un editor para la única propiedad de la clase, la petición POST va a contener los datos suficientes y ASP.NET MVC nos va a poder crear un objeto WallMessageModel con los datos entrados por el usuario.
                          </li>
                          <li>
                            El uso de ModelState.IsValid para comprobar si el modelo recibido es correcto: esto aplica las reglas de Data Annotations y devuelve true si el modelo las cumple o no. Si el usuario ha entrado un texto vació Model.IsValid será false. Si este es el caso &ldquo;simplemente&rdquo; mandamos el modelo &ldquo;de vuelta&rdquo; a la misma vista. Esto renderizará de nuevo la vista, con los valores rellenados por el usuario, y el mensaje de error (gracias al uso de Html.ValidationMessageFor). Por otro lado si todo ha ido bien, manda el usuario a la vista &ldquo;Publish&rdquo; y le pasa el modelo.
                          </li>
                        </ol>
                        
                        <p>
                          <strong>5. La vista de publicación</strong>
                        </p>
                        
                        <p>
                          Sólo nos queda crear la vista que publica el texto en facebook. Para ello vamos a usar el método <a target="_blank" href="http://developers.facebook.com/docs/?u=facebook.jslib.FB.Connect.streamPublish" rel="noopener noreferrer">streamPublish</a> del API de facebook. Este método publica una variable de tipo <em>attachment</em> que tiene varios campos (p.ej. el texto, una URL, el tipo de attachment por si queremos publicar imágenes, ...). La descripción completa de todos los campos está en la <a target="_blank" href="http://wiki.developers.facebook.com/index.php/Attachment_%28Streams%29" rel="noopener noreferrer">definición de attachment en la wiki de facebook developer</a>.
                        </p>
                        
                        <p>
                          A modo de ejemplo vamos a usar los campos:
                        </p>
                        
                        <ul>
                          <li>
                            name: Título del post
                          </li>
                          <li>
                            href: Una URL que apunta a &ldquo;quien ha realizado el post&rdquo;.
                          </li>
                          <li>
                            caption: El subtítulo del post
                          </li>
                          <li>
                            description: El texto largo del post.
                          </li>
                        </ul>
                        
                        <p>
                          En mi caso dejo los 3 primeros fijos y en el cuarto pongo el valor que ha entrado el usuario:
                        </p>
                        
                        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">&lt;script type=<span style="color: #006080">"text/javascript"</span>&gt;<br />FB.init(<span style="color: #006080">"TU_CLAVE_API"</span>, <span style="color: #006080">"/xd_receiver.htm"</span>);<br />FB.ensureInit(function() {<br />    var attachment = { <span style="color: #006080">'name'</span>: <span style="color: #006080">'TestGeeks'</span>, <span style="color: #006080">'href'</span>: <span style="color: #006080">'http://geeks.ms'</span>, <span style="color: #006080">'caption'</span>: <span style="color: #006080">'{*actor*} ha hecho algo'</span>,<br />        <span style="color: #006080">'description'</span>: <span style="color: #006080">'&lt;%= Model.Text %&gt;'</span><br />    };<br />    <br />    FB.Connect.streamPublish(<span style="color: #006080">''</span>, attachment);<br />});<br />&lt;/script&gt;<br />&lt;h2&gt;Mensaje publicado&lt;/h2&gt;</pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              Fijaos en el uso de {*actor*}: esto se sustituye por el nombre del usuario al que publicamos el mensaje...
                            </p>
                            
                            <p>
                              Si ejecutais el código vereis como al entrar un texto en el textbox y pulsar enter, facebook va a pedir confirmación del texto a publicar en el muro. Si la aceptáis... el mensaje será publicado en el muro!
                            </p>
                            
                            <p>
                              Notáis el tintineo de los euros que van cayendo??? 🙂
                            </p>
                            
                            <p>
                              Saludos!!!
                            </p>
                            
                            <p>
                              Os dejo el <a target="_blank" href="http://cid-6521c259e9b1bec6.skydrive.live.com/self.aspx/BurbujasNet/ZipsPosts/WallPublish.zip" rel="noopener noreferrer">zip con todo el código</a> (en mi skydrive). Para cualquier duda que tengáis... ya sabéis: contactad conmigo!!!
                            </p>