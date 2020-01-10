---
title: 'ASP.NET MVC3 Beta: Mis impresiones'

author: eiximenis

date: 2010-10-07T14:19:00+00:00
geeks_url: /?p=1537
geeks_visits:
  - 1986
geeks_ms_views:
  - 1086
categories:
  - Uncategorized

---
Buenooo… ayer fue un día movidito en Microsoft: [anunciaron de golpe][1] la beta 2 de WebMatrix, la beta de MVC3 y un gestor de paquetes OSS para Visual Studio llamado NuPack. También he visto a través del [Web PI][2] que está la CTP2 de Compact SQL 4.

<!--more-->

MVC1 salió con 5 (creo) previews antes de la beta, con MVC2 juraria que hicieron un par o tres, y con MVC3 sólo un preview1 y luego ya la beta… a ese ritmo MVC4 cuando salga lo hará ya con el SP1 incorporado. 😛 Esos de Microsoft van cada vez más rápido.

Bueno, aquí van un poco mis impresiones sobre la Beta 3 de MVC 🙂

Me centraré básicamente en lo que ha cambiado desde la preview1 hasta la Beta.

**1. Creación de aplicaciones**

Cuando le damos a nuevo proyecto en el Visual Studio, nos sale _una sola_ opción de MVC3:&#160; ASP.NET MVC 3 Web Application y una vez la seleccionamos es cuando nos deja elegir si queremos una aplicación vacía o la estándard (con los controladores Account y Home) y que View Engine queremos usar. Son las mismas opciones que teníamos en MVC2 (excepto lo del view engine) pero mejor organizadas.

Pero vamos… nada nuevo bajo el sol 😀

**2. Donde está IServiceLocator?**

En [mi post sobre la preview1][3] comentaba que el soporte para IoC se basaba en la interfaz [IServiceLocator][4], un proyecto de codeplex para proporcionar una interfaz común a distintos contenedores IoC y así permitir independizarnos de ellos. Decía también que la Preview1 contenía una implementación propia de IServiceLocator en lugar de usar la del assembly que está en codeplex, pero que se esperaba que en la versión final eso no fuera así. 

Pues bien… al final eso ha cambiado un poco: Ahora, a través de la clase _DependencyResolver_ podemos indicar a MVC3 que debe usar para resolver sus dependencias. Y tenemos tres posibilidades:

  1. Pasarle un objeto que implemente una interfaz nueva llamada _IDependencyResolver_ (propia de MVC3). Dicha interfaz está definida:
