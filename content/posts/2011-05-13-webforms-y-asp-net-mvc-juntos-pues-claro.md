---
title: ¿Webforms y ASP.NET MVC juntos? Pues claro!

author: eiximenis

date: 2011-05-13T18:15:00+00:00
geeks_url: /?p=1568
geeks_visits:
  - 11664
geeks_ms_views:
  - 2978
categories:
  - Uncategorized

---
En el grupo de <a target="_blank" href="http://www.linkedin.com/groups/AUGES-3889150" rel="noopener noreferrer">linkedin de AUGES</a>, <a target="_blank" href="http://www.linkedin.com/groups/Primer-debate-iniciar-AUGES-Es-3889150.S.52141354" rel="noopener noreferrer">en uno de los debates que tenemos abierto</a>, Javier Giners pregunta estrategias de migración de Webforms hacia ASP.NET MVC. Yo le responde que depende de como esté arquitecturada la aplicación pero que tenga presente que ASP.NET MVC y webforms pueden convivir juntos en **una misma aplicación web**. No se trata de que una aplicación web hecha en webforms se comunique fácilmente con otra hecha en ASP.NET MVC no. Se trata de que ambas tecnologías pueden combinarse para crear una sola aplicación web.

Y ese es el objetivo de este post 😉

<!--more-->

**1. En el inicio sólo existía webforms**

Para este post he empezado por desarrollar una aplicación webforms. Es muy chorra, lo único que tiene es una página con una grid que a través de un LinqDataSource se conecta a una base de datos y muestra los datos de una tabla (llamada en un alarde de originalidad &ldquo;Datos&rdquo;).

No voy a comentar nada del proyecto Webforms, al final del post hay el enlace para que os lo descarguéis, pero ya veréis que es muy simplón. Esta es una captura de pantalla:

