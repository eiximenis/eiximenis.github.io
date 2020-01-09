---
title: 'Facebook Connect (iii): Eh! Que esta parte de mi web sólo es para usuarios de facebook! (ASP.NET MVC)'
description: 'Facebook Connect (iii): Eh! Que esta parte de mi web sólo es para usuarios de facebook! (ASP.NET MVC)'
author: eiximenis

date: 2009-12-15T17:34:38+00:00
geeks_url: /?p=1483
geeks_visits:
  - 3999
geeks_ms_views:
  - 1115
categories:
  - Uncategorized

---
Este post va a ser cortito… En los dos primeros posts de esta serie hemos visto <a href="http://geeks.ms/blogs/etomas/archive/2009/12/10/facebook-connect-si-est-225-s-en-facebook-bienvenido-a-mi-web.aspx" target="_blank" rel="noopener noreferrer">como podemos autenticar (logon) a un usuario de facebook</a> en nuestra web y <a href="http://geeks.ms/blogs/etomas/archive/2009/12/14/facebook-connect-ii-adi-243-s-amigo-adi-243-s-o-como-hacer-el-logout.aspx" target="_blank" rel="noopener noreferrer">como podemos desautenticarlo (logoff)</a>.

Yo uso ASP.NET MVC para mis desarrollos web, no voy a enumerar ahora las ventajas que en **mi opinión** tiene MVC sobre Webforms, sinó comentaros como podemos evitar el acceso de usuarios que no estén autenticados en facebook a ciertas regiones de nuestra web.

En ASP.NET MVC tenemos lo que se llaman _filtros_. A grandes rasgos un filtro es código que se ejecuta _antes y/o después_ de cada petición y que puede alterar el comportamiento por defecto de la petición. Digamos que es un modo rápido y sencillo de aplicar técnicas AOP en aplicaciones MVC. Los filtros se enganchan a las peticiones a las que afectan mediante el uso de atributos (que se aplican a las _acciones_ de los controladores, en MVC cada petición es atendida por una accion \_método\_ de un controlador).

Qué cosas podemos hacer con filtros? Pues imaginad: hacer log de las peticiones, tratar los errores que se den de una forma coherente, comprimir el resultado de la petición, o porque no, validar que el usuario esté autenticado.

