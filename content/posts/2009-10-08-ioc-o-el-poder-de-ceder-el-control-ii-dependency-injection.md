---
title: 'IoC o el poder de ceder el control (ii): Dependency Injection'

author: eiximenis

date: 2009-10-08T12:11:04+00:00
geeks_url: /?p=1471
geeks_visits:
  - 3858
geeks_ms_views:
  - 1301
categories:
  - Uncategorized

---
Hace ya algún tiempecillo publiqué por aquí un post sobre [IoC][1], titulado [IoC o el poder de ceder el control][2]. En el post mencionaba dos de los patrones clásicos asociados con IoC, el _service locator_ y la inyección de dependencias (_dependency injection_), pero luego sólo me centraba en Service Locator. Un par de comentarios en dicho post decían si era posible algo similar pero explicando la inyección de dependencias, así que a ello vamos 😉

<!--more-->

**Dependencias de una clase**

Para entender como funciona la inyección de dependencias tenemos que tener claro que entendemos por dependencias de una clase: Básicamente una clase tiene dependencias con todas las otras clase que _utilice_, ya sea reciba objetos de dicha clase como parámetros, los devuelva como valores de retorno o cree variables locales o de clase.

Las dependencias no son nada malo y de hecho no son evitables: es evidente que las clases cooperan unas con otras para realizar alguna acción conjunta, así que es lógico que nuestro código depende de otras clases. Lo que debe preocuparnos es el _acoplamiento_ de nuestro código con estas dependencias, o dicho de otro modo: cuanto nos costaría cambiar _nuestra_ clase para que en lugar de depender de una clase X, dependiese de _otra_ clase Y que ofrece la misma funcionalidad. Si has de modificar muchas líneas de código es que tienes un alto acoplamiento (y eso sí que es malo). El objetivo de la inyección de dependencias es facilitarte conseguir un acoplamiento lo más bajo posible.

**Alto acoplamiento: uso directo de clases**

