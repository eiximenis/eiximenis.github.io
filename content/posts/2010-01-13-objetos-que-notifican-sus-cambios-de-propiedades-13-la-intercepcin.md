---
title: 'Objetos que notifican sus cambios de propiedades (1/3): La intercepción'

author: eiximenis

date: 2010-01-13T13:35:00+00:00
geeks_url: /?p=1487
geeks_visits:
  - 2118
geeks_ms_views:
  - 510
categories:
  - Uncategorized

---
**Nota:** Este post es el primer post de la <a target="_blank" href="/blogs/etomas/archive/2010/01/12/objetos-que-notifican-sus-cambios-de-propiedades-0-3-introducci-243-n.aspx" rel="noopener noreferrer">serie Objetos que notifican sus cambios de propiedades</a>__.

En este post vamos a ver como configurar la intercepción de Unity, para poder inyectar nuestro código cada vez que se modifiquen las propiedades de un objeto.

<!--more-->

Los que desarrolléis en WPF sabréis que existe una interfaz llamada <a target="_blank" href="http://msdn.microsoft.com/es-es/library/system.componentmodel.inotifypropertychanged.aspx" rel="noopener noreferrer">INotifyPropertyChanged</a>, que se puede implementar para notificar a la interfaz de usuario de que las propiedades de un objeto (generalmente ligado a la interfaz) han modificado, y que por lo tanto la interfaz debe actualizar sus datos.

