---
title: ASP.NET MVC–Traduciendo las validaciones de CompareAttribute
author: eiximenis

date: 2014-10-21T09:29:43+00:00
geeks_url: /?p=1685
geeks_visits:
  - 1123
geeks_ms_views:
  - 1210
categories:
  - Uncategorized

---
Muy buenas! Seguimos ese _tour de force_ sobre las traducciones de los mensajes de validación de Data Annotations.

En el primer post de esta serie vimos <a href="http://geeks.ms/blogs/etomas/archive/2014/10/09/asp-net-mvc-traducir-los-mensajes-de-error-de-dataannotations-otra-vez.aspx" target="_blank" rel="noopener noreferrer">como crear adaptadores de atributos</a> para permitirnos fácilmente y a nivel centralizado establecer las propiedades ErrorMessageResourceName y ErrorMessageResourceType.

El post terminaba con una lista de los distintos adaptadores que tiene ASP.NET MVC y que podíamos usar como clases base. Hay adaptadores definidos para casi todos los atributos de Data Annotations (Required, StringLength,…) pero **no hay ninguno para el CompareAttribute**. El atributo Compare no se usa mucho, ya que valida que **dos** propiedades tengan el mismo valor. El clásico uso es en formularios de registro donde el usuario debe introducir una contraseña dos veces para evitar que haya ningún error.

Pero el hecho de que ASP.NET MVC no incluya ningún adaptador base para dicho atributo no nos impide crearnos el nuestro y aplicar la misma técnica para traducir los mensajes de validación de dicho atributo. Para ello derivaremos de la clase DataAnnotationsModelValidator<T> (siendo T el tipo del atributo, en este caso el CompareAttribute).

La principal diferencia **es que ahora debemos generar las validaciones de cliente,** sobreescribiendo el método GetClientValidationRules().

Pese a todo, el código sigue siendo muy sencillo:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:a46904b4-3edf-4d34-b24b-3b19db8259d2" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2.5em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">LocalizedCompareAdapter</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#4ec9b0">DataAnnotationsModelValidator</span><span style="background:#1e1e1e;color:#b4b4b4"><</span><span style="background:#1e1e1e;color:#dcdcdc">System</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ComponentModel</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">DataAnnotations</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#4ec9b0">CompareAttribute</span><span style="background:#1e1e1e;color:#b4b4b4">></span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> LocalizedCompareAdapter(</span><span style="background:#1e1e1e;color:#4ec9b0">ModelMetadata</span><span style="background:#1e1e1e;color:#dcdcdc"> metadata, </span><span style="background:#1e1e1e;color:#4ec9b0">ControllerContext</span><span style="background:#1e1e1e;color:#dcdcdc"> context, </span><span style="background:#1e1e1e;color:#4ec9b0">CompareAttribute</span><span style="background:#1e1e1e;color:#dcdcdc"> attribute)</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">: </span><span style="background:#1e1e1e;color:#569cd6">base</span><span style="background:#1e1e1e;color:#dcdcdc">(metadata, context, attribute)</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc">attribute</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ErrorMessageResourceName </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"Compare"</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc">attribute</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ErrorMessageResourceType </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">typeof</span><span style="background:#1e1e1e;color:#dcdcdc">(Resources</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#4ec9b0">Messages</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
        <li>
          &nbsp;
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">override</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#b8d7a3">IEnumerable</span><span style="background:#1e1e1e;color:#b4b4b4"><</span><span style="background:#1e1e1e;color:#4ec9b0">ModelClientValidationRule</span><span style="background:#1e1e1e;color:#b4b4b4">></span><span style="background:#1e1e1e;color:#dcdcdc"> GetClientValidationRules()</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li style="background: #111111">
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">var</span><span style="background:#1e1e1e;color:#dcdcdc"> other </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> Attribute</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">OtherProperty;</span>
        </li>
        <li>
                  <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">return</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc">[] { </span><span style="background:#1e1e1e;color:#569cd6">new</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">ModelClientValidationEqualToRule</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#569cd6">base</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ErrorMessage, other) };</span>
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