El nivel de acoplamiento mayor es cuando nuestros métodos trabajan con parámetros cuyo tipo es una clase:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> X<br />{<br />    <span style="color: #0000ff">public</span> MyLogClass Logger { get; <span style="color: #0000ff">private</span> set;}<br />    <span style="color: #0000ff">public</span> X (MyLogClass logger)<br />    {<br />        <span style="color: #0000ff">this</span>.Logger = logger;<br />    }<br />}</pre>
  
  <p>
    </div> 
    
    <p>
      La clase X tiene un alto acoplamiento con la clase MyLogClass. Si quisiéramos cambiar MyLogClass por otra clase distinta, llamésmole MyLogClass2 que tenga la misma funcionalidad deberemos modificar la clase X para que la propiedad Logger sea de tipo MyLogClass, asi como modificar el constructor… Parece sencillo, pero tened en cuenta que nuestra clase X será llamada por varias clases distintas. Todas las clases que crean un objeto de X, crearán un objeto MyLogClass para pasarlo como parámetro al constructor: deberemos cambiar también todas estas clases.
    </p>
    
    <p>
      Un cambio que debería ser fácil y que debería afectar sólo a una clase, se convierte, por culpa de alto acoplamiento, en un cambio complejo, que afecta a multitud de clases.
    </p>
    
    <p>
      <strong>Acoplamiento medio: Interfaces</strong>
    </p>
    
    <p>
      Las interfaces ayudan solucionar el problema. Podemos definir la clase X para que trabaje con interfaces:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> X<br />{<br />    <span style="color: #0000ff">public</span> ILogger Logger { get; <span style="color: #0000ff">private</span> set;}<br />    <span style="color: #0000ff">public</span> X (ILogger logger)<br />    {<br />        <span style="color: #0000ff">this</span>.Logger = logger;<br />    }<br />}</pre>
      
      <p>
        </div> 
        
        <p>
          Ahora la clase X no tiene dependencia alguna con MyLogClass. Podemos utilizar MyLogClass, MyLogClass2 o cualquier clase que implemente ILogger.
        </p>
        
        <p>
          Pero el problema no está resuelto al 100%. Para crear objetos de la clase X, debemos pasarle en el constructor un objeto <em>de una clase</em> que implemente ILogger:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">MyLogClass logger = <span style="color: #0000ff">new</span> MyLogClass();<br />X x = <span style="color: #0000ff">new</span> X(logger);</pre>
          
          <p>
            </div> 
            
            <p>
              Es decir la clase X no depende de MyLogClass, pero todas aquellas clases que crean objetos de la clase X sí, ya que deben crear un MyLogClass para pasarlo como parámetro al constructor. De nuevo modificar MyLogClass por MyLogClass2 implica localizar todos aquellos sitios donde se crean objetos de X y modificarlo.
            </p>
            
            <p>
              <strong>Acoplamiento bajo: Interfaces + Factoría</strong>
            </p>
            
            <p>
              Llegados a este punto alguien puede tener la idea “<em>hey! porque no creamos una factoria de ILogger, que sea la responsable de crear los objetos?</em>”. Es una gran idea ya que <em>mueve</em> todas las dependencias a la clase en UN sólo sitio, la factoría:
            </p>
            
            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> <span style="color: #0000ff">class</span> ILoggerFactory<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">static</span> ILogger GetLogger()<br />    {<br />        <span style="color: #0000ff">return</span> <span style="color: #0000ff">new</span> MyLogClass();<br />    }<br />}<br /><br /><span style="color: #008000">// ... Luego en cualquier otro sitio ...</span><br />X x = <span style="color: #0000ff">new</span> X (ILoggerFactory.GetLogger());</pre>
              
              <p>
                </div> 
                
                <p>
                  Ahora si en lugar de querer usar MyLogClass queremos usar MyLogClass2 sólo debemos modificar la factoría.
                </p>
                
                <p>
                  Hey! Y todo eso <strong>sin</strong> usar IoC… entonces para que el post? Bueno… imagina que <em>por cualquier razón</em>, debes modificar el constructor de X para que acepte algún otro parámetro:
                </p>
                
                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> X<br />{<br />    <span style="color: #0000ff">public</span> X (ILogger log, IFormatter frm);    <br />}</pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Sigue imaginando que, por la razón que sea, no puedes seguir teniendo el constructor con un solo parámetro ILogger, ya que no puedes asignar ningún valor por defecto a IFormatter. Pues bien… en este caso de nuevo debes volver a localizar todas las llamadas al constructor de X y modificarlas para pasar el nuevo parámetro&#160; (que por supuesto sacarás de otra factoría que crearás).
                    </p>
                    
                    <p>
                      Por suerte no estamos en un callejón sin salida: la inyección de dependencias viene para solucionar este <em>pequeño</em> problema.
                    </p>
                    
                    <p>
                      <strong>Acoplamiento muy bajo: Inyección de dependencias</strong>
                    </p>
                    
                    <p>
                      La inyección de dependencias se basa en el mismo principio que la factoría: No creas tu los objetos directamente, sinó que delegas esta responsabilidad en alguien. La diferencia respecto a la factoría tradicional, es que este alguien es un contenedor de IoC, capaz de <em>crear</em> todos aquellos parámetros necesarios e inyectarlos en el constructor. Si añades un parámetro nuevo, apenas deberás hacer nada: el contenedor de IoC <em>automáticamente</em> sabrá inyectar este nuevo parámetro.
                    </p>
                    
                    <p>
                      Vamos a ver un ejemplo usando <a href="http://msdn.microsoft.com/en-us/library/dd203104.aspx">Unity</a>, el contenedor IoC de la gente de Patterns & Practices. Primero comento muy rápidamente los conceptos básicos de Unity.
                    </p>
                    
                    <p>
                      La gracia está en <em>mapear</em> un tipo a una interfaz, con esto le decimos al contenedor que cuando pidamos objetos de una interfaz nos devuelva objetos de una clase determinada. Esto en Unity se consigue con el método RegisterType:
                    </p>
                  </p>
                  
                  <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                    <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">IUnityContainer container = <span style="color: #0000ff">new</span> UnityContainer();<br />container.RegisterType&lt;ILogger, MyLogClass&gt;();</pre>
                    
                    <p>
                      </div>
                    </p>
                    
                    <p>
                      Cuando pidamos un ILogger, Unity nos devolverá un MyLogClass… Y como le pedimos a Unity un ILogger? Pues usando el método Resolve:
                    </p></p> 
                    
                    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">ILogger logger = container.Resolve&lt;ILogger&gt;();</pre>
                      
                      <p>
                        </div>
                      </p>
                      
                      <p>
                        Hasta aquí todo muy parecido a la factoría. Ahora viene lo bueno: Si tenemos mappings registrados en Unity para las interfaces, Unity puede inyectar estos mappings en cualquier constructor. Es decir:
                      </p></p> 
                      
                      <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                        <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">container.RegisterType&lt;ILogger, MyLogClass&gt;();<br />container.RegisterType&lt;IFormatter, MyFormatClass&gt;();<br />X x = container.Resolve&lt;X&gt;();</pre>
                        
                        <p>
                          </div>
                        </p>
                        
                        <p>
                          Con las dos primeras líneas hemos configurado nuestro contenedor de IoC para que sepa que devolver cuando se le pida un ILogger y un IFormatter. Con la tercera línea estamos pidiendo un objeto de tipo X. Entonces Unity hace lo siguiente:
                        </p>
                        
                        <ol>
                          <li>
                            Mira si tiene algún mapeo que mapee X a una clase en concreto (en principio Unity no sabe que X es una clase y no una interfaz).
                          </li>
                          <li>
                            Al no tenerlo, deduce que es una clase y que debe crear un objeto de la clase X. Para ello inspecciona la clase X, y ve que el constructor requiere dos parámetros, un ILogger y un IFormatter. <ol>
                              <li>
                                Unity resuelve el primer parámetro
                              </li>
                              <li>
                                Unity resuelve el segundo parámetro
                              </li>
                              <li>
                                Unity pasa los valores de los dos parámetros resueltos al constructor de la clase X y devuelve el objeto X creado. Es decir, Unity <strong>inyecta</strong> los parámetros necesarios en el constructor.
                              </li>
                            </ol>
                          </li>
                        </ol>
                        
                        <p>
                          Es decir, lo que Unity <em>hace por nosotros</em> es equivalente a si hubieramos hecho:
                        </p>
                        
                        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">ILogger p1 = container.Resolve&lt;ILogger&gt;();<br />IFormatter p2 = container.Resolve&lt;IFormatter&gt;();<br />X x = <span style="color: #0000ff">new</span> X(p1, p2);</pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              Llegados a este punto… si se modificase el constructor de la clase X para hacer aparecer un tercer parámetro… tan sólo debemos hacer <strong>una</strong> modificación en nuestro código: Donde configuramos el contenedor de Unity, poner una llamada más a RegisterType(), para que Unity sepa que devolver cuando se encuentre un parámetro de dicho tipo. Pero dado que en nuestro códgo siempre obtenemos instancias de X llamando a container.Resolve<X>(), no deberemos modificar <strong>ninguna</strong> línea más de nuestro código.
                            </p>
                            
                            <p>
                              Llegados a este punto, comentaros dos cosillas:
                            </p>
                            
                            <ol>
                              <li>
                                Lo que hemos visto se llama inyección de dependencias en el constructor, y es uno de los mecanismos más normales. Otra forma común de inyectar dependencias es usar propiedades: es decir, el contenedor IoC al crear el objeto, inyecta valores en todas las propiedades que nosotros le indiquemos.
                              </li>
                              <li>
                                Que ocurre en aquellos objetos que por cualquier razón NO pueden ser creados por el contenedor (p.ej. objetos que un framework cree por nostros)? Podemos hacer que estos objetos reciban inyección de dependencias?
                              </li>
                            </ol>
                            
                            <p>
                              La respuesta al punto (2) la da el punto (1): Podemos utilizar inyección de dependencias en propiedades y luego, una vez tenemos el objeto ya creado, decirle al contenedor IoC que inyecte las dependencias pendientes. Esto en Unity se hace mediante el método BuildUp, que toma un objeto e inyecta las dependencias pendientes. Por ejemplo imaginad que deserializamos un objeto y queremos que el objeto deserializado reciba dependencias del contenedor de IoC. Es evidente que no podemos poner las dependencias en el constructor (porque no controlamos <em>quien</em> crea el objeto), pero podemos poner las dependencias en propiedades y una vez tenemos el objeto indicarle al contenedor de IoC que inyecte dichas propiedades. Esto en Unity se consigue mediante el método BuildUp.
                            </p>
                            
                            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> X<br />{<br />    <span style="color: #008000">// Código variado...</span><br />    <br />    <span style="color: #008000">// El atributo Dependency indica a Unity que la propiedad debe</span><br />    <span style="color: #008000">// ser inyectada</span><br />    [Dependency()]<br />    [XmlIgnore()]<br />    <span style="color: #0000ff">public</span> ILogger Logger { get; set;}    <br />}<br /><br /><span style="color: #008000">// En cualquier otro sitio...</span><br /><br />X x = <span style="color: #0000ff">null</span>;<br />XmlSerializer ser = <span style="color: #0000ff">new</span> XmlSerializer(<span style="color: #0000ff">typeof</span>(X));<br />x = (X)ser.Deserialize(myStream);<br /><span style="color: #008000">// Inyectamos las propiedades</span><br />container.BuildUp(x);</pre>
                              
                              <p>
                                </div> 
                                
                                <p>
                                  Cuando leemos el stream myStream, el propio XmlSerializer nos crea un objeto de la clase X. Luego al llamar al método BuildUp, es cuando el contenedor de IoC inyectará las propiedades.
                                </p>
                                
                                <p>
                                  Así es como funciona (más o menos :p) la inyección de dependencias.
                                </p>
                                
                                <p>
                                  Un saludo a todos!
                                </p>

 [1]: http://es.wikipedia.org/wiki/Inversi%C3%B3n_de_Control
 [2]: http://geeks.ms/blogs/etomas/archive/2008/10/28/ioc-o-el-poder-de-ceder-el-control.aspx