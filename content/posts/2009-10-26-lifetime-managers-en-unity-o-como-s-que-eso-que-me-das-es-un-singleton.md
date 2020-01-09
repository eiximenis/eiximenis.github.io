---
title: Lifetime Managers en Unity o ¿como sé que eso que me das es un singleton?
description: Lifetime Managers en Unity o ¿como sé que eso que me das es un singleton?
author: eiximenis

date: 2009-10-26T17:31:51+00:00
geeks_url: /?p=1476
geeks_visits:
  - 1704
geeks_ms_views:
  - 1372
categories:
  - Uncategorized

---
Los que leais habitualmente mi blog (¡muchas gracias!) habreis visto que tengo [varias entradas sobre unity][1] el contenedor IoC de la gente de patterns & practices. En ellas he ido comentando varios aspectos más o menos avanzados del contenedor y de los patrones IoC associados.

En este post quiero hablaros un poco de los “lifetime managers”, objetos que le indican a Unity si cuando debe resolver un objeto debe crear uno nuevo o bien devolver uno existente.

Resumiendo **mucho** podemos afirmar que:

  1. Con RegisterType<IA, A>() lo que hacemos es registrar un mapeo de la interfaz IA a la clase A: cada vez que pidamos un objeto IA, usando Resolve<IA>(), el contenedor nos devolverá _un nuevo_ objeto A.
  2. Con RegisterInstance<IA>(IA instance) lo que hacemos es registrar un singleton de la interfaz IA. Cada vez que pidamos un objeto IA, el contenedor nos devolverá _el mismo_ objeto: el que hemos pasado como parámetro a RegisterInstance.

La realidad es, como casi siempre, un poco más compleja. Unity no distingue solamente los casos “crear un objeto cada vez” o “devolver siempre el mismo objeto”, sino que la decisión de si se debe crear un objeto nuevo o no se deriva en otra clase: el lifetime manager. De serie con Unity vienen 3 lifetime managers distintos:

  * TransientLifetimeManager: Cada vez que tengamos que devolver un objeto crearemos _uno nuevo._
  * ContainerControlledLifetimeManager: Devolveremos siempre el mismo objeto (singleton).
  * PerThreadLifetimeManager: Devolveremos siempre el mismo objeto, pero crearemos un objeto para cada thread (singleton a nivel de thread).
  * ExternallyControlledLifetimeManager: Cada vez que tengamos que devolver un objeto devolveremos el mismo, si este sigue vivo (es decir el garbage collector no lo ha “recogido”). Obviamente Unity no mantiene una referencia al objeto sino una WeakReference, ya que en caso contrario el objeto estaría vivo “para siempre” dentro de Unity.

