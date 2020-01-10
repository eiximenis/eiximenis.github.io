---
title: 'ASP.NET: Obtener el ID del usuario actual'

author: eiximenis

date: 2010-12-16T13:24:20+00:00
geeks_url: /?p=1546
geeks_visits:
  - 24339
geeks_ms_views:
  - 7450
categories:
  - Uncategorized

---
Buenas!

No se vosotros, pero yo cuando desarrollo mis aplicaciones, si uso FKs de la otabla de usuarios, las hago en base al ID del usuario, nunca en base a su nombre. Así pues, saber el ID del usuario actualmente autenticado en mi aplicación es algo fundamental.

<!--more-->

Primero, para saber el **nombre** del usuario autenticado podemos usar:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">string</span> userName = HttpContext.Current.User.Identity.Name;</pre>
  
  <p>
    </div>
  </p>
  
  <p>
    El tema está en que las mentes pensantes que parieron el sistema de proveedores de autenticación en ASP.NET no tuvieron a bien a poner un campo para guardar el ID.
  </p>
  
  <p>
    Por suerte obtenerlo es trivial:
  </p>
  
  <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">int</span> i = (<span style="color: #0000ff">int</span>)Membership.GetUser().ProviderUserKey;<br /></pre>
    
    <p>
      </div> 
      
      <p>
        <strong><u>Ojo con ese código:</u></strong> Yo uso un membership provider propio, ya que mi base de datos usa ints para los IDs de usuarios. Si usáis el membership provider que viene por defecto en ASP.NET, el cast lo debéis hacer a Guid y no int.
      </p>
      
      <p>
        Bueno todo muy bonito, pero antes que descorchéis el cava: eso <strong>hace una llamada a la base de datos</strong>. Además se trae <em>todos los campos</em> del registro correspondiente de la tabla de usuarios (que si usáis el membership provider que viene por defecto tiene la hostia y pico). O sea que cuidado con usar eso a mansalva… 🙂
      </p>
      
      <p>
        <strong>Eeeerrr… ¿se puede SIN necesidad de acceder a la base de datos?</strong>
      </p>
      
      <p>
        Bueno… esa es la gran pregunta, no nos vamos a engañar 😉 Hay varias maneras de poder acceder al ID del usuario <em>sin</em> hacer un round-trip a la base de datos, pero a si a bote pronto se me ocurren dos:
      </p>
      
      <ol>
        <li>
          Guardarlo en una variable de sesión: Podemos guardar el ID en una variable de sesión y consultarla cuando la necesitamos. Para una escalabilidad máxima podéis no usar <em>sticky sessions</em> y en el Session_Start guardar dicha variable con el ID. Si no usáis sticky sessions un mismo usuario puede iniciar sesión en varios IIS a la vez, pero en nuestro caso no es problemático (simplemente se consultará el ID del usuario cada vez que inicie sesión). Eso sí, estoy asumiendo que <strong>no</strong> guardáis nada más en la sesión (es decir que funcionalmente <em>no</em> dependéis de la sesión).
        </li>
        <li>
          Guardarlo en la cookie de autenticación del usuario. Esto no es, ni de lejos tan sencillo como el punto anterior, pero ya que hemos llegado hasta aquí…
        </li>
      </ol>
      
      <p>
        <strong>Modificar la cookie de autenticación</strong>
      </p>
      
      <p>
        Primero debemos hacer que cuando se cree la cookie de autenticación se añada el ID del usuario. En mi caso, como siempre, uso ASP.NET MVC, así que modificaré el código de AccountController (que es el que genera VS). Para aplicaciones webforms ese código debe colocarse cuando se va a hacer login del usuario.
      </p>
      
      <p>
        En el código por defecto que genera VS para autenticar un usuario se usa:
      </p>
      
      <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
        <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> SignIn(<span style="color: #0000ff">string</span> userName, <span style="color: #0000ff">bool</span> createPersistentCookie)<br />{<br />    <span style="color: #0000ff">if</span> (String.IsNullOrEmpty(userName)) <span style="color: #0000ff">throw</span> <span style="color: #0000ff">new</span> ArgumentException(<span style="color: #006080">"Value cannot be null or empty."</span>, <span style="color: #006080">"userName"</span>);<br /><br />    FormsAuthentication.SetAuthCookie(userName, createPersistentCookie);<br />}</pre>
        
        <p>
          </div> 
          
          <p>
            Este código está en la clase <em>FormsAuthenticationService</em> (dentro de AccountsModel.cs) y no tiene ningún secreto: lo que hace es crear la cookie de autenticación de ASP.NET.
          </p>
          
          <p>
            En nuestro caso vamos a modificar ese código por el siguiente:
          </p>
          
          <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
            <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> SignIn(<span style="color: #0000ff">string</span> userName, <span style="color: #0000ff">bool</span> createPersistentCookie)<br />{<br />    <span style="color: #0000ff">if</span> (String.IsNullOrEmpty(userName)) <span style="color: #0000ff">throw</span> <span style="color: #0000ff">new</span> ArgumentException(<span style="color: #006080">"Value cannot be null or empty."</span>, <span style="color: #006080">"userName"</span>);<br />    FormsAuthentication.SetAuthCookie(userName, createPersistentCookie);<br />    <span style="color: #0000ff">int</span> id = 100;        <span style="color: #008000">// Aquí va el ID del usuario que pillaríamos de la BBDD</span><br />    <span style="color: #0000ff">string</span> userData = <font color="#0000ff">id.ToString();<span style="color: #0000ff"></span></font><br />    FormsAuthenticationTicket ticket = <span style="color: #0000ff">new</span> FormsAuthenticationTicket(1, userName, DateTime.Now, DateTime.Now.AddMinutes(30), createPersistentCookie, userData); <br />    <span style="color: #0000ff">string</span> encTicket = FormsAuthentication.Encrypt(ticket); <br />    HttpCookie faCookie = <span style="color: #0000ff">new</span> HttpCookie(FormsAuthentication.FormsCookieName, encTicket); <br />    HttpContext.Current.Response.Cookies.Add(faCookie);<br />}</pre>
            
            <p>
              </div> 
              
              <p>
                Lo que hacemos es crear una cookie, con datos adicionales (el ID del usuario).
              </p>
              
              <p>
                Ahora lo que nos toca es la otra parte: reemplazar el valor de HttpContext.Current.User.Identity por uno propio que tenga el ID. Para ello usamos el evento Post<em>Authenticate_Request</em>:
              </p>
              
              <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">protected</span> <span style="color: #0000ff">void</span> Application_PostAuthenticateRequest(<span style="color: #0000ff">object</span> sender, EventArgs e)<br />{<br />    HttpCookie authCookie = Request.Cookies[FormsAuthentication.FormsCookieName];<br />    <span style="color: #0000ff">if</span> (authCookie != <span style="color: #0000ff">null</span>)<br />    {<br />        FormsAuthenticationTicket authTicket = FormsAuthentication.Decrypt(authCookie.Value);<br />        CustomIdentity identity = <span style="color: #0000ff">new</span> CustomIdentity(authTicket.Name, authTicket.UserData);<br />        GenericPrincipal newUser = <span style="color: #0000ff">new</span> GenericPrincipal(identity, <span style="color: #0000ff">new</span> <span style="color: #0000ff">string</span>[] {});<br />        Context.User = newUser;<br />    }<br />}</pre>
                
                <p>
                  </div> 
                  
                  <p>
                    Recogemos la cookie de autenticación, desencriptamos el ticket de autenticación por forms y con los datos (el nombre y el UserData) creamos un objeto de tipo CustomIdentity, clase nuestra que nos implementa IIdentity. Luego la incrustamos dentro de un GenericPrincipal y lo establecemos a la propiedad User del HttpContext.
                  </p>
                  
                  <p>
                    <strong>Nota:</strong> El segundo parámetro del constructor de GenericPrincipal es el array de roles a los que pertenece el usuario. En mi caso no uso roles, así que le asigno un array vacío.
                  </p>
                  
                  <p>
                    La clase CustomIdentity es tal y como sigue:
                  </p>
                  
                  <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">class</span> CustomIdentity : IIdentity<br />  {<br /><br />      <span style="color: #0000ff">public</span> CustomIdentity(<span style="color: #0000ff">string</span> name, <span style="color: #0000ff">string</span> id)<br />      {<br />          IsAuthenticated = <span style="color: #0000ff">true</span>;<br />          Name = name;<br />          Id = Int32.Parse(id);<br />          AuthenticationType = <span style="color: #006080">"Forms"</span>;<br />      }<br /><br />      <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> AuthenticationType { get; <span style="color: #0000ff">private</span> set; }<br />      <span style="color: #0000ff">public</span> <span style="color: #0000ff">bool</span> IsAuthenticated { get; <span style="color: #0000ff">private</span> set; }<br />      <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Name { get; <span style="color: #0000ff">private</span> set;}<br />      <span style="color: #0000ff">public</span> <span style="color: #0000ff">int</span> Id { get; <span style="color: #0000ff">private</span> set; }<br />  }</pre>
                    
                    <p>
                      </div> 
                      
                      <p>
                        De esta manera, ahora podemos al Id del usuario, desde un controlador:
                      </p>
                      
                      <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                        <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">CustomIdentity ci = (CustomIdentity)ControllerContext.HttpContext.User.Identity;<br /><span style="color: #0000ff">int</span> IdUsuario = ci.Id;</pre>
                        
                        <p>
                          </div> 
                          
                          <p>
                            Un misterio con el que me he encontrado es que el código de PostAuthenticateRequest si se pone en AuthenticateRequest (que parece que debería funcionar igual), se queja diciendo que la clase “CustomIdentity” no es serializable. No tengo muy claro porque ocurre eso y eso si que parece ser propio de MVC. Aquí hay más información al respecto: <a title="http://stackoverflow.com/questions/1884030/implementing-a-custom-identity-and-iprincipal-in-mvc" href="http://stackoverflow.com/questions/1884030/implementing-a-custom-identity-and-iprincipal-in-mvc">http://stackoverflow.com/questions/1884030/implementing-a-custom-identity-and-iprincipal-in-mvc</a>
                          </p>
                          
                          <p>
                            Y Listos!
                          </p>
                          
                          <p>
                            Con esto podemos acceder al ID de nuestros usuarios sin necesidad de usar para nada la base de datos. Además, dado que estamos usando el sistema de autenticación de ASP.NET (no hacemos nada <em>raro</em>), nos siguen funcionando los filtros de autenticación como [Authorize].
                          </p>
                          
                          <p>
                            Un saludo!
                          </p>
                          
                          <p>
                            <strong>Referencia:</strong> <a title="http://stackoverflow.com/questions/1064271/asp-net-mvc-set-custom-iidentity-or-iprincipal" href="http://stackoverflow.com/questions/1064271/asp-net-mvc-set-custom-iidentity-or-iprincipal">http://stackoverflow.com/questions/1064271/asp-net-mvc-set-custom-iidentity-or-iprincipal</a>
                          </p>