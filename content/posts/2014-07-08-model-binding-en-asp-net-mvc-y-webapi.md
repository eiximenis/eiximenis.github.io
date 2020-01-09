---
title: Model binding en ASP.NET MVC y WebApi
description: Model binding en ASP.NET MVC y WebApi
author: eiximenis

date: 2014-07-08T13:25:35+00:00
geeks_url: /?p=1675
geeks_visits:
  - 1630
geeks_ms_views:
  - 2615
categories:
  - Uncategorized

---
Una de las confusiones más habituales con la gente que viene de MVC y pasa a WebApi es que **el funcionamiento del model binding** (es decir rellenar los parámetros de los controladores a partir de los datos de la request) **es distinto entre ambos frameworks**. La confusión viene porque **a primera vista todo parece que funcione igual** pero realmente hay diferencias muy profundas entre ambos frameworks.

Veamos, pues, algunas de las diferencias que hay entre ambos frameworks 

**ASP.NET MVC Value Providers primero y Model Binders después**

En ASP.NET MVC el proceso de model binding está compuesto de dos pasos:

  1. Primero existe una serie de clases, llamadas _value providers_ que se encargan de leer la petición HTTP y colocar los datos _en un sitio común_ (como un diccionario). Así **no importa si un dato (p. ej. Id) está en querystring o en formdata).** Simplemente, se obtendrá ese Id y se colocará en el ese sitio común. Hay varios value providers porque cada value provider puede analizar una parte de la petición. Así hay un value provider que analiza la querystring, un par que analizan formdata (uno si el content-type es application/x-www-form-urlencoded y otro si es application/json) y así sucesivamente.
  2. Una vez los datos de la petición están en este lugar común entra en acción el model binder. El model binder recoge los datos de ese sitio común y los utiliza para crear los objetos que son pasados como parámetros en la acción del controlador. Cada parámetro es enlazado por un model binder distinto dependiendo del tipo del parámetro (aunque por defecto casi todos los tipos se enlazan usando el DefaultModelBinder, nosotros podemos crear model binders propios y vincularlos a tipos de parámetros).