Tomemos la siguiente interfaz IA, y la clase A que la implementa:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">interface</span> IA<br />{<br />    Guid Id {get;}<br />}<br /><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> A : IA<br />{<br />    <span style="color: #0000ff">public</span> Guid Id {get; <span style="color: #0000ff">private</span> set;}<br />    <span style="color: #0000ff">public</span> A()<br />    {<br />        <span style="color: #0000ff">this</span>.Id = Guid.NewGuid();<br />    }<br />}</pre>
  
  <p>
    </div> 
    
    <p>
      Cada objeto A creado tendrá su propio Id único.
    </p>
    
    <p>
      Ahora miremos el siguiente código:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">IUnityContainer container = <span style="color: #0000ff">new</span> UnityContainer();<br /><br />container.RegisterType&lt;IA, A&gt;(<span style="color: #0000ff">new</span> ContainerControlledLifetimeManager());<br />IA a1 = container.Resolve&lt;IA&gt;();<br />IA a2 = container.Resolve&lt;IA&gt;();<br />Console.WriteLine(<span style="color: #006080">"a1: "</span> + a1.Id.ToString());<br />Console.WriteLine(<span style="color: #006080">"a2: "</span> + a2.Id.ToString());</pre>
      
      <p>
        </div> 
        
        <p>
          Si ejecutáis el siguiente código observareis que el ID es el mismo: Unity nos ha devuelto el mismo objeto para las dos llamadas a container.Resolve<IA>(). Esto ha sido porque hemos especificado un ContainerControlledLifetimeManager como parámetro a la llamada RegisterType.
        </p>
      </p>
      
      <p>
        Si ahora modificamos el ContainerControlledLifetimeManager por un ExternallyControlledLifetimeManager el resultado es el mismo: ambos Resolve reciben el mismo objeto.
      </p>
      
      <p>
        Ahora bien, si forzamos una recolección del Garbage Collector:
      </p>
      
      <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
        <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">IUnityContainer container = <span style="color: #0000ff">new</span> UnityContainer();<br />container.RegisterType&lt;IA, A&gt;(<span style="color: #0000ff">new</span> ExternallyControlledLifetimeManager());<br />IA a1 = container.Resolve&lt;IA&gt;();<br />Console.WriteLine(<span style="color: #006080">"a1: "</span> + a1.Id.ToString());<br />a1 = <span style="color: #0000ff">null</span>;      <span style="color: #008000">// Importante! Si no es null, el GC no puede recojer el objeto!</span><br />GC.Collect();<br />IA a2 = container.Resolve&lt;IA&gt;();<br />Console.WriteLine(<span style="color: #006080">"a2: "</span> + a2.Id.ToString());</pre>
        
        <p>
          </div> 
          
          <p>
            Ahora podemos observar como la segunda llamada a Resolve ha obtenido un objeto distinto al de la primera llamada (ya que el Garbage Collector ha eliminado el primer objeto).
          </p>
          
          <p>
            <strong>Crear nuestros propios lifetime managers</strong>
          </p>
          
          <p>
            Ahora que hemos visto que la realidad es un poco más divertida, la siguiente pregunta es: podemos crear nuestros propios lifetime managers? Y la respuesta es sí!
          </p>
          
          <p>
            Para ello simplemente debemos crearnos una clase que herede LifetimeManager y que implemente los métodos:
          </p>
          
          <ul>
            <li>
              SetValue: Que invoca Unity cuando ha creado el objeto. En este método podemos guardarnos el objeto creado.
            </li>
            <li>
              GetValue: Donde devolvemos el objeto o bien null, para que Unity cree uno de nuevo (y luego nos invoque SetValue).
            </li>
            <li>
              RemoveValue: Cuando se elimina un objeto
            </li>
          </ul>
          
          <p>
            Una nota importante sobre RemoveValue: Unity nunca llama a este método, está ahí para que nosotros podamos eliminar objetos de Unity, siempre y cuando tengamos acceso al Lifetime manager.
          </p>
          
          <p>
            Veamos un posible ejemplo de un Lifetime manager:
          </p>
          
          <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
            <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> CustomLifetimeManager : LifetimeManager<br />  {<br />      <span style="color: #0000ff">private</span> <span style="color: #0000ff">object</span>[] values;<br />      <span style="color: #0000ff">int</span> idx = 0;<br /><br />      <span style="color: #0000ff">public</span> CustomLifetimeManager(<span style="color: #0000ff">int</span> instances)<br />      {<br />          values = <span style="color: #0000ff">new</span> <span style="color: #0000ff">object</span>[instances];<br />          idx = -1;<br />      }<br /><br />      <span style="color: #0000ff">public</span> <span style="color: #0000ff">override</span> <span style="color: #0000ff">void</span> SetValue(<span style="color: #0000ff">object</span> newValue)<br />      {<br />          values[idx] = newValue;<br />      }<br /><br />      <span style="color: #0000ff">public</span> <span style="color: #0000ff">override</span> <span style="color: #0000ff">object</span> GetValue()<br />      {<br />          idx = (idx + 1) % values.Length;<br />          <span style="color: #0000ff">object</span> <span style="color: #0000ff">value</span> = values[idx];<br />          <span style="color: #0000ff">return</span> <span style="color: #0000ff">value</span>;<br />      }<br /><br />      <span style="color: #0000ff">public</span> <span style="color: #0000ff">override</span> <span style="color: #0000ff">void</span> RemoveValue()<br />      {<br />          <span style="color: #0000ff">object</span> <span style="color: #0000ff">value</span> = values[idx];<br />          values[idx] = <span style="color: #0000ff">null</span>;<br />          idx = (idx + 1) % values.Length;<br />          <span style="color: #0000ff">if</span> (<span style="color: #0000ff">value</span> <span style="color: #0000ff">is</span> IDisposable)<br />          {<br />              ((IDisposable)<span style="color: #0000ff">value</span>).Dispose();<br />          }<br />      }<br />  }</pre>
            
            <p>
              </div> 
              
              <p>
                El código se comenta casi solo, no? Este lifetime manager guarda x instancias del objeto, eso significa que Unity nos devolverá hasta x objetos distintos, y luego empezará a repetirlos.
              </p>
              
              <p>
                P.ej. dado el siguiente código:
              </p>
              
              <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">IUnityContainer container = <span style="color: #0000ff">new</span> UnityContainer();<br />CustomLifetimeManager lft = <span style="color: #0000ff">new</span> CustomLifetimeManager(3);<br /><br />container.RegisterType&lt;IA, A&gt;(lft);<br /><br />IA a1 = container.Resolve&lt;IA&gt;();<br />IA a2 = container.Resolve&lt;IA&gt;();<br />IA a3 = container.Resolve&lt;IA&gt;();<br /><span style="color: #008000">// Repes!</span><br />IA a4 = container.Resolve&lt;IA&gt;();<br />IA a5 = container.Resolve&lt;IA&gt;();<br />IA a6 = container.Resolve&lt;IA&gt;();</pre>
                
                <p>
                  </div> 
                  
                  <p>
                    Hemos configurado nuestro CustomLifetimeManager para que nos de hasta tres objetos distintos. Los tres primeros Resolve recibirán cada uno un objeto nuevo distinto… pero luego el cuarto resolve recibirá de nuevo el primer objeto, el quinto recibirá el segundo y así sucesivamente: hemos creado un <em>pool</em> de objetos.
                  </p>
                  
                  <p>
                    Como veis es realmente fácil, crearos vuestros propios lifetime managers, lo que os permite personalizar al máximo cuando Unity debe crear un objeto nuevo o devolver uno ya existente!
                  </p>
                  
                  <p>
                    Un saludo!
                  </p>

 [1]: http://geeks.ms/blogs/etomas/archive/tags/unity/default.aspx