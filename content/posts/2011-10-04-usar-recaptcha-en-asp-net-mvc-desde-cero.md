---
title: Usar Recaptcha en ASP.NET MVC (desde cero)
author: eiximenis

date: 2011-10-04T20:19:54+00:00
geeks_url: /?p=1578
geeks_visits:
  - 3975
geeks_ms_views:
  - 2397
categories:
  - Uncategorized

---
Buenas! En este post vamos a ver como usar <a href="http://www.google.com/recaptcha" target="_blank" rel="noopener noreferrer">Recaptcha</a> en ASP.NET MVC. Pero, antes que nada permitidme una aclaración: Si estás buscando integrar rápidamente Recaptcha en tu proyecto que sepas que puedes usar <a href="http://mvcrecaptcha.codeplex.com/" target="_blank" rel="noopener noreferrer">MvcRecaptcha</a> o también el <a href="http://www.dotnetcurry.com/ShowArticle.aspx?ID=611" target="_blank" rel="noopener noreferrer">helper que viene en MVC3</a>. Pero vamos a ver como hacerlo desde cero. ¿Por que? Pues simplemente porque me parece un buen ejemplo didáctico. Pero insisto: ya hay soluciones hechas, eso es sólo para ver como _podríamos_ hacerlo desde cero

Añadir el captcha en una vista es sumamente sencillo: basta con incluir un tag <script> y dejar que él haga todo. También se puede crear usando javascript (lo que es útil si se quiere crear el captcha sólo si se cumplen ciertas condiciones en tiempo de ejecución), pero no vamos a verlo aquí (todos los detalles están en <http://code.google.com/intl/ca/apis/recaptcha/docs/display.html> en el apartado de “Ajax API”).

Para añadir recaptcha en nuestra página basta simplemente con añadir el siguiente código script:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">&lt;script type=<span style="color: #006080">"text/javascript"</span><br />     src=<span style="color: #006080">"http://www.google.com/recaptcha/api/challenge?k=CLAVE_PUBLICA"</span>&gt;<br />  &lt;/script&gt;</pre>
  
  <p>
    </div> 
    
    <p>
      Este tag <script> renderizará el captcha en la posición donde se incluya.
    </p>
    
    <p>
      Vamos a crearnos un helper que nos genere este tag script. El código es trivial:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">class</span> RecaptchaExtensions<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> IHtmlString Recaptcha(<span style="color: #0000ff">this</span> HtmlHelper @<span style="color: #0000ff">this</span>)<br />    {<br />        <span style="color: #0000ff">return</span> Recaptcha(@<span style="color: #0000ff">this</span>, <span style="color: #006080">"RecaptchaPublicKey"</span>);<br />    }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> IHtmlString Recaptcha(<span style="color: #0000ff">this</span> HtmlHelper @<span style="color: #0000ff">this</span>, <span style="color: #0000ff">string</span> publicKeyId)<br />    {<br />        var publicKey = ConfigurationManager.AppSettings[publicKeyId];<br />        <span style="color: #0000ff">return</span> DoRecaptcha(@<span style="color: #0000ff">this</span>, publicKey);<br />    }<br /><br />    <span style="color: #0000ff">private</span> <span style="color: #0000ff">static</span> IHtmlString DoRecaptcha(<span style="color: #0000ff">this</span> HtmlHelper @<span style="color: #0000ff">this</span>, <span style="color: #0000ff">string</span> publicKey)<br />    {<br />        var tagBuilder = <span style="color: #0000ff">new</span> TagBuilder(<span style="color: #006080">"script"</span>);<br />        tagBuilder.Attributes.Add(<span style="color: #006080">"type"</span>, <span style="color: #006080">"text/javascript"</span>);<br />        tagBuilder.Attributes.Add(<span style="color: #006080">"src"</span>, <span style="color: #0000ff">string</span>.Concat(<span style="color: #006080">"http://www.google.com/recaptcha/api/challenge?k="</span>, publicKey));<br /><br />        <span style="color: #0000ff">return</span> MvcHtmlString.Create(tagBuilder.ToString(TagRenderMode.Normal));<br />    }<br />}<br /></pre>
      
      <p>
        </div> 
        
        <p>
          El método que realmente realiza el trabajo es el método privado DoRecaptcha, que usa un objeto <a href="http://msdn.microsoft.com/en-us/library/system.web.mvc.tagbuilder.aspx" target="_blank" rel="noopener noreferrer">TagBuilder</a> para construir el tag <script>. Fijaos en que el valor de retorno de las funciones del helper es <a href="http://msdn.microsoft.com/en-us/library/system.web.ihtmlstring.aspx" target="_blank" rel="noopener noreferrer">IHtmlString</a>.
        </p>
        
        <p>
          La función Recaptcha del helper recibe un parámetro que es el nombre del <appSetting> donde hay la clave pública de Recaptcha (hay una versión sin parámetreos que usa el <appSetting> cuya clave sea <em>RecaptchaPublicKey</em>.
        </p>
        
        <p>
          Usar el helper es muy sencillo:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">&lt;div&gt;<br />    Necesitamos asegurarnos que eres humano. Actualmente sólo aceptamos<br />    &lt;i&gt;Humanos estándar&lt;/i&gt;:<br />    @Html.Recaptcha()<br />&lt;/div&gt;<br /></pre>
          
          <p>
            </div> 
            
            <p>
              Perfecto! Estamos listos para lo realmente interesante: Comprobar que el resultado que entra el usuario es válido.
            </p>
            
            <p>
              Para ello, si consultamos <a href="http://code.google.com/intl/ca/apis/recaptcha/docs/verify.html" target="_blank" rel="noopener noreferrer">la página donde se describe el proceso de verificación</a> veremos que necesitamos 4 valores:
            </p>
            
            <ol>
              <li>
                La IP del cliente
              </li>
              <li>
                La clave <em>privada</em> de Recaptcha
              </li>
              <li>
                Dos valores adicionales, llamados <em>challenge </em>y <em>response</em> que nos envía recaptcha (son campos añadidos al formulario). Los nombres de los dos campos son <em>recaptcha_challenge_field</em> y <em>recaptcha_response_field</em>.
              </li>
            </ol>
            
            <p>
              Bueno, para validar que el usuario ha dado de alta el captcha, lo podríamos hacer de muchas maneras, pero yo he escogido un <strong>filtro de acción</strong>. Eso me va a permitir decorar la acción del controlador de la siguiente manera:
            </p>
            
            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">[HttpPost]<br />[Recaptcha(Name=<span style="color: #006080">"Captcha"</span>)]<br /><span style="color: #0000ff">public</span> ActionResult Register(RegisterModel model)<br />{<br /><span style="color: #008000">//...</span><br />}<br /></pre>
              
              <p>
                </div> 
                
                <p>
                  Si la validación con Recaptcha es errónea el flitro dejará un error en ModelState con la clave indicada en el parámetro <em>Name </em>(aquí el mensaje es fijo, pero por supuesto podría ser variable)<em>. </em>El filtro lo configuraremos para que se ejecute <em>antes</em> de la acción, por lo que, dentro del método <em>Register</em> podremos usar ModelState.IsValid para preguntar si todo está correcto (incluyendo el captcha).
                </p>
                
                <p>
                  El uso de un filtro de acción es interesante porque elimina toda esa lógica de comprobación de la acción del controlador.
                </p>
                
                <p>
                  Bueno, si revisamos de nuevo la documentación de Recaptcha, vemos que debemos usar los 4 valores mencionados anteriormente y realizar un POST a la dirección <a href="http://www.google.com/recaptcha/api/verify">http://www.google.com/recaptcha/api/verify</a>. La respuesta de este POST nos indicará si la validación ha sido correcta (la primera línea valdrá true) o ha sido incorrecta (valdrá false). ¡Y ya está!
                </p>
                
                <p>
                  Para crear el filtro, derivamos de la clase ActionFilterAttribute y redefinimos el método OnActionExecuting, para que se ejecute justo ANTES de la acción del controlador:
                </p>
                
                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> RecaptchaAttribute : ActionFilterAttribute<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Name { get; set; }<br /><br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">override</span> <span style="color: #0000ff">void</span> OnActionExecuting(ActionExecutingContext filterContext)<br />    {<br />        var request = filterContext.RequestContext.HttpContext.Request;<br />        var challenge = request.Form[<span style="color: #006080">"recaptcha_challenge_field"</span>];<br />        var response = request.Form[<span style="color: #006080">"recaptcha_response_field"</span>];<br />        <span style="color: #0000ff">const</span> <span style="color: #0000ff">string</span> postUrl = <span style="color: #006080">"http://www.google.com/recaptcha/api/verify"</span>;<br />        var result = PerformPost(request.UserHostAddress, challenge, response, postUrl);<br />        <span style="color: #0000ff">if</span> (!result)<br />        {<br />            filterContext.Controller.ViewData.ModelState.AddModelError<br />               (Name ?? <span style="color: #0000ff">string</span>.Empty, <span style="color: #006080">"Recaptcha incorrecto"</span>);<br />        }<br />    }<br /><br /> }<br /></pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Este es el código básico: Recogemos los dos campos recaptcha_challenge_field y recaptcha_response_field, realizamos el POST y si el resultado NO es correcto añadimos un error usando el método AddModelError de ModelState.
                    </p>
                    
                    <p>
                      El método PerformPost sería tal y como sigue:
                    </p>
                    
                    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">private</span> <span style="color: #0000ff">bool</span> PerformPost(<span style="color: #0000ff">string</span> remoteip, <span style="color: #0000ff">string</span> challenge, <span style="color: #0000ff">string</span> response, <span style="color: #0000ff">string</span> postUrl)<br />{<br />    var request = WebRequest.Create(postUrl);<br />    request.Method = <span style="color: #006080">"POST"</span>;<br />    request.ContentType = <span style="color: #006080">"application/x-www-form-urlencoded"</span>;<br />    var stream = request.GetRequestStream();<br />    var privateKey = ConfigurationManager.AppSettings[<span style="color: #006080">"RecaptchaPrivateKey"</span>];<br />    <span style="color: #0000ff">using</span> (var sw = <span style="color: #0000ff">new</span> StreamWriter(stream))<br />    {<br />        <span style="color: #0000ff">const</span> <span style="color: #0000ff">string</span> data = <span style="color: #006080">"privatekey={0}&remoteip={1}&challenge={2}&response={3}"</span>;<br />        sw.Write(data, privateKey, remoteip, challenge, response);<br />    }<br />    var recaptchaResponse = request.GetResponse();<br />    <span style="color: #0000ff">string</span> recaptchaData = <span style="color: #0000ff">null</span>;<br />    var recaptchaStream = recaptchaResponse.GetResponseStream();<br />    <span style="color: #0000ff">if</span> (recaptchaStream != <span style="color: #0000ff">null</span>)<br />    {<br />        <span style="color: #0000ff">using</span> (var sr = <span style="color: #0000ff">new</span> StreamReader(recaptchaStream))<br />        {<br />            recaptchaData = sr.ReadToEnd();<br />        }<br />        <span style="color: #0000ff">return</span> ParseResponse(recaptchaData);<br />    }<br />    <span style="color: #0000ff">else</span> <span style="color: #0000ff">return</span> <span style="color: #0000ff">false</span>;<br />}<br /></pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          Usamos la clase WebRequest para realizar una petición POST con los campos indicados. Fijaos en la definición de la variable data que contiene las variables en el formato típico de post: <em>nombre=valor&nombre=valor&</em>… Luego simplemente volcamos esa variable en el stream de la request del objeto WebRequest.
                        </p>
                        
                        <p>
                          Finalmente recogemos la respuesta, la guardamos toda en una cadena y la parseamos con el método ParseResponse que es tal y como sigue:
                        </p>
                        
                        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">private</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">bool</span> ParseResponse(<span style="color: #0000ff">string</span> recaptchaData)<br />{<br />    var reader = <span style="color: #0000ff">new</span> StringReader(recaptchaData);<br />    var first = reader.ReadLine();<br />    var result = <span style="color: #0000ff">false</span>;<br />    <span style="color: #0000ff">if</span> (first != <span style="color: #0000ff">null</span>)<br />    {<br />        first = first.ToLowerInvariant();<br />        <span style="color: #0000ff">bool</span>.TryParse(first, <span style="color: #0000ff">out</span> result);<br />    }<br /><br />    <span style="color: #0000ff">return</span> result;<br />}<br /></pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              Más simple imposible: leemos la primera línea y miramos si es true o false. Esa primera línea nos indica si ha ido bien o mal la validación del captcha.
                            </p>
                            
                            <p>
                              Y listos! Por supuesto en la vista podemos usar Html.ValidationMessage para añadir el mensaje de error en caso de que la validación del captcha sea incorrecta:
                            </p>
                            
                            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">@Html.ValidationMessage("Captcha")<br /><br /></pre>
                              
                              <p>
                                </div> 
                                
                                <p>
                                  El lugar donde coloquemos este llamada a Htm.ValidationMessage es donde aparecerá el mensaje de error en caso de que la validación del captcha sea incorrecta. Por supuesto el parámetro de ValidationMessage es la misma cadena que el valor del atributo Name del ActionFilter (en mi caso <em>Captcha</em>).
                                </p>
                                
                                <p>
                                  Nos falta ver el código de la acción del controlador, pero no tiene ningún secreto:
                                </p>
                                
                                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">[HttpPost]<br />[Recaptcha(Name=<span style="color: #006080">"Captcha"</span>)]<br /><span style="color: #0000ff">public</span> ActionResult Register(RegisterModel model)<br />{<br />    <span style="color: #0000ff">if</span> (ModelState.IsValid)<br />    {<br />        <span style="color: #008000">// Creamos el usuario y lo autenticamos</span><br />    }<br />    <span style="color: #008000">// Si llegamos aquí hay algun error (puede ser el captcha</span><br />    <span style="color: #008000">// puede ser cualquier otro).</span><br />    <span style="color: #0000ff">return</span> View(model);<br />}<br /></pre>
                                  
                                  <p>
                                    </div> 
                                    
                                    <p>
                                      Una prueba rápida nos permite ver que efectivamente si el usuario falla el captcha aparece el mensaje de error:
                                    </p>
                                    
                                    <p>
                                      <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_63393F3D.png"><img style="background-image: none; border-bottom: 0px; border-left: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_6E162085.png" width="244" height="88" /></a>
                                    </p>
                                    
                                    <p>
                                      Y eso es todo!
                                    </p>
                                    
                                    <p>
                                      En este post hemos visto como usar un ActionFilter para integrar la validación de Recaptcha en nuestro site de forma sencilla y fácil.
                                    </p>
                                    
                                    <p>
                                      Insisto en lo que os he dicho al principio: hay soluciones ya hechas para integrar Recaptcha, pero a veces está bien ver las cosas desde cero, saber como funcionan e intentar ver como afrontarlas, no? Porque si siempre nos lo dan todo masticado… que gracia tiene?
                                    </p>
                                    
                                    <p>
                                      Un saludo! 😀
                                    </p>