Ya hay en ASP.NET MVC, un filtro que comprueba si el usuario está autenticado:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">[Authorize]<br /><span style="color: #0000ff">public</span> ActionResult ChangePassword()<br />{<br />    ViewData[<span style="color: #006080">"PasswordLength"</span>] = MembershipService.MinPasswordLength;<br />    <span style="color: #0000ff">return</span> View();<br />}</pre>
  
  <p>
    </div> 
    
    <p>
      El filtro <em>Authorize</em> comprueba que el usuario está autenticado, y si no es el caso lo redirige a la página de login. De esta manera la acción <em>ChangePassword</em> sólo puede ser ejecutada por usuarios autenticados.
    </p>
    
    <p>
      En <strong>mi caso</strong>, los usuarios pueden entrar en mi web usando una cuenta de facebook o bien una cuenta propia de mi web (autorización estándard de ASP.NET). Pero hay ciertas acciones que sólo están disponibles si el usuario se autentica via facebook. Así que me puse manos a la obra para crear un filtro parecido a [Authorize] pero que validase si el usuario está autenticado en facebook.
    </p>
    
    <p>
      <strong>Creando el filtro…</strong>
    </p>
    
    <p>
      Crear un filtro es realmente sencillo… el único punto delicado es saber <em>que</em> tipo de filtro queremos crear, ya que existen cuatro tipos:
    </p>
    
    <ol>
      <li>
        Authorization filters: Para implementar autorizaciones
      </li>
      <li>
        Action filters: Para implementar acciones a realizar antes o después de una acción de un controlador.
      </li>
      <li>
        Result filters: Para implementar acciones a realizar antes o después de que se haya ejecutado la vista asociada a una acción.
      </li>
      <li>
        Exception filters: Para gestionar errores producidos durante el tratamiento de una petición.
      </li>
    </ol>
    
    <p>
      Para crearnos nuestro propio filtro derivamos de la clase <em>FilterAttribute</em> e implementamos una interfaz u otra. En mi caso, dado que quiero implementar un <em>authorization filter</em> debo implementar la interfaz <em>IAuthorizationFilter</em>, así que manos a la obra!
    </p>
    
    <p>
      La interfaz es muy sencilla: tiene un sólo método:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">void</span> OnAuthorization(AuthorizationContext filterContext)</pre>
      
      <p>
        </div> 
        
        <p>
          En este método es donde comprobaremos que el usuario está autorizado. Si no lo está usaremos las propiedades del objeto <em>filterContext</em> para modificar la respuesta y generar una respuesta nueva (p.ej. una redirección). En caso que el usuario esté autorizado no hacemos nada y dejamos que se procese la acción del controlador.
        </p>
        
        <p>
          El código para saber si un usuario está autenticado en facebook, es muy simple:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">ConnectSession cs = <span style="color: #0000ff">new</span> ConnectSession(appKey, secretKey);<br /><span style="color: #0000ff">if</span> (!cs.IsConnected())<br />{<br />    <span style="color: #008000">// No está conectado via facebook</span><br />}</pre>
          
          <p>
            </div> 
            
            <p>
              Lo que yo quiero es <em>redirigir</em> los usuarios no autenticados via facebook a otra acción. En lugar de redirigirlos a la acción de login (como hace [Authorize]) yo quiero poder decidir en cada caso a que acción redirigir el usuario. Es decir, poder tener algo como:
            </p>
            
            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">[FacebookAuthorize(<span style="color: #006080">"Index"</span>, <span style="color: #006080">"Home"</span>)]<br /><span style="color: #0000ff">public</span> ActionResult LinkFacebookUser()<br />{<br />    <span style="color: #0000ff">return</span> View();<br />}</pre>
              
              <p>
                </div> 
                
                <p>
                  Si el usuario no está autorizado en facebook, quiero redirigir la petición a la acción Index del controlador Home.
                </p>
                
                <p>
                  El código base del método OnAuthorization queda tal y como sigue:
                </p>
                
                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> OnAuthorization(AuthorizationContext filterContext)<br />{<br />    <span style="color: #0000ff">string</span> appKey = <span style="color: #0000ff">this</span>.AppKey ?? ConfigurationManager.AppSettings[<span style="color: #006080">"ApiKey"</span>];<br />    <span style="color: #0000ff">string</span> secretKey = <span style="color: #0000ff">this</span>.SecretKey ?? ConfigurationManager.AppSettings[<span style="color: #006080">"Secret"</span>];<br /><br />    ConnectSession cs = <span style="color: #0000ff">new</span> ConnectSession(appKey, secretKey);<br />    <span style="color: #0000ff">if</span> (!cs.IsConnected())<br />    {<br />        <span style="color: #008000">// Redirigir usuario</span><br />    }    <br />}</pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Sólo nos queda lo más “fácil”: redirigir el usuario en caso de éste no se haya autenticado via facebook!
                    </p>
                    
                    <p>
                      <strong>Redirigiendo el usuario</strong>
                    </p>
                    
                    <p>
                      Para redirigir el usuario debemos crear un nuevo resultado y asignarlo a la propiedad <em>Result</em> del objeto filterContext recibido como parámetro. No hay ningún resultado de tipo “RedirectToActionResult”, en su lugar tenemos que usar el tipo <em>RedirectToRouteResult</em>, de la siguiente manera:
                    </p>
                    
                    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">var</span> rvalues = <span style="color: #0000ff">new</span> RouteValueDictionary();<br />rvalues[<span style="color: #006080">"action"</span>] = <span style="color: #0000ff">this</span>.Action;<br /><span style="color: #0000ff">if</span> (!<span style="color: #0000ff">string</span>.IsNullOrEmpty(<span style="color: #0000ff">this</span>.Controller))<br />{<br />    rvalues[<span style="color: #006080">"controller"</span>] = <span style="color: #0000ff">this</span>.Controller;<br />}<br /><br />filterContext.Result = <span style="color: #0000ff">new</span> RedirectToRouteResult(rvalues);</pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          Creamos un <em>RouteValueDictionary</em> y asignamos las entradas “action” y “controller” a la acción y controlador donde queremos redirigir el usuario, y a partir de este <em>RouteValueDictionary</em> creamos el <em>RedirectToRouteResult</em> que asignamos a la propiedad Result.
                        </p>
                        
                        <p>
                          Y listos! Con esto ya tenemos nuestro propio filtro creado!
                        </p>
                        
                        <p>
                          Os dejo aquí el código fuente completo, para que lo podáis examinar libremente!
                        </p>
                        
                        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #008000">/// &lt;summary&gt;</span><br /><span style="color: #008000">/// ActionFilter de MVC per redirigir la petició a un altre vista si l'usuari</span><br /><span style="color: #008000">/// no està autenticat a Facebook</span><br /><span style="color: #008000">/// &lt;/summary&gt;</span><br /><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> FacebookAuthorize : FilterAttribute, IAuthorizationFilter<br />{<br /><br />    <span style="color: #008000">/// &lt;summary&gt;</span><br />    <span style="color: #008000">/// Clau pública de l'API</span><br />    <span style="color: #008000">/// &lt;/summary&gt;</span><br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> AppKey { get; set; }<br /><br />    <span style="color: #008000">/// &lt;summary&gt;</span><br />    <span style="color: #008000">/// Clau privada de l'API</span><br />    <span style="color: #008000">/// &lt;/summary&gt;</span><br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> SecretKey { get; set; }<br /><br />    <span style="color: #008000">/// &lt;summary&gt;</span><br />    <span style="color: #008000">/// Controlador al qual es transfereix la petició (&lt;c&gt;null&lt;/c&gt; significa</span><br />    <span style="color: #008000">/// el controlador actual).</span><br />    <span style="color: #008000">/// &lt;/summary&gt;</span><br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Controller { get; <span style="color: #0000ff">private</span> set; }<br /><br />    <span style="color: #008000">/// &lt;summary&gt;</span><br />    <span style="color: #008000">/// Acció a la que es transfereix la petició</span><br />    <span style="color: #008000">/// &lt;/summary&gt;</span><br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Action { get; <span style="color: #0000ff">private</span> set; }<br /><br />    <span style="color: #008000">/// &lt;summary&gt;</span><br />    <span style="color: #008000">/// Construeix un objecte &lt;c&gt;FacebookAuthorize&lt;/c&gt; que redirigirà la petició</span><br />    <span style="color: #008000">/// a l'acció &lt;paramref name="action"/&gt; del controlador actual si l'usuari no</span><br />    <span style="color: #008000">/// està autenticat a facebook.</span><br />    <span style="color: #008000">/// &lt;/summary&gt;</span><br />    <span style="color: #008000">/// &lt;param name="action"&gt;Acció a la qual cal redirigir.&lt;/param&gt;</span><br />    <span style="color: #0000ff">public</span> FacebookAuthorize(<span style="color: #0000ff">string</span> action)<br />    {<br />        <span style="color: #0000ff">this</span>.Action = action;<br />    }<br /><br />    <span style="color: #008000">/// &lt;summary&gt;</span><br />    <span style="color: #008000">/// Construeix un objecte &lt;c&gt;FacebookAuthorize&lt;/c&gt; que redirigirà la petició</span><br />    <span style="color: #008000">/// a l'acció &lt;paramref name="action"/&gt; del controlador &lt;paramref name="controller" /&gt; </span><br />    <span style="color: #008000">/// si l'usuari no està autenticat a facebook.&lt;/param&gt;</span><br />    <span style="color: #008000">/// &lt;/summary&gt;</span><br />    <span style="color: #008000">/// &lt;param name="action"&gt;Acció a la qual cal redirigir.&lt;/param&gt;</span><br />    <span style="color: #008000">/// &lt;param name="controller"&gt;Controlador al qual cal redirigir.&lt;/param&gt;</span><br />    <span style="color: #0000ff">public</span> FacebookAuthorize(<span style="color: #0000ff">string</span> action, <span style="color: #0000ff">string</span> controller)<br />    {<br />        <span style="color: #0000ff">this</span>.Action = action;<br />        <span style="color: #0000ff">this</span>.Controller = controller;<br />    }<br /><br />    <span style="color: #008000">/// &lt;summary&gt;</span><br />    <span style="color: #008000">/// Valida la petició</span><br />    <span style="color: #008000">/// &lt;/summary&gt;</span><br />    <span style="color: #008000">/// &lt;param name="filterContext"&gt;Contexte de ASP.NET MVC&lt;/param&gt;</span><br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> OnAuthorization(AuthorizationContext filterContext)<br />    {<br />        <span style="color: #0000ff">string</span> appKey = <span style="color: #0000ff">this</span>.AppKey ?? ConfigurationManager.AppSettings[<span style="color: #006080">"ApiKey"</span>];<br />        <span style="color: #0000ff">string</span> secretKey = <span style="color: #0000ff">this</span>.SecretKey ?? ConfigurationManager.AppSettings[<span style="color: #006080">"Secret"</span>];<br /><br />        ConnectSession cs = <span style="color: #0000ff">new</span> ConnectSession(appKey, secretKey);<br />        <span style="color: #0000ff">if</span> (!cs.IsConnected())<br />        {<br />            <span style="color: #0000ff">var</span> rvalues = <span style="color: #0000ff">new</span> RouteValueDictionary();<br />            rvalues[<span style="color: #006080">"action"</span>] = <span style="color: #0000ff">this</span>.Action;<br />            <span style="color: #0000ff">if</span> (!<span style="color: #0000ff">string</span>.IsNullOrEmpty(<span style="color: #0000ff">this</span>.Controller))<br />            {<br />                rvalues[<span style="color: #006080">"controller"</span>] = <span style="color: #0000ff">this</span>.Controller;<br />            }<br /><br />            filterContext.Result = <span style="color: #0000ff">new</span> RedirectToRouteResult(rvalues);<br />        }    <br />    }<br />}</pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              Un saludo!!!!
                            </p>
                            
                            <p>
                              PD: Al final el post no ha sido tan cortito… aiinsss! Si es que cuando empiezo a escribir no paro!! 😛
                            </p>