Esta interfaz define un solo evento, llamado <a target="_blank" href="http://msdn.microsoft.com/es-es/library/system.componentmodel.inotifypropertychanged.propertychanged.aspx" rel="noopener noreferrer">PropertyChanged</a> que debe lanzarse para informar del cambio de propiedad. Es responsabilidad de cada clase lanzar el evento cuando sea oportuno:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public class</span> A : INotifyPropertyChanged<br />{<br />    <span style="color: #0000ff">private</span> <span style="color: #0000ff">string</span> _name;<br /><br />    <span style="color: #008000">// Evento definido por la interfaz</span><br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">event</span> PropertyChangedEventHandler PropertyChanged;<br /><br />    <span style="color: #008000">// Lanza el evento "PropertyChanged"</span><br />    <span style="color: #0000ff">private</span> <span style="color: #0000ff">void</span> NotifyPropertyChanged(<span style="color: #0000ff">string</span> info)<br />    {<br />        var handler = <span style="color: #0000ff">this</span>.PropertyChanged;<br />        <span style="color: #0000ff">if</span> (handler != <span style="color: #0000ff">null</span>)<br />        {<br />            handler(<span style="color: #0000ff">this</span>, <span style="color: #0000ff">new</span> PropertyChangedEventArgs(info));<br />        }<br />    }<br />    <span style="color: #008000">// Propiedad que informa de sus cambios</span><br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Name<br />    {<br />        get { <span style="color: #0000ff">return</span> _name; }<br />        set<br />        {<br />            <span style="color: #0000ff">if</span> (_name != <span style="color: #0000ff">value</span>)<br />            {<br />                _name = <span style="color: #0000ff">value</span>;<br />                NotifyPropertyChanged(<span style="color: #006080">"Name"</span>); <br />            }<br />        }<br />    }<br />}</pre>
  
  <p>
    </div> 
    
    <p>
      Este código es pesado de realizar en clases con muchas propiedades y es fácil cometer errores... además no podemos utilizar las auto-propiedades. Vamos a ver como con el sistema de intercepción de Unity podemos hacer que este evento se lance de forma <em>automática</em>.
    </p>
    
    <p>
      <strong>1. Configurando el sistema de intercepción de Unity</strong>
    </p>
    
    <p>
      Para usar el sistema de intercepción de Unity, debéis añadir los assemblies Microsoft.Practices.ObjectBuilder2, Microsoft.Practices.Unity y Microsoft.Practices.Unity.Interception a vuestro proyecto (los tres assemblies forman parte de <a target="_blank" href="http://www.codeplex.com/unity/" rel="noopener noreferrer">Unity</a>).
    </p>
    
    <p>
      El primer paso es crear una clase que implemente la interfaz <em>ICallHandler</em>, esta clase es la encargada de proporcionarnos un punto dónde inyectar el código:
    </p>
    
    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">class</span> AutoPropertyChangedHandler : ICallHandler<br />{<br />    <span style="color: #0000ff">public</span> IMethodReturn Invoke(IMethodInvocation input, GetNextHandlerDelegate getNext)<br />    {<br />        <span style="color: #008000">// Aquí podremos inyectar nuestro código</span><br />        IMethodReturn msg = getNext()(input, getNext);<br />        <span style="color: #0000ff">return</span> msg;<br />    }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">int</span> Order { get; set; }<br />}</pre>
      
      <p>
        </div> 
        
        <p>
          El siguiente paso es crear un atributo que permita indicar a Unity que ICallHandler debe usar cuando se opere con objetos de la clase. Esta clase debe derivar de <em>HandlerAttribute</em> y debe redefinir el método <em>CreateHandler</em> para devolver una instancia del ICallHandler a utilizar:
        </p>
        
        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">class</span> AutoPropertyChangedAttribute : HandlerAttribute<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">override</span> ICallHandler CreateHandler(IUnityContainer container)<br />    {<br />        <span style="color: #0000ff">return</span> <span style="color: #0000ff">new</span> AutoPropertyChangedHandler();<br />    }<br />}<br /></pre>
          
          <p>
            </div> 
            
            <p>
              El código es trivial, eh?? Simplemente devuelve un <em>AutoPropertyChangeHandler</em>, de esa manera Unity usará este AutoPropertyChangeHandler para todas las clases que estén decoradas con el atributo <em>AutoPropertyChangedAttribute</em>. De esta manera es como le indicamos a Unity qué ICallHandler debe usar por cada tipo de clase.
            </p>
            
            <p>
              Finalmente queda configurar el propio contenedor. Para ello debemos debemos añadir la extensión de intercepción a Unity y indicarle que interceptor queremos utilizar para cada clase:
            </p>
            
            <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
              <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">IUnityContainer container = <span style="color: #0000ff">new</span> UnityContainer();<br />container.AddNewExtension&lt;Interception&gt;();<br />container.Configure&lt;Interception&gt;().SetInterceptorFor&lt;A2&gt;(<span style="color: #0000ff">new</span> VirtualMethodInterceptor());<br /></pre>
              
              <p>
                </div> 
                
                <p>
                  Le he indicado a Unity que para la clase A2 utilice el interceptor VirtualMethodInterceptor. Este interceptor puede interceptar todas las llamadas a métodos virtuales.
                </p>
                
                <p>
                  Finalmente ya podemos definir la clase A2:
                </p>
                
                <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">[AutoPropertyChanged()]<br /><span style="color: #0000ff">public class</span> A2 : INotifyPropertyChanged<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">event</span> PropertyChangedEventHandler PropertyChanged;<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">virtual</span> <span style="color: #0000ff">string</span> Name { get; set; }<br />}<br /></pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Fijaos en las diferencias entre A2 y la clase A antigua:
                    </p>
                    
                    <ol>
                      <li>
                        A2 está decorada con el atributo <em>AutoPropertyChangedAttribute</em> que hemos definido antes.
                      </li>
                      <li>
                        La propiedad Name es <strong>virtual</strong>
                      </li>
                      <li>
                        No tenemos ningún código adicional para lanzar el evento PropertyChanged.
                      </li>
                    </ol>
                    
                    <p>
                      Con eso el mecanismo de intercepción está listo. Si obteneis una instancia de A2 usando el método Resolve del contenedor y miráis con el debugger de que tipo <em>real</em> es el objeto, veréis que no es de tipo A2, sinó de un tipo <em>raro</em>: el proxy que crea Unity para poder interceptar los métodos virtuales (comparad el wach para a2 y a2newed:
                    </p>
                    
                    <p>
                      <a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_6CD319CE.png"><img height="53" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_691160FF.png" alt="image" border="0" title="image" style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" /></a>
                    </p>
                    
                    <p>
                      <strong>2. Implementando el código en nuestro Handler</strong>
                    </p>
                    
                    <p>
                      Vamos a modificar el método Invoke de AutoNotifyPropertyHandler para que lance el evento cada vez que se llame a un set de una propiedad... La clase completa queda tal y como sigue: Básicamente en el método Invoke, miramos si el nombre del método que se ha llamado empieza por &ldquo;set_&rdquo;, y si es el caso asumimos que es el Setter de una propiedad. Recogemos el valor <em>actual</em> de la propiedad usando reflection y si no son el mismo, lanzamos el evento <em>PropertyChanged</em> usando reflection. Y listos! 🙂
                    </p>
                    
                    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">class</span> AutoPropertyChangedHandler : ICallHandler<br />{<br />    <span style="color: #0000ff">public</span> IMethodReturn Invoke(IMethodInvocation input, GetNextHandlerDelegate getNext)<br />    {<br />        <span style="color: #0000ff">bool</span> raiseEvt = <span style="color: #0000ff">false</span>;<br />        <span style="color: #0000ff">string</span> propName = <span style="color: #0000ff">null</span>;<br />        INotifyPropertyChanged inpc = input.Target <span style="color: #0000ff">as</span> INotifyPropertyChanged;<br />        <span style="color: #0000ff">if</span> (inpc != <span style="color: #0000ff">null</span>)<br />        {<br />            <span style="color: #008000">// Si el nombre del método empieza por "set_" es un Setter de propiedad</span><br />            <span style="color: #0000ff">if</span> (input.MethodBase.Name.StartsWith(<span style="color: #006080">"set_"</span>))<br />            {<br />                propName = input.MethodBase.Name.Substring(4);<br />                MethodInfo getter = input.Target.GetType().GetProperty(propName).GetGetMethod();<br />                <span style="color: #0000ff">object</span> oldValue = getter.Invoke(input.Target, <span style="color: #0000ff">null</span>);<br />                <span style="color: #0000ff">object</span> newValue = input.Arguments[0];<br />                <span style="color: #008000">// Si los valores de newValue y oldValue son distintos</span><br />                <span style="color: #008000">// debemos lanzar el evento (la comparación la hacemos por</span><br />                <span style="color: #008000">// Equals si es posible).</span><br />                raiseEvt = newValue == <span style="color: #0000ff">null</span> ?<br />                    newValue != oldValue :<br />                    !newValue.Equals(oldValue);<br />            }<br />        }<br />        IMethodReturn msg = getNext()(input, getNext);<br />        <span style="color: #008000">// Si el setter no produce excepción, lanzamos el evento</span><br />        <span style="color: #0000ff">if</span> (raiseEvt && msg.Exception == <span style="color: #0000ff">null</span>)<br />        {<br />            RaiseEvent(inpc, propName);<br />        }<br />        <span style="color: #0000ff">return</span> msg;<br />    }<br /><br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">int</span> Order { get; set; }<br /><br />    <span style="color: #008000">// Método que lanza el evento PropertyChanged usando reflection</span><br />    <span style="color: #0000ff">private</span> <span style="color: #0000ff">void</span> RaiseEvent(INotifyPropertyChanged inpc, <span style="color: #0000ff">string</span> propertyName)<br />    {<br />        Type type = inpc.GetType();<br />        FieldInfo evt = <span style="color: #0000ff">null</span>;<br />        <span style="color: #008000">// Buscamos el evento (no estará en el propio Type ya que el propio Type</span><br />        <span style="color: #008000">// será un proxy, pero iteraremos por los tipos base hasta encontrarlo)</span><br />        <span style="color: #0000ff">do</span><br />        {<br />            evt = type.GetField(<span style="color: #006080">"PropertyChanged"</span>, BindingFlags.Instance | BindingFlags.NonPublic);<br />            type = type.BaseType;<br />        } <span style="color: #0000ff">while</span> (evt == <span style="color: #0000ff">null</span> && type.BaseType != <span style="color: #0000ff">null</span>);<br />        <span style="color: #008000">// Invocamos el evento</span><br />        <span style="color: #0000ff">if</span> (evt != <span style="color: #0000ff">null</span>)<br />        {<br />            MulticastDelegate mcevt = evt.GetValue(inpc) <span style="color: #0000ff">as</span> MulticastDelegate;<br />            <span style="color: #0000ff">if</span> (mcevt != <span style="color: #0000ff">null</span>)<br />            {<br />                mcevt.DynamicInvoke(inpc, <span style="color: #0000ff">new</span> PropertyChangedEventArgs(propertyName));<br />            }<br />        }<br />    }<br />}</pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          Ahora podemos comprobar como se lanza el evento automáticamente al modificar una propiedad de A2.
                        </p>
                        
                        <p>
                          Un saludo!!!!
                        </p>
                        
                        <p>
                          <strong>PD: </strong>Dije en el post introductorio que no pondría código, pero finalmente os incluyo el <a target="_blank" href="http://cid-6521c259e9b1bec6.skydrive.live.com/self.aspx/BurbujasNet/ZipsPosts/AutoNotifPropertiesPost1.zip" rel="noopener noreferrer">zip con el código de este post</a> (en SkyDrive).
                        </p>
                        
                        <p>
                          PD2: Echad un post a <em><a target="_blank" href="http://shecht.wordpress.com/2009/12/12/inotifypropertychanged-with-unity-interception-aop/" rel="noopener noreferrer">INotifyPropertyChanged with Unity Interception AOP</a></em> del blog de Dmitry Shechtman que me ha servido de inspiración para el ejemplo (yo tenía pensado uno mucho más tonto en este primer post).
                        </p>