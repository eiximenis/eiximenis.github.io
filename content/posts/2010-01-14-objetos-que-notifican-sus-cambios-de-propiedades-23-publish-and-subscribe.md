---
title: 'Objetos que notifican sus cambios de propiedades (2/3): Publish and subscribe'

author: eiximenis

date: 2010-01-14T10:04:00+00:00
geeks_url: /?p=1488
geeks_visits:
  - 1574
geeks_ms_views:
  - 582
categories:
  - Uncategorized

---
**Nota:** Este post es el segundo post de la <a target="_blank" href="/blogs/etomas/archive/2010/01/12/objetos-que-notifican-sus-cambios-de-propiedades-0-3-introducci-243-n.aspx" rel="noopener noreferrer">serie Objetos que notifican sus cambios de propiedades</a>__.

En el post anterior vimos como configurar Unity para que no tener que añadir código adicional para implementar la interfaz <a target="_blank" href="http://msdn.microsoft.com/es-es/library/system.componentmodel.inotifypropertychanged.aspx" rel="noopener noreferrer">INotifyPropertyChanged</a>. En este post quiero hablaros de un patrón que se utiliza mucho cuando hablamos de aplicaciones _complejas_: el patrón del <a target="_blank" href="http://en.wikipedia.org/wiki/Publish/subscribe" rel="noopener noreferrer">publicador &ndash; suscriptor</a>. En este patrón tenemos básicamente dos conceptos:

<!--more-->

  1. El publicador: Cuando un objeto quiere _notificar_ algo al respecto de su estado, se limita a publicar un mensaje con la información deseada.
  2. El suscriptor: Los subscriptores reciben todos aquellos mensajes a los que están suscritos, con _independencia_ de quien los haya publicado.

