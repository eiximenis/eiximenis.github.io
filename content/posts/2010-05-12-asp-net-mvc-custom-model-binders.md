---
title: 'ASP.NET MVC: Custom Model Binders'

author: eiximenis

date: 2010-05-12T15:13:00+00:00
geeks_url: /?p=1510
geeks_visits:
  - 3618
geeks_ms_views:
  - 2438
categories:
  - Uncategorized

---
Seguimos esa serie donde intentamos _bucear_ un poco por algunas interioridades de ASP.NET MVC, intentando ver como funcionan por dentro algunas de las características de ese framework tan apasionante como és ASP.NET MVC. Si <a target="_blank" href="/blogs/etomas/archive/2010/05/07/asp-net-mvc-valueproviders.aspx" rel="noopener noreferrer">en el primer post de la serie vimos lo que eran los value providers</a>__ y <a target="_blank" href="/blogs/etomas/archive/2010/05/10/asp-net-mvc-el-defaultmodelbinder.aspx" rel="noopener noreferrer">en el segundo post vimos como funcionaba el DefaultModelBinder</a>__ en el post de hoy veremos como podemos crear Model Binders propios (lo que a su vez, nos ayudará a entender todavía más como funciona el <a target="_blank" href="http://msdn.microsoft.com/en-us/library/system.web.mvc.defaultmodelbinder.aspx" rel="noopener noreferrer">DefaultModelBinder</a>).

<!--more-->

Bueno, para empezar dejemos claro un punto fundamental:

  * En ASP.NET MVC **no estamos limitados a un solo Model Binder**, podemos tener muchos model binders, de hecho uno **por cada tipo de modelo**. 

