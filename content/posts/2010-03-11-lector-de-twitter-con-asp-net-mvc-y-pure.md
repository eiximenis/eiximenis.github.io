---
title: Lector de Twitter con ASP.NET MVC y Pure
description: Lector de Twitter con ASP.NET MVC y Pure
author: eiximenis

date: 2010-03-11T18:36:00+00:00
geeks_url: /?p=1501
geeks_visits:
  - 2416
geeks_ms_views:
  - 1137
categories:
  - Uncategorized

---
Hola a todos! El otro día os <a target="_blank" href="/blogs/etomas/archive/2010/03/09/trasteando-con-pure.aspx" rel="noopener noreferrer">comentaba lo mucho que me está gustando pure</a>... Hoy, a modo de ejemplo, os presento como realizar fácilmente un lector de Twitter que muestre los últimos tweets...

**El proyecto asp.net mvc**

No voy a mencionar mucha cosa del proyecto asp.net mvc en general puesto que al final hay un enlace con todo el código. Consta de dos controladores:

  1. Home: Que tiene las acciones Index, Login y Tweets. La acción Index redirige a Login o bien a Tweets en función de si el usuario ha entrado o no login y password de twitter. La acción Login muestra un formulario de login y redirige a la acción tweets, que es la que hace el trabajo.
  2. La acción Tweets se limita a mostrar una vista que es la que hace gran parte del trabajo.

**1. La vista Home/Tweets**

