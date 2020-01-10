---
title: 'Backbone: El misterioso caso del sync que no se lanzaba'

author: eiximenis

date: 2013-06-13T13:16:39+00:00
geeks_url: /?p=1645
geeks_visits:
  - 1201
geeks_ms_views:
  - 863
categories:
  - Uncategorized

---
Muy buenas! Estos días he estado resolviendo un misterio que me sucedía con un proyecto utilizando Backbone.

En concreto, se supone que, a partir de la versión 1.0, cuando se guarda un modelo de Backbone al servidor (usando p. ej. _save_) si la operación tiene éxito, el modelo nos lanza el evento _sync_ para informarnos, precisamente, del éxito de la operación.

Así, una secuencia típica de operaciones, se supone que es:

  1. El usuario hace click en un enlace, botón, o en cualquier elemento del DOM para que se “guarde” algo. 
  2. La vista de Backbone recoge este evento y hace lo que tenga que hacer (validaciones, modificaciones de UI en cliente) para terminar modificando el modelo según los datos de la UI. 
  3. Una vez el modelo está actualizado la vista llama a save() del modelo. 
  4. Alguien (usualmente la propia vista) recoge el _sync_ y hace lo que tenga que hacer (mostrar un mensaje, redirigir el usuario a otra acción del enrutador, etc). 

Esa es la idea, sencilla ¿no? Llamaas al método save() del modelo y este te lanza el evento sync cuando el guardado en el servidor ha sido completado.

Todo muy bonito **salvo que a mi no me iba**. No se me lanzaba el evento sync, a pesar de que la llamada al servidor se completaba y no lanzaba ningún error.

Aquí un pequeño inciso para los que no conozcáis Backbone: el método save() del modelo lo que hace es una petición (usualmente POST) a una url (que se especifica al definir la “clase” del modelo) con los datos a guardar. La idea es que Backbone se integre muy fácilmente con una API estilo REST que tengamos. En mi caso, era una API hecha con WebApi. Pongo aquí el código más relevante del método de servidor invocado:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #569cd6">void</span> <span style="color: white">PostOne</span>(<span style="color: #4ec9b0">TaskDto</span> <span style="color: white">task</span>)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #608b4e">// Guardar datos...</span>
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Repito: El método se invocaba correctamente y el objeto TaskDto era guardado. De hecho en la ventana de Network de Chrome tenía lo siguiente:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; display: inline; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_6F4D0284.png" width="484" height="237" />][1] 

Cuando ya estaba seguro (si es que se puede estarlo alguna vez) de que el error _no_ estaba en mi código, empecé a fijarme en la respuesta enviada al navegador. ASP.NET WebApi devuelve código HTTP 204 (No Content) en los métodos que devuelven void. Tiene toda la lógica y es un uso coherente de los códigos HTTP, el 204 significa precisamente esto: Ha ido todo bien (es un 2xx) pero no hay datos adicionales al respecto.

Entonces me surgió la duda… ¿Y si Backbone **no** entiende el 204? Por supuesto esto sería un “error” de Backbone, pero lo probé. Mi forma de probarlo, muy pedrestre, fue “obligar” a WebApi a devolver algún resultado. En este caso entonces ya no se usa el 204 (no puede usarse el 204 si hay contenido en la respuesta) si no el más genérico 200.

Modifiqué el método de WebApi:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #569cd6">string</span> <span style="color: white">PostOne</span>(<span style="color: #4ec9b0">TaskDto</span> <span style="color: white">task</span>)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #608b4e">// Guardar datos...</span>
  </p>
  
  <p style="margin: 0px">
    <p style="margin: 0px">
      <span style="color: #608b4e"></span>
    </p>
    
    <p>
      &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: #d69d85">""</span>;
    </p>
    
    <p style="margin: 0px">
      }
    </p>
  </p>
</div>

¡Genial! con eso ya tenía mi código de respuesta 200:

&#160;[<img title="image" style="border-left-width: 0px; border-right-width: 0px; border-bottom-width: 0px; display: inline; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_6C579DD1.png" width="484" height="106" />][2] 

Y…. ¡**funcionó!** El evento sync se me lanzaba tal y como se supone que debe ocurrir.

Luego pensé en otra cosa: “¿Y si no es el código 204 lo que BackBone no entiende si no una respuesta vacía?”… Para probar mi teoría modifiqué de nuevo el método _PostOne_:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #4ec9b0">HttpResponseMessage</span> <span style="color: white">PostOne</span>(<span style="color: #4ec9b0">TaskDto</span> <span style="color: white">task</span>)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #608b4e">// Guardar datos...</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: white">Request</span><span style="color: #b4b4b4">.</span><span style="color: white">CreateResponse</span>(<span style="color: #b8d7a3">HttpStatusCode</span><span style="color: #b4b4b4">.</span><span style="color: white">OK</span>);
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Ahora estaba mandando el código 200 pero sin nada (ni una cadena vacía) en la respuesta. Pues bien… **el evento sync no se lanzaba**. Es decir, el problema NO era usar un código distinto a 200, el problema para Backbone era una respuesta sin cuerpo. 

Así, que bueno… ya que Backbone parece insistir que quiere un cuerpo en&#160; la respuesta, al final terminé con esta implementación:

<div style="font-size: 10pt; font-family: consolas; background: #1e1e1e; color: #dcdcdc">
  <p style="margin: 0px">
    <span style="color: #569cd6">public</span> <span style="color: #4ec9b0">HttpResponseMessage</span> <span style="color: white">PostOne</span>(<span style="color: #4ec9b0">TaskDto</span> <span style="color: white">task</span>)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #608b4e">// Guardar datos...</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #569cd6">return</span> <span style="color: white">Request</span><span style="color: #b4b4b4">.</span><span style="color: white">CreateResponse</span>(<span style="color: #b8d7a3">HttpStatusCode</span><span style="color: #b4b4b4">.</span><span style="color: white">Created</span>, <span style="color: #569cd6">true</span>);
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

El código Created és el 201, que mira, me parecía un poco más correcto que usar el genérico 200, ya que realmente esta API siempre añadía una tarea.

Y así, todos contentos: Backbone ya tiene su cuerpo en la respuesta y yo mi evento sync que se lanzaba.

Así que recuerda: A Backbone no le gustan las respuestas sin contenido cuando se guarda el modelo.

¡Saludos!

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_46FDD030.png
 [2]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_5B536FEE.png