En el método GetClientValidationRules devolvemos las validaciones de cliente. En este caso, queremos comparar dos propiedades así que devolvemos una ModelClientValidationEqualToRule, a la cual le pasamos el nombre de la **otra propiedad**. Recuerda que el [Compare] se aplica a _una_&#160; propiedad (p. ej. ConfirmPassword) y se coloca el nombre de la otra propiedad (p. ej. Password):

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:a35a0912-0387-4ae0-964d-5886c138c4e5" class="wlWriterEd
itableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  </p> 
  
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> Password { </span><span style="background:#1e1e1e;color:#569cd6">get</span><span style="background:#1e1e1e;color:#dcdcdc">; </span><span style="background:#1e1e1e;color:#569cd6">set</span><span style="background:#1e1e1e;color:#dcdcdc">; }</span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#dcdcdc">[</span><span style="background:#1e1e1e;color:#4ec9b0">Compare</span><span style="background:#1e1e1e;color:#dcdcdc">(</span><span style="background:#1e1e1e;color:#d69d85">"Password"</span><span style="background:#1e1e1e;color:#dcdcdc">)]</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">string</span><span style="background:#1e1e1e;color:#dcdcdc"> ConfirmPassword { </span><span style="background:#1e1e1e;color:#569cd6">get</span><span style="background:#1e1e1e;color:#dcdcdc">; </span><span style="background:#1e1e1e;color:#569cd6">set</span><span style="background:#1e1e1e;color:#dcdcdc">; }</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

En este caso es la propiedad ConfirmPassword la que está decorada con el CompareAttribute, por lo tanto esta es la contendrá la regla de validación en cliente. De hecho el código HTML generado por los helpers Html.TextBoxFor para esas dos propiedades es: 

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:3245774f-2f10-4d00-8f03-b30563bf0a00" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">input</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">id</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"Password"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">name</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"Password"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">type</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"text"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">value</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">""</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#808080">/></span>
        </li>
        <li style="background: #111111">
          <span style="background:#1e1e1e;color:#808080"><</span><span style="background:#1e1e1e;color:#569cd6">input</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">data-val</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"true"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">data-val-equalto</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"Los valores de ConfirmPassword y Password deben ser iguales"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span>
        </li>
        <li>
                 <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#9cdcfe">data-val-equalto-other</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"Password"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">id</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"ConfirmPassword"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">name</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"ConfirmPassword"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">type</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">"text"</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#9cdcfe">value</span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#c8c8c8">""</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#808080">/></span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Se puede ver que el <input /> que se corresponde a Password no tiene validaciones aplicadas y que es el <input /> que corresponde a ConfirmPassword el que tiene las validaciones de cliente aplicadas.

Podemos ver que el mensaje de error (data-val-equalto) está en castellano porque en mi fichero de recursos (Messages.es.resx) tengo la entrada “Compare”que es la que usa nuestro adaptador de recursos:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_6359A0BF.png" width="504" height="45" />][1]

¡Y listos! Hemos visto como el hecho de que ASP.NET MVC&#160; no provea un adaptador base no nos impide usar DataAnnotationsModelValidator<T> para crearnos nuestro propio adaptador, con la salvedad de que debemos indicar las validaciones de cliente a generar (las de servidor no son necesarias).

**PD:** Por supuesto debemos registrar ese adaptador de atributo como vimos en el post dedicado a los adaptadores:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:99fed560-c8a0-4cbb-a4a7-f6bdc283aa9a" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#4ec9b0">DataAnnotationsModelValidatorProvider</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">RegisterAdapter(</span>
        </li>
        <li
style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">typeof</span><span style="background:#1e1e1e;color:#dcdcdc"> (System</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ComponentModel</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">DataAnnotations</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#4ec9b0">CompareAttribute</span><span style="background:#1e1e1e;color:#dcdcdc">),</span>
        </li>
        <li>
              <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#569cd6">typeof</span><span style="background:#1e1e1e;color:#dcdcdc"> (</span><span style="background:#1e1e1e;color:#4ec9b0">LocalizedCompareAdapter</span><span style="background:#1e1e1e;color:#dcdcdc">));</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Un saludo!

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_4416F9EC.png