Supongamos dos clases como las siguientes:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:0a1dc3c8-b34a-44cb-ad2c-5850fd4ed081" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Customer</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
             <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">int</span><span style="background:#1e1e1e;color:#dcdcdc"> Id { </span><span style="background:#1e1e1e;color:#569cd6">get</span><span style="background:#1e1e1e;color:#dcdcdc">; </span><span style="background:#1e1e1e;color:#569cd6">set</span><span style="background:#1e1e1e;color:#dcdcdc">; }</span>
        </li>
        <li style="background: #111111">
             <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> Name {  </span><span style="background:#1e1e1e;color:#569cd6">get</span><span style="background:#1e1e1e;color:#dcdcdc">; </span><span style="background:#1e1e1e;color:#569cd6">set</span><span style="background:#1e1e1e;color:#dcdcdc">; }</span>
        </li>
        <li>
             <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> Address{ </span><span style="background:#1e1e1e;color:#569cd6">get</span><span style="background:#1e1e1e;color:#dcdcdc">; </span><span style="background:#1e1e1e;color:#569cd6">set</span><span style="background:#1e1e1e;color:#dcdcdc">; }</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Product</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li style="background: #111111">
             <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">int</span><span style="background:#1e1e1e;color:#dcdcdc"> Id { </span><span style="background:#1e1e1e;color:#569cd6">get</span><span style="background:#1e1e1e;color:#dcdcdc">; </span><span style="background:#1e1e1e;color:#569cd6">set</span><span style="background:#1e1e1e;color:#dcdcdc">; }</span>
        </li>
        <li>
             <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> Name { </span><span style="background:#1e1e1e;color:#569cd6">get</span><span style="background:#1e1e1e;color:#dcdcdc">; </span><span style="background:#1e1e1e;color:#569cd6">set</span><span style="background:#1e1e1e;color:#dcdcdc">; }</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Y una vista que envíe los datos:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:fb8c636e-c647-4d97-ab8b-209b1fbc8cef" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">form</span><span style="background:#1e1e1e;color:#808080">></span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">input</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">type</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"text"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">name</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"Id"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#808080">/><</span><span style="background:#1e1e1e;color:#569cd6">br</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#808080">/></span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"><</spa n><span style="background:#1e1e1e;color:#569cd6">input</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">type</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"text"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">name</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"Name"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#808080">/><</span><span style="background:#1e1e1e;color:#569cd6">br</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#808080">/></span></li> 
          
          <li style="background: #111111">
                <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">input</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">type</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"text"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">name</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"Address"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#808080">/><</span><span style="background:#1e1e1e;color:#569cd6">br</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#808080">/></span>
          </li>
          <li>
                <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">input</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">type</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"submit"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">value</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"send"</span><span style="background:#1e1e1e;color:#808080">/></span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#808080"></</span><span style="background:#1e1e1e;color:#569cd6">form</span><span style="background:#1e1e1e;color:#808080">></span>
          </li></ol></div> </p></div> </p></div> 
          
          <p>
            Ahora si tenemos un controlador que tiene una acción con dos parámetros (Customer y Product) ¿qué es lo que recibimos? Pues lo siguiente:
          </p>
          
          <p>
            <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_509AC605.png"><img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_39031892.png" width="504" height="208" /></a>
          </p>
          
          <p>
            Es decir el valor de las propiedades Id y Name se ha enlazado a ambos parámetros. Es lógico: los value providers han recogido 3 valores de la petición resultado de enviar el formulario (Id, Name y Address) y los han colocado en el “sitio común”. Luego cuando el model binder del parámetro <em>customer</em> debe crear el objeto usa los valores de dicho sitio común para enlazar las propiedades… lo mismo que el model binder del parámetro <em>product</em>. De ahí que las propiedades que se llamen igual tengan el mismo valor.
          </p>
          
          <p>
            Por supuesto esta posibilidad está contemplada en ASP.NET MVC, de forma que el model binder entiende de “prefijos”. Así si modifico uno de los campos de texto para que sea:
          </p>
          
          <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:8a6de4ab-ca9e-41b5-8fbf-9371ca2e2b03" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
            <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
              <div style="background: #ddd; max-height: 300px; overflow: auto">
                <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
                  <li>
                    <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">input</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">type</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"text"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">name</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"customer.Name"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#808080">/></span>
                  </li>
                </ol>
              </div></p>
            </div></p>
          </div>
          
          <p>
            Ahora el valor del campo de la petición llamado “customer.Name” se enlazará solo a la propiedad Name del parámetro customer (product.Name será null).
          </p>
          
          <p>
            El modelo de value providers + model binders es muy versátil y potente.
          </p>
          
          <p>
            <strong>WebApi – Básicamente MediaTypeFormatters (básicamente)</strong>
          </p>
          
          <p>
            En WebApi la aproximación <strong>es radicalmente distinta</strong>. Primero <strong>hay una distinción clara, clarísima, sobre si el parámetro de la acción se enlaza desde el cuerpo de la petición o desde cualquier otro sitio (p. ej. querystring</strong>).
          </p>
          
          <p>
            En MVC el model binder no sabe de donde vienen los datos que usa para enlazar los parámetros del controlador, ya que los saca siempre del “sitio común” (donde lo dejan los value providers). WebApi usa una orientación distinta pero la regla de oro fundamental es: <strong>El cuerpo de la petición puede ser leído una sola vez</strong>.
          </p>
          
          <p>
            En efecto en WebApi el cuerpo de la petición HTTP es un stream forward-only, lo que significa que puede ser leído una única vez (en ASP.NET MVC el cuerpo está cacheado ya que distintos value providers pueden leerlo). Eso está hecho por temas de rendimiento.
          </p>
          
          <p>
            Así en WebApi se usa un paradigma basado básicamente en el content-type. Aparecen unos entes nuevos (los media type formatters) encargados de leer el cuerpo de la petición. Pero SOLO UN media type formatter puede leer el cuerpo (recuerda: puede ser leído una sola vez). Y como sabe WebApi qué media type formatter procesa el cuerpo de la petición? Pues basándose en el content-type.
          </p>
          
          <p>
            El media type formatter lee pues el cuerpo de la petición <strong>y rellena un objeto de .NET (del tipo indicado)</strong> en base a dichos datos. Efectivamente: el media type formatter lee de la petición y enlaza las propiedades. Eso implica una restricción importante: <strong>tan solo un parámetro de la acción puede ser obtenido a partir de los datos del cuerpo de una petición.</strong>
          </p>
          
          <p>
            Veamos la diferencia: he creado un controlador WebApi con un método análogo al anterior que recibe (via POST) un Customer y un Product:
          </p>
          
          <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:7de9e7ee-6b81-4dbe-9862-06e7104ddf48" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
            <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
              <div style="background: #ddd; max-height: 300px; overflow: auto">
                <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
                  <li>
                    <span style="background:#1
