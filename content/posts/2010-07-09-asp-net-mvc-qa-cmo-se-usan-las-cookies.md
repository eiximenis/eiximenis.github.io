---
title: 'ASP.NET MVC Q&A: ¿Cómo se usan las cookies?'
description: 'ASP.NET MVC Q&A: ¿Cómo se usan las cookies?'
author: eiximenis

date: 2010-07-09T13:49:56+00:00
geeks_url: /?p=1525
geeks_visits:
  - 5981
geeks_ms_views:
  - 3367
categories:
  - Uncategorized

---
Buenoooo… sigo esta serie que se inició a raíz de las preguntas que recibí en el webcast que realicé sobre ASP.NET MVC para la gente del [DotNetClub de la UOC][1]. Si queréis ver que otros posts de esta serie hay (o habrá) echad un vistazo al [post que sirve de índice][2].

En este caso la pregunta fue “_Cómo y desde donde acceder a las cookies en una aplicación MVC_”. Este post está muy relacionado con el mismo que hablaba sobre el acceso a la sesión. En particular las normas sobre desde donde acceder serían las mismas:

  1. No acceder a las cookies desde las vistas. Las vistas deben ser agnósticas sobre la procedencia de los datos que muestran. 
  2. (Intentar) no acceder desde el modelo: Esto “ata” el modelo a “objetos http”. Intenta mantener el modelo libre de esas dependencias, y si por alguna razón no puedes encapsula todo este acceso en clases helper y preferiblemente inyecta esas clases usando un contenedor IoC. 
  3. Acceder desde los controladores. Los controladores _ya están atados_ a todos los objetos http, así que es uno de los mejores lugares para acceder a las cookies.

**Acceso a las cookies de forma manual**

