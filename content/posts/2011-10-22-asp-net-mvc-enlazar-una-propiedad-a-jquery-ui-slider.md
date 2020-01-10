---
title: 'ASP.NET MVC: Enlazar una propiedad a jQuery UI Slider'

author: eiximenis

date: 2011-10-22T10:32:13+00:00
geeks_url: /?p=1581
geeks_visits:
  - 3803
geeks_ms_views:
  - 1305
categories:
  - Uncategorized

---
¡Hola! Un compañero me ha preguntado si era posible enlazar una propiedad (de tipo int) a un control [slider de jQuery UI][1]. La verdad es que sí que es posible y vamos a ver en este post una posible solución que de hecho es extrapolable a otras situaciones parecidas que podáis tener.

<!--more-->

**Templated helpers al rescate**

En ASP.NET MVC2 introdujeron el concepto de _templated helpers_ un mecanismo para construir la interfaz de usuario a partir del tipo de datos del modelo. Simplificando un poco, si colocamos en la carpeta _DisplayTemplates_ y _EditorTemplates_ una vista parcial ASP.NET MVC usará esta vista automáticamente cada vez que se use el método Html.DisplayFor o Html.EditorFor respectivamente.

Si tenemos un Modelo de tipo X que tiene una propiedad, llamémosle Foo, cuyo tipo sea BarType, si hacemos:

<div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
  <div style="background: #fff; max-height: 300px; overflow: auto">
    <ol style="padding-bottom: 0px; margin: 0px; padding-left: 5px; padding-right: 0px; background: #ffffff; padding-top: 0px">
      <li>
        <span style="background: #ffff00">@</span>Html.EditorFor(x=>x.Foo)
      </li>
    </ol>
  </div></p>
</div>

ASP.NET MVC buscará la vista EditorTemplates/BarType (el nombre de la vista es el tipo de la propiedad usada en EditorFor). 

Como esta regla del nombre de tipo puede ser demasiado genérica, también es posible usar el atributo [UIHint] indicando el nombre del template (la vista parcial) a usar para editar o mostrar los datos:

<div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
  <div style="background: #ddd; max-height: 300px; overflow: auto">
    <ol style="padding-bottom: 0px; margin: 0px 0px 0px 2em; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
      <li>
        <span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> <span style="color: #2b91af">FooModel</span>
      </li>
      <li style="background: #f3f3f3">
        {
      </li>
      <li>
        &#160;&#160;&#160; [<span style="color: #2b91af">UIHint</span>(<span style="color: #a31515">"FooEdit"</span>)]
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; <span style="color: #0000ff">public</span> <span style="color: #2b91af">BarType</span> Foo { <span style="color: #0000ff">get</span>; <span style="color: #0000ff">set</span>; }
      </li>
      <li>
        }
      </li>
    </ol>
  </div></p>
</div>

Ahora la llamada a Html.EditorFor, buscará la vista llamada FooEdit en EditorTamplates.

Vamos pues a crear el _template_ para editar y visualizar una propiedad de tipo _int_ usando el slider de jQuery UI.

Para ello creamos una vista parcial en Views/Shared/EditorTemplates y le damos el nombre que queramos, en mi caso slider.cshtml. Para crear un slider basta con tener un <div> y luego llamar al método slider() (doy por supuesto que jQuery UI se ha descargado y está referenciada). Así que empezaremos con este código:

<div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
  <div style="background: #ddd; max-height: 300px; overflow: auto">
    <ol style="padding-bottom: 0px; margin: 0px 0px 0px 2em; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
      <li>
        <span style="color: #0000ff"><</span><span style="color: #800000">script</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text/javascript"></span>
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; $(document).ready(<span style="color: #0000ff">function</span> () {
      </li>
      <li>
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; $(<span style="color: #800000">"#slider"</span>).slider();
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; });
      </li>
      <li>
        <span style="color: #0000ff"></</span><span style="color: #800000">script</span><span style="color: #0000ff">></span>
      </li>
      <li style="background: #f3f3f3">
        <span style="color: #0000ff"><</span><span style="color: #800000">div</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="slider"></span>
      </li>
      <li>
        <span style="color: #0000ff"></</span><span style="color: #800000">div</span><span style="color: #0000ff">></span>
      </li>
    </ol>
  </div></p>