e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">DataController</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#4ec9b0">ApiController</span>
                  </li>
                  <li style="background: #111111">
                    <span style="background:#1e1e1e;color:#dcdcdc">{</span>
                  </li>
                  <li>
                        <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">bool</span><span style="background:#1e1e1e;color:#dcdcdc"> Post(</span><span style="background:#1e1e1e;color:#4ec9b0">Product</span><span style="background:#1e1e1e;color:#dcdcdc"> product, </span><span style="background:#1e1e1e;color:#4ec9b0">Customer</span><span style="background:#1e1e1e;color:#dcdcdc"> customer)</span>
                  </li>
                  <li style="background: #111111">
                        <span style="background:#1e1e1e;color:#dcdcdc">{</span>
                  </li>
                  <li>
                            <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">true</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
                  </li>
                  <li style="background: #111111">
                        <span style="background:#1e1e1e;color:#dcdcdc">}</span>
                  </li>
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc">}</span>
                  </li>
                </ol>
              </div></p>
            </div></p>
          </div>
          
          <p>
            Y he modificado la vista para que haga post del formulario a dicho controlador. Existe un media type formatter que se encarga de leer los datos cuando el content-type es application/x-www-form-urlencoded (de hecho, realmente hay dos pero tampoco es necesario entrar en más detalles). ¿Y cual es el resultado? Pues <strong>un error.</strong> Concretamente una System.InvalidOperationException con el mensaje: Can&#8217;t bind multiple parameters (&#8216;product&#8217; and &#8216;customer&#8217;) to the request&#8217;s content.
          </p>
          
          <p>
            La razón es que tenemos <strong>dos parámetros a rellenar basándonos en el cuerpo de la petición</strong>. Pero dicho cuerpo puede ser leído una sola vez por un media type formatter. Y un media type formatter tan solo puede enlazar un objeto.
          </p>
          
          <p>
            Es ahí donde entra el atributo FromUri. Dicho atributo se aplica a parámetros de la accion para indicar que esos parámetros se deben enlazar a partir de datos encontrados en la URL (querystring):
          </p>
          
          <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:41061d97-f979-4d10-b88b-7add6c4f0dc7" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
            <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
              <div style="background: #ddd; max-height: 300px; overflow: auto">
                <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">bool</span><span style="background:#1e1e1e;color:#dcdcdc"> Post(</span><span style="background:#1e1e1e;color:#4ec9b0">Product</span><span style="background:#1e1e1e;color:#dcdcdc"> product, [</span><span style="background:#1e1e1e;color:#4ec9b0">FromUri</span><span style="background:#1e1e1e;color:#dcdcdc">] </span><span style="background:#1e1e1e;color:#4ec9b0">Customer</span><span style="background:#1e1e1e;color:#dcdcdc"> customer)</span>
                  </li>
                  <li style="background: #111111">
                    <span style="background:#1e1e1e;color:#dcdcdc">{</span>
                  </li>
                  <li>
                        <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">true</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
                  </li>
                  <li style="background: #111111">
                    <span style="background:#1e1e1e;color:#dcdcdc">}</span>
                  </li>
                </ol>
              </div></p>
            </div></p>
          </div>
          
          <p>
            Si ahora ejecutamos de nuevo podemos ver que recibe el controlador son:
          </p>
          
          <p>
            <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_74567E50.png"><img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_28F6DA8C.png" width="504" height="124" /></a>
          </p>
          
          <p>
            Podemos ver como <em>customer</em> no tiene datos (a pesar de haber un campo Address en el cuerpo de la petición) porque customer se enlaza a partir de los datos de la URL (querystring).
          </p>
          
          <p>
            Por defecto WebApi usa siempre un media type formatter cuando el parámetro es un tipo complejo (una clase) a no ser que haya el atributo FromUri aplicado. Si el parámetro es un tipo simple (p. ej. int o un string) intenta enlazarlo a partir de la URL a no ser que haya aplicado el atributo contrario [FromBody]. Eso sí recuerda que <strong>solo un parámetro puede ser enlazado desde el cuerpo de la petición</strong>.
          </p>
          
          <p>
            <strong>WebApi – ModelBinders y Value Providers</strong>
          </p>
          
          <p>
            Cuando se enlaza un parámetro que no viene del cuerpo de la petición (es decir que viene de la URL) WebApi usa entonces el mismo modelo que MVC. Es decir primero los value providers recogen los datos de la petición (<strong>excepto el cuerpo</strong>) y los colocan en un sitio común y luego los model binders usan los datos de este sitio cómún para enlazar los parámetros).
          </p>
          
          <p>
            P. ej. modifico el controlador de WebApi para que acepte GET y enlace ambos parámetros desde la URL:
          </p>
          
          <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:111c6492-738e-4fbe-8d1a-9785bc3b8a00" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
            <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
              <div style="background: #ddd; max-height: 300px; overflow: auto">
                <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
                  <li>
                    <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">bool</span><span style="background:#1e1e1e;color:#dcdcdc"> Get([</span><span style="background:#1e1e1e;color:#4ec9b0">FromUri</span><span style="background:#1e1e1e;color:#dcdcdc">]</span><span style="background:#1e1e1e;color:#4ec9b0">Product</span><span style="background:#1e1e1e;color:#dcdcdc"> product, [</span><span style="background:#1e1e1e;color:#4ec9b0">FromUri</span><span style="background:#1e1e1e;color:#dcdcdc">] </span><span style="background:#1e1e1e;color:#4ec9b0">Customer</span><span style="background:#1e1e1e;color:#dcdcdc"> customer)</span>
                  </li>
                  <li style="background: #111111">
                    <span style="background:#1e1e1e;color:#dcdcdc">{</span>
                  </li>
                  <li>
                        <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">true</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
                  </li>
                  <li style="background: #111111">
                    <span style="background:#1e1e1e;color:#dcdcdc">}</span>
                  </li>
                </ol>
              </div></p>
            </div></p>
          </div>
          
          <p>
            Ahora si realizo una llamada GET a /api/Data/10?name=test lo que obtengo es:
          </p>
          
          <p>
            <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_288AA797.png"><im g title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_5D2B03D2.png" width="504" height="135" /></a>
          </p>
          
          <p>
            Los value providers recogen los datos de la URL (hay dos, uno para los route values (como /10) y otro para la querystring) y los dejan en el sitio común. Luego los model binders usan esos datos para enlazar tanto <em>product</em> como <em>customer</em> y es por eso que obtengo los datos duplicados (es el caso análogo al caso inicial de ASP.NET MVC).
          </p>
          
          <p>
            En resumen: hemos visto cuatro pinceladas de como ASP.NET MVC y WebApi enlazan los parámetros a los controladores haciendo especial énfasis en las diferencias. Dejamos para un post posterior el ver como funciona este tema en vNext pues recuerda que en vNext WebApi y MVC son un solo framework 🙂
          </p>
          
          <p>
            Saludos!
          </p>