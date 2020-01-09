---
title: 'Facebook Connect: Si estás en facebook bienvenido a mi web…'
description: 'Facebook Connect: Si estás en facebook bienvenido a mi web…'
author: eiximenis

date: 2009-12-10T09:52:00+00:00
geeks_url: /?p=1481
geeks_visits:
  - 24585
geeks_ms_views:
  - 2939
categories:
  - Uncategorized

---
Antón Molleda comentaba en un post de su blog ([[WLMT] Socializándonos][1]), las ventajas que ofrece integrar nuestras aplicaciones an alguna de las redes sociales existentes. &Eacute;l comentará sus experiencias con el Windows Live Messenger Toolkit en [su blog][2], así que yo voy a comentaros cuatro cosillas sobre [Facebook Connect][3], el mecanismo de integración que nos ofrece Facebook.

Con Facebook Connect lo que obtenemos es la posibilidad de que los usuarios registrados en Facebook puedan entrar con su login en nuestra web (o aplicación, aunque yo me cocentraré en web), y no sólo eso, sinó que (previa aceptación del usuario) nuestra aplicación pueda publicar contenido en facebook.

En el caso que me ocupa estoy desarrollando una web y me interesa que los usuarios que están en facebook puedan entrar en ella sin necesidad de registrarse en mi web. Mi web seguirá manteniendo su propio sistema de registro de usuarios, puesto que no es obligatorio tener cuenta en facebook para entrar en ella. La página de login tendría un aspecto tal como (queda _prohibido_ cualquier comentario sobre la estética :p):

[<img height="244" width="236" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_36925FBE.png" alt="image" border="0" title="image" style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" />][4] 

**1. Crear una aplicación en Facebook para Connect**

El primer paso para poder usar Facebook Connect es crear la aplicación facebook que nos va a dar acceso a facebook. Para ello, entramos en facebook y seguimos los siguientes pasos:

  * Nos dirigimos a <http://www.facebook.com/developers/createapp.php> para crear una nueva aplicación facebook. 
  * En el apartado _Basic_ entramos el nombre de la aplicación. El resto de valores los podemos dejar por defecto (aunque si soys detallistas siempre podéis añadir un icono a vuestra aplicación :p). Ahora viene lo importante... 
  * En el apartado _Connect_ debéis entrar el _dominio_ donde está vuestra web... Si estais desarrollando seguramente no tendréis vuestra web colgada. Según tengo entendido **localhost no és un dominio válido**, así que os &ldquo;inventais&rdquo; un nombre y entrais en el archivo hosts este nombre con la ip 127.0.0.1. Además si vuestro servidor web no escucha por el puerto 80 (lo normal cuando estamos con VS) debemos entrar el número de puerto que vamos a usar y configurar VS para que arranque cassini siempre con el mismo puerto. P.ej. en el apartado _Connect_ de mi aplicación facebook yo tengo entrado <http://wofserver:12345>: 

> [<img height="59" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_1AA12AC6.png" alt="image" border="0" title="image" style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" />][5]

> Y luego tengo el VS configurado para que me arranque cassini siempre por el puerto 12345:
> 
> [<img height="85" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_3DA17669.png" alt="image" border="0" title="image" style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" />][6] 

> Y recordad la entrada en el fichero hosts para mapear vuestro &ldquo;dominio&rdquo; al 127.0.0.1

  * En el apartado _Basic_ de vuestra aplicación hay dos identificadores que vamos a necesitar: 
      * API Key: Es la clave pública de la aplicación y que debe usarse en la gran mayoría de las llamadas al API de facebook 
      * Secret: La clave secreta. Debe usarse en algunas llamadas al API, y como su nombre indica no debemos compartirla con nadie 😉 
  * Una vez creada la aplicación, estamos listos para interactuar con facebook. 

**2. Preparando nuestra aplicación web para hablar con facebook**