</div>

Con esto creamos el slider pero dado que estamos en modo edición, necesitamos alguna manera para _guardar_ el valor que el usuario seleccione. Una forma rápida de hacerlo es tener un _hidden_ que mantenga en todo momento el valor que el usuario seleccione en el slider. Y aquí nos surge la primera duda: que valor ha de tener el atributo _name_ para que luego ASP.NET MVC sea capaz de reconocerlo y enlazarlo a la propiedad del viewmodel? El problema es que el valor del atributo name depende del _nombre_ de la propiedad que estamos enlazando así que hemos de recuperar este nombre… Por suerte podemos saber este nombre, usando la propiedad HtmlFieldPrefix de la propiedad TemplateInfo del ViewData. Es decir, podemos generar el campo hidden así:

<div style="background: #fff; max-height: 300px; overflow: auto">
  <ol style="padding-bottom: 0px; margin: 0px; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
    <li>
      <span style="color: #0000ff"><</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="hidden"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="</span><span style="background: #ffff00">@</span><span style="color: #0000ff">ViewData.TemplateInfo.HtmlFieldPrefix"</span><span style="color: #0000ff">/></span>
    </li>
  </ol>
</div>

Finalmente sólo nos queda suscrbirnos al evento change del slider y actualizar el campo hidden. Para ello deberemos añadir un _id_ (he usado slider_hidden) al campo hidden y usar el siguiente código para crear el slider:

