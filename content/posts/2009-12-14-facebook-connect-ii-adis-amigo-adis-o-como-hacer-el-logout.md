---
title: 'Facebook Connect (ii): Adiós, amigo adiós (o como hacer el logout).'
author: eiximenis

date: 2009-12-14T16:06:12+00:00
geeks_url: /?p=1482
geeks_visits:
  - 3353
geeks_ms_views:
  - 1536
categories:
  - Uncategorized

---
Hola a todos! Este es el segundo post de una serie que iré haciendo contando _mis_ experiencias con Facebook Connect. En el primer post vimos [como usar facebook connect para implementar un single sign on en nuestra web][1] (o sea que los usuarios puedan entrar en nuestra web usando el login y password de facebook).

Ahora viene la segunda parte… da igual lo buena que sea tu web, llegará un momento en que el usuario querrá irse y no creo que le guste mucho que dejemos su sesión abierta :p. Tened presente que cuando usamos connect, cuando el usuario abre la sesión en _nuestra_ web, también la abre en facebook y viceversa: cuando cerramos la sesión en nuestra web también cerramos su sesión de facebook.

**Método 1 (que no funciona)**

En el post anterior, introduje FDT ([Facebook Developer Toolkit][2]), un conjunto de clases que permiten llamar a la API REST de facebook desde código C# (en lugar de usar la API javascript del propio facebook). Una de las clases que incorpora FDT es la ConnectSession, que encapsula una conexión de Facebook Connect.

Pues bien: ConnectSession tiene un método Logout, así que lo más sencillo es hacer lo siguiente:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">var</span> fbc = <span style="color: #0000ff">this</span>.GetConnectSession();<br /><span style="color: #0000ff">if</span> (fbc.IsConnected())<br />{<br />    fbc.Logout();<br />}</pre>
  
  <p>
    </div> 
    
    <p>
      ¡Y listos! ¿Listos? Pues no… esto no funciona. Lo podeis comprobar fácilmente: si <strong>sin cerrar</strong> el navegador abrís otra pestaña y os vais a facebook, veréis que entráis directamente sin que os pida las credenciales. Todavía estáis autenticados en facebook.
    </p>
    
    <p>
      Después de hacer algunas pruebas lo siguiente que hice, fue navegar por el código fuente de FDC y mirar que hacia <a href="http://facebooktoolkit.codeplex.com/SourceControl/changeset/view/39697#422535" target="_blank" rel="noopener noreferrer">el método Logout de la clase ConnectSession</a>, y eso es lo que me encontré:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #008000">/// &lt;summary&gt;</span><br /><span style="color: #008000">/// Logs out user</span><br /><span style="color: #008000">/// &lt;/summary&gt;</span><br /><span style="color: #0000ff">public</span> <span style="color: #0000ff">override</span> <span style="color: #0000ff">void</span> Logout()<br />{<br />}<br /></pre>
      
      <p>
        </div> 
        
        <p>
          Vaya… pues claro que no me funcionaba! ¿Porque está vacío este método? Bueno, pues FDT proporciona distintos tipos de sesiones para dar soporte a varias funcionalidades (p.ej. aquí nos centramos en Connect, pero se puede usar FDT para desarrollar aplicaciones que funcionen <em>dentro</em> de facebook). Según sea el uso que le demos a FDT usaremos una clase de sesión u otra, pero todas derivan de FacebookSesion, clase abstracta que define <em>todos</em> los métodos que todas las sesiones deben implementar. El problema está en que algunos métodos no pueden ser implementados en según que clases derivadas, debido a que Facebook no ofrece las mismas funcionalidades según el tipo de aplicación que estemos desarrollando…
        </p>
        
        <p>
          … no se si me he explicado bien! Resumiendo: Facebook <strong>no</strong> ofrece API REST para cerrar una sesión de Connect. Por eso la clase ConnectSession no implementa el método Logout. Quizá un throw new NotImplementedException hubiese sido mejor, ya que almenos daría una pista visual y rápida de lo que realmente ocurre, en lugar de tener un método vacío que no hace nada…
        </p>
        
        <p>
          <strong>Método 2 (el que sí funciona)</strong>
        </p>
        
        <p>
          Para hacer el logout del usuario, debemos usar el API Javascript de Facebook. De hecho, básicamente, debemos llamar al método FB.Connect.logout, <em>sólo</em> que hay que tener presentes un par de puntos:
        </p>
        
        <ol>
          <li>
            <em>Antes</em> de llamar a FB.Connect.logout debemos llamar a FB.Init para inicializar el API de Facebook
          </li>
          <li>
            <em>Antes</em> de llamar a FB.Init debemos llamar a FB_RequireFeatures para decidir <em>que</em> parte del API de facebook queremos inicializar.
          </li>
        </ol>
        
        <p>
          FB_RequireFeatures es una función <em>asíncrona</em>, pero por suerte le podemos pasar el método a ejecutar cuando la ejecución haya tenido lugar, así que lo más normal es tener algo como:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">function</span> PerformFBLogout() {<br />    FB_RequireFeatures([<span style="color: #006080">"XFBML"</span>], <span style="color: #0000ff">function</span>() {<br />        FB.init(<span style="color: #006080">"TU_CLAVE_DEL_API"</span>, <span style="color: #006080">"/xd_receiver.htm"</span>);<br />        FB.Connect.logout(<span style="color: #0000ff">function</span>() {<br />            window.location=<span style="color: #006080">"&lt;%= Url.Action("</span>LogOff<span style="color: #006080">","</span>Account<span style="color: #006080">") %&gt;"</span>;<br />         });<br />    });<br />}</pre>
          
          <p>
            </div> 
            
            <p>
              Este código llama a FB_RequireFeatures, y le pasa la función a ejecutar cuando FB_RequireFeatures se haya ejecutado asíncronamente. Entonces podemos llamar a FB.Init y posteriormente a FB.Connect.logout. A FB.Connect.logout le pasamos una nueva función con el código a ejecutar cuando el usuario se haya desconectado. En mi caso cuando el usuario se haya desconectado realizo una redirección a la acción “LogOff” del controlador “Account” (sí, yo uso ASP.NET MVC). En la acción LogOff del controlador Account <em>desautentico</em> el usuario.de ASP.NET y muestro la vista de inicio.
            </p>
            
            <p>
              Para que este código funcione, debe estar en una página que tenga el siguiente tag <em>script</em> <strong>en el body (no en el head)</strong>:
            </p>
            
            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">&lt;script src=<span style="color: #006080">"http://static.ak.connect.facebook.com/js/api_lib/v0.4/FeatureLoader.js.php"</span> type=<span style="color: #006080">"text/javascript"</span>&gt;&lt;/script&gt;</pre>
              
              <p>
                </div> 
                
                <p>
                  (Yo este tag <em>script</em> lo tengo colocado en mi página .master)
                </p>
                
                <p>
                  Bueno… pues eso es todo por el momento! Ya os iré contando más batallitas con Connect!!! 😉
                </p>

 [1]: http://geeks.ms/blogs/etomas/archive/2009/12/10/facebook-connect-si-est-225-s-en-facebook-bienvenido-a-mi-web.aspx
 [2]: http://www.codeplex.com/FacebookToolkit