La clase estática <a target="_blank" href="http://msdn.microsoft.com/en-us/library/system.web.mvc.modelbinders.aspx" rel="noopener noreferrer">ModelBinders</a> es la que mantiene el conjunto de model binders que estén registrados en el sistema. Para registrar un Model Binder propio simplemente llamamos al método Add() de la colección _Binders_ expuesta por dicha clase:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">ModelBinders.Binders.Add(<span style="color: #0000ff">typeof</span>(FooClass), <span style="color: #0000ff">new</span> FooBinder());</pre>
  
  <p>
    </div> 
    
    <p>
      El método Add() espera el <em>tipo de modelo</em> y el binder a usar para objetos de dicho tipo. Si para un tipo de modelo <strong>no</strong> existe model binder definido, se usará el model binder predeterminado, cuyo valor por defecto es el <em>DefaultModelBinder </em>pero que también podemos cambiar:
    </p>
    
    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">ModelBinders.Binders.DefaultBinder = <span style="color: #0000ff">new</span> CustomModelBinder();</pre>
      
      <p>
        </div> 
        
        <p>
          <strong>1. Un ejemplo sencillo... un Model Binder para objetos simples</strong>
        </p>
        
        <p>
          Vamos a crear un Model Binder propio tan sencillo como (probablemente) inútil: un model binder propio para objetos de tipo string, que simplementa convierta los valores en mayúsculas.
        </p>
        
        <p>
          El trato del framework de ASP.NET MVC con los model binders es muy simple: la interfaz <a target="_blank" href="http://msdn.microsoft.com/en-us/library/system.web.mvc.imodelbinder.aspx" rel="noopener noreferrer">IModelBinder</a> que deben implementar todos los model binders define un sólo método: <em><a target="_blank" href="http://msdn.microsoft.com/en-us/library/system.web.mvc.imodelbinder.bindmodel.aspx" rel="noopener noreferrer">BindModel</a></em>. Como se las apañe el model binder internamente le da igual al framework. Por suerte para nosotros la clase <em>DefaultModelBinder</em> es muy extensible, de forma que cuando implementamos un model binder propio, lo más normal (aunque <strong>no es obligatorio</strong>) es derivar de dicha clase. Así que antes vamos a ver que es lo que hace el <em>DefaultModelBinder</em> cuando debe enlazar un modelo:
        </p>
        
        <p>
          &nbsp;<a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_6EC8BF21.png"><img height="301" width="402" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_2B310DFF.png" alt="Elace de un modelo con el DefaultModelBinder" border="0" title="Elace de un modelo con el DefaultModelBinder" style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" /></a>
        </p>
        
        <p>
          Nota:<strong> No </strong>estan todas las funciones que usa el CustomModelBinder (faltan las relacionadas con la validación del modelo, pero no quiero hablar hoy de validaciones).
        </p>
        
        <p>
          <strong>Todas</strong> estas funciones son virtuales y por lo tanto pueden ser redefinidas en clases derivadas. Aquí tienes una breve descripción de cada método:
        </p>
        
        <ul>
          <li>
            BindModel: El único método de IModelBinder, su responsabilidad es devolver el modelo creado y enlazado.
          </li>
          <li>
            CreateModel: Crea una instancia del modelo
          </li>
          <li>
            GetTypeDescriptor: Obtiene la información del tipo del modelo
          </li>
          <li>
            GetModelProperties: Obtiene información sobre las propiedades del modelo
          </li>
          <li>
            BindProperty: Método que enlaza <em>una propiedad</em> concreta del modelo
          </li>
          <li>
            GetPropertyValue: Vista en el post anterior, obtiene el valor de una propiedad. Para ello usará el model binder asociado al tipo de la propiedad y llamará a su BindModel.
          </li>
          <li>
            SetProperty: Vista en el post anterior, establece el valor de una propiedad
          </li>
        </ul>
        
        <p>
          Volviendo a nuestro caso (un model binder que convierta las cadenas en mayúsculas) vamos a derivar de <em>DefaultModelBinder</em> y vamos a redefinir el método BindModel. Nos basta con este, puesto que un objeto string no tiene <em>propiedades</em> que se puedan enlazar, así que no se llamaría nunca a BindProperty de nuestro model binder: en BindModel vamos a hacer todo el trabajo:
        </p>
        
        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> StringBinder : DefaultModelBinder<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">override</span> <span style="color: #0000ff">object</span> BindModel(ControllerContext controllerContext, ModelBindingContext bindingContext)<br />    {<br />        <span style="color: #0000ff">object</span> o = <span style="color: #0000ff">base</span>.BindModel(controllerContext, bindingContext);<br />        <span style="color: #0000ff">return</span> o <span style="color: #0000ff">as</span> <span style="color: #0000ff">string</span> != <span style="color: #0000ff">null</span> ? ((<span style="color: #0000ff">string</span>)o).ToUpper() : o;<br />    }<br />}</pre>
          
          <p>
            </div> 
            
            <p>
              El código es trivial: llamamos a la implementación de BindModel del <em>DefaultModelBinder</em> (de esa manera todo el trabajo va a realizarlo el CustomModelBinder) y luego simplemente convertimos el resultado a mayúsculas.
            </p>
            
            <p>
              Sólo nos queda registrar nuestro model binder, usando la clase ModelBinders. Esto suele hacerse en Global.asax en el Application_Start:
            </p>
            
            <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
              <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">ModelBinders.Binders.Add(<span style="color: #0000ff">typeof</span>(<span style="color: #0000ff">string</span>), <span style="color: #0000ff">new</span> StringBinder());</pre>
              
              <p>
                </div> 
                
                <p>
                  Y listos... ahora <strong>cualquier</strong> cadena que se enlace será convertida a mayúsculas, pero atención! No es necesario que el controlador reciba una cadena: si la acción del controlador recibe <em>cualquier</em> clase que tenga una propieda de tipo <em>string</em>, dicha propiedad se enlazará usando nuestro model binder, por lo que dicha propiedad será convertida a mayúsculas.
                </p>
                
                <p>
                  <strong>2. Otro ejemplo, colecciones a partir de una sola cadena</strong>
                </p>
                
                <p>
                  Vamos ahora a crear un model binder para una clase de modelo específica. La clase es tal como sigue:
                </p>
                
                <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> Pedido<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Nombre {get; set;}<br />    <span style="color: #0000ff">public</span> IEnumerable&lt;<span style="color: #0000ff">string</span>&gt; Bebidas {get; set;}<br />}</pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Como ya sabemos, podemos enlazar colecciones con N campos en la petición que tengan el mismo nombre, o con N campos que tengan <em>nombre[0], nombre[1],..., nombre[N-1]</em>. Pero en este caso, vamos a crear una vista que tenga <strong>un solo campo</strong> que se llame <em>Bebidas</em>:
                    </p>
                    
                    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">&lt;</span><span style="color: #800000">form</span> <span style="color: #ff0000">action</span><span style="color: #0000ff">="/Pedido/Nuevo"</span> <span style="color: #ff0000">method</span><span style="color: #0000ff">="post"</span><span style="color: #0000ff">&gt;</span><br />       <span style="color: #0000ff">&lt;</span><span style="color: #800000">fieldset</span><span style="color: #0000ff">&gt;</span><br />           <span style="color: #0000ff">&lt;</span><span style="color: #800000">legend</span><span style="color: #0000ff">&gt;</span>Fields<span style="color: #0000ff">&lt;/</span><span style="color: #800000">legend</span><span style="color: #0000ff">&gt;</span><br />           <span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">class</span><span style="color: #0000ff">="editor-label"</span><span style="color: #0000ff">&gt;</span><br />               <span style="color: #0000ff">&lt;</span><span style="color: #800000">label</span> <span style="color: #ff0000">for</span><span style="color: #0000ff">="Nombre"</span><span style="color: #0000ff">&gt;</span>Nombre<span style="color: #0000ff">&lt;/</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span><br />           <span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br />           <span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">class</span><span style="color: #0000ff">="editor-field"</span><span style="color: #0000ff">&gt;</span><br />               <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="Nombre"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="Nombre"</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">=""</span> <span style="color: #0000ff">/&gt;</span><br />           <span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br />           <span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">class</span><span style="color: #0000ff">="editor-label"</span><span style="color: #0000ff">&gt;</span><br />               <span style="color: #0000ff">&lt;</span><span style="color: #800000">label</span> <span style="color: #ff0000">for</span><span style="color: #0000ff">="Bebidas"</span><span style="color: #0000ff">&gt;</span>Bebidas<span style="color: #0000ff">&lt;/</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span><br />           <span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br />           <span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span> <span style="color: #ff0000">class</span><span style="color: #0000ff">="editor-field"</span><span style="color: #0000ff">&gt;</span><br />               <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="Bebidas"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="Bebidas"</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">=""</span> <span style="color: #0000ff">/&gt;</span><br />           <span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br />       <span style="color: #0000ff">&lt;/</span><span style="color: #800000">fieldset</span><span style="color: #0000ff">&gt;</span><br />       <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="submit"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="Pedir"</span> <span style="color: #0000ff">/&gt;</span><br />   <span style="color: #0000ff">&lt;/</span><span style="color: #800000">form</span><span style="color: #0000ff">&gt;</span><br /></pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          Fijaos que sólo tenemos un <input type=&rdquo;text&rdquo; name=&rdquo;Bebidas&rdquo;>. Cuando hacemos submit del formulario los datos de la petición POST quedan de la siguiente forma:
                        </p>
                        
                        <p>
                          <code>Content-Type: application/x-www-form-urlencoded &lt;br /></code><code>Content-Length: 51 &lt;br /></code><code>Nombre=eiximenis&&lt;span style="color: #ff0000;">&lt;strong>Bebidas=coca-cola%2C+fanta%2C+agua&lt;/strong>&lt;/span></code>
                        </p>
                        
                        <p>
                          Fijaos que existe un solo campo POST llamado Bebidas, cuyo valor es la cadena &ldquo;coca-cola, fanta, agua&rdquo;. Si usáis del DefaultModelBinder para enlazar este modelo, el controlador recibirá un objeto <em>Pedido</em> cuyo nombre será correcto, y la propiedad Bebidas será un IEnumerable<string> <strong>con un solo</strong> elemento (con valor coca-cola, fanta, agua):
                        </p>
                        
                        <p>
                          <img height="88" width="370" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_226105B3.png" alt="image" border="0" title="image" style="border-right-width: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px" />
                        </p>
                        
                        <p>
                          <strong>Nota:</strong> Fíjate como el nombre <strong>se ha convertido a mayúsculas</strong>, ya que el DefaultModelBinder ha usado el StringBinder que hicimos antes para enlazar la propiedad Nombre que es de tipo String!
                        </p>
                        
                        <p>
                          Bueno... esperar que el DefaultModelBinder <em>entienda</em> que un campo separado por comas es realmente una lista de cadenas es esperar demasiado, así que vamos a hacer un CustomBinder que haga esto:
                        </p>
                        
                        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> PedidoBinder : DefaultModelBinder<br />{<br />    <span style="color: #0000ff">protected</span> <span style="color: #0000ff">override</span> <span style="color: #0000ff">object</span> GetPropertyValue(ControllerContext controllerContext, ModelBindingContext bindingContext, System.ComponentModel.PropertyDescriptor propertyDescriptor, IModelBinder propertyBinder)<br />    {<br />        <span style="color: #0000ff">object</span> <span style="color: #0000ff">value</span> = <span style="color: #0000ff">base</span>.GetPropertyValue(controllerContext, bindingContext, propertyDescriptor, propertyBinder);<br />        <span style="color: #0000ff">object</span> retVal = <span style="color: #0000ff">value</span>;<br />        <span style="color: #0000ff">if</span> (propertyDescriptor.Name == <span style="color: #006080">"Bebidas"</span> && <span style="color: #0000ff">value</span> <span style="color: #0000ff">as</span> IEnumerable&lt;<span style="color: #0000ff">string</span>&gt; != <span style="color: #0000ff">null</span>)<br />        {<br />            retVal = ((IEnumerable&lt;<span style="color: #0000ff">string</span>&gt;)<span style="color: #0000ff">value</span>).First().Split(<span style="color: #0000ff">new</span> <span style="color: #0000ff">char</span>[] { <span style="color: #006080">','</span> }, StringSplitOptions.RemoveEmptyEntries);<br />        }<br />        <span style="color: #0000ff">return</span> retVal;<br />    }<br />}</pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              El código es muy simple... A diferencia del caso anterior, ahora redefinimos el método <em>GetPropertyValue </em>(dado que la clase <em>Pedido</em> si que tiene propiedades). Simplemente miramos si estamos obteniendo la propiedad &ldquo;Bebidas&rdquo; y si es el caso:
                            </p>
                            
                            <ol>
                              <li>
                                Cogemos el valor que ha obtenido el <em>DefaultModelBinder</em>
                              </li>
                              <li>
                                Lo convertimos a IEnumerable<string>, ya que sabemos que se trata de un IEnumerable<string> con una sola cadena
                              </li>
                              <li>
                                Sobre esa cadena, devolvemos el resultado de hacer el Split (lo que devuelve un array, que a su vez es otro IEnumerable<string>).
                              </li>
                            </ol>
                            
                            <p>
                              Y listos, ahora sí que el enlace se realiza correctamente:
                            </p>
                            
                            <p>
                              <img height="121" width="374" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_283BA94C.png" alt="image" border="0" title="image" style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" />
                            </p>
                            
                            <p>
                              Bien, vamos a ver ahora <em>otra</em> manera en como podríamos codificar <strong>ese mismo</strong> model binder: vamos a implementar directamente la interfaz <em>IModelBinder</em> (no es lo mejor en este caso, pero vamos a aprender algunas cosillas más haciendolo).
                            </p>
                            
                            <p>
                              <strong>3. Implementando IModelBinder</strong>
                            </p>
                            
                            <p>
                              Insisto: En muchas ocasiones es mejor derivar de DefaultModelBinder en lugar de implementar directamente la interfaz IModelBinder (para aprovechar parte de lo que el DefaultModelBinder ya hace por nosotros).
                            </p>
                            
                            <p>
                              Aquí tenemos una posible implementación:
                            </p>
                            
                            <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                              <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> PedidoBinderInterfaz : IModelBinder<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">object</span> BindModel(ControllerContext controllerContext, ModelBindingContext bindingContext)<br />    {<br />        Pedido pedido = <span style="color: #0000ff">new</span> Pedido();<br /><br />        pedido.Nombre = bindingContext.ValueProvider.GetValue(<span style="color: #006080">"Nombre"</span>).AttemptedValue <span style="color: #0000ff">as</span> <span style="color: #0000ff">string</span>;<br />        IEnumerable&lt;<span style="color: #0000ff">string</span>&gt; bebidas = bindingContext.ValueProvider.GetValue(<span style="color: #006080">"Bebidas"</span>).RawValue <span style="color: #0000ff">as</span> IEnumerable&lt;<span style="color: #0000ff">string</span>&gt;;<br />        <span style="color: #0000ff">if</span> (bebidas != <span style="color: #0000ff">null</span>)<br />        {<br />            pedido.Bebidas = ((IEnumerable&lt;<span style="color: #0000ff">string</span>&gt;)bebidas).First().Split(<span style="color: #0000ff">new</span> <span style="color: #0000ff">char</span>[] { <span style="color: #006080">','</span> }, StringSplitOptions.RemoveEmptyEntries);<br />        }<br /><br />        <span style="color: #0000ff">return</span> pedido;<br />    }<br />}</pre>
                              
                              <p>
                                </div> 
                                
                                <p>
                                  Fijate que dado que sabemos que el modelo es de tipo <em>Pedido</em> (puesto que este model binder sólo se registra para objetos de tipo <em>Pedido</em>) podemos crear directamente un Pedido y enlazar sus dos propiedades (Nombre y Bebidas).
                                </p>
                                
                                <p>
                                  Y ahora lo que quería que vierais: Para <strong>preguntar a los value providers por el valor de un campo de la petición</strong>, se usa el método <em>GetValue</em> de la propiedad <em>ValueProvider</em> del <em>bindingContext</em>. La propiedad <em>ValueProvider</em> es un objeto de la clase <a target="_blank" href="http://msdn.microsoft.com/en-us/library/system.web.mvc.valueprovidercollection.aspx" rel="noopener noreferrer"><em>ValueProviderCollection</em></a><em>&nbsp;</em>pero nosotros lo recibimos como un <a target="_blank" href="http://msdn.microsoft.com/en-us/library/system.web.mvc.ivalueprovider.aspx" rel="noopener noreferrer">IValueProvider</a>. Eso es muy interesante: realmente el objeto es una colección de value providers, pero nosotros la vemos <strong>como un solo value provider </strong>y simplemente llamamos al método GetValue(). La propia clase se encarga de iterar por los value providers que contiene y encontrar el primero que nos pueda devolver el valor indicado.
                                </p>
                                
                                <p>
                                  Igual viendo el código pensáis que tampoco hay para tanto, que implementar la interfaz IModelBinder tampoco es tan complicado... bueno, fijaos en que lo que <strong>no</strong> hace este Model Binder y que si que hace el <em>DefaultModelBinder</em>:
                                </p>
                                
                                <ol>
                                  <li>
                                    No estamos <strong>validando</strong> el modelo... Siempre podemos meter el código de validación dentro de <em>BindModel </em>y tampoco seria muy costoso hacerlo, cierto... pero perdemos la capacidad de usar DataAnnotations p.ej. (Si queremos soportar DataAnnotations entonces si que debemos empezar a tirar código)
                                  </li>
                                  <li>
                                    No estamos usando los model binders concretos para las propiedades... me explico: si enlazáis un modelo <em>Pedido</em> con este model binder, la propiedad <em>Nombre</em> la estamos enlazando nosotros directamente, <strong>sin usar el StringBinder</strong> que teníamos registrado... efectivamente, el valor aparece en minúsculas:
                                  </li>
                                </ol>
                                
                                <p>
                                  <img height="121" width="370" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_672D29E7.png" alt="image" border="0" title="image" style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" />
                                </p>
                                
                                <p>
                                  Bueno... espero que os esté gustando esta serie de posts sobre temas &ldquo;internos&rdquo; de ASP.NET MVC... aún nos quedan varias cosas por destripar!!!
                                </p>
                                
                                <p>
                                  Un saludo!!
                                </p>
                                
                                <p>
                                  PD: El código de ejemplo lo podéis descargar <a target="_blank" href="http://cid-6521c259e9b1bec6.skydrive.live.com/self.aspx/BurbujasNet/ZipsPosts/CustomBinderMvc.zip" rel="noopener noreferrer">aquí</a> (link a mi skydrive).
                                </p>