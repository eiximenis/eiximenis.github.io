---
title: ¿MVP e IoC trabajando juntos? ¡Pues claro!

author: eiximenis

date: 2009-10-13T12:09:01+00:00
geeks_url: /?p=1472
geeks_visits:
  - 1879
geeks_ms_views:
  - 951
categories:
  - Uncategorized

---
Un comentario de Galcet en mi post “[Como independizar tu capa lógica de tu capa de presentación][1]” decía que el entendía por separado los conceptos de IoC y los de MVC pero que no veía como podían trabajar juntos… El motivo de este post es para comentar precisamente esto: no sólo cómo MVC e IoC pueden trabajar juntos sinó las ventajas que la combinación de ambos patrones nos aporta.

<!--more-->

Galcet no comentaba si se refería a aplicaciones desktop o web. En este post voy a tratar aplicaciones de escritorio (por lo que me centraré en el patrón MVP más que en el MVC dado que, en mi opinión, MVP aplica mejor que MVC en aplicaciones desktop). En aplicaciones web, si usamos ASP.NET MVC el tema se simplifica mucho, dado que ASP.NET MVC está “preparado” para que sea muy fácil crear nuestros controladores mediante cualquier contenedor IoC.

**La filosofía CAB + SCSF**

CAB (o [Composite UI Application Block][2]) es un framework para el desarrollo de aplicaciones de escritorio winforms con interfaz de usuario compleja. Se basa en el patrón MVP y se compone de varios assemblies y una guía de buenas prácticas. Aunque puede usarse sola, suele combinarse con SCSF ([Smart Client Software Factory][3]), un conjunto de guías, librerías y buenas prácticas para la creación de aplicaciones winforms complejas. La recomendación de SCSF para la interfaz de usuario es usar CAB, y de hecho _extiende_ CAB. No voy a hablar aquí ni de CAB ni de SCSF (hay varios tutoriales en internet), sino de como CAB y SCSF afrontan el uso de IoC junto con MVP.