La clave para habilitar la comunicación con facebook está en un archivo html llamado _xd_receiver.htm_. El código de dicho archivo es tal y como sigue (siempre es igual):

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">&lt;!</span><span style="color: #800000">DOCTYPE</span> <span style="color: #ff0000">html</span> <span style="color: #ff0000">PUBLIC</span> <span style="color: #0000ff">"-//W3C//DTD XHTML 1.0 Strict//EN"</span> <span style="color: #0000ff">"http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd"</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">html</span> <span style="color: #ff0000">xmlns</span><span style="color: #0000ff">="http://www.w3.org/1999/xhtml"</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">head</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">title</span><span style="color: #0000ff">&gt;</span>xd<span style="color: #0000ff">&lt;/</span><span style="color: #800000">title</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">head</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">body</span><span style="color: #0000ff">&gt;</span><br /><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">script</span> <span style="color: #ff0000">src</span><span style="color: #0000ff">="http://static.ak.facebook.com/js/api_lib/v0.4/XdCommReceiver.js"</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text/javascript"</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">script</span><span style="color: #0000ff">&gt;</span><br /><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">body</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">html</span><span style="color: #0000ff">&gt;</span><br /></pre>
  
  <p>
    </div> 
    
    <p>
      Podemos poner el archivo donde queramos, pero yo os recomiendo que lo pongáis en el directorio raíz de vuestra web. Es decir en mi caso <a href="http://wofserver:12345/xd_receiver.htm">http://wofserver:12345/xd_receiver.htm</a>
    </p>
    
    <p>
      &nbsp;
    </p>
    
    <p>
      &nbsp;
    </p>
    
    <p>
      &nbsp;
    </p>
    
    <p>
      &nbsp;
    </p>
    
    <p>
      &nbsp;
    </p>
    
    <p>
      <strong>3. Preparando el entorno para renderizar XFBML</strong>
    </p>
    
    <p>
      Para ayudarnos a integrarnos con facebook, éste incorpora una API de tags conocida como XFBML (XHtml FaceBook Markup Language). Usando estos tags podemos p.ej. renderizar el botón de login, o el nombre del usuario conectado.
    </p>
    
    <p>
      Para ello debemos indicar que nuestras páginas son XHTML e importar el espacio de nombres fb que es el que usan los tags XFBML. Para ello modificamos las etiquetas <html> y <!DOCTYPE>:
    </p>
    
    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">&lt;!</span><span style="color: #800000">DOCTYPE</span> <span style="color: #ff0000">html</span> <span style="color: #ff0000">PUBLIC</span> <span style="color: #0000ff">"-//W3C//DTD XHTML 1.0 Strict//EN"</span> <span style="color: #0000ff">"http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd"</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">html</span> <span style="color: #ff0000">xmlns</span><span style="color: #0000ff">="http://www.w3.org/1999/xhtml"</span><br /> <span style="color: #ff0000">xmlns:fb</span><span style="color: #0000ff">="http://www.facebook.com/2008/fbml"</span><span style="color: #0000ff">&gt;</span><br /></pre>
      
      <p>
        </div> 
        
        <p>
          Ya casi estamos... sólo nos queda referenciar el uso del fichero javascript que habilita el API de facebook (así como XFBML), para ello debemos añadir el siguiente tag <script> <strong>justo después de abrir el tag <body></strong>:
        </p>
        
        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">&lt;</span><span style="color: #800000">script</span> <span style="color: #ff0000">src</span><span style="color: #0000ff">="http://static.ak.connect.facebook.com/js/api_lib/v0.4/FeatureLoader.js.php"</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text/javascript"</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">script</span><span style="color: #0000ff">&gt;</span></pre>
          
          <p>
            </div> 
            
            <p>
              Evidentemente os recomiendo utilizar una <em>master page</em> para tener todos estos cambios en todas vuestras páginas (estos tags deben estar en cualquier página que quiera usar el API de facebook).
            </p>
            
            <p>
              <strong>4. Usando XFBML</strong>
            </p>
            
            <p>
              Antes de poder usar XFBML hemos de <em>incializar</em> nuestra conexión con facebook y habilitar XFBML. Yo tengo el código puesto en la pantalla de login, pero esto ya dependerá de cada uno. Para ello debemos usar el siguiente código javascript:
            </p>
            
            <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
              <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">&lt;script type=<span style="color: #006080">"text/javascript"</span>&gt;<br />    FB_RequireFeatures([<span style="color: #006080">"XFBML"</span>], <span style="color: #0000ff">function</span>() {<br />    FB.Facebook.init(<span style="color: #006080">"CLAVE_API"</span>, <span style="color: #006080">"/xd_receiver.htm"</span>);<br />    });<br />&lt;/script&gt;</pre>
              
              <p>
                </div> 
                
                <p>
                  Ahora sí: el uso de XFBML es super simple. P.ej. para renderizar el botón de login basta con usar el tag <fb:login-button>:
                </p>
                
                <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">&lt;</span><span style="color: #800000">fb:login-button</span> <span style="color: #ff0000">v</span><span style="color: #0000ff">="2"</span> <span style="color: #ff0000">size</span><span style="color: #0000ff">="medium"</span> <br /><span style="color: #ff0000">onlogin</span><span style="color: #0000ff">="window.location.reload(true);"</span><span style="color: #0000ff">&gt;</span><br />Login WoF with facebook<br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">fb:login-button</span><span style="color: #0000ff">&gt;</span></pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      En este caso se renderiza el botón de login, y cuando el usuario se haya identificado se forzará un refresco de la página. El texto que colocamos dentro del tag <fb:login-button> será el texto del botón:
                    </p>
                    
                    <p>
                      <a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_4EA5A44C.png"><img height="26" width="178" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_3BF0AA95.png" alt="image" border="0" title="image" style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" /></a>
                    </p>
                    
                    <p>
                      Hay muchos tags XFBML, en esta dirección (<a href="http://wiki.developers.facebook.com/index.php/XFBML" title="http://wiki.developers.facebook.com/index.php/XFBML">http://wiki.developers.facebook.com/index.php/XFBML</a>) tenéis la lista completa.
                    </p>
                    
                    <p>
                      &nbsp;
                    </p>
                    
                    <p>
                      Cuando el usuario haga click en el botón de iniciar sesión en facebook pueden ocurrir dos cosas:
                    </p>
                    
                    <ol>
                      <li>
                        Que el usuario ya estuviese identificado en facebook. En este caso se disparará el evento onlogin del botón.
                      </li>
                      <li>
                        Que el usuario no estuviese identificado en facebook. En este caso le saldrá la ventana para que se identifique. Una vez identificado se disparará el evento onlogin:
                      </li>
                    </ol>
                    
                    <blockquote>
                      <p>
                        <a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_7B4E5E25.png"><img height="221" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_0BE65914.png" alt="image" border="0" title="image" style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" /></a>&nbsp;
                      </p>
                    </blockquote>
                    
                    <p>
                      <strong>5. Accediendo a información del usuario</strong>
                    </p>
                    
                    <p>
                      Llegados a este punto ya nos hemos integrado con el login del usuario... Pero esto no sirve de mucho si no somos capaces de extraer información del usuario (p.ej su nombre, o bien si ya había un usuario identificado en facebook). Para ello podemos usar el API de facebook (obviamente basada en javascript), pero yo voy a contaros como se puede hacerlo usando el &ldquo;<a href="http://www.codeplex.com/FacebookToolkit">Facebook Developer Toolkit</a>&rdquo; que os podéis descargar de Codeplex.
                    </p>
                    
                    <p>
                      Una vez descargado basta con referenciar el assembly &ldquo;Facebook.dll&rdquo;, y luego podemos utilizar la clase <em>ConnectSession</em> para averiguar si un usuario está conectado en facebook:
                    </p>
                    
                    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">var fbc= <span style="color: #0000ff">new</span> ConnectSession(<span style="color: #006080">"&lt;CLAVE_API&gt;"</span>,<span style="color: #006080">"&lt;CLAVE_SECRETA&gt;"</span>);<br /><span style="color: #0000ff">if</span> (fbc.IsConnected()) <br />{<br />    <span style="color: #008000">// Usuaro conectado en facebook. En fbc.UserId tenemos el ID unico del</span><br />    <span style="color: #008000">// usuario en facebook.</span><br />}<br /><span style="color: #0000ff">else</span><br />{<br />    <span style="color: #008000">// No hay usuario conectado en facebook</span><br />}<br /></pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          A partir de aquí, lo que hagamos ya dependerá de nuestras necesidades. P.ej. en mi caso, pregunto al usuario se desea usar mi web con su login de facebook y si responde afirmativamente creo un usuario <em>especial</em> en mi tabla de usuarios, vinculado a dicho id de facebook y autentico este usuario en mi sitio web...
                        </p>
                        
                        <p>
                          Espero que este post os haya servido para daros una idea de como funciona Facebook Connect. En siguientes posts espero ir contando alguna cosilla más al respecto! 😉
                        </p>
                        
                        <p>
                          Saludos!!!
                        </p>

 [1]: /blogs/amolleda/archive/2009/12/07/wlmt-socializ-225-ndonos.aspx
 [2]: /blogs/amolleda
 [3]: http://developers.facebook.com/connect.php
 [4]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_78E57801.png
 [5]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_69EE231A.png
 [6]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_0A45B30D.png