Este patrón se diferencia del modelo de eventos estándard de .NET, en que para realizar una suscripción a un tipo de mensaje no es necesario tener referencia alguna a quien pueda publicar este mensaje. En el sistema de eventos no és así: si quiero recibir información sobre el click de un botón, debo tener _una referencia_ a este botón, para poder registrar la función gestora del evento:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">button1.Click += <span style="color: #0000ff">new</span> EventHandler(button1_Click);<br /></pre>
  
  <p>
    </div> 
    
    <p>
      Este modelo de eventos directos tiene sus limitaciones y da en Winforms bastantes quebraderos de cabeza (especialmente cuando tenemos un formulario con un usercontrol formado por varios usercontrols que a su vez están formados por más usercontrols y queremos propagar un evento del usercontrol&nbsp; más <em>interno</em> al formulario). Es cierto que WPF introduce dos mejoras interesantes como los <em><a target="_blank" href="http://msdn.microsoft.com/en-us/library/ms742806.aspx" rel="noopener noreferrer">routed events</a></em> (que ayudan precisamente a solventar este problema de usercontrols anidados) y los <a target="_blank" href="http://msdn.microsoft.com/en-us/library/ms752308.aspx" rel="noopener noreferrer"><em>commands</em></a>,<em>&nbsp;</em>pero ninguno de ambos mecanismos ofrece la misma flexibilidad que el modelo de publicación &ndash; suscripción.
    </p>
    
    <p>
      Créeme: si desarrollas una aplicación compleja, ya sea en winforms o en WPF, te beneficiará mucho el uso de un modelo de publicación &ndash; suscripción (no en vano tanto CAB+SCSF como PRISM incorporan uno).
    </p>
    
    <p>
      Vamos a ver como podemos implementarnos uno que, aunque sencillito, sea lo suficientemente funcional...
    </p>
    
    <p>
      <strong>1. El notificador de mensajes</strong>
    </p>
    
    <p>
      Lo primero que debemos crear es el notificador de mensajes, es decir el objeto que usamos para <em>publicar</em> un mensaje y el que usamos también para informar a que tipo de mensajes queremos <em>suscribirnos</em>.
    </p>
    
    <p>
      El notificador de mensajes va a tener esta interfaz:
    </p>
    
    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">interface</span> ICommandNotifier<br />{<br />    <span style="color: #008000">/// &lt;summary&gt;</span><br />    <span style="color: #008000">/// Devuelve la lista de los commands actuales. Los commands se</span><br />    <span style="color: #008000">/// añaden automáticamente cuando se realiza un publish de cualquier</span><br />    <span style="color: #008000">/// tipo nuevo.</span><br />    <span style="color: #008000">/// &lt;/summary&gt;</span><br />    IEnumerable&lt;Type&gt; Commands { get; }<br />    <span style="color: #008000">/// &lt;summary&gt;</span><br />    <span style="color: #008000">/// Añade una suscripción al tipo de command TPayload</span><br />    <span style="color: #008000">/// &lt;/summary&gt;</span><br />    <span style="color: #008000">/// &lt;typeparam name="TPayload"&gt;Tipo de command al que nos suscribimos&lt;/typeparam&gt;</span><br />    <span style="color: #008000">/// &lt;param name="func"&gt;Acción a ejecutar cuando se publique el command&lt;/param&gt;</span><br />    <span style="color: #008000">/// &lt;param name="filterFunc"&gt;Método que se evalúa sobre el payload para determinar</span><br />    <span style="color: #008000">/// si el command se pasa o no al suscriptor.&lt;/param&gt;</span><br />    <span style="color: #008000">/// &lt;returns&gt;Token de suscripción&lt;/returns&gt;</span><br />    SubscriptionToken Subscribe&lt;TPayload&gt;(Action&lt;TPayload&gt; func, Func&lt;TPayload, <span style="color: #0000ff">bool</span>&gt; filterFunc);<br /><br />    <span style="color: #008000">/// &lt;summary&gt;</span><br />    <span style="color: #008000">/// Publica un command. El tipo de command es el tipo de la clase del payload.</span><br />    <span style="color: #008000">/// &lt;/summary&gt;</span><br />    <span style="color: #008000">/// &lt;param name="payload"&gt;Payload (datos) del commanad&lt;/param&gt;</span><br />    <span style="color: #0000ff">void</span> Publish(<span style="color: #0000ff">object</span> payload);<br /></pre>
      
      <p>
        </div> 
        
        <p>
          Básicamente sólo tiene un método para suscribirse a un determinado tipo de mensajes y otro método para publicarlos. Una implementación más compleja nos permitiría también eliminar suscripciones (es decir cuando ya no me interesa seguir recibiendo notificaciones de determinados commands)... pero eso lo dejamos como ejercicio 🙂
        </p>
        
        <p>
          La implementación tampoco es excesivamente compleja (no pongo el código aquí, ya que lo tenéis en el zip que adjunto al final del post). Básicamente lo que hace es:
        </p>
        
        <ol>
          <li>
            Mantiene una lista de todos los tipos de mensajes que se hayan lanzado. Lo que determina si un mensaje es de un tipo u otro es su clase (en la implementación una lista de objetos CommandInfo).
          </li>
          <li>
            Por cada mensaje de esa lista mantiene una lista con todos los suscriptores (en la implementación objetos de la clase AllTimeSubscriber).
          </li>
          <li>
            Por cada suscriptor de cada mensaje mantiene básicamente dos delegates: <ol>
              <li>
                El delegate que sirve para decidir si se envía este mensaje a este suscriptor (parámetro <em>filterFunc</em> del método Subscribe)
              </li>
              <li>
                El delegate que debe invocarse en el suscriptor (parámetro <em>func</em> del método Subscribe).
              </li>
            </ol>
          </li>
        </ol>
        
        <p>
          Sólo un apunte: el notificador de mensajes vamos a registrarlo en Unity como un singleton, eso significa que existirá sólo uno y que estará vivo durante <strong>toda</strong> la ejecución del programa. Por lo tanto, si guardamos directamente los delegates en el notificador de mensajes, impedirá al garbage collector actuar sobre los suscriptores (recordad que un delegate mantiene una <em>referencia</em> a un <em>objeto en concreto</em> y a un método). Para solucionar esto me he creado una clase, que he llamado WeakDelegate, que tiene la misma información que un delegate, pero usa una <a target="_blank" href="http://msdn.microsoft.com/es-es/library/system.weakreference.aspx" rel="noopener noreferrer">WeakReference</a> para apuntar al objeto (el suscriptor) y de esta manera permitir actuar al garbage collector. Recordad: siempre que guardeis referencias en un singleton considerad el uso de WeakReference!
        </p>
        
        <p>
          <strong>2. Cambiar la implementación de nuestro handler</strong>
        </p>
        
        <p>
          Una vez tenemos un notificador de mensajes, sólo debemos cambiar la implementación de nuestro <em>ICallHandler</em> (clase <em>AutoPropertyChangedHandler</em>) de Unity, para usar dicho notificador. Para ello en el método <em>Invoke</em> en lugar de llamar al método RaiseEvent (para lanzar el evento PropertyChanged) como hacíamos en el post anterior, vamos a usar el notificador para publicar un mensaje de tipo <em>PropertyChangedCommand</em>:
        </p>
        
        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #008000">// Si el setter no produce excepción, publicamos un command de tipo PropertyChangedCommand</span><br /><span style="color: #0000ff">if</span> (raiseEvt && msg.Exception == <span style="color: #0000ff">null</span>)<br />{<br />    cmdNotifier.Publish(<span style="color: #0000ff">new</span> PropertyChangedCommand(propName, input.Target));<br />}</pre>
          
          <p>
            </div> 
            
            <p>
              La clase PropertyChangedCommand es una clase que nos hemos creado nosotros que no hace nada más que guardar el nombre de la propiedad que ha cambiado y el objeto sobre el cual ha cambiado la propiedad.
            </p>
            
            <p>
              Como recibe la clase AutoPropertyChangedHandler el notificador de mensajes? Pues se le pasa en el constructor:
            </p>
            
            <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
              <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> AutoPropertyChangedHandler(ICommandNotifier cmdNotifier)<br />{<br />    <span style="color: #0000ff">this</span>.cmdNotifier = cmdNotifier;<br />}<br /></pre>
              
              <p>
                </div> 
                
                <p>
                  Ahora sólo debemos modificar la clase <em>AutoPropertyChangedAttribute</em> para que cuando cree el objeto AutoPropertyChangedHandler&nbsp; le pase el notificador de mensajes. La forma más fácil es aprovechar que en AutoPropertyChangedAttribute tenemos acceso a Unity, para devolver el objeto AutoPropertyChangedHandler usando Resolve y que de esa manera Unity inyecte el notificador de mensajes:
                </p>
                
                <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">override</span> ICallHandler CreateHandler(IUnityContainer container)<br />{<br />    <span style="color: #0000ff">return</span> container.Resolve&lt;AutoPropertyChangedHandler&gt;();<br />}</pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      <strong>3. El suscriptor</strong>
                    </p>
                    
                    <p>
                      Finalmente nos queda crear el suscriptor. Los suscriptores son clases normales que usan el notificador de mensajes para suscribirse a tipos de mensajes. P.ej. el siguiente suscriptor se suscribe a los mensajes cuyo tipo sea PropertyChangedCommand:
                    </p>
                    
                    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> Suscriptor(ICommandNotifier cmdNotif)<br />{<br />    cmdNotif.Subscribe&lt;PropertyChangedCommand&gt;(<span style="color: #0000ff">this</span>.DoPropertyChangedCommand, <span style="color: #0000ff">this</span>.CanDoPropertyChangedCommand);<br />}<br /><br /><span style="color: #0000ff">private</span> <span style="color: #0000ff">void</span> DoPropertyChangedCommand(PropertyChangedCommand payload)<br />{<br />    Console.WriteLine(<span style="color: #006080">"Propiedad {0} modificada"</span>, payload.PropertyName);<br />}<br /><br /><span style="color: #0000ff">private</span> <span style="color: #0000ff">bool</span> CanDoPropertyChangedCommand(PropertyChangedCommand payload)<br />{<br />    <span style="color: #0000ff">bool</span> retVal = !payload.PropertyName.Equals(<span style="color: #006080">"Name"</span>);<br />    Console.WriteLine(<span style="color: #006080">"CanDoPropertyChanged con prop {0} devuelve {1}"</span>, payload.PropertyName, retVal);<br />    <span style="color: #0000ff">return</span> retVal;<br />}</pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          Fíajos en la función <em>CanDoPropertyChangedCommand</em>: esta función se evalúa cada vez que alguien publica un command y sólo en el caso que devuelva <em>true</em> se ejecutará la función <em>DoPropertyChangedCommand</em> que es la que &ldquo;procesa&rdquo; el mensaje. En este caso, este suscriptor está interesado en recibir todos los cambios de calquier propiedad excepto &ldquo;Name&rdquo;.
                        </p>
                        
                        <p>
                          Finalmente sólo nos queda crear un suscriptor y probar el código. En el método Main() tengo:
                        </p>
                        
                        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">container.RegisterType&lt;ICommandNotifier, CommandNotifier&gt;(<span style="color: #0000ff">new</span> ContainerControlledLifetimeManager());<br />A2 a2 = container.Resolve&lt;A2&gt;();<br />Suscriptor subs = container.Resolve&lt;Suscriptor&gt;();<br />a2.Name = <span style="color: #006080">"edu"</span>;<br />a2.Edad = 10;<br /><span style="color: #008000">// Registramos el notificador como singleton</span><br />container.RegisterType&lt;ICommandNotifier, CommandNotifier&gt;(<span style="color: #0000ff">new</span> ContainerControlledLifetimeManager());<br /><span style="color: #008000">// Creamos un A2...</span><br />A2 a2 = container.Resolve&lt;A2&gt;();<br /><span style="color: #008000">// ... y un suscriptor</span><br />Suscriptor subs = container.Resolve&lt;Suscriptor&gt;();<br />a2.Name = <span style="color: #006080">"edu"</span>;<br />a2.Edad = 10;<br /></pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              Y listos! Sí lo ejecutais veréis que la salida es:
                            </p>
                            
                            <p>
                              <span style="font-family: Courier New;">CanDoPropertyChanged con prop Name devuelve False <br />CanDoPropertyChanged con prop Edad devuelve True <br />Propiedad Edad modificada</span>
                            </p>
                            
                            <p>
                              Ya tenemos implementado nuestro propio publicador-suscriptor!
                            </p>
                            
                            <p>
                              Os dejo <a target="_blank" href="http://cid-6521c259e9b1bec6.skydrive.live.com/self.aspx/BurbujasNet/ZipsPosts/AutoNotifPropertiesPost2.zip" rel="noopener noreferrer">un zip con todo el código</a> (en skydrive).
                            </p>
                            
                            <p>
                              Un saludo!
                            </p>