---
title: Caliburn… ¿sientes el poder de MVVM en tus manos?
author: eiximenis

date: 2010-02-12T17:22:00+00:00
geeks_url: /?p=1496
geeks_visits:
  - 4895
geeks_ms_views:
  - 1167
categories:
  - Uncategorized

---
Los más frikis de por aquí, sabréis que Caliburn (_<a target="_blank" href="http://la.wikipedia.org/wiki/Caliburnus" rel="noopener noreferrer">Caliburnus</a>_ para ser exactos) era el nombre de una poderosa espada que luego alguien decidió rebautizar como <a target="_blank" href="http://es.wikipedia.org/wiki/Excalibur" rel="noopener noreferrer"><em>Excalibur</em></a>... Como frikis hay en todas partes y en eso de la informática pues quizás más, <a target="_blank" href="http://caliburn.codeplex.com/" rel="noopener noreferrer">Caliburn</a> también resulta ser el nombre de un framework para aplicaciones Silverlight y WPF. Dicho así parece ser lo mismo que <a target="_blank" href="http://www.codeplex.com/CompositeWPF/" rel="noopener noreferrer">PRISM</a> y en cierta manera ambos frameworks tienen el mismo objetivo y comparten muchas características. Por ejemplo ambos frameworks se abstraen del contendor IoC a utilizar (es decir requieren uno, pero no se atan a ninguno), ambos dan soporte a vistas compuestas y ambos tienen el concepto de módulo... entonces ¿en que se diferencian? Pues en como se enfocan para llegar al objetivo. El objetivo de este post es **iniciar una serie de posts** (no se de cuantos :P) para hablar sobre Caliburn y compararlo con PRISM. Hoy, pero vamos a empezar por lo más básico... 🙂

**1. Preparando el entorno**

No es muy complicado preparar el entorno para trabajar con Caliburn: basta con <a target="_blank" href="http://caliburn.codeplex.com/releases/view/34985" rel="noopener noreferrer">descargarse el framework (actualmente está en v1 RTW)</a>. Caliburn (de nuevo al igual que PRISM) existe en dos sabores: Silverlight y WPF. Ambas versiones son esencialmente la misma salvando las diferencias técnológicas que existen entre Silverlight y WPF. Vamos a optar en este caso por una aplicación WPF.

Abrimos VS2008 y creamos una nueva aplicación WPF. El siguiente paso es añadir referencias a los ensablados de Caliburn que estarán en el directorio Bin/NET-3.5/debug (o release, hay ambas versiones). **Nota:** Yo os recomiendo que os descargueis el codigo fuente y compileis Caliburn... Así será más fácil depurar! 

Si en lugar de WPF nuestra aplicación fuese Silverlight entonces debemos ir al directorio Bin/Silverlight-2.0 o Silverlight-3.0 según sea necesario. Para empezar vamos a usar Caliburn.Core.dll, Caliburn.PresentationFramework.dll y Microsoft.Practices.ServiceLocation.dll.

Ahora sí! Ya estamos listos para desenvainar la espada ...

**2. Empezando con Caliburn**

Caliburn tiene una idea muy clara sobre como se debe organizar la IU de tu aplicación: usando MVVM. Eso significa que vamos a tener un grupo de clases llamadas _ViewModels_ que van a ser los que tengan **toda** la información sobre los datos a mostrar. A cada _ViewModel_ le corresponderá una vista (_View_). El enlace entre las vistas y los _ViewModels_ será a través de Bindings... por supuesto que todo esto se puede hacer sin Caliburn, pero Caliburn nos da herramientas para que sea un poco más fácil.

