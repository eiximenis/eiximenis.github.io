---
title: ASP.NET MVC–Traduciendo las validaciones implícitas

author: eiximenis

date: 2014-10-13T14:49:28+00:00
geeks_url: /?p=1684
geeks_visits:
  - 1336
geeks_ms_views:
  - 1532
categories:
  - Uncategorized

---
En el post anterior vimos <a href="http://geeks.ms/blogs/etomas/archive/2014/10/09/asp-net-mvc-traducir-los-mensajes-de-error-de-dataannotations-otra-vez.aspx" target="_blank" rel="noopener noreferrer">como localizar los mensajes de validación de Data Annotations en ASP.NET MVC</a> usando adaptadores de atributos. Pero además de esos mensajes ASP.NET MVC **tiene algunos mensajes de traducción implícitos**.

P. ej. si tenemos una propiedad de tipo Int y le intentamos poner un valor no numérico ASP.NET MVC mostrará un mensaje de error. Este mensaje de error no proviene de Data Annotations, por lo que no podemos usar la técnica descrita en el post anterior para traducirlo.

Vamos a ver como traducir los mensajes implícitos. Para ello basta con crear un fichero de recursos (en App_GlobalResources) con la entrada **PropertyValueInvalid** y MVC usará dicho mensaje para mostrar el error. Puedes usar {0} para mostrar el valor inválido y {1} para mostrar el nombre de la propiedad.

La entrada **PropertyValueInvalid** la usa el model binder cuando se encuentra un valor inválido para una propiedad. P. ej. si tenemos una propiedad definida como int e intentamos asignarle una cadena.

Debemos configurar el model binder para que use el fichero de recursos que hemos creado usando la propiedad ResourceClassKey:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:43b591e6-87e0-4b90-8f54-2d75f0b90daa" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#4ec9b0">DefaultModelBinder</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ResourceClassKey </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"Messages"</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

El valor de ResourceClassKey es el nombre del fichero de recursos a usar.

**Traducir el mensaje implícito de propiedad requerida**

El atributo [Required] de Data Annotations nos permite especificar que el valor de una propiedad es obligatorio. Pero MVC trata automáticamente algunos campos como obligatorios: las propiedades cuyo tipo es un tipo por valor son tratadas como obligatorias automáticamente. Eso es lógico: si tengo una propiedad declarada como int, es obvio que debe tener siempre un valor, ya que int no admite el valor null.

Pero el tratamiento exacto que se da en esos casos depende del valor de la propiedad AddImplicitRequiredAttributeForValueTypes de la clase DataAnnotationsModelValidatorProvider:

  * Si vale true (valor por defecto) **se añade automáticamente un atributo [Required] a cada propiedad cuyo tipo sea un tipo por valor**. 
  * Si vale false **el tratamiento lo realiza el default model binder**. 

Si estamos en el primer caso, eso significa que dicho mensaje realmente tenemos que tratarlo a través de un adaptador de atributo porque, a todos los efectos, es como si la propiedad tuviese un atributo [Required] colocado.&#160; Así nos podríamos crear un adaptador para el atributo Required, <a href="http://geeks.ms/blogs/etomas/archive/2014/10/09/asp-net-mvc-traducir-los-mensajes-de-error-de-dataannotations-otra-vez.aspx" target="_blank" rel="noopener noreferrer">tal y como se comentaba en el post anterior</a>.

Si estamos en el segundo caso, entonces es el model binder quien tratará ese caso. Para mostrar el mensaje (que por defecto es un insulso “A value is required”) usará la entrada **PropertyValueRequired** del fichero de recursos que hayamos especificado mediante la propiedad ResourceClassKey.

Así vemos que el DefaultModelBinder usa dos claves del fichero de recursos:

  * PropertyValueInvalid: Cuando se asigna un valor incorrecto a una propiedad 
  * PropertyValueRequired: Cuando no se asigna valor a una propiedad cuyo tipo es un tipo por valor (siempre y cuando AddImplicitRequiredAttributeForValueTypes valga false). 

**Nota:** Esas dos son las dos únicas claves que usa el Default Model Binder.

**Validación en cliente**

Si tienes habilitada la validación en cliente, las cosas se ponen un poco más interesante. Hasta ahora hemos visto como traducir los mensajes implícitos que usa el Default Model Binder, pero la validación en cliente es independiente del Model Binder y es necesario un paso más.

Si la tienes habilitada verás que no te funciona… Aparecen otros mensajes de errores. P. ej. en el caso de introducir un valor no numérico en una propiedad numérica (un int p. ej.) ahora aparece un mensaje “The field xxx must be a number”.

Mirando el código fuente de la página puedes ver que este mensaje es de la validación en cliente:

[<img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_4EBC39C5.png" width="504" height="33" />][1]

¿Quien ha generado este mensaje y de donde lo saca?

Pues bien, estos mensajes de errores los genera la clase ClientDataTypeModelValidatorProvider que es la encargada de gestionar las validaciones en cliente.

Por suerte dicha clase expone también una propiedad ResourceCssKey, al igual que el DefaultModelBinder que podemos usar para especificarle un fichero de recursos:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:600c5608-a259-4348-b6af-8f26e82342e6" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#4ec9b0">ClientDataTypeModelValidatorProvider</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ResourceClassKey </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#d69d85">"Messages"</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Ahora la pregunta a responder es: ¿qué claves espera encontrar en este fichero de recursos?

Pues las siguientes:

  * FieldMustBeNumeric: En el caso que se entre una cadena en campos que deban ser numéricos 
  * FieldMustBeDate: En el caso de que se entre una cadena incorrecta en campos que deban ser fechas. 

Si te preguntas “qué es un campo numérico”, pues cualquiera cuyo tipo en la propiedad .NET equivalente sea: byte, sbyte, short, ushort, int, uint, long, ulong, float, double y decimal **o las versiones Nullable** de esos tipos.

Un campo fecha es aquel cuya propiedad se haya declarado como DateTime (o DateTime?) (y que **no** tenga aplicado el atributo [<a href="http://msdn.microsoft.com/en-us/library/system.
componentmodel.dataannotations.datatypeattribute(v=vs.110).aspx" target="_blank" rel="noopener noreferrer">DataType</a>] con el valor “Time”).

Por lo tanto nos basta con agregar esas dos recursos a nuestro fichero de recursos para poder traducir los mensajes implícitos.

Vale. Un apunte final: si te fijas en el código HTML para el <input /> de la propiedad Age, verás que no ha generado el data-val-required. Eso es porque tenia la línea:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:ef952a9f-c59a-45db-908d-89c5ef82ad0a" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1d1d1d; margin: 0 0 0 2em; padding: 0 0 0 5px;">
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc"></span><span style="background:#1e1e1e;color:#4ec9b0">DataAnnotationsModelValidatorProvider</span><span style="background:#1e1e1e;color:#b4b4b4">.</span>
        </li>
        <li style="background: #111111">
              <span style="background:#1e1e1e;color:#dcdcdc">AddImplicitRequiredAttributeForValueTypes </span><span style="background:#1e1e1e;color:#b4b4b4">=</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">false</span><span style="background:#1e1e1e;color:#dcdcdc">;</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Recuerda que eso hace que MVC no añada automáticamente un [Required]. La validación en cliente no entiende de campos requeridos si no hay un [Required]. Es por ello que no se genera el código en cliente para asegurarse que deba entrar un valor en este campo. Si elimino esa línea y ejecuto de nuevo ahora si que me aparece la validación de campo obligatorio:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_37FCF23C.png" width="504" height="25" />][2]

Si te preguntas ahora de donde sale el mensaje usado para la validación de campo obligatorio en el cliente, la respuesta es que del atributo [Required] que MVC ha añadido automáticamente a la propiedad.

Por lo tanto, si usas un adaptador de atributo para el [Required] este se aplicará (y dado que se aplica también en servidor verás el mismo mensaje en el cliente que en el servidor).

**En resumen…**

Resumiendo, los mensajes implícitos de MVC son generados por:

  1. El DefaultModelBinder en el servidor
  2. El ClientDataTypeModelValidatorProvider en la validación en cliente

Ambos usan una propiedad (ResourceCssKey) para especificar el fichero de recursos a utilizar. Y dentro de ese fichero de recursos podemos colocar las claves:

  1. PropertyValueInvalid: Cuando se asigna un valor inválido a una propiedad. Usado por el DefaultModelBinder
  2. PropertyValueRequired: Cuando no se ha informado el valor de una propiedad que es obligatoria porque su tipo es un tipo por valor (usado por el DefaultModelBinder solo si AddImplicitRequiredAttributeForValueTypes&#160; es false).
  3. FieldMustBeNumeric: Cuando se introduce un valor no numérico en una propiedad numérica (usada por ClientDataTypeModelValidatorProvider. El equivalente en servidor sería PropertyValueInvalid).
  4. FieldMustBeDate: Cuando se introduce un valor que no es una fecha en una propiedad de tipo fecha (usada por ClientDataTypeModelValidatorProvider. El equivalente en servidor sería PropertyValueInvalid).

Espero que te haya sido útil! En otro post iremos un paso más allá y veremos como personalizar al máximo esos mensajes de error. Pero ya te avanzo que nos meteremos hasta la cocina y el baño de ASP.NET MVC.

Saludos!

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_2CD0D741.png
 [2]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_035C9601.png