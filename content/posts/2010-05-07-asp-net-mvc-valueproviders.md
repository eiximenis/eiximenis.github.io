---
title: 'ASP.NET MVC: ValueProviders'
author: eiximenis

date: 2010-05-07T11:12:58+00:00
geeks_url: /?p=1508
geeks_visits:
  - 4225
geeks_ms_views:
  - 4259
categories:
  - Uncategorized

---
Hola! Hoy quiero comentar un aspecto de ASP.NET MVC2 que no sé hasta que punto es conocido, y son los llamados _Value Providers_.

> **Disclaimer:** Este post será largo y puede ser un poco denso y asumo conocimientos básicos de ASP.NET MVC. Tampoco tengas reparos en leerte este post en más de un dia si quieres… había pensado dividirlo en dos posts, pero al final he preferido meterlo todo en uno.

Si habéis trabajado un poco con ASP.NET MVC, sabréis que si tenéis un controlador con esta acción:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">[HttpPost()]<br /><span style="color: #0000ff">public</span> ActionResult Create(Producto p)<br />{<br />    <span style="color: #008000">// Hacer algo con producto</span><br />    <span style="color: #0000ff">return</span> View();<br />}</pre>
  
  <p>
    </div> 
    
    <p>
      ASP.NET MVC es capaz de hacer binding entre los parámetros de la Request y las propiedades de la clase Producto para instanciar un objeto Producto para tí y pasarlo al controlador.
    </p>
    
    <p>
      Es decir, si la clase producto está definida como:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> Producto<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">int</span> Codigo { get; set; }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Nombre { get; set; }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">int</span> Precio { get; set; }<br />}</pre>
      
      <p>
        </div> 
        
        <p>
          Y la Request tiene estos campos, ASP.NET MVC hace el binding por nosotros… P.ej. si usamos un formulario como el siguiente para enviar los datos:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">&lt;form action=<span style="color: #006080">"/Productos/Create"</span> method=<span style="color: #006080">"post"</span>&gt;<br />&lt;fieldset&gt;    <br />        &lt;input id=<span style="color: #006080">"Codigo"</span> name=<span style="color: #006080">"Codigo"</span> type=<span style="color: #006080">"text"</span> <span style="color: #0000ff">value</span>=<span style="color: #006080">""</span> /&gt;<br />        &lt;input id=<span style="color: #006080">"Nombre"</span> name=<span style="color: #006080">"Nombre"</span> type=<span style="color: #006080">"text"</span> <span style="color: #0000ff">value</span>=<span style="color: #006080">""</span> /&gt;        <br />        &lt;input id=<span style="color: #006080">"Precio"</span> name=<span style="color: #006080">"Precio"</span> type=<span style="color: #006080">"text"</span> <span style="color: #0000ff">value</span>=<span style="color: #006080">""</span> /&gt;<br />        &lt;input type=<span style="color: #006080">"submit"</span> <span style="color: #0000ff">value</span>=<span style="color: #006080">"Create"</span> /&gt;<br />&lt;/fieldset&gt;<br />&lt;/form&gt;<br /><br /></pre>
          
          <p>
            </div> 
            
            <p>
              La Request tiene los campos “Codigo”, “Nombre” y “Precios” (los <em>name</em> de los input). Al enviarse el formulario genera una Request con los siguientes datos POST (se pueden ver fácilmente usando firebug):
            </p>
            
            <p>
              <code>Content-Type: application/x-www-form-urlencoded&lt;br />
    &lt;br /></code><code>Content-Length: 31&lt;br />
    &lt;br /></code><code>&lt;strong>Codigo=1&Nombre=Silla&Precio=12&lt;/strong></code>
            </p>
            
            <p>
              La última línea es la que tiene los campos con los valores, que son usados por ASP.NET MVC para realizar el binding con la clase Producto.
            </p>
            
            <p>
              El responsable de recoger los valores y realizar el binding con el modelo es el <em>ModelBinder </em>pero <strong>no es reponsabilidad del ModelBinder saber <em>donde </em>están los valores</strong>. P.ej. haced una prueba… Si al formulario anterior le quitáis uno de los campos (p.ej. el campo Precio) y modificáis el action del <form> para que quede <em><form action=”/Productos/Create?Precio=10” method=”post></em> y realizáis la petición <strong>veréis que sigue funcionando</strong>.
            </p>
            
            <p>
              En este caso si miramos con firebug la petición, no tiene el parámetro post “Precio”:
            </p>
            
            <p>
              <code>Content-Type: application/x-www-form-urlencoded&lt;br />
    &lt;br /></code><code>Content-Length: 21&lt;br />
    &lt;br /></code><code>&lt;strong>Codigo=1&Nombre=Silla&lt;/strong></code>
            </p>
            
            <p>
              Pero si que dicho parámetro está en la URL, que ahora es <em>/Productos/Create<strong>?Precio=10</strong></em>. Pero esto <strong>no afecta</strong> al ModelBinder que es capaz de recoger el parámeteo con independencia de dónde se encuentre dentro de la Request. ¿Y cómo es posible? Pues gracias a los Value Providers.
            </p>
            
            <p>
              Pero antes sigamos jugando un poco, para tener todas las piezas encima de la mesa… que ocurre si <strong>sin modificar el atributo action del tag <form></strong> volvemos a añadir el <input name=”Precio”> al formulario? Es decir tenemos un formulario con los tres campos (Codigo, Nombre y Precio) pero además tenemos el campo Precio <em>otra vez </em>en la url (?Precio=100). Pueden ocurrir tres cosas:
            </p>
            
            <ol>
              <li>
                Que ASP.NET MVC de error, porque el campo “Precio” está repetido (una vez en la URL y otra en los parámetros POST) de la petición.
              </li>
              <li>
                Que tenga prioridad el valor del parámetro en la URL
              </li>
              <li>
                Que tenga prioriodad el valor del parámetro POST
              </li>
            </ol>
            
            <p>
              Bien, lo que ocurre es que <strong>tiene prioridad el valor del parámetro POST</strong>. Es decir el valor de la propiedad Precio del objeto Producto que reciba el controlador será el que haya entrado el usuario y no el que se indica en la URL. Pero… ¿por que?
            </p>
            
            <p>
              Cuando una petición es tratada por el framework, primero se pasa por una serie de <em>value providers</em> que inspeccionan cada uno dicha petición y guardan los valores que cada uno de ellos entiende. P.ej. existe un value provider para inspeccionar los valores POST y otro distinto para inspeccionar los valores de la URL. ASP.NET MVC mantiene una colección de value providers y pasa la request por todos ellos.
            </p>
            
            <p>
              P.ej. asumamos en nuestro caso que tenemos una colección con dos value providers (luego veremos que <strong>esto no es exactamente así, pero me vale esa simplificación por ahora</strong>). Llamemosle PostValueProvider al primero y UrlValueProvider al segundo. Supongamos que nos llega la petición que hemos comentado:
            </p>
            
            <ul>
              <li>
                Parámetros en la URL: ?Precio=100
              </li>
              <li>
                Parámetros POST: Codigo=1&Nombre=Silla&Precio=12
              </li>
            </ul>
            
            <p>
              La request pasa por el primer value provider (supongamos que es el PostValueProvider) que la inspecciona, ve que hay tres parámetros llamados Código, Nombre y Precio y se los guarda con sus respectivos valores. Luego la request pasa por el siguiente value provider, el UrlValueProvider que inspecciona la request y ve que hay un parámetro llamado Precio y se lo guarda junto con su valor.
            </p>
            
            <p>
              Ahora es el turno del ModelBinder: el ModelBinder detecta que debe crear un objeto Producto que tiene tres propiedades: Código, Nombre y Precio, así que pregunta a los value providers por los valores de estos campos. Esta frase es la clave:<strong> pregunta a los value providers, no a un value provider en particular</strong>. Pongamos que cuando el ModelBinder necesita el valor del campo “Precio” se limita a preguntar simplemente el valor de dicho campo, y el ModelBinder espera que haya un sólo valor para dicho campo. Si como es el caso dos value providers han guardado el valor para dicho campo, sólo uno responde… cual? Pues el que esté primero en la lista de value providers que ASP.NET MVC mantiene. Ni más ni menos 🙂
            </p>
            
            <p>
              <strong>Creación de un Value Provider propio</strong>
            </p>
            
            <p>
              Como ya sabréis ASP.NET MVC es muy extensible, así que obviamente uno se puede crear sus propios Value Providers, para permitir tratar peticiones que tengan datos codificados de forma extraña o en otros sitios donde pueden estar los datos (p.ej. en cookies). Crear un value provider es muy simple, basta crear una clase que implemente la interfaz <a href="http://msdn.microsoft.com/en-us/library/system.web.mvc.ivalueprovider.aspx" target="_blank" rel="noopener noreferrer">IValueProvider</a>. Este interfaz define dos métodos:
            </p>
            
            <ol>
              <li>
                ContainsPrefix –> No se como explicar fácilmente lo que significa <em>exactamente</em> sin entrar en demasiados detalles sobre como funciona el ModelBinder, así que permitidme una simplificación. Este método devuelve si el value provider tiene valor para un campo en concreto. Es decir si el value provider tiene un valor para el campo “Precio” entonces ContainsPrefix(“Precio”) debe devolver true. Insisto: No es tan simple, pero para entender el concepto del post nos basta así.
              </li>
              <li>
                GetValue –> Devuelve el valor que se le pide. Es decir GetValue(“Precio”) devuelve el valor correspondiente al campo “Precio” que tenga este Value Provider. Este método sólo se llama si ContainsPrefix devuelve true.
              </li>
            </ol>
            
            <p>
              Vamos a ver como podemos implementar un ValueProvider propio… P.ej. vamos a implementar un Value Provider que lea valores de los <appSettings> del web.config (de acuerdo, quizá no es brutalmente útil, pero como ejemplo servirá).
            </p>
            
            <p>
              El código podría ser el siguiente:
            </p>
            
            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> AppSettingsValueProvider : IValueProvider<br />{<br />    <span style="color: #0000ff">private</span> Dictionary&lt;<span style="color: #0000ff">string</span>, ValueProviderResult&gt; values;<br /><br /><br />    <span style="color: #0000ff">public</span> AppSettingsValueProvider()<br />    {<br />        values = <span style="color: #0000ff">new</span> Dictionary&lt;<span style="color: #0000ff">string</span>, ValueProviderResult&gt;();<br />        <span style="color: #0000ff">foreach</span> (<span style="color: #0000ff">string</span> key <span style="color: #0000ff">in</span> ConfigurationManager.AppSettings.AllKeys)<br />        {<br />            <span style="color: #0000ff">string</span> appSetting = ConfigurationManager.AppSettings[key];<br />            values.Add(key, <span style="color: #0000ff">new</span> ValueProviderResult(appSetting, appSetting, CultureInfo.InvariantCulture));<br />        }<br />    }<br /><br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">bool</span> ContainsPrefix(<span style="color: #0000ff">string</span> prefix)<br />    {<br />        <span style="color: #0000ff">return</span> values.ContainsKey(prefix);<br />    }<br /><br />    <span style="color: #0000ff">public</span> ValueProviderResult GetValue(<span style="color: #0000ff">string</span> key)<br />    {<br />        ValueProviderResult <span style="color: #0000ff">value</span>;<br />        values.TryGetValue(key, <span style="color: #0000ff">out</span> <span style="color: #0000ff">value</span>);<br />        <span style="color: #0000ff">return</span> <span style="color: #0000ff">value</span>;<br />    }<br />}</pre>
              
              <p>
                </div> 
                
                <p>
                  En el constructor se inicializa el diccionario <em>values</em> con los valores leídos de los <appSettings> del web.config. El método GetValue de IValueProvider, no devuelve un object con el valor directo del campo pedido, sinó que devuelve un <em>ValueProviderResult</em> una clase que tiene tres campos (rawValue, attemptedValue y culture).&#160; El campo rawValue es lo que <em>realmente</em> se ha leído de la petición, mientras que el campo attemptedValue es el valor de rawValue convertido a una string. En mi caso dado que <appSettings> contiene ya string, tanto rawValue como attemptedValue tienen el mismo valor.
                </p>
                
                <p>
                  Bien! Ya tenemos un value provider… ahora ha llegado el momento de decirle a ASP.NET MVC que lo use… y aquí es donde debemos explicar la simplificación que hice antes 🙂
                </p>
                
                <p>
                  <strong>Factorías de Value Providers</strong>
                </p>
                
                <p>
                  Os acordáis cuando dije que ASP.NET MVC mantenía una colección de value providers y que pasaba la request a cada uno de ellos para que pudiesen procesarla y guardarse los campos necesarios? Dije que esto no era exactamente así, sinó una simplificación… Pues bien, la verdad es que ASP.NET MVC <strong>no</strong> mantiene una colección de value providers, sinó una colección de <strong>factorías de value providers</strong>.
                </p>
                
                <p>
                  Si te preguntas porque una factoría en lugar de guardar directamente los value providers… es para darte más control sobre <em>como</em> se crean los value providers: un value provider actúa sobre los datos de la petición actúal, por lo que por cada petición <strong>deben </strong>crearse todos los value providers de nuevo… La interfaz IValueProvider no define ningún mecanismo para pasarle la Request al value provider. Tampoco tenemos acceso a ningún tipo de contexto ni nada parecido… Piensa, si ASP.NET MVC debe crear los value providers a cada petición, cómo le pasa los datos de la request?
                </p>
                
                <p>
                  La solución pasa por usar <em>factorías de value providers</em> es decir, clases cuya única responsabilidad es crear los value providers a cada petición.
                </p>
                
                <p>
                  Veamos como sería nuestra factoría que cree objetos de nuestro <em>AppSettingsValueProvider</em>:
                </p>
                
                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> AppSettingsValueProviderFactory : ValueProviderFactory<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">override</span> IValueProvider GetValueProvider(ControllerContext controllerContext)<br />    {<br /><br />        <span style="color: #0000ff">return</span> <span style="color: #0000ff">new</span> AppSettingsValueProvider();<br />    }<br />}</pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Basta con derivar de <em>ValueProviderFactory</em> y redefinir el método GetValueProvider. En este méotdo tenemos acceso al <em>ControllerContext</em> que a su vez nos da acceso a la request http. Ahora si queréis pasáis la request http (o lo que queráis) al value provider.
                    </p>
                    
                    <p>
                      Una vez tenemos la factoría debemos decirle a ASP.NET MVC que la use. Eso se hace registrando nuestra factoría usando la clase estática <a href="http://msdn.microsoft.com/en-us/library/system.web.mvc.valueproviderfactories.aspx" target="_blank" rel="noopener noreferrer">ValueProviderFactories</a>:
                    </p>
                    
                    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">ValueProviderFactories.Factories.Add(<span style="color: #0000ff">new</span> AppSettingsValueProviderFactory());</pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          Usualmente esto se coloca en el Application_Start de Global.asax.
                        </p>
                        
                        <p>
                          Fijaos en un detalle: La factoría <strong>se crea sólo una vez</strong> (y la creamos nosotros, no el framework) y luego para cada petición se llama al método GetValueProvider de la factoría… método que también hemos hecho nosotros y donde se devuelve el value provider. De esta manera <strong>controlamos la creación de los value providers </strong>(lo que nos permitiría usar, si quisiéramos, mecanismos de inyección de dependencias).
                        </p>
                        
                        <p>
                          Y listos! Ya estamos. Si modificamos la vista de datos para que <strong>no</strong> tenga el campo “Precio” (ni como POST ni como GET) y añadimos en el web.config:
                        </p>
                        
                        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">&lt;</span><span style="color: #800000">appSettings</span><span style="color: #0000ff">&gt;</span><br />  <span style="color: #0000ff">&lt;</span><span style="color: #800000">add</span> <span style="color: #ff0000">key</span><span style="color: #0000ff">="Precio"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="999"</span><span style="color: #0000ff">/&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">appSettings</span><span style="color: #0000ff">&gt;</span></pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              Veréis que el controlador recibe un Producto con el nombre y código que hayáis indicado y un Precio de 999. Dado que nuestra factoría se ha añadido la última de la lista, si volvéis a poner el Precio en el formulario, tendrá prioridad el que haya entrado el usuario (ya que la factoría que crea el value provider con los datos POST está <em>antes</em> de nuestra factoría).
                            </p>
                            
                            <p>
                              Os dejo un proyecto de demo. La vista inicial muestra tres enlaces: Formulario con los tres campos, formulario con dos campos pero precio en la url y formulario con dos campos. Al hacer submit del formulario se nos muestran los detalles del producto entrado. Veréis como en el último caso (formulario con dos campos) aparece el valor de precio 999 que es el que se ha sacado del web.config (en el primer caso tiene preferencia el valor entrado por el usuario y en el segundo el valor de la url).
                            </p>
                            
                            <p>
                              Os podeis <a href="http://cid-6521c259e9b1bec6.skydrive.live.com/self.aspx/BurbujasNet/ZipsPosts/ValueProvidersMvc.zip" target="_blank" rel="noopener noreferrer">descargar el proyecto desde aquí</a> (enlace a mi skydrive).
                            </p>
                            
                            <p>
                              Espero que te haya quedado más o menos clara la idea de los value providers… en un post próximo hablaré de los ModelBinders para que podamos tener la foto completa!
                            </p>
                            
                            <p>
                              Saludos!
                            </p>