[<img height="113" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_342EE23F.png" alt="image" border="0" title="image" style="background-image: none; border-right-width: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" />][1]

Fijaos que la URL termina en .aspx (es un Webform) y también que hay usuario registrado. La configuración de seguridad en web.config está:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">&lt;</span><span style="color: #800000">location</span> <span style="color: #ff0000">path</span><span style="color: #0000ff">="VerDatos.aspx"</span><span style="color: #0000ff">&gt;</span><br />   <span style="color: #0000ff">&lt;</span><span style="color: #800000">system.web</span><span style="color: #0000ff">&gt;</span><br />     <span style="color: #0000ff">&lt;</span><span style="color: #800000">authorization</span><span style="color: #0000ff">&gt;</span><br />       <span style="color: #0000ff">&lt;</span><span style="color: #800000">deny</span> <span style="color: #ff0000">users</span> <span style="color: #0000ff">="?"</span> <span style="color: #0000ff">/&gt;</span><br />     <span style="color: #0000ff">&lt;/</span><span style="color: #800000">authorization</span><span style="color: #0000ff">&gt;</span><br />   <span style="color: #0000ff">&lt;/</span><span style="color: #800000">system.web</span><span style="color: #0000ff">&gt;</span><br /> <span style="color: #0000ff">&lt;/</span><span style="color: #800000">location</span><span style="color: #0000ff">&gt;</span><br /></pre>
  
  <p>
    </div> 
    
    <p>
      Vamos, que todo permitido excepto VerDatos.aspx que sólo lo pueden ver usuarios registrados.
    </p>
    
    <p>
      Y listos, ya tenemos una aplicación webforms!
    </p>
    
    <p>
      <strong>2. Preparando la entrada de MVC</strong>
    </p>
    
    <p>
      Empieza el show. Ahora queremos implementar el método de dar de alta un usuario, y para ello queremos que la funcionalidad de introducir un dato nuevo, esté hecho en ASP.NET MVC.
    </p>
    
    <p>
      Lo primero es modificar el web.config para &ldquo;añadir soporte&rdquo; a ASP.NET MVC. Si estáis en VS2010 no necesitás hacer mucha cosa, copiar apenas tres pedacitos.
    </p>
    
    <p>
      El primer pedacito es para añadir las referencias a los assemblies propios de ASP.NET MVC. Se debe poner dentro del tag <compilation>:
    </p>
    
    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #008000">&lt;!-- 1: Añadimos assemblies que se usan en ASP.NET MVC --&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">assemblies</span><span style="color: #0000ff">&gt;</span><br />  <span style="color: #0000ff">&lt;</span><span style="color: #800000">add</span> <span style="color: #ff0000">assembly</span><span style="color: #0000ff">="System.Web.Abstractions, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35"</span> <span style="color: #0000ff">/&gt;</span><br />  <span style="color: #0000ff">&lt;</span><span style="color: #800000">add</span> <span style="color: #ff0000">assembly</span><span style="color: #0000ff">="System.Web.Helpers, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35"</span> <span style="color: #0000ff">/&gt;</span><br />  <span style="color: #0000ff">&lt;</span><span style="color: #800000">add</span> <span style="color: #ff0000">assembly</span><span style="color: #0000ff">="System.Web.Routing, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35"</span> <span style="color: #0000ff">/&gt;</span><br />  <span style="color: #0000ff">&lt;</span><span style="color: #800000">add</span> <span style="color: #ff0000">assembly</span><span style="color: #0000ff">="System.Web.Mvc, Version=3.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35"</span> <span style="color: #0000ff">/&gt;</span><br />  <span style="color: #0000ff">&lt;</span><span style="color: #800000">add</span> <span style="color: #ff0000">assembly</span><span style="color: #0000ff">="System.Web.WebPages, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35"</span> <span style="color: #0000ff">/&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">assemblies</span><span style="color: #0000ff">&gt;</span><br /></pre>
      
      <p>
        </div> 
        
        <p>
          El segundo pedazo de código es para registrar los namespaces propios de ASP.NET MVC. Para ello dentro de la etiqueta <system.web> que cuelga directamente de <configuration> ponemos:
        </p>
        
        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #008000">&lt;!-- 2: Registramos namespaces de ASP.NET MVC --&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">pages</span><span style="color: #0000ff">&gt;</span><br />  <span style="color: #0000ff">&lt;</span><span style="color: #800000">namespaces</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">add</span> <span style="color: #ff0000">namespace</span><span style="color: #0000ff">="System.Web.Helpers"</span> <span style="color: #0000ff">/&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">add</span> <span style="color: #ff0000">namespace</span><span style="color: #0000ff">="System.Web.Mvc"</span> <span style="color: #0000ff">/&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">add</span> <span style="color: #ff0000">namespace</span><span style="color: #0000ff">="System.Web.Mvc.Ajax"</span> <span style="color: #0000ff">/&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">add</span> <span style="color: #ff0000">namespace</span><span style="color: #0000ff">="System.Web.Mvc.Html"</span> <span style="color: #0000ff">/&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">add</span> <span style="color: #ff0000">namespace</span><span style="color: #0000ff">="System.Web.Routing"</span> <span style="color: #0000ff">/&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">add</span> <span style="color: #ff0000">namespace</span><span style="color: #0000ff">="System.Web.WebPages"</span><span style="color: #0000ff">/&gt;</span><br />  <span style="color: #0000ff">&lt;/</span><span style="color: #800000">namespaces</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">pages</span><span style="color: #0000ff">&gt;</span><br /></pre>
          
          <p>
            </div> 
            
            <p>
              Y finalmente sólo nos queda añadir las referencias a ASP.NET MVC en el proyecto. Para ello le dais a Add Reference &ndash;> Browse y donde tengáis ASP.NET MVC3 instalado y añadís la referencia a System.Web.Mvc.dll (Si quereis usar Razor también podemos añadir la referencia a System.Web.WebPages.dll).
            </p>
            
            <p>
              <strong>3. Inicializando ASP.NET MVC</strong>
            </p>
            
            <p>
              Para inicializar ASP.NET MVC es muy sencillo, basta con añadir algunas líneas en Global.asax.cs:
            </p>
            
            <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
              <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">private</span> <span style="color: #0000ff">void</span> MvcInit()<br />{<br />    AreaRegistration.RegisterAllAreas();<br />    RegisterGlobalFilters(GlobalFilters.Filters);<br />    RegisterRoutes(RouteTable.Routes);<br />}<br /><br /><span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">void</span> RegisterGlobalFilters(GlobalFilterCollection filters)<br />{<br />    filters.Add(<span style="color: #0000ff">new</span> HandleErrorAttribute());<br />}<br /><br /><span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">void</span> RegisterRoutes(RouteCollection routes)<br />{<br />    routes.IgnoreRoute(<span style="color: #006080">"{resource}.axd/{*pathInfo}"</span>);<br />    routes.MapRoute(<span style="color: #006080">"Default"</span>, <span style="color: #006080">"{controller}/{action}/{id}"</span>,<br />        <span style="color: #0000ff">new</span> {action = <span style="color: #006080">"index"</span>, id = UrlParameter.Optional});<br />}<br /></pre>
              
              <p>
                </div> 
                
                <p>
                  Y añadir la llamada a ese método <em>MvcInit()</em> dentro del Application_Start.
                </p>
                
                <p>
                  Con eso ya tenemos ASP.NET MVC inicializado dentro de nuestro proyecto webforms Y hemos dado de alta las rutas <em>estándard</em> de ASP.NET MVC. Si ahora ejecutamos de nuevo vemos que todo sigue funcionando.
                </p>
                
                <blockquote>
                  <p>
                    <strong>Nota: </strong>¿Te sorprende que siga funcionando todo? Ahora ya tenemos ASP.NET MVC y las URLs de tipo /controlador/acción ya están habilitadas gracias a la tabla de rutas. Si conoces algo de ASP.NET MVC igual te preguntas: si ya tenemos URLs <em>bonitas... cómo es que siguen funcionando las URLs que terminan con .aspx</em>?
                  </p>
                  
                  <p>
                    La respuesta es muy sencilla: Las rutas, en principio, <strong>no se aplican si existe un fichero físico que coincida con la URL</strong>. La razón no es facilitar la interoperabilidad con webforms (aunque ayuda), la razón es mucho más simple: Si no fuese así, deberíamos crear controladores para devolver imágenes, css, ficheros javascript y multitud de elementos estáticos más.
                  </p>
                  
                  <p>
                    Así pues recuerda: Si existe un fichero en la ruta física que indica la URL, las rutas no se tienen en cuenta (a menos que se indique lo contrario). Es por eso que ahora una URL /VerDatos.aspx nos funciona, porque tenemos un fichero físico llamado VerDatos.aspx en la raíz de la aplicación web.
                  </p>
                </blockquote>
                
                <p>
                  <strong>4. Creando un controlador y una vista</strong>
                </p>
                
                <p>
                  Bueno... ahora ya podemos crear el controlador. Dado que partimos de un proyecto de Webforms <strong>no tenemos</strong> el soporte de tooling de Visual Studio disponible (no hay el menú &ldquo;Add Controller&rdquo; p.ej.). Por suerte añadirlo es muy simple.
                </p>
                
                <p>
                  Para ello abrimos el fichero .csproj con un editor de texto y busca el tag <ProjectGuids>. En mi caso tenía ese valor:
                </p>
                
                <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">&lt;</span><span style="color: #800000">ProjectTypeGuids</span><span style="color: #0000ff">&gt;</span>{349c5851-65df-11da-9384-00065b846f21};{fae04ec0-301f-11d3-bf4b-00c04f79efbc}<span style="color: #0000ff">&lt;/</span><span style="color: #800000">ProjectTypeGuids</span><span style="color: #0000ff">&gt;</span></pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Estos GUIDs son los que indican el <em>tipo</em> (<em>o tipos</em>) del proyecto y por lo tanto indican que herramientas aplica Visual Studio. Como no sabía el GUID de un proyecto de ASP.NET MVC3, cree uno de vacío y miré esa misma línea. En el caso de ASP.NET MVC3 el valor de <em>ProjectTypeGuids</em> es:
                    </p>
                    
                    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">&lt;</span><span style="color: #800000">ProjectTypeGuids</span><span style="color: #0000ff">&gt;</span>{E53F8FEA-EAE0-44A6-8774-FFD645390401};{349c5851-65df-11da-9384-00065b846f21};{fae04ec0-301f-11d3-bf4b-00c04f79efbc}<span style="color: #0000ff">&lt;/</span><span style="color: #800000">ProjectTypeGuids</span><span style="color: #0000ff">&gt;</span></pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          Veo que de los tres GUIDs que hay en la segunda línea, dos son los mismos que antes y que hay uno de nuevo (<em>{E53F8FEA-EAE0-44A6-8774-FFD645390401}</em>), así que simplemente copio el GUID nuevo en la etiqueta ProjectTypeGuids del .csproj de Webforms.
                        </p>
                        
                        <p>
                          &iexcl;Y voilá! Ya tenemos el tooling de VS2010 en nuestro proyecto webforms!
                        </p>
                        
                        <blockquote>
                          <p>
                            <strong>Nota importante: </strong>El <strong>orden</strong> de los GUIDs dentro de ProjectTypeGuids parece que importa! El GUID &ldquo;<em>nuevo</em>&rdquo; (el que hemos copiado del proyecto MVC3 al de Webforms) tiene que estar en la primera posición. Si no está en la primera posición no podréis abrir el proyecto en VS2010!
                          </p>
                        </blockquote>
                        
                        <p>
                          Ahora añadimos manualmente las carpetas <em>Controllers</em> y <em>Views</em>. Una vez creada ya podemos añadir el controlador. En este caso vamos a añadir un controlador para añadir datos nuevos:
                        </p>
                        
                        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> DatosController : Controller<br />{<br />    <span style="color: #0000ff">public</span> ActionResult Add()<br />    {<br />        <span style="color: #0000ff">return</span> View();<br />    }<br /><br />    [HttpPost]<br />    <span style="color: #0000ff">public</span> ActionResult Add(DatosViewModel datos)<br />    {<br />        <span style="color: #0000ff">if</span> (ModelState.IsValid)<br />        {<br />            <span style="color: #0000ff">using</span> (var database = <span style="color: #0000ff">new</span> DatosDataContext())<br />            {<br />                var nuevoDato = <span style="color: #0000ff">new</span> Dato();<br />                nuevoDato.id = datos.Id;<br />                nuevoDato.nombre = datos.Nombre;<br />                nuevoDato.twitter = datos.Twitter;<br />                database.Datos.InsertOnSubmit(nuevoDato);<br />                database.SubmitChanges();<br />            }<br />            <span style="color: #0000ff">return</span> Redirect(<span style="color: #006080">"/"</span>);<br />        }<br />        <span style="color: #0000ff">else</span><br />        {<br />            <span style="color: #0000ff">return</span> View();<br />        }<br />    }<br />}</pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              DatosViewModel es una clase que me he creado yo, que me servirá para hacer binding de los datos que entre el usuario:
                            </p>
                            
                            <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                              <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> DatosViewModel<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">int</span> Id { get; set; }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Nombre { get; set; }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Twitter { get; set; }<br />}</pre>
                              
                              <p>
                                </div> 
                                
                                <p>
                                  Ahora toca crear la vista de alta. Antes que nada nos toca crear la vista Layout (la equivalente de la master en Razor).
                                </p>
                                
                                <blockquote>
                                  <p>
                                    <strong>Nota: </strong>Podríamos usar el View Engine de aspx para reutilizar la master. Pero vamos a ver como hacerlo en Razor, porque (en mi opinión) Razor es una mejora sustancial respecto el View Engine de aspx.
                                  </p>
                                </blockquote>
                                
                                <p>
                                  Podemos crear un página de Layout y establecerla en cada vista (lo mismo que hacemos en el caso de webforms o el view engine de aspx) o bien podemos <strong>no</strong> establecer el Layout en cada vista y hacerlo a través del _ViewStart.cshtml (que se ejecuta <em>antes</em> de ejecutar cada vista).
                                </p>
                                
                                <p>
                                  Creamos un archivo llamado _ViewStart.cshtml que esté en /Views y que tenga el código:
                                </p>
                                
                                <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                                  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">@{<br />    Layout = <span style="color: #006080">"~/Views/Shared/Layout.cshtml"</span>;<br />}</pre>
                                  
                                  <p>
                                    </div> 
                                    
                                    <p>
                                      Con eso establecemos /Views/Shared/Layout.cshtml como la página de layout de todas nuestras vistas.
                                    </p>
                                    
                                    <p>
                                      Ahora podemos crear la página (Add New Item &ndash;> MVC3 Layout Page (Razor)). El código que VS2010 nos genera por defecto ya es suficiente:
                                    </p>
                                    
                                    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                                      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">&lt;!</span><span style="color: #800000">DOCTYPE</span> <span style="color: #ff0000">html</span><span style="color: #0000ff">&gt;</span><br /><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">html</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">head</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">title</span><span style="color: #0000ff">&gt;</span>@ViewBag.Title<span style="color: #0000ff">&lt;/</span><span style="color: #800000">title</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">head</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">body</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br />        @RenderBody()<br />    <span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">body</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">html</span><span style="color: #0000ff">&gt;</span></pre>
                                      
                                      <p>
                                        </div> 
                                        
                                        <p>
                                          Recordad de guardar esa vista en /Views/Shared con el nombre de Layout.cshtml.
                                        </p>
                                        
                                        <p>
                                          Ahora ya podemos añadir nuestra vista. Añadimos una vista llamada &ldquo;Add&rdquo; que esté en /Views/Datos. En mi caso he creado la vista con esos parámetros:
                                        </p>
                                        
                                        <p>
                                          <img height="484" width="491" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_2AF2A6FE.png" alt="image" border="0" title="image" style="background-image: none; border-bottom: 0px; border-left: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" />
                                        </p>
                                        
                                        <p>
                                          Y el código que por defecto genera VS2010 ya es suficiente 😉
                                        </p>
                                        
                                        <p>
                                          Ahora ya podemos probarlo, a ver que tal... Para ello, el Webform VerDatos.aspx tiene al final un <asp:Hyperlink> al que le añado la propiedad NavigateUrl con valor &ldquo;/Datos/Add&rdquo;. Ahora ya podemos probarlo!
                                        </p>
                                        
                                        <p>
                                          <strong>5. Zas! En toda la boca! :p</strong>
                                        </p>
                                        
                                        <p>
                                          Si al probar el proyecto os aparece un error como este:
                                        </p>
                                        
                                        <h4>
                                          <i><span style="background-color: #ffff00;">Compilation Error</span></i>
                                        </h4>
                                        
                                        <p>
                                          <span style="background-color: #ffff00;"><b>
                                        </p>
                                        
                                        <p>
                                          <code></code>
                                        </p>
                                        
                                        <pre><span style="background-color: #ffff00;">Line 1:  @model WebApp.DatosDataContext
Line 2:
Line 3:  @{</span></pre>
                                        
                                        <p>
                                          Tranquilos... es normal.&nbsp; Nos falta configurar ASP.NET para que use también el motor de Razor. Eso se hace... en el web.config. Pero no en el web.config general, sino en un web.config que esté dentro de /Views.
                                        </p>
                                        
                                        <p>
                                          La forma más rápida de tenerlo es cojerlo de un proyecto nuevo de MVC3 que os creeis, pero si estáis perezoso, este os puede servir (recordad, va en View/web.config):
                                        </p>
                                        
                                        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                                          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">&lt;?</span><span style="color: #800000">xml</span> <span style="color: #ff0000">version</span><span style="color: #0000ff">="1.0"</span>?<span style="color: #0000ff">&gt;</span><br /><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">configuration</span><span style="color: #0000ff">&gt;</span><br />  <span style="color: #0000ff">&lt;</span><span style="color: #800000">configSections</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">sectionGroup</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="system.web.webPages.razor"</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="System.Web.WebPages.Razor.Configuration.RazorWebSectionGroup, System.Web.WebPages.Razor, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35"</span><span style="color: #0000ff">&gt;</span><br />      <span style="color: #0000ff">&lt;</span><span style="color: #800000">section</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="host"</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="System.Web.WebPages.Razor.Configuration.HostSection, System.Web.WebPages.Razor, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35"</span> <span style="color: #ff0000">requirePermission</span><span style="color: #0000ff">="false"</span> <span style="color: #0000ff">/&gt;</span><br />      <span style="color: #0000ff">&lt;</span><span style="color: #800000">section</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="pages"</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="System.Web.WebPages.Razor.Configuration.RazorPagesSection, System.Web.WebPages.Razor, Version=1.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35"</span> <span style="color: #ff0000">requirePermission</span><span style="color: #0000ff">="false"</span> <span style="color: #0000ff">/&gt;</span><br />    <span style="color: #0000ff">&lt;/</span><span style="color: #800000">sectionGroup</span><span style="color: #0000ff">&gt;</span><br />  <span style="color: #0000ff">&lt;/</span><span style="color: #800000">configSections</span><span style="color: #0000ff">&gt;</span><br /><br />  <span style="color: #0000ff">&lt;</span><span style="color: #800000">system.web.webPages.razor</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">host</span> <span style="color: #ff0000">factoryType</span><span style="color: #0000ff">="System.Web.Mvc.MvcWebRazorHostFactory, System.Web.Mvc, Version=3.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35"</span> <span style="color: #0000ff">/&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">pages</span> <span style="color: #ff0000">pageBaseType</span><span style="color: #0000ff">="System.Web.Mvc.WebViewPage"</span><span style="color: #0000ff">&gt;</span><br />      <span style="color: #0000ff">&lt;</span><span style="color: #800000">namespaces</span><span style="color: #0000ff">&gt;</span><br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">add</span> <span style="color: #ff0000">namespace</span><span style="color: #0000ff">="System.Web.Mvc"</span> <span style="color: #0000ff">/&gt;</span><br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">add</span> <span style="color: #ff0000">namespace</span><span style="color: #0000ff">="System.Web.Mvc.Ajax"</span> <span style="color: #0000ff">/&gt;</span><br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">add</span> <span style="color: #ff0000">namespace</span><span style="color: #0000ff">="System.Web.Mvc.Html"</span> <span style="color: #0000ff">/&gt;</span><br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">add</span> <span style="color: #ff0000">namespace</span><span style="color: #0000ff">="System.Web.Routing"</span> <span style="color: #0000ff">/&gt;</span><br />      <span style="color: #0000ff">&lt;/</span><span style="color: #800000">namespaces</span><span style="color: #0000ff">&gt;</span><br /><br />    <span style="color: #0000ff">&lt;/</span><span style="color: #800000">pages</span><span style="color: #0000ff">&gt;</span><br />  <span style="color: #0000ff">&lt;/</span><span style="color: #800000">system.web.webPages.razor</span><span style="color: #0000ff">&gt;</span><br /><br />  <span style="color: #0000ff">&lt;</span><span style="color: #800000">appSettings</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">add</span> <span style="color: #ff0000">key</span><span style="color: #0000ff">="webpages:Enabled"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="false"</span> <span style="color: #0000ff">/&gt;</span><br />  <span style="color: #0000ff">&lt;/</span><span style="color: #800000">appSettings</span><span style="color: #0000ff">&gt;</span><br /><br />  <span style="color: #0000ff">&lt;</span><span style="color: #800000">system.web</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">httpHandlers</span><span style="color: #0000ff">&gt;</span><br />      <span style="color: #0000ff">&lt;</span><span style="color: #800000">add</span> <span style="color: #ff0000">path</span><span style="color: #0000ff">="*"</span> <span style="color: #ff0000">verb</span><span style="color: #0000ff">="*"</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="System.Web.HttpNotFoundHandler"</span><span style="color: #0000ff">/&gt;</span><br />    <span style="color: #0000ff">&lt;/</span><span style="color: #800000">httpHandlers</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">pages</span><br />        <span style="color: #ff0000">validateRequest</span><span style="color: #0000ff">="false"</span><br />        <span style="color: #ff0000">pageParserFilterType</span><span style="color: #0000ff">="System.Web.Mvc.ViewTypeParserFilter, System.Web.Mvc, Version=3.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35"</span><br />        <span style="color: #ff0000">pageBaseType</span><span style="color: #0000ff">="System.Web.Mvc.ViewPage, System.Web.Mvc, Version=3.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35"</span><br />        <span style="color: #ff0000">userControlBaseType</span><span style="color: #0000ff">="System.Web.Mvc.ViewUserControl, System.Web.Mvc, Version=3.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35"</span><span style="color: #0000ff">&gt;</span><br />      <span style="color: #0000ff">&lt;</span><span style="color: #800000">controls</span><span style="color: #0000ff">&gt;</span><br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">add</span> <span style="color: #ff0000">assembly</span><span style="color: #0000ff">="System.Web.Mvc, Version=3.0.0.0, Culture=neutral, PublicKeyToken=31BF3856AD364E35"</span> <span style="color: #ff0000">namespace</span><span style="color: #0000ff">="System.Web.Mvc"</span> <span style="color: #ff0000">tagPrefix</span><span style="color: #0000ff">="mvc"</span> <span style="color: #0000ff">/&gt;</span><br />      <span style="color: #0000ff">&lt;/</span><span style="color: #800000">controls</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;/</span><span style="color: #800000">pages</span><span style="color: #0000ff">&gt;</span><br />  <span style="color: #0000ff">&lt;/</span><span style="color: #800000">system.web</span><span style="color: #0000ff">&gt;</span><b
r />  <span style="color: #0000ff">&lt;</span><span style="color: #800000">system.webServer</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">validation</span> <span style="color: #ff0000">validateIntegratedModeConfiguration</span><span style="color: #0000ff">="false"</span> <span style="color: #0000ff">/&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">handlers</span><span style="color: #0000ff">&gt;</span><br />      <span style="color: #0000ff">&lt;</span><span style="color: #800000">remove</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="BlockViewHandler"</span><span style="color: #0000ff">/&gt;</span><br />      <span style="color: #0000ff">&lt;</span><span style="color: #800000">add</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="BlockViewHandler"</span> <span style="color: #ff0000">path</span><span style="color: #0000ff">="*"</span> <span style="color: #ff0000">verb</span><span style="color: #0000ff">="*"</span> <span style="color: #ff0000">preCondition</span><span style="color: #0000ff">="integratedMode"</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="System.Web.HttpNotFoundHandler"</span> <span style="color: #0000ff">/&gt;</span><br />    <span style="color: #0000ff">&lt;/</span><span style="color: #800000">handlers</span><span style="color: #0000ff">&gt;</span><br />  <span style="color: #0000ff">&lt;/</span><span style="color: #800000">system.webServer</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">configuration</span><span style="color: #0000ff">&gt;</span><br /></pre>
                                          
                                          <p>
                                            </div> 
                                            
                                            <p>
                                              Si ahora lo probáis el error es otro: un error de compilación de que deberíamos añadir una referencia a System.Data.Linq. Recordad que cuando ASP.NET compila una vista, usa las referencias que esten indicadas en el web.config. Así pues añadimos la siguiente línea dentro del tag <assemblies> del web.config principal:
                                            </p>
                                            
                                            <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                                              <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">&lt;</span><span style="color: #800000">add</span> <span style="color: #ff0000">assembly</span><span style="color: #0000ff">="System.Data.Linq, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089"</span> <span style="color: #0000ff">/&gt;</span></pre>
                                              
                                              <p>
                                                </div> 
                                                
                                                <p>
                                                  Ok. Ya estamos listos!Ya Aparece nuestra vista Razor.
                                                </p>
                                                
                                                <p>
                                                  Para evitar que cualquiera pueda añadir usuarios, recordad de colocar [Authorize] en el controlador! Aunque el login se haya hecho con webforms, como son la misma aplicación, un usuario autenticado en webforms lo está en MVC y viceversa! 😀
                                                </p>
                                                
                                                <p>
                                                  Por supuesto hay muchos más temas que podríamos tratar, pero al menos espero que este post os sirva para ver que Webforms y MVC van de la mano sin ningún problema!
                                                </p>
                                                
                                                <p>
                                                  Un saludo!
                                                </p>
                                                
                                                <p>
                                                  PD: Os dejo el código del proyecto en: <a href="http://cid-6521c259e9b1bec6.office.live.com/self.aspx/BurbujasNet/ZipsPosts/webformsAndMvc.zip" title="http://cid-6521c259e9b1bec6.office.live.com/self.aspx/BurbujasNet/ZipsPosts/webformsAndMvc.zip"><span style="font-size: xx-small;">http://cid-6521c259e9b1bec6.office.live.com/self.aspx/BurbujasNet/ZipsPosts/webformsAndMvc.zip</span></a>
                                                </p>

 [1]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_30153047.png