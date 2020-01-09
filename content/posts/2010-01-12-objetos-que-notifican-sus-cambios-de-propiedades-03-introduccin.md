---
title: 'Objetos que notifican sus cambios de propiedades (0/3): Introducción'
description: 'Objetos que notifican sus cambios de propiedades (0/3): Introducción'
author: eiximenis

date: 2010-01-12T13:01:08+00:00
geeks_url: /?p=1486
geeks_visits:
  - 2330
geeks_ms_views:
  - 1178
categories:
  - Uncategorized

---
Hola a todos!!! Como ha ido la despedida del 2009 y la bienvenida del 2010!!! Espero que os hayáis portado bien y que los reyes os hayan traído muuuuchos regalitos!

En este post quiero dejar de lado la serie que estaba haciendo sobre facebook connect, para ver como, gracias a Unity, podemos crear objetos que nos notifiquen cuando cambian sus propiedades, sin que nosotros debamos añadir (casi) ningún código adicional!

Pienso que es un muy buen ejemplo del poder de usar un contenedor IoC, además de resolver una situación que se da muchas veces: quiero enterarme de los cambios sobre las propiedades de un objeto, pero no quiero codificar dicho objeto de ninguna forma especial (es decir, no poner ningún tipo de código a la clase de cuyos cambios de propiedad deseo enterarme).

**Donde queremos llegar…**

Cuando hablamos de contenedores IoC, nos vienen a la cabeza dos grandes patrones: Dependency Injection y Service Locator… pero hay otra poderosísima razón para usarlos: las _intercepciones_. Esta capacidad permite “enchufar” código a los objetos en tiempo de ejecución, lo que permite añadir capacidades de <a href="http://en.wikipedia.org/wiki/Aspect-oriented_programming" target="_blank" rel="noopener noreferrer">AOP</a>.

En este caso vamos a configurar la intercepción de Unity, para enchufar código cada vez que se lea/modifique una propiedad de un objeto. Dicho código nos notificará la lectura o escritura de la propiedad.

Además vamos a hacerlo configurable, de forma que no recibamos notificaciones de todas las propiedades, sino sólo de aquellas que nos interesen! La configuración de què propiedades queremos recibir notificación vamos a tenerla en _otra_ clase. El objetivo es llegar a un código como el que sigue:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">[PropertyNotifier(<span style="color: #0000ff">typeof</span>(UfoNotifications))]<br /><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> Ufo<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">virtual</span> <span style="color: #0000ff">string</span> Name { get; set; }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">virtual</span> <span style="color: #0000ff">int</span> Edad { get; set; }<br />}</pre>
  
  <p>
    </div> 
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> UfoNotifications : PropertyNotifications&lt;Ufo&gt;<br />{<br />    <span style="color: #0000ff">public</span> UfoNotifications()<br />    {<br />        <span style="color: #0000ff">this</span>.OnProperty(x =&gt; x.Name).Set.Notify.WithParameters();<br />        <span style="color: #0000ff">this</span>.OnProperty(x =&gt; x.Name).Get.Notify.WithoutParameters();<br />        <span style="color: #0000ff">this</span>.OnProperty(x =&gt; x.Edad).Get.Notify.WithoutParameters();<br />    }<br />}</pre>
      
      <p>
        </div> 
        
        <p>
          El primer código nos define una clase Ufo. Es una clase totalmente normal, salvo por dos detalles:
        </p>
        
        <ol>
          <li>
            El atributo PropertyNotifier nos indica que queremos recibir notificaciones cuando se lean/modifiquen propiedades de dicha clase, y indica que la configuración sobre cuales son las propiedades que deseamos notificar se encuentra en la clase <em>UfoNotifications</em>
          </li>
          <li>
            Todas las propiedades son virtuales.
          </li>
        </ol>
        
        <p>
          El segundo código contiene la configuración sobre qué propiedades y qué notificaciones queremos recibir. En este caso usamos una aproximación tipo <a href="http://en.wikipedia.org/wiki/Fluent_interface" target="_blank" rel="noopener noreferrer"><em>fluent interface</em></a><em>&#160;</em>cuya ventaja principal es que el código es mucho más fácil de leer (y de escribir).
        </p>
        
        <p>
          Esta serie va a constar de tres posts que publicaré en breve (a medida que publique los posts iré modificando éste para añadir los enlaces):
        </p>
        
        <ol>
          <li>
            <a href="http://geeks.ms/blogs/etomas/archive/2010/01/13/objetos-que-notifican-sus-cambios-de-propiedades-1-3-la-intercepci-243-n.aspx" target="_blank" rel="noopener noreferrer">Como configurar el mecanismo de intercepción de Unity para enchufar nuestro código</a> <strong>(Añadido el 13/01/2009)</strong>.
          </li>
          <li>
            <a href="http://geeks.ms/blogs/etomas/archive/2010/01/14/objetos-que-notifican-sus-cambios-de-propiedades-2-3-publish-and-subscribe.aspx" target="_blank" rel="noopener noreferrer">Como podemos notificar los cambios de propiedades, sin obligar a que quien quiera recibirlos tenga que tener una referencia al objeto que los notifica</a> (lo que nos impide usar eventos tradicionales de .NET) <strong>(Añadido el 14/01/2009)</strong>.
          </li>
          <li>
            Como crear la <em>fluent interface</em>&#160; para configurar las notificacioes
          </li>
        </ol>
        
        <p>
          <strike>No voy a adjuntar código en cada post porque sólo tengo la solución completa implementada… en el último post adjuntaré <strong>todo</strong> el código, junto con un ejemplo.</strike> Al final sí que adjunto un zip en cada post 🙂
        </p>
        
        <p>
          Nos leemos!
        </p>