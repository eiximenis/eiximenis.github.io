---
title: Mostrar un formulario modal con ASP.NET MVC y Ajax (ii)

author: eiximenis

date: 2009-04-15T09:52:00+00:00
geeks_url: /?p=1446
geeks_visits:
  - 5838
geeks_ms_views:
  - 2264
categories:
  - Uncategorized

---
En el post anterior ([Mostrar un formulario modal con ASP.NET MVC y Ajax][1]) comenté como había usado [SimpleModal][2] en una aplicación ASP.NET MVC para mostrar un formulario modal al usuario.

En este post voy a comentar como podemos comunicar nuestro formulario modal con nuestros controladores, para así poder validar (parcialmente o totalmente) el formulario desde servidor, sin necesidad de hacer un _submit_, usando Ajax.

<!--more-->

En mi caso p.ej. a medida que el usuario va entrando un posible nickname, se le indica si dicho nickname ya está ocupado o no (aunqué en la versión final probablemente sea un boton &ldquo;comprobar nick&rdquo;, ya que una llamada Ajax cada vez que se entre un carácter en un textbox quizá es demasiado... pero eso no afecta al sentido del post). Para realizar las llamadas Ajax cada vez que el usuario pulse una tecla en el campo &ldquo;nick&rdquo; del formulario modal vamos a usar jQuery.

Lo primero que debemos cambiar respecto al post anterior es la forma en como abrimos el formulario modal: el método _modal_ de SimpleModal, admite varios parámetros, uno de los cuales es una función de _callback_ que SimpleModal llamará. A través de esta función vamos a registrar una función gestora del evento _KeyPress_ usando jQuery. Recordáis la función show_popup() que teníamos en la vista Index.aspx? Su código ahora queda así:

```js
function show_popup() {
    $("#popup").modal( { onOpen: popup_open});
}
```

Hemos añadido el parámetro onOpen con el nombre de una función javascript que SimpleModal nos llamará cuando deba abrir el formulario modal.

Una particularidad de SimpleModal es que si usamos la función de callback onOpen, debemos encargarnos nosotros de abrir el formulario modal. Así el código de la función popup_open queda así:

```js
function popup_open(dialog) {
    dialog.overlay.show(1);
    dialog.container.show(1);
    dialog.data.show(1);
}
```

Debemos llamar al método [show][4] (un método estándar definido en jQuery) con los tres elementos que componen el formulario modal de SimpleModal: Overlay (que lo que hace es inhabilitar el resto de la pantalla), Container (que muestra el borde del formulario) y Data (que muestra el contenido del formulario).

Otra forma &ldquo;más jQuery&rdquo; de realizar lo mismo es encadenar las llamadas: Generalmente todos los métodos jQuery aceptan un parámetro _callback_ con código a realizar cuando se termine el método. Así también podríamos escribir la función popup_open como:

```js
function popup_open(dialog) {
    dialog.overlay.show(1,function() {
        dialog.container.show(1,function() {
            dialog.data.show(1);
        });
    });
}
```

El siguiente paso es añadir el código para suscribirnos al evento onKeyPress del textbox cuyo id era &ldquo;nick&rdquo;. jQuery unifica todos los eventos de los distintos browsers en un conjunto de eventos propio, lo que permite más fácilmente desarrollar aplicaciones cross-browser. El método keypress de un _objeto jQuery_ permite suscribir un _callback_ al evento de pulsación de una tecla. Un _objeto jQuery_ es un objeto javascript que se obtiene generalmente usando la función selector (comúnmente llamada $) de jQuery. Así la llamada:

```js
$("#popup")
```

Me devuelve el _objeto jQuery_ asociado al elemento DOM cuyo ID sea &ldquo;popup&rdquo;. Esto **no** es equivalente a document.getElementById(&ldquo;popup&rdquo;) que me devuelve el objeto DOM directamente... es mucho mejor, ya que sobre el objeto jQuery puedo usar todas las propiedades de jQuery (como el método show() que hemos visto antes o el método modal() que define SimpleModal)!

Así pues, para suscribirnos al keypress del textbox cuyo ID es &ldquo;nick&rdquo; usando jQuery, el código de popup_open queda:

```js
function popup_open(dialog) {
    dialog.overlay.show(1, function() {
        dialog.container.show(1, function() {
            dialog.data.show(1, function() {
                $("#nick").keypress(function(e) { });
            });
        });
    });
}
```