Para establecer una cookie basta con usar la clase [HttpCookie][3] y la [colección Cookies que está en la Response][4]:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">string</span> valor = <span style="color: #006080">"valor de cookie"</span>;<br />HttpCookie cookie1 = <span style="color: #0000ff">new</span> HttpCookie(<span style="color: #006080">"foo_one"</span>, valor);<br />ControllerContext.HttpContext.Response.SetCookie(cookie1);<br /></pre>
  
  <p>
    </div> 
    
    <p>
      Aquí estamos estableciendo la cookie llamada <em>foo_one</em> con el valor contenido en la variable valor. La colección Cookies que está en la Response contiene las cookies que se envían del servidor al cliente.
    </p>
    
    <p>
      Para leer una cookie que el cliente nos envía, debemos acceder a <a href="http://msdn.microsoft.com/en-us/library/system.web.httprequest.cookies.aspx">la colección cookies, pero ahora debemos usar la que está en la Request</a>:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">var c1 = ControllerContext.HttpContext.Request.Cookies[<span style="color: #006080">"foo_one"</span>];<br /><span style="color: #0000ff">if</span> (c1 != <span style="color: #0000ff">null</span>)<br />{<br />   <span style="color: #0000ff">string</span> valor = c1.Value;<br />}</pre>
      
      <p>
        </div> 
        
        <p>
          En el código recuperamos la cookie <em>foo_one</em> que nos ha enviado el cliente.
        </p>
        
        <p>
          <strong>Acceso a las cookies via Value Provider</strong>
        </p>
        
        <p>
          Bien… que os parece este código?
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> ActionResult GeCookieManual()<br />{<br />    var c1 = ControllerContext.HttpContext.Request.Cookies[<span style="color: #006080">"foo_one"</span>];<br />    var c2 = ControllerContext.HttpContext.Request.Cookies[<span style="color: #006080">"foo_two"</span>];<br />    FooData fd = <span style="color: #0000ff">null</span>;<br /><br />    <span style="color: #0000ff">if</span> (c1 != <span style="color: #0000ff">null</span> && c2 != <span style="color: #0000ff">null</span>)<br />    {<br />        fd = <span style="color: #0000ff">new</span> FooData()<br />        {<br />            OneValue = c1.Value,<br />            AnotherValue = c2.Value<br />        };<br />    }<br /><br />    <span style="color: #0000ff">return</span> View(<span style="color: #006080">"Details"</span>, fd);<br />}</pre>
          
          <p>
            </div> 
            
            <p>
              Fijaos en lo que hace… obtiene dos cookies y con ellas crea un objeto de la clase <em>FooModel</em>.
            </p>
            
            <p>
              ¿Y bien? ¿No os suena mucho al <em>binding</em> todo eso? Si los datos estuviesen en la querystring (GET) o bien en POST ASP.NET MVC seria capaz de crear automáticamente el objeto <em>FooModel</em>… y con las cookies no puede?
            </p>
            
            <p>
              <p>
                Pues no. Al menos no “de serie”… pero por suerte ASP.NET MVC es tan extensible que introducirle esta capacidad es muy, muy fácil…
              </p>
              
              <p>
                La clave está en crearnos un <a href="http://msdn.microsoft.com/en-us/library/system.web.mvc.ivalueprovider.aspx">IValueProvider</a><em></em> propio, que trabaje con las cookies. Ya he comentado en anteriores posts que los value providers son los encargados de “parsear” la request del cliente, recoger los datos y dejarlos <em>todos en un mismo sitio</em> para que el model binder pueda realizar el binding. Por defecto hay value providers para querystring y para POST (además de alguno más rarito que no nos importa ahora), pero no lo hay para cookies. Veamos como podríamos crearlo.
              </p>
              
              <p>
                No voy a entrar mucho en detalle, solo comentaré que es necesario crear la clase que implementa IValueProvider y luego crear la factoría (la clase que creará nuestro value provider). En <a href="http://geeks.ms/blogs/etomas/archive/2010/05/07/asp-net-mvc-valueproviders.aspx">mi post sobre Value Providers</a> tenéis más detalles.
              </p>
              
              <p>
                La clase que implementa IValueProvider es muy simple:
              </p>
              
              <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> CookieValueProvider : IValueProvider<br /> {<br />     <span style="color: #0000ff">private</span> <span style="color: #0000ff">readonly</span> HttpCookieCollection cookies;<br /><br />     <span style="color: #0000ff">public</span> CookieValueProvider(HttpCookieCollection cookies)<br />     {<br />         <span style="color: #0000ff">this</span>.cookies = cookies;<br />     }<br /><br />     <span style="color: #0000ff">public</span> <span style="color: #0000ff">bool</span> ContainsPrefix(<span style="color: #0000ff">string</span> prefix)<br />     {<br />         <span style="color: #0000ff">return</span> <span style="color: #0000ff">this</span>.cookies[prefix] != <span style="color: #0000ff">null</span>;<br />     }<br /><br />     <span style="color: #0000ff">public</span> ValueProviderResult GetValue(<span style="color: #0000ff">string</span> key)<br />     {<br />         var cookie = <span style="color: #0000ff">this</span>.cookies[key];<br />         <span style="color: #0000ff">return</span> cookie != <span style="color: #0000ff">null</span> ?<br />             <span style="color: #0000ff">new</span> ValueProviderResult(cookie.Value, cookie.Value, CultureInfo.CurrentUICulture) : <span style="color: #0000ff">null</span>;<br />     }<br /> }</pre>
                
                <p>
                  </div> 
                  
                  <p>
                    Básicamente <em>ContainsPrefix</em> debe devolver si la clave existe dentro de este value provider y <em>GetValue</em> debe devolver el valor asociado a la clave. En nuestro caso simplemente debemos mirar dentro de la colección de cookies que recibimos del constructor.
                  </p>
                  
                  <p>
                    Ahora toca la factoria, que lo único que debe hacer es crear un <em>CookieValueProvider</em> y pasarle la colección Cookies de la Request:
                  </p>
                  
                  <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">override</span> IValueProvider GetValueProvider(ControllerContext controllerContext)<br />{<br />    <span style="color: #0000ff">return</span> <span style="color: #0000ff">new</span> CookieValueProvider(controllerContext.HttpContext.Request.Cookies);<br />}</pre>
                    
                    <p>
                      </div> 
                      
                      <p>
                        <p>
                          Y finalmente, en el Application_Start (en Global.asax) registramos esa factoría de value providers:
                        </p>
                        
                        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">ValueProviderFactories.Factories.Add(<span style="color: #0000ff">new</span> CookieValueProviderFactory());</pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              Ahora si tenemos la clase FooData deifnida como:
                            </p>
                            
                            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> FooData<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> OneValue { get; set; }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> AnotherValue { get; set; }<br />}</pre>
                              
                              <p>
                                </div> 
                                
                                <p>
                                  <strong>Y si hemos establecido cookies que se llaman igual que las propiedades</strong> (OneValue y AnotherValue), podemos obtener los valores dejando que actúe el binding de ASP.NET MVC:
                                </p>
                                
                                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> ActionResult GetCookieAuto(FooData data)<br />{<br />    <span style="color: #008000">// data está rellenado con el valor de las cookies</span><br />}</pre>
                                  
                                  <p>
                                    </div> 
                                    
                                    <p>
                                      Y listos! Nuestro value provider obtiene el valor de las cookies y el model binder se encarga del resto! 🙂
                                    </p>
                                    
                                    <p>
                                      Os dejo el link de un <a href="http://www.dotnetguy.co.uk/post/2010/03/07/ASPNET-MVC-working-smart-with-cookies-e28093-Custom-ValueProvider.aspx">post de donde vi hace algún tiempo el value provider para cookies en ASP.NET MVC</a>.
                                    </p>
                                    
                                    <p>
                                      Un saludo a todos!!!
                                    </p>
                                  </p>

 [1]: http://uoc.dotnetclubs.com/
 [2]: http://geeks.ms/blogs/etomas/archive/2010/06/29/webcast-asp-net-mvc-uoc-dotnetclub-material.aspx
 [3]: http://msdn.microsoft.com/en-us/library/system.web.httpcookie.aspx
 [4]: http://msdn.microsoft.com/en-us/library/system.web.httpresponse.cookies.aspx