<div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
  <div style="background: #ddd; max-height: 300px; overflow: auto">
    <ol style="padding-bottom: 0px; margin: 0px 0px 0px 2.5em; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
      <li>
        <span style="color: #0000ff"><</span><span style="color: #800000">script</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text/javascript"></span>
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160; $(document).ready(function () {
      </li>
      <li>
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; var options = {
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; change: function (event, ui) {
      </li>
      <li>
        &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; $("#slider_hidden").val(ui.value);
      </li>
      <li style="background: #f3f3f3">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }&#160;&#160;&#160;&#160;&#160;
      </li>
      <li>
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; };&#160;&#160;&#160;&#160;&#160;&#160;&#160;
      </li>
      <li>
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; $("#slider").slider(options); </ li> <li style="background: #f3f3f3">
          &#160;&#160;&#160; });
        </li>
        <li>
          <span style="color: #0000ff"></</span><span style="color: #800000">script</span><span style="color: #0000ff">></span>
        </li></ol></div> </p></div> 
        <p>
          Con eso ya podemos crear un ViewModel con una propiedad int, decorada con [UIHint(“silder”)] y observar como aparece el slider para editar la propiedad. P.ej. dado el siguiente ViewModel:
        </p>
        
        <div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
          <div style="background: #ddd; max-height: 300px; overflow: auto">
            <ol style="padding-bottom: 0px; margin: 0px 0px 0px 2.5em; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
              <li>
                <span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> <span style="color: #2b91af">Ratio</span>
              </li>
              <li style="background: #f3f3f3">
                {
              </li>
              <li>
                &#160;&#160;&#160; <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Texto { <span style="color: #0000ff">get</span>; <span style="color: #0000ff">set</span>; }&#160;
              </li>
              <li>
                &#160;&#160;&#160; [<span style="color: #2b91af">UIHint</span>(<span style="color: #a31515">"slider"</span>)]
              </li>
              <li style="background: #f3f3f3">
                &#160;&#160;&#160; <span style="color: #0000ff">public</span> <span style="color: #0000ff">int</span> Rating2 { <span style="color: #0000ff">get</span>; <span style="color: #0000ff">set</span>; }
              </li>
              <li style="background: #f3f3f3">
                }
              </li>
            </ol>
          </div></p>
        </div>
        
        <p>
          Una vista para editar un objeto Ratio sería tan sencilla como:
        </p>
        
        <div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
          <div style="background: #ddd; max-height: 300px; overflow: auto">
            <ol style="padding-bottom: 0px; margin: 0px 0px 0px 2.5em; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
              <li>
                <span style="background: #ffff00">@</span><span style="color: #0000ff">using</span> MvcSliderBinding.Models
              </li>
              <li style="background: #f3f3f3">
                <span style="background: #ffff00">@model </span><span style="color: #2b91af">Ratio</span>
              </li>
              <li>
                <span style="background: #ffff00">@{</span>
              </li>
              <li style="background: #f3f3f3">
                &#160;&#160;&#160; ViewBag.Title = <span style="color: #a31515">"title"</span>;&#160;&#160;&#160;
              </li>
              <li>
                }
              </li>
              <li style="background: #f3f3f3">
                <span style="background: #ffff00">@</span><span style="color: #0000ff">using</span> (Html.BeginForm())
              </li>
              <li>
                {
              </li>
              <li style="background: #f3f3f3">
                &#160;&#160;&#160; <span style="background: #ffff00">@</span>Html.LabelFor(x => x.Texto)
              </li>
              <li>
                &#160;&#160;&#160; <span style="background: #ffff00">@</span>Html.EditorFor(x => x.Texto)
              </li>
              <li style="background: #f3f3f3">
                &#160;&#160;&#160; <span style="color: #0000ff"><</span><span style="color: #800000">br</span> <span style="color: #0000ff">/></span>
              </li>
              <li>
                &#160;&#160;&#160; <span style="background: #ffff00">@</span>Html.LabelFor(x => x.Rating)
              </li>
              <li style="background: #f3f3f3">
                &#160;&#160;&#160; <span style="background: #ffff00">@</span>Html.EditorFor(x => x.Rating)
              </li>
              <li>
                &#160;&#160;&#160;&#160;
              </li>
              <li style="background: #f3f3f3">
                &#160;&#160;&#160; <span style="color: #0000ff"><</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="submit"</span> <span style="color: #0000ff">/></span>
              </li>
              <li>
                }
              </li>
            </ol>
          </div></p>
        </div>
        
        <p>
          Al usar @Html.EditorFor(x=>x.Rating), al ser Rating una propiedad decorada con UIHint(“slider”) se va a usar el editor template que hemos creado antes.
        </p>
        
        <p>
          Ya tenemos enlazado una propiedad con el slider de jQuery! Ahora vamos a pulir detalles…
        </p>
        
        <p>
          <strong>Preparándolo para que pueda haber más de un slider</strong>
        </p>
        
        <p>
          El template de edición que hemos creado NO admite ser repetido en una misma vista, ya que usa ids estáticos para el <div> que será el slider y el hidden que contiene el valor. Si tuviéramos un viewmodel que tiene dos propiedades y quisiéramos usar dos sliders no nos funcionaría bien. Para solucionarlo nos basta con asegurar que los IDs son siempre distintos. Hay varias maneras, una es usar como id un prefijo y el valor que nos da HtmlFieldPrefix, ya que ese se supone único. Otro es usar un GUID, como se muestra a continuación:
        </p>
        
        <div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
          <div style="background: #ddd; max-height: 300px; overflow: auto">
            <ol style="padding-bottom: 0px; margin: 0px 0px 0px 2.5em; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
              <li>
                <span style="background: #ffff00">@model </span><span style="color: #2b91af">Nullable</span><<span style="color: #0000ff">int</span>>
              </li>
              <li style="background: #f3f3f3">
                <span style="background: #ffff00">@{</span>
              </li>
              <li>
                &#160;&#160;&#160; <span style="color: #0000ff">var</span> suffix = <span style="color: #2b91af">Guid</span>.NewGuid().ToString();&#160;
              </li>
              <li style="background: #f3f3f3">
                }
              </li>
              <li>
                <span style="color: #0000ff"><</span><span style="color: #800000">script</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text/javascript"></span>
              </li>
              <li style="background: #f3f3f3">
                &#160;&#160;&#160; $(document).ready(<span style="color: #0000ff">function</span> () {
              </li>
              <li>
                &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #0000ff">var</span> options = {
              </li>
              <li>
                &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; value: <span style="background: #ffff00">@(</span>Model.HasValue ? Model.Value : min<span style="background: #ffff00">)</span>,
              </li>
              <li style="background: #f3f3f3">
                &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; change: <span style="color: #0000ff">function</span> (event, ui) {
              </li>
              <li>
                &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; $(<span style="color: #800000">"#</span><span style="background: #ffff00; color: #800000">@</span>suffix<span style="color: #800000">"</span>).val(ui.value);
              </li>
              <li style="background: #f3f3f3">
                &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }
              </li>
              <li>
                &#160;&#160;&#160;&#160;&#160;&#160;&#160; };&#160;&#160;
              </li>
              <li>
                &#160;&#160;&#160;&#160;&#160;&#160;&#160; $(<span style="color: #800000">"#slider_</span><span style="background: #ffff00; color: #800000">@(</span>suffix<span style="background: #ffff00; color: #800000">)</span><span style="color: #800000">"</span>).slider(options);
              </li>
              <li style="background: #f3f3f3">
                &#160;&#160;&#160; });
              </li>
              <li>
                <span style="color: #0000ff"></</span><span style="color: #800000">script</span><span style="color: #0000ff">></span>
              </li>
              <li style="background: #f3f3f3">
                <span style="color: #0000ff"><</span><span style="color: #800000">div</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="slider_</span><span style="background: #ffff00">@(</span><span style="color: #0000ff">suffix</span><span style="background: #ffff00">)</span><span style="color: #0000ff">"></span>
              </li>
              <li>
                <span style="color: #0000ff"></</span><span style="color: #800000">div</span><span style="color: #0000ff">></span>
              </li>
            </ol>
          </div></p>
        </div>
        
        <p>
          Guardamos e<br /> n suffix el GUID creado y lo añadimos al hidden y al div. Un detalle más que aprovecho para enseñaros es establecer el valor <em>inicial</em> del slider al valor que tenga la propiedad (que está en Model). Pero, si estamos <em>creando</em> el objeto el valor de Model será null. Es por ello que debo declarar que el modelo de la vista es de tipo Nullable<int>, en lugar de int, para poder aceptar esos valores nulos.
        </p>
        
        <p>
          <strong>Accediendo a la información del viewmodel</strong>
        </p>
        
        <p>
          Una cosa muy interesante al usar template helpers, es que nuestro template helper <em>puede acceder a información del viewmodel que define la propiedad</em>. Es decir, dentro del template helper, yo puedo saber <em>cual es la propiedad que estoy editando (en mi caso es Rating) y tengo información sobre la clase que define dicha propiedad (en mi caso Ratio</em>). A esos datos se puede acceder a través de ViewData.ModelMetadata:
        </p>
        
        <p>
          <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_6CD6150E.png"><img style="background-image: none; border-right-width: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_19129BF3.png" width="550" height="56" /></a>
        </p>
        
        <p>
          Podeis usar las propiedades de ViewData.ModelMetadata para realizar ciertas tareas, como analizar el ViewModel en busca de atributos que os puedan ayudar a definir como renderizar el template… lo que se os ocurra.
        </p>
        
        <p>
          Un ejemplo de esto, imaginad que tenemos una propiedad decorada con el atributo [Range], para indicar que acepta un rango de valores. Pues desde aquí podríais consultar dicho atributo Range (teneis acceso al Type del ViewModel y sabeis el nombre de la propiedad) y configurar el slider para que sólo acepte entradas en este rango.
        </p>
        
        <p>
          Es lo que he hecho yo, salvo que en lugar de usar reflection para acceder al atributo Range, lo que he hecho es <em>obtener los validadores </em>que hay asociados a la propiedad y generar código acorde a ellos. Para ello uso el método GetVaidators() de ModelMetadata:
        </p>
        
        <div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
          <div style="background: #ddd; max-height: 300px; overflow: auto">
            <ol style="padding-bottom: 0px; margin: 0px 0px 0px 2.5em; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
              <li>
                <span style="background: #ffff00">@model </span><span style="color: #2b91af">Nullable</span><<span style="color: #0000ff">int</span>>
              </li>
              <li style="background: #f3f3f3">
                <span style="background: #ffff00">@{</span>
              </li>
              <li>
                &#160;&#160;&#160; <span style="color: #0000ff">var</span> suffix = <span style="color: #2b91af">Guid</span>.NewGuid().ToString();
              </li>
              <li style="background: #f3f3f3">
                &#160;
              </li>
              <li>
                &#160;&#160;&#160; <span style="color: #0000ff">bool</span> hasRange = <span style="color: #0000ff">false</span>;
              </li>
              <li style="background: #f3f3f3">
                &#160;&#160;&#160; <span style="color: #0000ff">var</span> rangeVal = ViewData.ModelMetadata.GetValidators(ViewContext.Controller.ControllerContext).OfType<<span style="color: #2b91af">RangeAttributeAdapter</span>>().FirstOrDefault();
              </li>
              <li>
                &#160;&#160;&#160; <span style="color: #2b91af">ModelClientValidationRangeRule</span> rule = <span style="color: #0000ff">null</span>;
              </li>
              <li style="background: #f3f3f3">
                &#160;&#160;&#160; <span style="color: #0000ff">var</span> min = 0;
              </li>
              <li>
                &#160;&#160;&#160; <span style="color: #0000ff">var</span> max = -1;
              </li>
              <li style="background: #f3f3f3">
                &#160;&#160;&#160; <span style="color: #0000ff">if</span> (rangeVal != <span style="color: #0000ff">null</span>)
              </li>
              <li>
                &#160;&#160;&#160; {
              </li>
              <li style="background: #f3f3f3">
                &#160;&#160;&#160;&#160;&#160;&#160;&#160; rule = rangeVal.GetClientValidationRules().OfType<<span style="color: #2b91af">ModelClientValidationRangeRule</span>>().FirstOrDefault();
              </li>
              <li>
                &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #0000ff">if</span> (rule != <span style="color: #0000ff">null</span>)
              </li>
              <li style="background: #f3f3f3">
                &#160;&#160;&#160;&#160;&#160;&#160;&#160; {
              </li>
              <li>
                &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; min = <span style="color: #2b91af">Convert</span>.ToInt32(rule.ValidationParameters[<span style="color: #a31515">"min"</span>]);
              </li>
              <li style="background: #f3f3f3">
                &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; max = <span style="color: #2b91af">Convert</span>.ToInt32(rule.ValidationParameters[<span style="color: #a31515">"max"</span>]);
              </li>
              <li>
                &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
              </li>
              <li style="background: #f3f3f3">
                &#160;&#160;&#160; }
              </li>
              <li>
                &#160;
              </li>
              <li style="background: #f3f3f3">
                }
              </li>
            </ol>
          </div></p>
        </div>
        
        <p>
          Llamo a GetValidatos y busco el validador de tipo RangeAttributeAdapter. Eso es lo mismo que buscar si existe un atributo [Range], salvo que es más genérico (aunque no entraremos en detalles, simplemente comentar que DataAnnotations es <em>una</em> manera de añadir validadores, pero pueden haber más)<em>. </em>Si existe el validador de rango, obtengo su configuración, en concreto sus valores mínimo y máximo, y me los guardo. Con esto ahora tengo información para generar un slider que sólo acepte valores en este rango:
        </p>
        
        <div style="border-bottom: #000080 1px solid; border-left: #000080 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; color: #000; font-size: 10pt; border-top: #000080 1px solid; border-right: #000080 1px solid">
          <div style="background: #ddd; max-height: 300px; overflow: auto">
            <ol style="padding-bottom: 0px; margin: 0px 0px 0px 2.5em; padding-left: 5px; padding-right: 0px; white-space: nowrap; background: #ffffff; padding-top: 0px">
              <li>
                <span style="color: #0000ff"><</span><span style="color: #800000">script</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text/javascript"></span>
              </li>
              <li style="background: #f3f3f3">
                &#160;&#160;&#160; $(document).ready(<span style="color: #0000ff">function</span> () {
              </li>
              <li>
                &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #0000ff">var</span> options = {
              </li>
              <li style="background: #f3f3f3">
                &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="background: #ffff00">@</span><span style="color: #0000ff">if</span> (rule!=<span style="color: #0000ff">null</span>)
              </li>
              <li>
                &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; {
              </li>
              <li style="background: #f3f3f3">
                &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="background: #ffff00">@:</span>min: <span style="background: #ffff00">@</span>min,
              </li>
              <li>
                &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="background: #ffff00">@:</span>max: <span style="background: #ffff00">@</span>max,
              </li>
              <li style="background: #f3f3f3">
                &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }
              </li>
              <li>
                &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; value: <span style="background: #ffff00">@(</span>Model.HasValue ? Model.Value : min<span style="background: #ffff00">)</span>,
              </li>
              <li style="background: #f3f3f3">
                &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; change: <span style="color: #0000ff">function</span> (event, ui) {
              </li>
              <li>
                &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; $(<span style="color: #800000">"#</span><span style="background: #ffff00; color: #800000">@</span>suffi<br /> x<span style="color: #800000">"</span>).val(ui.value);
              </li>
              <li style="background: #f3f3f3">
                &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; }
              </li>
              <li>
                &#160;&#160;&#160;&#160;&#160;&#160;&#160; };&#160;&#160;&#160;&#160;&#160;&#160;&#160;
              </li>
              <li>
                &#160;&#160;&#160;&#160;&#160;&#160;&#160; $(<span style="color: #800000">"#slider_</span><span style="background: #ffff00; color: #800000">@(</span>suffix<span style="background: #ffff00; color: #800000">)</span><span style="color: #800000">"</span>).slider(options);
              </li>
              <li style="background: #f3f3f3">
                &#160;&#160;&#160; });
              </li>
            </ol>
          </div></p>
        </div>
        
        <p>
          Listos! Ahora tenemos un editor que además respeta el validador de rango que tenga la propiedad.
        </p>
        
        <p>
          Y así podríamos ir perfilando este template de edición con todo lo que necesitáramos para adaptarlo a nuestras necesidades… ¡Simple, sencillo y potentísimo!
        </p>
        
        <p>
          Os dejo un proyecto VS2010 con la implementación del template de edición y uno de visualización para que podáis verlo en acción: <a title="https://skydrive.live.com/?cid=6521c259e9b1bec6&sc=documents&uc=1&id=6521C259E9B1BEC6%21167#" href="https://skydrive.live.com/?cid=6521c259e9b1bec6&sc=documents&uc=1&id=6521C259E9B1BEC6%21167#">https://skydrive.live.com/?cid=6521c259e9b1bec6&sc=documents&uc=1&id=6521C259E9B1BEC6%21167#</a>
        </p>
        
        <p>
          Espero que os haya sido útil! Un saludo!
        </p>

 [1]: http://docs.jquery.com/UI/Slider