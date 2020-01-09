---
title: 'ASP.NET MVC Q&A: Cómo usar la sesión?'
author: eiximenis

date: 2010-06-30T18:54:52+00:00
geeks_url: /?p=1522
geeks_visits:
  - 23065
geeks_ms_views:
  - 3833
categories:
  - Uncategorized

---
Hola a todos! Este es el primer post de la serie que “nace” a raíz de las preguntas que se me realizaron durante el Webcast de ASP.NET MVC que realizé el pasado 28 de Junio.

Una de las preguntas fue precisamente _si se podia usar la sesión_. La respuesta corta que di en el Webcast fue “_sí: la sesión funciona exactamente igual que en Webforms y en mi opinión el sitio donde usarla es en los controladores_”. Ahora viene la respuesta larga… 🙂

Acceder a la sesión desde un controlador es trivial:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> HomeController : Controller<br />{<br />    <span style="color: #0000ff">public</span> ActionResult Index()<br />    {<br />        Session[<span style="color: #006080">"data"</span>] = <span style="color: #006080">"Datos en sesión"</span>;<br />        <span style="color: #0000ff">return</span> View();<br />    }<br />}</pre>
  
  <p>
    </div> 
    
    <p>
      Como se puede ver el controlador tiene acceso directo a la sesión a través de la propiedad Session declarada en la clase <em><a href="http://msdn.microsoft.com/en-us/library/system.web.mvc.controller.aspx" target="_blank" rel="noopener noreferrer">Controller</a></em>.
    </p>
    
    <p>
      Desde las vistas también tenemos acceso a la sesión:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">&lt;div&gt;<br />    Datos en sesión: &lt;%: <span style="color: #0000ff">this</span>.Session[<span style="color: #006080">"data"</span>] %&gt;<br />&lt;/div&gt;</pre>
      
      <p>
        </div> 
        
        <p>
          <strong>1. NO accedas a la sesión desde las vistas</strong>
        </p>
        
        <p>
          ASP.NET MVC expone un modelo extremadamente sencillo y potente: toda petición es enrutada a un controlador y el controlador hace lo que tenga que hacer para terminar pasándole un conjunto de datos a la vista (usando lo que se llama un <em>ViewModel</em> que no es nada más que una clase propia que encapsula los datos que la vista requiere). La vista accede a su <em>ViewModel</em> a través de la propiedad <em>Model</em>.
        </p>
        
        <p>
          La vista debe ser agnóstica sobre donde proceden los datos: le basta saber que los tiene disponibles en su ViewModel. Deja que sea el controlador si es preciso quien acceda a la sesión, cree un ViewModel con los datos y se los pasa a la vista.
        </p>
        
        <p>
          Otro sitio donde uno podría usar Session es en el Modelo, es decir dejar la lógica de si un dato debe almacenarse en sesión o no en el propio Modelo. De esta aproximación lo que mi no me convence es que termina <em>atando</em> nuestro modelo al objeto <a href="http://msdn.microsoft.com/en-us/library/system.web.httpsessionstatebase.aspx" target="_blank" rel="noopener noreferrer">HttpSessionStateBase</a>. Yo personalmente prefiero que todo acceso a sesión esté en los controladores, dados que es más natural que estos estén atados a “objetos típicos de http”.
        </p>
        
        <p>
          <strong>2. ¿Para qué quieres la sesión?</strong>
        </p>
        
        <p>
          Asegúrate que usas la sesión para lo que pertoca. En la sesión se guardan datos <em>relativos a un usuario</em> para un periodo de tiempo <em>relativamente largo</em>, generalmente hasta que el usuario <em>se desconecta</em> de nuestra aplicación. <strong>No</strong> hagas nunca esto:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> HomeController : Controller<br />{<br />    <span style="color: #0000ff">public</span> ActionResult Index()<br />    {<br />        Session[<span style="color: #006080">"data"</span>] = <span style="color: #006080">"Datos en sesión"</span>;<br />        <span style="color: #0000ff">return</span> RedirectToAction(<span style="color: #006080">"OtraAccion"</span>, <span style="color: #006080">"OtroControlador"</span>);<br />    }<br />}<br /><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> OtroControladorController : Controller<br />{<br />    <span style="color: #0000ff">public</span> ActionResult OtraAccion()<br />    {<br />        var datos = Session[<span style="color: #006080">"data"</span>] <span style="color: #0000ff">as</span> <span style="color: #0000ff">string</span>;<br />        <span style="color: #008000">// hacemos algo con los datos</span><br />        Session.Remove(<span style="color: #006080">"data"</span>);<br />        <span style="color: #0000ff">return</span> View();<br />    }<br />}</pre>
          
          <p>
            </div> 
            
            <p>
              En este código el controlador <em>Home</em> pone datos en la sesión y luego redirige a la acción <em>OtraAccion</em> del controlador <em>OtroControlador</em> que recoge los datos de la sesión, hace algo con ellos (p.ej. crear un ViewModel), los elimina de la sesión y llama a su vista.
            </p>
            
            <p>
              Aquí se está usando la sesión para compartir datos temporales entre controladores. Dado que este es un escenario relativamente habitual ASP.NET MVC proporciona un mecanismo propio para ello: el TempData.
            </p>
            
            <p>
              El código anterior quedaría mejor con:
            </p>
            
            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> HomeController : Controller<br />{<br />    <span style="color: #0000ff">public</span> ActionResult Index()<br />    {<br />        TempData[<span style="color: #006080">"data"</span>] = <span style="color: #006080">"Datos temporales"</span>;<br />        <span style="color: #0000ff">return</span> RedirectToAction(<span style="color: #006080">"OtraAccion"</span>, <span style="color: #006080">"OtroControlador"</span>);<br />    }<br />}<br /><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> OtroControladorController : Controller<br />{<br />    <span style="color: #0000ff">public</span> ActionResult OtraAccion()<br />    {<br />        var datos = TempData[<span style="color: #006080">"data"</span>] <span style="color: #0000ff">as</span> <span style="color: #0000ff">string</span>;<br />        <span style="color: #008000">// hacemos algo con los datos</span><br />        <span style="color: #0000ff">return</span> View();<br />    }<br />}</pre>
              
              <p>
                </div> 
                
                <p>
                  TempData funciona de un método <em>algo raro</em>: Cuando una propiedad de TempData es leída <em>se marca para eliminación</em> y dicha eliminación sucede al fin de la request. Eso significa que a efectos prácticos los datos de TempData son datos <strong>para leer una sola vez.</strong> Una buena solución pues para pasar datos temporales entre controladores. Y sí: un dato almacenado en TempData que nunca es consultado <strong>permanece</strong> entre requests. Esta capacidad permite dar soporte a escenarios con varias redirecciones, pero debemos andarnos con cuidado: si ponemos un valor en TempData que jamás es consultado, ahí se queda ocupando espacio… ¿Y espacio de donde? Pues si esto de que el dato permanece entre requests te suena mucho a “sesión”, no vas desencaminado: internamente TempData se guarda dentro de Session. Para más información sobre TempData léete <a href="http://www.variablenotfound.com/2010/02/tempdata-en-aspnet-mvc-2.html" target="_blank" rel="noopener noreferrer">este brutal post de José M. Aguilar donde lo cuenta mejor que yo</a>.
                </p>
                
                <p>
                  <strong>3. Considera no usar sesión</strong>
                </p>
                
                <p>
                  Otro tema importante a tener en cuenta es evitar el uso de la sesión. La sesión lleva consiguo ciertas implicaciones. Tenemos tres modos de sesión en ASP.NET (y por lo tanto en ASP.NET MVC):
                </p>
                
                <ul>
                  <li>
                    InProc: La sesión se guarda en el propio servidor web. No se comparte entre servidores web de una misma web farm, por lo que si tenemos una web farm debemos usar <a href="http://dev.fyicenter.com/Interview-Questions/JavaScript/What_does_the_term_sticky_session_mean_in_a_web_.html" target="_blank" rel="noopener noreferrer">sticky sessions</a> lo que limita la esacabilidad.
                  </li>
                  <li>
                    StateServer: La sesión se guarda en un único servidor. Se comparte entre diversos los servidores de la web farm, pero añade un único punto de fallo: si se cae el servidor de sesión, se cae toda nuestra aplicación.
                  </li>
                  <li>
                    SQLServer: La sesión se guarda en base de datos. Se comparte entre los servidores de la web farm, a costa de perder rendimiento y de perder parte de escalabilidad (siempre es más difícil escalar un SQL Server que añadir un servidor web adicional).
                  </li>
                </ul>
                
                <p>
                  Así pues, ten presente el precio que pagas por usar la sesión. Con eso no digo que la sesión esté prohibida (ni mucho menos), simplemente que consideres <em>si la necesitas de verdad</em>.
                </p>
                
                <p>
                  <strong>4. El punto (3) te echa para atrás pero necesitas usar sesión?</strong>
                </p>
                
                <p>
                  Quieres usar la sesión, pero no estás satisfecho con ninguna de las tres alternativas que te proporciona por defecto ASP.NET? Ningún problema, puedes crearte tu propio proveedor de sesión que use algún mecanismo que satisfaga tus necesidades… Ya, no es tarea fácil verdad? Por suerte (o por desgracia) Microsoft no para de sacar productos, APIs y cosas nuevas y hay una muy interesante: <strong><a href="http://msdn.microsoft.com/es-es/windowsserver/ee695849(en-us).aspx" target="_blank" rel="noopener noreferrer">Velocity</a></strong>.
                </p>
                
                <p>
                  Velocity (incluída dentro de lo que llama Windows Server AppFabric) es una cache distribuída. Lo bueno es que viene con un proveedor de sesión para ASP.NET. Por lo tanto, puedes utilizar Velocity para almacenar tu sesión! Esto implica que la sesión, al estar en la cache distribuída, se comparte entre los servidores web del webfarm sin ser un único punto de fallo (como StateServer) ni penalizar tanto el rendimiento (como SQLServer). Así pues, es una muy buena opción para usar como proveedor de sesión (no solo para ASP.NET MVC, también para ASP.NET tradicional).
                </p>
                
                <p>
                  En <a href="http://stephenwalther.com/blog/archive/2008/08/28/asp-net-mvc-tip-39-use-the-velocity-distributed-cache.aspx" target="_blank" rel="noopener noreferrer">este post Stephen Walther tienes más información sobre como usar Velocity en ASP.NET MVC</a>.
                </p>
                
                <p>
                  <strong>5. Y sobre unit testing?</strong>
                </p>
                
                <p>
                  Una de las ventajas de MVC respecto Webforms es la capacidad de probar nuestras aplicaciones usando unit testing. Generalmente lo que más nos interesa pobar son los controladores y el modelo, dado que las vistas no contienen ninguna lógica (tan solo presentación).
                </p>
                
                <p>
                  Si nuestros controladores usan la sesión, necesitan un HttpContext para funcionar, pero durante tus pruebas untarias no vas a tenerlo. Imagina que tengo este controlador:
                </p>
                
                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> HomeController : Controller<br />{<br />    <span style="color: #0000ff">public</span> ActionResult Index()<br />    {<br />        Session[<span style="color: #006080">"data"</span>] = <span style="color: #006080">"eiximenis"</span>;<br />        var data = Session[<span style="color: #006080">"data"</span>] <span style="color: #0000ff">as</span> <span style="color: #0000ff">string</span>;<br />        <span style="color: #0000ff">return</span> View();<br />    }<br />}</pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Y un test unitario para probar algo sobre dicha acción:
                    </p>
                    
                    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">[TestMethod]<br /><span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> TestMethod1()<br />{<br />    HomeController hc = <span style="color: #0000ff">new</span> HomeController(); <br />    var result = hc.Index() <span style="color: #0000ff">as</span> ViewResult; <br />    Assert.AreEqual(<span style="color: #0000ff">string</span>.Empty, result.ViewName);<br />}</pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          Al ejecutar este UnitTest… patapaaaam! Nos va a lanzar una NullReferenceException, porque como no estamos ejecutando los tests bajo el motor de ASP.NET no existe Session ni nada parecido (por lo que el método Index de HomeController se encuentra un <em>null</em> cuando accede a la propiedad <em>Session</em>). ¿Y entonces? Pues debemos, de alguna manera u otra conseguir crear <em>Mocks</em> de dichos objetos.
                        </p>
                        
                        <p>
                          Hay varias maneras de hacer esto, pero una de las más sencillas es usar el <a href="http://mvccontrib.codeplex.com/wikipage?title=TestHelper" target="_blank" rel="noopener noreferrer">TestHelper</a> de <a href="http://mvccontrib.codeplex.com/" target="_blank" rel="noopener noreferrer">MvcContrib</a>. Veamos como podemos testear este método usando MvcContrib. Para ello nos <a href="http://mvccontrib.codeplex.com/wikipage?title=Download&referringTitle=Home" target="_blank" rel="noopener noreferrer">descargamos MvcContrib</a> (para la versión de ASP.NET MVC que estés usando) y:
                        </p>
                        
                        <ol>
                          <li>
                            Agregamos una referencia a MvcContrib.TestHelper.dll
                          </li>
                          <li>
                            Agregamos otra referencia a Rhino.Mocks.dll. <strong>Nota:</strong> Técnicamente esta referencia <strong>no</strong> es necesaria que la agreguemos en nuestro proyecto. El requisito a cumplir es que Rhino.Mocks.dll esté disponible en tiempo de ejecución del test (MvcContrib.TestHelper.dll depende de ella). P. ej. otra forma de conseguir esto es agregar Rhino.Mocls.dll a la solución y usar [<a href="http://msdn.microsoft.com/en-us/library/microsoft.visualstudio.testtools.webtesting.deploymentitemattribute(VS.80).aspx" target="_blank" rel="noopener noreferrer">DeploymentItem</a>].
                          </li>
                        </ol>
                        
                        <p>
                          Ahora el código del test queda así:
                        </p>
                        
                        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">[TestMethod]<br /><span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> TestMethod1()<br />{<br />    TestControllerBuilder tb = <span style="color: #0000ff">new</span> TestControllerBuilder();<br />    HomeController hc = tb.CreateController&lt;HomeController&gt;();<br />    var result = hc.Index() <span style="color: #0000ff">as</span> ViewResult;<br />    Assert.AreEqual(<span style="color: #0000ff">string</span>.Empty, result.ViewName);<br />}</pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              Usamos la clase TestControllerBuilder que es capaz de crear controladores con todos los “objetos http” inicializados (con Mocks).
                            </p>
                            
                            <p>
                              Los Mocks de MvcContrib no solo hacen que el código del controlador no de un error, también simulan la funcionalidad de la clase real. Por lo tanto podemos probar que el método Index guarda algo en la sesión:
                            </p>
                            
                            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">[TestMethod]<br /><span style="color: #0000ff">public</span> <span style="color: #0000ff">void</span> TestMethod1()<br />{<br />    TestControllerBuilder tb = <span style="color: #0000ff">new</span> TestControllerBuilder();<br />    HomeController hc = tb.CreateController&lt;HomeController&gt;();<br />    hc.Index();<br />    Assert.AreEqual(<span style="color: #006080">"pepe"</span>, hc.HttpContext.Session[<span style="color: #006080">"data"</span>]);<br />}</pre>
                              
                              <p>
                                </div> 
                                
                                <p>
                                  ¿Que os parece? MvcContrib es una manera elegante y fácil de poder probar nuestros controladores que tienen dependencias contra “objetos http” como la sesión.
                                </p>
                                
                                <p>
                                  Un saludo a todos!!! 😉
                                </p>