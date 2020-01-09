---
title: 'Opinión: De los atributos y su uso…'
description: 'Opinión: De los atributos y su uso…'
author: eiximenis

date: 2010-08-14T20:22:16+00:00
geeks_url: /?p=1530
geeks_visits:
  - 1324
geeks_ms_views:
  - 788
categories:
  - Uncategorized

---
Sí, ya sé: estamos en Agosto y lo que más seduce ahora mismo es darse un bañito en la playa y salir de copas a rebentar los mojitos del bar, así que los que podáis hacedlo sin dudar… Total, este post tampoco se largará a ninguna parte luego… 🙂

Los que no estéis de vacaciones o bien prefiráis leer geeks.ms en pleno Agosto (hay de todo en la viña del señor) a ver que os parece este post… es _mi_ opinión sobre el uso que se da a los atributos y los “problemas” que a mi parecer conlleva dicho uso. Dado que es un post de opinión vuestros comentarios sobre vuestras opiniones serán muy bien recibidos (de hecho siempre lo son, simplemente en este post lo serán más si cabe).

He estado observando en determinados casos un uso de los atributos que no me termina de convencer… Para aclarar conceptos entiendo por atributos aquellas clases que derivan de _[System.Attribute][1]_ y que usamos para decorar nuestras clases, métodos, propiedades… 

Voy a centrar este artículo de opinión en [Data Annotations][2], pero no es el único caso que tiene este “defecto”… 

En mi opinión los atributos deberían representar sólamente _metadatos_, es decir mera información, sin comportamiento alguno definido… O dicho de otro modo no deberían tener ningún método, sólo propiedades.

Cojamos el siguiente código de ejemplo, que es lo más simple del mundo: una clase, con una sola propiedad decorada con [Required]. Dicho atributo indica que la propiedad que se decora con él debe tener un valor:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">class</span> Foo<br />{<br />    [Required]<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Bar { get; set; }<br />}<br /></pre>
  
  <p>
    </div> 
    
    <p>
      Bien, ahora dado el siguiente código:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">Foo foo = <span style="color: #0000ff">new</span> Foo();<br />foo.Bar = <span style="color: #0000ff">string</span>.Empty;<br /><span style="color: #0000ff">bool</span> b = ((RequiredAttribute)foo.GetType().<br />    GetProperty(<span style="color: #006080">"Bar"</span>).GetCustomAttributes(<span style="color: #0000ff">false</span>)[0]).<br />    IsValid(foo.Bar);<br /></pre>
      
      <p>
        </div> 
        
        <p>
          Cuál es el valor de b, en este punto?
        </p>
        
        <p>
          La respuesta rápida es… depende de la implementación del método IsValid que está en RequiredAttribute. En este caso el valor de IsValid es false, porque string.Empty asume que no es un valor válido.
        </p>
        
        <p>
          Y aquí es donde, a mi, me chirría todo un poco… Yo estoy marcando un valor como requerido (usando [Required]) pero no entiendo porque la implementación de Required decide <strong>que significa ser requerido dentro de mi aplicación</strong>. Es decir el mero de hecho de declararlo como requerido me obliga a una implementación <em>concreta</em> de que signfica “ser requerido”.
        </p>
        
        <p>
          En mi lógica de negocio requerido puede significar que no valga null (siendo string.Empty un valor válido, cosa que no acepta el RequiredAttribute). Me diréis que me puedo crear mis propios atributos y tendréis toda la razón del mundo pero entonces si aplico a la propiedad Bar de la clase Foo un atributo mío, estoy cambiando los metadatos de dicha clase… que pasa si la clase es externa o bien quiero usarla en distintos proyectos (donde el concepto de requerido puede ser diferente)?
        </p>
        
        <p>
          No se si sería mejor separar RequiredAttribute en dos clases: una que sea el atributo que marque el elemento como requerido y otro que sea “que significa que un elemento sea requerido (es decir el método IsValid)” (dejadme llamar AttributeHandler a estas clases por ponerles nombre). Obviamente el framework debería proporcionar un AttributeHandler asociado a cada atributo y debería darme un mecanismo para que yo pudiese indicar que en mi proyecto el AttributeHandler para la clase RequiredAttribute es la clase que yo quiera…
        </p>
        
        <p>
          No sé si me entiende por donde voy… 🙂
        </p>
        
        <p>
          Un (caluroso) saludo a todos!
        </p>
        
        <p>
          PD: Un efecto colateral de tener atributos con cierto comportamiento es que estos empiezan a tener dependencias a terceros componentes (imaginaros un atributo tipo [Log]… que usa para generar los ficheros de log?). Y dado que los atributos los crea el CLR eso hace difícil inyectarles dependencias… Si el código que tiene el comportamiento (p.ej. el que hace log) estuviese en otras clases que no se creasen automáticamente por el CLR sería más fácil inyectarles dependencias).
        </p>

 [1]: http://msdn.microsoft.com/en-us/library/system.attribute.aspx
 [2]: http://msdn.microsoft.com/en-us/library/dd901590(VS.95).aspx