Para empezar vamos a crear un ViewModel y una vista y vamos a dejar que la magia de Caliburn nos lo una. Para ello, **creamos una carpeta llamada ViewModel** en nuestro proyecto. Es importante el nombre de esta carpeta, puesto que Caliburn asume que lo que en ella esté son ViewModels o vistas. Dentro de dicha carpeta creamos una clase tal y como sigue:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> UserViewModel<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Nombre { get; set; }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Foto { get; set; }<br />}</pre>
  
  <p>
    </div> 
    
    <p>
      Ya tenemos el ViewModel de un usuario: su nombre y su foto. Ahora el siguiente paso es crear una vista. Para ello añadid un <em>User Control</em> que se llame UserView. El nombre de nuevo es importante: Caliburn asumirá que UserView es la vista para los <em>ViewModels</em> de tipo UserViewModel. Poned el user control <strong>fuera</strong> de la carpeta ViewModel. El código xaml puede ser algo parecido a:
    </p>
    
    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">&lt;</span><span style="color: #800000">UserControl</span> <span style="color: #ff0000">x:Class</span><span style="color: #0000ff">="CaliburnDemo.UserView"</span><br />    <span style="color: #ff0000">xmlns</span><span style="color: #0000ff">="http://schemas.microsoft.com/winfx/2006/xaml/presentation"</span><br />    <span style="color: #ff0000">xmlns:x</span><span style="color: #0000ff">="http://schemas.microsoft.com/winfx/2006/xaml"</span><br />    <span style="color: #ff0000">Height</span><span style="color: #0000ff">="300"</span> <span style="color: #ff0000">Width</span><span style="color: #0000ff">="300"</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">Grid</span><span style="color: #0000ff">&gt;</span><br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">Label</span> <span style="color: #ff0000">Height</span><span style="color: #0000ff">="28"</span> <span style="color: #ff0000">Margin</span><span style="color: #0000ff">="25,47,28,0"</span> <span style="color: #ff0000">VerticalAlignment</span><span style="color: #0000ff">="Top"</span> <span style="color: #ff0000">Content</span><span style="color: #0000ff">="{Binding Nombre}"</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">Label</span><span style="color: #0000ff">&gt;</span><br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">Image</span> <span style="color: #ff0000">Margin</span><span style="color: #0000ff">="109,81,127,0"</span> <span style="color: #ff0000">Stretch</span><span style="color: #0000ff">="Fill"</span>  <span style="color: #ff0000">Width</span><span style="color: #0000ff">="64"</span> <span style="color: #ff0000">Height</span><span style="color: #0000ff">="64"</span> <span style="color: #ff0000">VerticalAlignment</span><span style="color: #0000ff">="Top"</span> <br />               <span style="color: #ff0000">Source</span><span style="color: #0000ff">="{Binding Foto}"</span><span style="color: #0000ff">/&gt;</span><br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">Label</span> <span style="color: #ff0000">Height</span><span style="color: #0000ff">="28"</span> <span style="color: #ff0000">Margin</span><span style="color: #0000ff">="85,13,101,0"</span> <span style="color: #ff0000">VerticalAlignment</span><span style="color: #0000ff">="Top"</span><span style="color: #0000ff">&gt;</span>Datos del Usuario:<span style="color: #0000ff">&lt;/</span><span style="color: #800000">Label</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;/</span><span style="color: #800000">Grid</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">UserControl</span><span style="color: #0000ff">&gt;</span></pre>
      
      <p>
        </div> 
        
        <p>
          Finalmente sólo nos queda un paso: modificar nuestra aplicación para que derive de <em>CaliburnApplication</em>. Para ello, en App.cs modificad la clase para que derive de CaliburnApplication:
        </p>
        
        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">partial</span> <span style="color: #0000ff">class</span> App : CaliburnApplication<br />{<br />    <span style="color: #0000ff">protected</span> <span style="color: #0000ff">override</span> <span style="color: #0000ff">object</span> CreateRootModel()<br />    {<br />        <span style="color: #0000ff">return</span> <span style="color: #0000ff">new</span> UserViewModel();<br />    }<br />}</pre>
          
          <p>
            </div> 
            
            <p>
              Fijaos que redefinimos el método CreateRootModel: Este método (definido en CaliburnApplication) es el punto de entrada de nuestra aplicación. El tipo de ViewModel que creemos determinará el tipo de vista a utilizar y los datos iniciales a mostrar. Un detalle: Fijaos que <strong>no</strong> vamos a crear una ventana nunca (nuestra vista UserView es un UserControl). No hay problema, porque si el ViewModel inicial no es una ventana, Caliburn la va a crear para nosotros.
            </p>
            
            <p>
              Hemos modificado la clase base de la aplicación en el fichero .cs y debemos hacer lo mismo en App.xaml:
            </p>
            
            <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
              <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">&lt;</span><span style="color: #800000">caliburn:CaliburnApplication</span> <span style="color: #ff0000">x:Class</span><span style="color: #0000ff">="CaliburnDemo.App"</span><br />    <span style="color: #ff0000">xmlns</span><span style="color: #0000ff">="http://schemas.microsoft.com/winfx/2006/xaml/presentation"</span><br />    <span style="color: #ff0000">xmlns:x</span><span style="color: #0000ff">="http://schemas.microsoft.com/winfx/2006/xaml"</span><br />    <span style="color: #ff0000">xmlns:caliburn</span><span style="color: #0000ff">="clr-namespace:Caliburn.PresentationFramework.ApplicationModel;assembly=Caliburn.PresentationFramework"</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">Application.Resources</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;/</span><span style="color: #800000">Application.Resources</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">caliburn:CaliburnApplication</span><span style="color: #0000ff">&gt;</span></pre>
              
              <p>
                </div> 
                
                <p>
                  No hay secreto: Declaro el namespace <em>caliburn</em> y modifico el tag raiz para que en lugar de Application sea CaliburnApplication que es nuestra nueva clase base. También elimino el StartupUri ya que no es necesario.
                </p>
                
                <p>
                  Y listos... ya podemos ejecutar!
                </p>
                
                <p>
                  <strong>3. Plaf! La primera en la frente!</strong>
                </p>
                
                <p>
                  Si has seguido mis indicaciones te vas a encontrar algo parecido a esto (si no has compilado Caliburn quizá simplemente te salga una excepción en lugar de esta &ldquo;preciosa&rdquo; ventana).
                </p>
                
                <p>
                  <a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_46BB08FA.png"><img height="149" width="345" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_761951F9.png" alt="image" border="0" title="image" style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" /></a>
                </p>
                
                <p>
                  Hombre... no es muy bonito que digamos... cual es el problema? Fácil: la vista no está situada en el lugar que toca... Recordáis que os dije que la pusierais <em><strong>fuera</strong> </em>de la carpeta ViewModel? Pues debe ir dentro... Así, que moved UserView <em>dentro</em> de la carpeta ViewModel y <strong>modificad</strong> el namespace para que incluya ViewModel:
                </p>
                
                <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">namespace</span> CaliburnDemo.<strong><span style="text-decoration: underline;">ViewModel</span></strong><br />{<br />}</pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      De hecho lo importante es el namespace, no la ubicación física del archivo xaml, así que si no quereis moverlo no lo hagáis per el namespace de la clase UserView debe ser el mismo que el de UserViewModel.
                    </p>
                    
                    <p>
                      Ahora sí que sí! Si ejecutamos vemos una triste ventana... pero es <em>nuestra</em> ventana:
                    </p>
                    
                    <p>
                      <a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_092A58D9.png"><img height="244" width="228" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_63C0656A.png" alt="image" border="0" title="image" style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" /></a>
                    </p>
                    
                    <p>
                      Está vacía porque el ViewModel que hemos creado lo está, pero eso tiene fácil arreglo modificando el método <em>CreateRootModel</em> para que el UserViewModel creado tenga datos:
                    </p>
                    
                    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">protected</span> <span style="color: #0000ff">override</span> <span style="color: #0000ff">object</span> CreateRootModel()<br />{<br />    <span style="color: #0000ff">return</span> <span style="color: #0000ff">new</span> UserViewModel()<br />    {<br />        Nombre = <span style="color: #006080">"Edu"</span>,<br />        Foto = <span style="color: #006080">"/CaliburnDemo;component/avatar.png"</span><br />    };<br />}</pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          (La foto está añadida como Resource en el proyecto, de ahí esta ruta).
                        </p>
                        
                        <p>
                          Ahora si que vemos ya nuestros datos:
                        </p>
                        
                        <p>
                          <a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_06A1271B.png"><img height="167" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_0C0F97BF.png" alt="image" border="0" title="image" style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" /></a>
                        </p>
                        
                        <p>
                          &iexcl;Mola! Que es lo que ha hecho Caliburn por nosotros? Pues a partir de un ViewModel ha creado la vista correspondiente y ha asignado el ViewModel como DataContext de la vista...
                        </p>
                        
                        <p>
                          Ok... no es nada que no podamos hacer nosotros mismos con pocas líneas de código... pero esto es sólo el principio! En sucesivos posts iremos viendo otras cosillas de Caliburn. Obviamente si alguien ha trabajado con Caliburn y/o con PRISM y quiere contar sus opiniones... adelante!
                        </p>
                        
                        <p>
                          Un saludo a todos!
                        </p>
                        
                        <p>
                          PD: Dejo el <a target="_blank" href="http://cid-6521c259e9b1bec6.skydrive.live.com/self.aspx/BurbujasNet/ZipsPosts/CaliburnDemo1.zip" rel="noopener noreferrer">código del proyecto en este ficherillo zip!</a> (en mi skydrive)
                        </p>