Ahora sólo nos rellenar la función anónima que pasamos como parámetro a la llamada a keypress con el código que realice una petición Ajax a un controlador para que compruebe si el nick que se ha entrado está libre o no. Para ello vamos a usar la función [getJSON de jQuery][5], que lo que hace es realizar una petición Ajax a la URL especificada, esperar la respuesta en [formato JSON][6], deserializar la respuesta en un objeto Javascript y ejecutar el método de _callback_ que nosotros le indiquemos.

Así, pues usando getJSON el código de popup_open queda así:

```js
function popup_open(dialog) {
    dialog.overlay.show(1, function() {
        dialog.container.show(1, function() {
            dialog.data.show(1, function() {
                $("#nick").keypress(function(e) {
                    if (e.which != 13 && e.which != 8 
                        && e.which != 0) {
                        var str = this.value + 
                            String.fromCharCode(e.which);
                        var url = "/Account/Check";
                        $.getJSON(url, { nick: escape(str) }, 
                        function(data) {
                            if (data.existeix) {
                                $("#invalid_nick").show();
                            }
                            else {
                                $("#invalid_nick").hide();
                            }
                        });
                    }
                });
            });
        });
    });
}
```

Dentro de la función gestora del evento `keypress`:

  1. Miramos que la tecla pulsada NO sea enter, backspace o tabulador
  2. Llamamos a getJSON con la URL /Account/Check (parámetro 1), con el valor del textbox codificado como parámetro (parámetro 2) y la función de callback que queremos ejecutar cuando recibamos la respuesta del servidor (parámetro 3 que es un método anónimo). 
      1. Dentro del método anónimo, miramos si el valor del campo &ldquo;existeix&rdquo; del objeto recibido como parámetro es _true_ para mostrar u ocultar un objeto cuyo ID es &ldquo;invalid_nick&rdquo;.

¿Que nos queda por hacer? Pues por un lado modificar la vista parcial que es el formulario modal (en mi caso era SignupPopup.ascx), para añadir un `<div>` con un id &ldquo;invalid_nick&rdquo; con un mensaje que ponga &ldquo;NICK INCORRECTO&rdquo; (o algo así). P.ej:

```js
<div id="invalid_nick" style="display:none">
    NICK IS INVALID
</div>
```

Inicialmente lo tenemos oculto (evidentemente usaríamos CSS y alguna imágen para hacerlo más &ldquo;bonito&rdquo;), puesto que lo mostramos via jQuery.

Por último lo que nos queda es hacer la función correspondiente en el controlador. En mi caso el controlador es AccountController y la acción es &ldquo;Check&rdquo; (como se puede deducir de la URL /Account/Check):

```csharp
public ActionResult Check(string nick)
{
    return new JsonResult() {
        Data = new {existeix = nick.Length % 2 == 0 }
    };
}
```

Esta acción en lugar de devolver una vista, devuelve un objeto serializado en JSON, usando JsonResult. Básicamente cuando queráis devolver un objeto codificado en JSON usando ASP.NET MVC:

  1. Creais un JsonResult.
  2. A la propiedad Data la asignais el objeto a serializar.

Esta función devuelve un objeto con una propiedad &ldquo;existeix&rdquo; que vale true si el nick tiene un número par de carácteres.

&iexcl;Ya lo tenemos todo listo! Ahora si vais tecleando carácteres en el formulario, se ve como se muestra o se oculta la etiqueta &ldquo;NICK IS INVALID&rdquo;.

¿Fácil, verdad?

Saludos!

PD: Recordais lo que os dije, que cuando usabamos el callback onOpen de SimpleModal debíamos &ldquo;abrir&rdquo; manualmente el overlay, el container y la data y que eso nos daba capacidades interesantes? Este &ldquo;interesantes&rdquo; viene por la API de [animación de jQuery][7]. P.ej. si podríamos cambiar los show() por llamadas a fadeIn para que la aparición del formulario sea más espectacular:

```js
function popup_open(dialog) {
    dialog.overlay.fadeIn('slow', function() {
        dialog.container.fadeIn('slow', function() {
            dialog.data.fadeIn('slow', function() {
                // A partir de aquí todo igual...
           });
        });
    });
}  
```

&iexcl;Y observad como se despliega suavemente el formulario! 😉

 [1]: /blogs/etomas/archive/2009/04/14/mostrar-un-formulario-modal-con-asp-net-mvc-y-ajax.aspx
 [2]: http://www.ericmmartin.com/projects/simplemodal/
 [4]: http://docs.jquery.com/Effects/show
 [5]: http://docs.jquery.com/Ajax/jQuery.getJSON
 [6]: http://www.json.org/
 [7]: http://docs.jquery.com/Effects