Esta vista, hace una llamada ajax a la acción List del controlador Tweeter (nuestro segundo controlador). Esta acción debe devolver la información json con los últimos tweets del usuario. El código, es gracias a jquery, trivial:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">$(document).ready(<span style="color: #0000ff">function</span>() {<br />    $.getJSON (<br />        <span style="color: #006080">"&lt;%= Url.Action("</span>List<span style="color: #006080">","</span>Twitter<span style="color: #006080">") %&gt;"</span>,<br />        <span style="color: #0000ff">function</span>(val) <br />        { <br />            renderPure(val); <br />        });<br />    });<br /></pre>
  
  <p>
    </div> 
    
    <p>
      El método <em>renderPure</em> será el que usara Pure para mostrar los datos.
    </p>
    
    <p>
      <strong>2. El controlador Twitter</strong>
    </p>
    
    <p>
      <a target="_blank" href="http://apiwiki.twitter.com/" rel="noopener noreferrer">Twitter dispone de una fantástica API REST</a> con la cual podemos hacer casi cualquier cosa. Esta API está preparada para devolver datos usando distintos formatos, entre ellos json.
    </p>
    
    <p>
      Algunas de las funciones de dicha API son públicas, pero otras (como la&nbsp; de ver los últimos tweets) requieren que nos autentiquemos en twitter. Hay dos formas de hacerlo: usar <a target="_blank" href="http://oauth.net/" rel="noopener noreferrer">OAuth</a> o bien <a target="_blank" href="http://en.wikipedia.org/wiki/Basic_access_authentication" rel="noopener noreferrer">basic authentication</a>.
    </p>
    
    <p>
      OAuth es un protocolo de autenticación, pensado para integrar distintas aplicaciones, y una de sus ventajas es que el usuario no debe dar su contraseña en ningún momento, además de que puede granularizar sus permisos (puede dar permisos de lectura pero no de escritura p.ej.). Es sin duda el método recomendado para integrarnos con Twitter. Si usais cualquier cliente Twitter (web o escritorio) que <strong>no</strong> os haya pedido el password y que además hayais tenido que darle permisos desde la web de twitter, entonces este cliente está usando OAuth.
    </p>
    
    <p>
      Basic authentication por otro lado es muy simple: se basa en poner una cabecera en la petición http, con nombre <em>Authorization </em>y cuyo valor sea <em>Basic login:password</em>, donde la cadena &ldquo;login:password&rdquo; está en <a target="_blank" href="http://en.wikipedia.org/wiki/Base64" rel="noopener noreferrer">Base64</a>. Es un método <strong>totalmente inseguro</strong> (salvo que se uso junto con SSL) y además el usuario debe proporcionar su usuario y su contraseña al cliente (lo que bueno... mucha confianza no genera). Pero es muy sencillo de implementar, y es el que yo he usado.
    </p>
    
    <p>
      El controlador Twitter tiene una sola acción List que realiza una llamada a la API REST de Twitter y pasa el resultado (json) <em>tal cual</em>:
    </p>
    
    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> ActionResult List()<br />{<br />    UserModel um = <span style="color: #0000ff">this</span>.Session[<span style="color: #006080">"user"</span>] <span style="color: #0000ff">as</span> UserModel;<br />    <span style="color: #0000ff">string</span> url = <span style="color: #006080">"http://api.twitter.com/1/statuses/friends_timeline.json"</span>;<br /><br />    <span style="color: #008000">// Realiza la petición a twitter</span><br />    HttpWebRequest req = (HttpWebRequest)WebRequest.Create(url);<br />    req.Headers.Add(<span style="color: #006080">"Authorization"</span>, <span style="color: #0000ff">string</span>.Format(<span style="color: #006080">"Basic {0}"</span>, um.Base64Data));<br />    req.Method = <span style="color: #006080">"GET"</span>;<br />    HttpWebResponse res = (HttpWebResponse)req.GetResponse();<br /><br />    <span style="color: #0000ff">string</span> data = <span style="color: #0000ff">null</span>;<br />    <span style="color: #0000ff">if</span> (res.StatusCode == HttpStatusCode.OK)<br />    {<br />        <span style="color: #0000ff">using</span> (StreamReader sr = <span style="color: #0000ff">new</span> StreamReader(res.GetResponseStream()))<br />        {<br />            data = sr.ReadToEnd();<br />        }<br />    }<br /><br />    <span style="color: #0000ff">return</span> !<span style="color: #0000ff">string</span>.IsNullOrEmpty(data) ? (ActionResult)<span style="color: #0000ff">this</span>.RawJson(data) : (ActionResult)View(<span style="color: #006080">"Error"</span>);<br />}<br /></pre>
      
      <p>
        </div> 
        
        <p>
          Fijaos que el código es muy simple: Recojemos un objeto <em>UserModel</em> de la sesión. Ese objeto contiene el login y password de twitter del usuario. Con esos datos construimos una petición a la URL de la API REST de Twitter y añadimos la cabecera <em>Authorization</em>. Finalmente leemos el valor devuelto (una cadena json, salvo error) y lo mandamos tal cual.
        </p>
        
        <p>
          El método RawJson es un método mío de extensión que devuelve un objeto RawJsonActionResult:
        </p>
        
        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> RawJsonActionResult RawJson(<span style="color: #0000ff">this</span> Controller @<span style="color: #0000ff">this</span>, <span style="color: #0000ff">string</span> data)<br />{<br />    <span style="color: #0000ff">return</span> <span style="color: #0000ff">new</span> RawJsonActionResult() { RawData = data };<br />}<br /></pre>
          
          <p>
            </div> 
            
            <p>
              La clase <em>RawJsonActionResult</em> es un ActionResult mío, que me permite devolver una cadena json. No uso el JsonResult de MVC porque ese está pensado para <em>construir</em> una cadena json a partir de un objeto .NET, y yo ya tengo la cadena json. Como podeis ver, el código es trivial:
            </p>
            
            <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
              <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> RawJsonActionResult : ActionResult<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> RawData { get; set; }<br /><br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">override</span> <span style="color: #0000ff">void</span> ExecuteResult(ControllerContext context)<br />    {<br />        <span style="color: #0000ff">var</span> response = context.HttpContext.Response;<br />        response.ContentType = <span style="color: #006080">"application/json"</span>;<br />        response.Write(RawData);<br />        response.Flush();<br />    }<br />}<br /></pre>
              
              <p>
                </div> 
                
                <p>
                  Quizá alguien se pregunta porque hago una llamada a un controlador mío (Twitter) que lo <strong>único</strong> que hace es consultar a la API REST de Twitter y mandarme el mismo resultado (sin tratar). La respuesta es para evitar problemas de cross-domain: de esta manera desde la vista Home/Tweets la única llamada ajax que hay es una llamada a otro controlador del mismo dominio, lo que seguro que no me va a generar ningún problema.
                </p>
                
                <p>
                  <strong>3. La vista Home/Tweets (ii)</strong>
                </p>
                
                <p>
                  Una vez la vista Home/Tweets recibe el objeto json (resultado de la llamada a la acción List del controlador Twitter), debemos generar el html necesario para &ldquo;dibujarlo&rdquo;... Aquí entra en acción PURE.
                </p>
                
                <p>
                  El objeto json que nos devuelve twitter es bastante grandote (aunque gracias a firebug se puede ver muy fácilmente). Por cada tweet yo voy a mostrar:
                </p>
                
                <ol>
                  <li>
                    El avatar de quien ha hecho el tweet
                  </li>
                  <li>
                    El texto del tweet
                  </li>
                  <li>
                    El nombre de usuario y el nombre real
                  </li>
                </ol>
                
                <p>
                  Para ello usaré la siguiente plantilla:
                </p>
                
                <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="purecontent"</span> <span style="color: #ff0000">style</span><span style="color: #0000ff">="display:none"</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">class</span><span style="color: #0000ff">="tweet"</span><span style="color: #0000ff">&gt;</span><br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">img</span> <span style="color: #ff0000">class</span><span style="color: #0000ff">="picprofile"</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">img</span><span style="color: #0000ff">&gt;</span><br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">p</span><span style="color: #0000ff">&gt;</span><br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">span</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="text"</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">span</span><span style="color: #0000ff">&gt;</span><br />        <span style="color: #0000ff">&lt;/</span><span style="color: #800000">p</span><span style="color: #0000ff">&gt;</span><br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">class</span><span style="color: #0000ff">="userdata"</span><span style="color: #0000ff">&gt;&lt;</span><span style="color: #800000">span</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="user"</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">span</span><span style="color: #0000ff">&gt;</span> (<span style="color: #0000ff">&lt;</span><span style="color: #800000">span</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="realname"</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">span</span><span style="color: #0000ff">&gt;</span>)<span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br /></pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Dentro del div &ldquo;purecontent&rdquo; se generaran tantos div &ldquo;tweet&rdquo; como tweets haya... Veamos el código del método renderPure:
                    </p>
                    
                    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">function</span> renderPure(data)<br />{<br />    <span style="color: #0000ff">var</span> directive = {<br />        <span style="color: #006080">'div.tweet'</span> : {<br />            <span style="color: #006080">'tweet&lt;-'</span>: {<br />                <span style="color: #006080">'span#text'</span> : <span style="color: #006080">'tweet.text'</span>,<br />                <span style="color: #006080">'span#text@id+'</span> : <span style="color: #006080">'tweet.id'</span>,<br />                <span style="color: #006080">'span#user'</span> : <span style="color: #006080">'tweet.user.screen_name'</span>,<br />                <span style="color: #006080">'span#user@id+'</span> : <span style="color: #006080">'tweet.id'</span>,<br />                <span style="color: #006080">'span#realname'</span> : <span style="color: #006080">'tweet.user.name'</span>,<br />                <span style="color: #006080">'span#realname@id+'</span> : <span style="color: #006080">'tweet.id'</span>,<br />                <span style="color: #006080">'img.picprofile@src'</span> : <span style="color: #006080">'tweet.user.profile_image_url'</span><br />            }<br />        }<br />    };<br />    $(<span style="color: #006080">"#purecontent"</span>).render(data, directive);<br />    $(<span style="color: #006080">"#purecontent"</span>).toggle();<br />}<br /></pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          Lo más importante (como no) es la directiva, que le indica a pure como renderizar los datos a partir de la plantilla y el objeto json. En concreto esa plantilla dice lo siguiente:
                        </p>
                        
                        <ol>
                          <li>
                            Por cada elemento del elemento json (tweet<-) haz lo siguiente: <ol>
                              <li>
                                En el span cuyo id es text (span#text) pon el valor de la propiedad text del elemento.
                              </li>
                              <li>
                                En este mismo span modifica su id para que el nuevo valor sea el id previo (text) más el valor de la propiedad Id del elemento.
                              </li>
                              <li>
                                Haz lo mismo con los spans user y realname, pero usando las propiedades screen_name y name del objeto user que está dentro del elemento.
                              </li>
                              <li>
                                Finalmente en el atributo src de la img cuya clase es picprofile (img.picprofile@src) coloca el valor de la propiedad profile_image_url del objeto user que está dentro del elemento.
                              </li>
                            </ol>
                          </li>
                        </ol>
                        
                        <p>
                          &nbsp;
                        </p>
                        
                        <p>
                          Que... fácil, eh? Al principio igual cuesta acostumbrarse pero una vez se le pilla el truquillo es muy potente y natural.
                        </p>
                        
                        <p>
                          Finalmente llamamos al método render de PURE y por último hacemos visible el div padre (<em>purecontent</em>) mediante el método toggle de jQuery...
                        </p>
                        
                        <p>
                          Y este es el resultado (por cierto <a target="_blank" href="/members/jmaguilar/default.aspx" rel="noopener noreferrer">José M. Aguilar</a>... ponte ya un avatar :p)
                        </p>
                        
                        <p>
                          <a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_6F4C778E.png"><img height="168" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_1CF9F4BA.png" alt="image" border="0" title="image" style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" /></a>
                        </p>
                        
                        <p>
                          <a target="_blank" href="http://cid-6521c259e9b1bec6.skydrive.live.com/self.aspx/BurbujasNet/ZipsPosts/TwitterDemo.zip" rel="noopener noreferrer">Os dejo un zip con el proyecto entero</a> para que le echeis un vistazo.
                        </p>
                        
                        <p>
                          Saludos!!!
                        </p>