<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">interface</span> IDependencyResolver<br />{<br />    <span style="color: #0000ff">object</span> GetService(Type serviceType);<br />    IEnumerable&lt;<span style="color: #0000ff">object</span>&gt; GetServices(Type serviceType);<br />}</pre>
  
  <p>
    </div> 
    
    <li>
      Pasarle dos delegates, el primero del cual se corresponde al método GetService y el segundo al método GetServices
    </li>
    <li>
      Pasarle un object. En este caso el object <em>debe</em> implementar la interfaz IServiceLocator. Pero (imagino que) ASP.NET MVC3 invocará los métodos via reflection, por lo que <em>cualquier</em> IServiceLocator vale, no es necesario usar el assembly de codeplex. Supongo que es para evitar tener una dependencia de MVC3 hacia un assembly “externo”.
    </li></ol> 
    
    <p>
      Veamos un ejemplo:
    </p>
    
    <p>
      P.ej. si usásemos Unity podríamos crear un IDependencyResolver propio:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> UnityDependencyResolver : IDependencyResolver<br />{<br />    <span style="color: #0000ff">private</span> IUnityContainer _ctr;<br /><br />    <span style="color: #0000ff">public</span> UnityDependencyResolver(IUnityContainer ctr)<br />    {<br />        _ctr = ctr;<br />    }<br /><br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">object</span> GetService(Type serviceType)<br />    {<br />        <span style="color: #0000ff">return</span> _ctr.Resolve(serviceType);<br />    }<br /><br />    <span style="color: #0000ff">public</span> IEnumerable&lt;<span style="color: #0000ff">object</span>&gt; GetServices(Type serviceType)<br />    {<br />        <span style="color: #0000ff">return</span> _ctr.ResolveAll(serviceType);<br />    }<br />}</pre>
      
      <p>
        </div> 
        
        <p>
          Y registrarlo en el global.asax:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">UnityContainer ctr = <span style="color: #0000ff">new</span> UnityContainer();<br />DependencyResolver.SetResolver(<span style="color: #0000ff">new</span> UnityDependencyResolver(ctr));</pre>
          
          <p>
            </div> 
            
            <p>
              Guay! Pero también podríamos usar la versión que admite un object y pasarle una implementación de IServiceLocator. La versió 2.0 de Unity ya viene con el assembly del service locator incorporado y un adaptador creado (si usais versiones anteriores de Unity os podéis <a href="http://commonservicelocator.codeplex.com/">descargar tanto la implementación de IServiceLocator como el adaptador para Unity desde codeplex</a>).
            </p>
            
            <p>
              En nuestro caso nos basta con agregar una referencia al ensamblado <em>Microsoft.Practices.ServiceLocation.dll</em> (que viene con Unity o os habéis descargado de codeplex) y usar:
            </p>
            
            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">UnityContainer ctr = <span style="color: #0000ff">new</span> UnityContainer();<br />DependencyResolver.SetResolver(<span style="color: #0000ff">new</span> UnityServiceLocator(ctr));</pre>
              
              <p>
                </div> 
                
                <p>
                  (Si usáis una versión de Unity anterior a la 2.0 necesitaréis también una referencia a <em>Microsoft.Practices.Unity.ServiceLocatorAdapter.dll</em> que también os habréis descargado de codeplex).
                </p>
                
                <p>
                  Listos! Con esto MVC3 ya usará nuestro DependencyResolver para crear los controladores, así que ya podremos inyectarles dependencias (fijaos que no ha sido necesario crear una factoría propia como hacíamos en MVC2).
                </p>
                
                <p>
                  <strong>3. IControllerActivator</strong>
                </p>
                
                <p>
                  IControllerActivator es una interfaz nueva de MVC3 cuya responsabilidad es devolver una instancia de un controlador de un determinado tipo. Antes esa responsabilidad era <em>una más</em> de las responsabilidades de la factoría de controladores: cuando en MVC2 queríamos usar DI generalmente implementábamos una factoría propia <em>derivada</em> de <em>DefaultControllerFactory </em>y refefiniamos el método GetControllerInstance:
                </p>
                
                <p>
                  P.ej. esa sería una factoría de controladores para usar IServiceLocator en MVC2:
                </p>
                
                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> ServiceLocatorControllerFactory : DefaultControllerFactory<br /> {<br />     <span style="color: #0000ff">private</span> <span style="color: #0000ff">readonly</span> IServiceLocator _sl;<br />     <span style="color: #0000ff">public</span> ServiceLocatorControllerFactory(IServiceLocator sl) <br />     {<br />         _sl = sl;<br />     }<br />     <span style="color: #0000ff">protected</span> <span style="color: #0000ff">override</span> IController GetControllerInstance(RequestContext requestContext, Type controllerType)<br />     {<br />         IController controller = <span style="color: #0000ff">null</span>;<br />         controller = _sl != <span style="color: #0000ff">null</span> ?  _sl.GetInstance(controllerType) <span style="color: #0000ff">as</span> IController : <br />             <span style="color: #0000ff">base</span>.GetControllerInstance(requestContext, controllerType);<br />         <span style="color: #0000ff">return</span> controller;<br />     }<br /> }</pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      En MVC3 han separado los dos comportamientos: Por un lado el decidir <em>que</em> tipo de controlador instanciar se queda en la factoría de controladores y por otro lado <em>instanciar</em> el controlador es responsabilidad de <em>IControllerActivator</em>.
                    </p>
                    
                    <p>
                      El uso de IControllerActivator nos permite definir <em>como</em> se crean nuestros controladores. Si no hay ningún IControllerActivator MVC3 usará el DependencyResolver (así que <strong>no</strong> es necesario implementar IControllerActivator sólo para DI). Si no hay DependencyResolver pues supongo que usará Reflection.
                    </p>
                    
                    <p>
                      Como MVC3 busca las cosas en varios sitios podemos pasarle el IControllerActivator a la DefaultControllerFactory (tiene un parámetro que acepta un IControllerActivator) <strong>o bien</strong>, registrarlo en el DependencyResolver. O sea podemos usar esto:
                    </p>
                    
                    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #008000">// Registramos un singleton con nuestro IControllerActivator</span><br />ctr.RegisterInstance&lt;IControllerActivator&gt;(<span style="color: #0000ff">new</span> CustomControllerActivator());<br />DependencyResolver.SetResolver(<span style="color: #0000ff">new</span> UnityServiceLocator(ctr));<br /><span style="color: #008000">// No registramos IControllerFactory por lo que se usará la DefaultControllerFactory</span><br /></pre>
                    </div>
                    
                    <p>
                      O bien esto:
                    </p>
                    
                    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #008000">// Registramos la DefaultControllerFactory como la factoría a usar</span><br /><span style="color: #008000">// pasándole el IControllerActivator deseado</span><br />ctr.RegisterInstance&lt;IControllerFactory&gt;<br />    (<span style="color: #0000ff">new</span> DefaultControllerFactory(<span style="color: #0000ff">new</span> CustomControllerActivator()));<br />DependencyResolver.SetResolver(<span style="color: #0000ff">new</span> UnityServiceLocator(ctr));</pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          Pero <strong>ambas</strong> cosas a la vez no (os dará error).
                        </p>
                        
                        <p>
                          <strong>4. Nuevos Helpers</strong>
                        </p>
                        
                        <p>
                          Bueno, ahora tenemos un conjunto de <em>Helpers</em> nuevos (bueno, nuevos para MVC, porque ya estaban en WebMatrix). Para usar esos helpers basta con agregar una referencia a System.Web.Helpers (viene ya por defecto):
                        </p>
                        
                        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">&lt;div&gt;<br />    El Hash de eiximenis es: @Crypto.Hash(<span style="color: #006080">"eiximenis"</span>)<br />&lt;/div&gt;</pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              Diria que <strong>no</strong> están disponibles todos los helpers que hay en WebMatrix… Espero que en la versión final sí estén todos, porque hay algunos que son una pasada!
                            </p>
                            
                            <p>
                              <strong>5. Ajax no “obtrusivo” (unobtrusive)</strong>
                            </p>
                            
                            <p>
                              El helper Ajax de MVC3 puede generar código Ajax que <strong>no</strong> está mezclado con el código HTML, aprovechando jQuery y una de las nuevas capacidades de HTML5… P.ej. el siguiente código en una vista:
                            </p>
                            
                            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">@Ajax.ActionLink(<span style="color: #006080">"Pulsa aquí para modif el DIV"</span>, <span style="color: #006080">"AjaxAction"</span>, <span style="color: #0000ff">new</span> AjaxOptions() { UpdateTargetId=<span style="color: #006080">"mydiv"</span>})<br />&lt;div id=<span style="color: #006080">"mydiv"</span>&gt;Div a modificar&lt;/div&gt;<br /></pre>
                              
                              <p>
                                </div> 
                                
                                <p>
                                  Nos genera el código HTML:
                                </p>
                                
                                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">&lt;</span><span style="color: #800000">a</span> <span style="color: #ff0000">data-ajax</span><span style="color: #0000ff">="true"</span> <span style="color: #ff0000">data-ajax-mode</span><span style="color: #0000ff">="replace"</span> <span style="color: #ff0000">data-ajax-update</span><span style="color: #0000ff">="#mydiv"</span> <span style="color: #ff0000">href</span><span style="color: #0000ff">="/Home/AjaxAction"</span><span style="color: #0000ff">&gt;</span>Pulsa aqu&#237; para modif el DIV<span style="color: #0000ff">&lt;/</span><span style="color: #800000">a</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="mydiv"</span><span style="color: #0000ff">&gt;</span>Div a modificar<span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span></pre>
                                  
                                  <p>
                                    </div> 
                                    
                                    <p>
                                      Fijaos que hay atributos “<em>data-ajax-*“ </em>que son interpretados por jQuery para realizar las acciones: de esa manera no hay ninguna llamada a ningún script mezclado con el código. Los atributos que empiezan por “data-“ son considerados especiales en HTML5 (<a href="http://www.javascriptkit.com/dhtmltutors/customattributes.shtml">custom HTML5 attributes</a>): HTML5 no admite que pongamos nuestros propios atributos a no ser que empiecen por data-. Esto permite separar el comportamiento (definiendolo en estos atributos especiales) del código.
                                    </p>
                                    
                                    <p>
                                      <strong>6. Soporte para Razor</strong>
                                    </p>
                                    
                                    <p>
                                      Pues… sigue bajo mínimos… Es decir, Razor está totalmente soportado en run-time (y ahora con sintaxis VB.NET también!) pero para VS2010 Razor sigue sin existir. Las vistas .cshtml (o .vbhtml) no tienen ni sintaxis coloreada ni tampoco intellisense. Pero es que ni tan siquiera las reconoce como HTML (o sea ni tenemos intellisense de HTML tampoco).
                                    </p>
                                    
                                    <p>
                                      Al margen de esto hay algunas novedades menores para Razor como el soporte de @model para definir el modelo, pero vamos… nada excepcionalmente nuevo.
                                    </p>
                                    
                                    <p>
                                      <strong>7. Conclusiones</strong>
                                    </p>
                                    
                                    <p>
                                      Las dos grandes novedades de esta Beta, respecto a la preview1, son algunas mejoras más en la inyección de dependencias (DependencyResolver, IControllerActivator) y el soporte para ajax no “obtrusivo”. El resto son mejoras menores (aunque interesantes).
                                    </p>
                                    
                                    <p>
                                      Sigo pensando que, del mismo modo que se llama MVC3 se podría llamar MVC2.5, porque no hay cambios espectacularmente grandes (excepto Razor, pero es <em>simplemente</em> otro ViewEngine, como muchos más que ya existían). Eso no es necesariamente malo: el framework ya está alcanzando cierto grado de madurez.
                                    </p>
                                    
                                    <p>
                                      Para finalizar, leeros las <a href="http://www.asp.net/learn/whitepapers/mvc3-release-notes">release notes de la beta de MVC3</a> donde se enumeran las novedades!
                                    </p>
                                    
                                    <p>
                                      Un saludo!
                                    </p>

 [1]: http://weblogs.asp.net/scottgu/archive/2010/10/06/announcing-nupack-asp-net-mvc-3-beta-and-webmatrix-beta-2.aspx
 [2]: http://www.microsoft.com/web/downloads/platform.aspx
 [3]: http://geeks.ms/blogs/etomas/archive/2010/07/28/asp-net-mvc-3-preview-1.aspx
 [4]: http://commonservicelocator.codeplex.com/