La filosofía de SCSF es que nosotros creamos las vistas y las vistas crean a su presenter asociado. El código equivalente usando Unity sería algo parecido a:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> View : UserControl, IView<br />{    <br />    <span style="color: #0000ff">public</span> View ()    <br />    {        <br />        InitializeComponents();        <br />        <span style="color: #008000">// Resto de código...    </span><br />    }    <br />    <span style="color: #0000ff">private</span> IMyPresenter _presenter;<br />    [Dependency()]<br />    <span style="color: #0000ff">public</span> IMyPresenter Presenter<br />    {<br />        get { <span style="color: #0000ff">return</span> _presenter;}<br />        set<br />        {<br />            _presenter = <span style="color: #0000ff">value</span>;<br />            _presenter.View = <span style="color: #0000ff">this</span>;<br />        }<br />    }<br />}<br /><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> MyPresenter : IPresenter<br />{<br />    <span style="color: #0000ff">public</span> IView View { get; set;}    <br />}<br /></pre>
</div>

El contenedor debe tener registrado el mapping entre las interfaces y las clases:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">container.RegisterType&lt;IPresenter, MyPresenter&gt;();<br /><br />container.RegisterType&lt;IView, View&gt;();<br /><span style="color: #008000">// Creamos la vista</span><br />var view = container.Resolve&lt;IView&gt;();</pre>
  
  <p>
    </div> 
    
    <p>
      Al llamar al método Resolve, Unity consulta sus mappings y llega a la conclusión de que debe crear un objeto View. La clase View tiene una propiedad “Presenter” decorada con [Dependency()], por lo que Unity debe inyectar un valor a esta propiedad. La propiedad es de tipo IMyPresenter, Unity consulta sus mappings y vee que debe crear un objeto de la clase MyPresenter. Luego en el setter de la propiedad “Presenter” la vista se asigna a si misma como vista del presenter recién creado.
    </p>
    
    <p>
      A lo mejor alguien se pregunta porque no usamos inyección de dependencias en el constructor del presenter, es decir que en lugar de que nuestro presenter declare una propiedad que reciba la vista y rellenar esta propiedad desde la propia vista, declaramos la vista en el constructor y que Unity se encargue de todo:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> View : UserControl, IView<br />{        <br />    <span style="color: #0000ff">public</span> View ()        <br />    {                <br />        InitializeComponents();                <br />        <span style="color: #008000">// Resto de código...        </span><br />    }        <br />    <span style="color: #0000ff">private</span> IMyPresenter _presenter;    <br />    [Dependency()]    <br />    <span style="color: #0000ff">public</span> IMyPresenter Presenter { get; set;}<br />}<br /><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> MyPresenter : IPresenter<br />{    <br />    <span style="color: #0000ff">public</span> MyPresenter (IView view)<br />    {<br />        <span style="color: #008000">// Nos guardamos la vista...</span><br />    }<br />}<br /></pre>
      
      <p>
        </div> 
        
        <p>
          ¿Qué problema hay en este código? A priori puede parecer que ninguno, pero si repasamos como actuaría Unity:
        </p>
        
        <ol>
          <li>
            Al llamar a Resolve<IView> Unity consulta sus mappings y ve que debe crear un objeto de la clase View
          </li>
          <li>
            Al inyectar la propiedad Presenter, Unity consulta sus mappings y ve que debe crear un objeto de la clase MyPresenter
          </li>
          <li>
            Al intentar crear un objeto MyPresenter, Unity observa que el constructor recibe un IView y que debe inyectarlo.
          </li>
          <li>
            Así pues, Unity consulta sus mappings y <strong>crea un nuevo</strong> objeto View que inyectará en el constructor del MyPresenter.
          </li>
        </ol>
        
        <p>
          Suponiendo que todo esto no terminase en un Stack Overflow, en todo caso el objeto MyPresenter recibiría un objeto View distinto del que devolvería la llamada a Resolve.
        </p>
        
        <p>
          <strong>El “problema” de Visual Studio</strong>
        </p>
        
        <p>
          ¿Porque ha optado CAB por esta filosofía? ¿Porque las vistas crean a los presenters y no al revés? ¿Porque la inyección de dependencias es por propiedades y no en el constructor? La respuesta a todos estos interrogantes se llama Visual Studio.
        </p>
        
        <p>
          Me explico: si tenemos una vista (o sea un UserControl) llamada View, cuando la arrastramos dentro de un formulario, o de un panel Visual Studio genera un código parecido a:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">View view1 = <span style="color: #0000ff">new</span> View();<br />panel1.Controls.Add(view1);</pre>
          
          <p>
            </div> 
            
            <p>
              ¿Observáis el problema? Visual Studio ha creado una instancia de la vista, usando new, no usando el método Resolve del contenedor de IoC, por lo tanto nos podemos olvidar de la inyección de dependencias, por lo que efectivamente nuestro presenter no estará creado.
            </p>
            
            <p>
              ¿Como podemos solucionar este problema? Bueno… todos los controladores IoC permiten “inicializar” un objeto ya creado, entendiendo por “inicializar” inyectar todas las dependencias que este objeto necesita (en el caso de nuestras vistas, la propiedad “Presenter”). En el caso de Unity este método se llama BuildUp, y se le pasa la instancia del objeto a inicializar. Lo que debemos hacer es inicializar todos los controles que estén en el formulario, lo que podemos hacer en el método Main:
            </p>
            
            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">[STAThread]<br /><span style="color: #0000ff">static</span> <span style="color: #0000ff">void</span> Main()<br />{<br />    Application.EnableVisualStyles();<br />    Application.SetCompatibleTextRenderingDefault(<span style="color: #0000ff">false</span>);<br />    Form1 frm = <span style="color: #0000ff">new</span> Form1();<br />    IUnityContainer container = <span style="color: #0000ff">new</span> UnityContainer();<br />    frm.BuildUpControls(container);<br />    Application.Run(frm);<br />}</pre>
              
              <p>
                </div> 
                
                <p>
                  Donde el método “BuildUpControls” es un método de extensión definido de la siguiente manera:
                </p>
                
                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">class</span> FormExtensions<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">void</span> BuildUpControls (<span style="color: #0000ff">this</span> Control self,<br />        IUnityContainer container)<br />    {<br />        container.BuildUp(self);<br />        <span style="color: #0000ff">foreach</span> (Control control <span style="color: #0000ff">in</span> self.Controls)<br />        {<br />            control.BuildUpControls(container);<br />        }<br />    }<br />}</pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      El método BuildUpControls va recorriendo recursivamente la colección “Controls” para llamar al método “BuildUp” del contenedor con todos los controles creados. En este caso no miramos nada más, por lo que inicializamos todos los controles (incluso las labels, los textboxes…), lo que es excesivo. Un refinamiento es inicializar sólo aquellos controles que sean “vistas”. Por ejemplo, CAB para saber que un control es una “vista” y que debe ser inicializado obliga a decorarlo con el atributo [SmartPart].
                    </p>
                    
                    <p>
                      Evidentemente si nosotros mismos añadimos en run-time una vista debemos inicializarla “manualmente”:
                    </p>
                    
                    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">Subview view = <span style="color: #0000ff">new</span> Subview();<br />view.BuildUpControls(container);<br /><span style="color: #008000">// O bien...</span><br />Subview view = container.Resolve&lt;Subview&gt;();</pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          Eso mismo ocurre en CAB: si en CAB creamos una vista de forma programática también debemos “inicializarla”. CAB gestiona la inicialización de una forma totalmente distinta, pero la <em>filosofía</em> es la misma (que es lo que intento contar).
                        </p>
                        
                        <p>
                          Pues bueno… hemos visto como podemos combinar el uso de un contenedor IoC (como siempre en mi caso Unity :p) junto con el patrón MVP. Fijaos que los presenters si que están creados por Unity, por lo que pueden recibir dependencias inyectadas en el constructor (p.ej. a un servicio de log).
                        </p>
                        
                        <p>
                          Un saludo!
                        </p>

 [1]: http://geeks.ms/blogs/etomas/archive/2009/09/17/como-independizar-tu-capa-de-l-243-gica-de-tu-capa-de-presentaci-243-n.aspx
 [2]: http://msdn.microsoft.com/en-us/library/aa480450.aspx
 [3]: http://msdn.microsoft.com/en-us/library/aa480482.aspx