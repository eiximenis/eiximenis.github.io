---
title: ASP.NET MVC6 (vNext)–ViewComponents
description: ASP.NET MVC6 (vNext)–ViewComponents
author: eiximenis

date: 2014-09-09T18:13:52+00:00
geeks_url: /?p=1678
geeks_visits:
  - 2194
geeks_ms_views:
  - 1410
categories:
  - Uncategorized

---
En ASP.NET vNext se unifican MVC y WebApi en una nueva API llamada MVC6. Aunque MVC6 _se parece_ a MVC5 _no es compatible con ella_, del mismo modo que WebApi se parece a MVC pero por debajo son muy distintas.

Ya hemos viso algunas de las novedades o cambios que trae MVC6 (temas de <a href="http://geeks.ms/blogs/etomas/archive/2014/07/14/asp-net-vnext-model-binding.aspx" target="_blank" rel="noopener noreferrer">model binding</a>, <a href="http://geeks.ms/blogs/etomas/archive/2014/06/26/asp-net-mvc-vnext-controladores-poco.aspx" target="_blank" rel="noopener noreferrer">controladores POCO</a>, …) y en este post vamos a explorar uno más: los ViewComponents.

Resumiendo: los **ViewComponents sustituyen a las vistas parciales**. Ya no existe este concepto en MVC6. De hecho, tampoco nos engañemos, desde razor la diferencia entre vistas parciales y vistas normales (a nivel del archivo .cshtml) es muy pequeña: se puede usar una vista parcial como vista normal tan solo cambiando el “return PartialView()” por un “return View()” (o viceversa). En el motor de vistas de ASPX eso no era así, ya que las vistas eran archivos .aspx y las vistas parciales eran archivos .ascx.

En Razor la única diferencia actual entre una vista parcial y una normal es que en la segunda se procesa el archivo de Layout (usualmente _Layout.cshtml) y en la primera no. Pero **no es el archivo .cshtml quien determina si es vista normal o parcial. Es el ActionResult devuelto.** Si devuelves un ViewResult el archivo .cshtml se procesará como vista normal. Si devuelves un PartialViewResult el archivo .cshtml se procesará como vista parcial.

El código “clásico” en MVC5 para tener una vista parcial era algo como:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:bc270a04-af8f-4040-9bf6-d5443b38fae8" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">ActionResult</span><span style="background:#1e1e1e;color:#dcdcdc"> Child()</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> PartialView();</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

El problema con este enfoque es que esta acción es enrutable, por lo que cualquiera puede ir a la URL que enrute esa acción (p. ej. Home/Child si suponemos HomeController) y recibirá el contenido HTML de la vista parcial. Eso, generalmente, no se desea (para algo la vista es parcial).

Para solventar esto, en MVC4 se añadió el atributo [ChildActionOnly] que evitaba que una acción se enrutase. Así si decoramos la acción con dicho atributo cuando el usuario navega a la URL que _debería_ enrutar dicha acción recibirá un error:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_1A756793.png" width="504" height="76" />][1]

La acción se puede invocar a través del helper Html.RenderAction:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:69249e24-4a6c-4880-8590-8614391d6088" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#ffffb3;color:#000000">@{</span><span style="background:#1e1e1e;color:#dcdcdc"> Html</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">RenderAction(</span><span style="background:#1e1e1e;color:#d69d85">"</span><span style="background:#1e1e1e;color:#ff3333">Child</span><span style="background:#1e1e1e;color:#d69d85">"</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#d69d85">"Home"</span><span style="background:#1e1e1e;color:#dcdcdc">); }</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

**Nota:** Se puede usar Html.Partial o Html.RenderPartial para renderizar una vista parcial directamente (sin pasar por un controlador). Eso es útil en el caso de que no haya lógica asociada a dicha vista parcial (si la hay, lo suyo es colocarla en la acción y usar Html.RenderAction).

Bueno… así tenemos las cosas hoy en día: básicamente colocamos las acciones “hijas” en un controlador (porque es _donde podemos_ colocar lógica) pero luego las quitamos del sistema de enrutamiento (con [ChildActionOnly]) y las llamamos indicando directamente que acción y que controlador es.

Realmente ¿**tiene sentido que las acciones hijas estén en un controlador?** No. Porque la responsabilidad del controlador es, básicamente, responder a peticiones del navegador y eso **no es una petición del navegador.**

Así en MVC6 se elimina el concepto de vista parcial y el PartialViewResult, y se sustituye por el concepto de ViewComponent. Ahora **lo que antes eran acciones hijas son clases propias que derivan de ViewComponent**:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:c5af0b45-4952-49ee-b2d9-d297f826de6b" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">[</span><span style="background:#1e1e1e;color:#4ec9b0">ViewComponent</span><span style="background:#1e1e1e;color:#dcdcdc">(Name </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"Child"</span><span style="background:#1e1e1e;color:#dcdcdc">)]</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">ChildComponent</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#4ec9b0">ViewComponent</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </sp an><span style="background:#1e1e1e;color:#569cd6">async</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">Task</span><span style="background:#1e1e1e;color:#dcdcdc"><</span><span style="background:#1e1e1e;color:#b8d7a3">IViewComponentResult</span><span style="background:#1e1e1e;color:#dcdcdc">> InvokeAsync()</span></li> 
          
          <li>
                <span style="background:#1e1e1e;color:#dcdcdc">{</span>
          </li>
          <li style="background: #111111">
                    <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> View();</span>
          </li>
          <li>
                <span style="background:#1e1e1e;color:#dcdcdc">}</span>
          </li>
          <li style="background: #111111">
            <span style="background:#1e1e1e;color:#dcdcdc">}</span>
          </li></ol></div> </p></div> </p></div> 
          
          <p>
            El atributo [ViewComponent] nos permite especificar el nombre que damos al componente. El siguiente paso es definir el método InvokeAsync que devuelve una Task<IViewComponentResult> con el resultado. La clase ViewComponent nos define el método View() que devuelve la vista asociada a dicho componente (de forma análoga al método View() de un controlador).
          </p>
          
          <p>
            La ubicación por defecto de la vista asociada a un componente es /Views/Shared/Components/[NombreComponente]/Default.cshtml. Es decir en mi caso tengo el fichero Default.cshtml en /Views/Shared/Components/Child:
          </p>
          
          <p>
            <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_2E8E841C.png"><img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_69E1E9DA.png" width="200" height="144" /></a>
          </p>
          
          <p>
            Por supuesto ahora tengo un sitio donde colocar la lógica (si la hubiera) de dicho componente: la propia clase ChildComponent.
          </p>
          
          <p>
            Finalmente nos queda ver como renderizamos el componente. Ya no tenemos Html.RenderAction, si no que en su lugar usamos la propiedad Component que tienen las vistas de MVC6:
          </p>
          
          <div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:4112d312-fbe3-49b3-886c-67281f00c4bd" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
            <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
              <div style="background: #ddd; max-height: 300px; overflow: auto">
                <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
                  <li>
                    <span style="background:#ffffb3;color:#000000">@</span><span style="background:#1e1e1e;color:#569cd6">await</span><span style="background:#1e1e1e;color:#dcdcdc"> Component</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">InvokeAsync(</span><span style="background:#1e1e1e;color:#d69d85">"Child"</span><span style="background:#1e1e1e;color:#dcdcdc">)</span>
                  </li>
                </ol>
              </div></p>
            </div></p>
          </div>
          
          <p>
            Simplemente le pasamos el nombre del componente (el mismo definido en el atributo [ViewComponent].
          </p>
          
          <p>
            Y listos 🙂
          </p>

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_4219530C.png