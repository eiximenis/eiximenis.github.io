---
title: Y el combate se decidió por KO (vi)–Validaciones
author: eiximenis

date: 2012-10-05T12:06:25+00:00
geeks_url: /?p=1613
geeks_visits:
  - 1715
geeks_ms_views:
  - 722
categories:
  - Uncategorized

---
Bueno… sigamos con otro post sobre <a href="http://geeks.ms/blogs/etomas/archive/tags/knockout/default.aspx" target="_blank" rel="noopener noreferrer">esta serie en la que vamos hablando de cosas sobre knockout</a>. Y en esta ocasión toca hablar de como validar los campos que tengamos enlazados en un formulario.

Dado que estamos moviendo toda nuestra lógica al viewmodel de cliente, es lógico asumir que las validaciones irán también en él, en lugar de estar “ancladas” al DOM como ocurre cuando usamos jQuery validate, p.ej. Si usamos knockout lo normal es tener los campos de nuestro formulario enlazados con propiedades de nuestro viewmodel.

**Extendiendo observables**

Lo primero que necesitamos para poder aplicar una validación en un viewmodel de knockout es poder colocar código en las propiedades del viewmodel. De esa forma podremos poner el código que queramos cuando se establezca el valor de dicha propiedad. Pues bien ello es posible, pero con una condición: que dichas propiedades sean _observables_. Para ello se usa la técnica de extender un observable a través del objeto ko.extenders.

Retomemos el ejemplo del post anterior y añadamos una vista de edición de cervezas. Para ello, creamos un método en el controlador de WebApi para que nos devuelva una cerveza:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: blue">public</span> <span style="color: #2b91af">Beer</span> GetById(<span style="color: blue">int</span> id)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">return</span> beers[id];
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Y luego creamos una vista para el controlador HomeController que llamaremos Edit. El código inicial puede ser algo como:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="background: yellow">@model </span><span style="color: blue">int</span>
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    <span style="background: yellow">@{</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; ViewBag.Title = <span style="color: #a31515">"Edit Beer"</span>;
  </p>
  
  <p style="margin: 0px">
    <span style="background: yellow">}</span>
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    <span style="background: yellow">@section scripts</span>
  </p>
  
  <p style="margin: 0px">
    <span style="background: yellow">{</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">script</span> <span style="color: red">type</span><span style="color: blue">="text/javascript"></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; $(document).ready(<span style="color: blue">function</span>() {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">var</span> getUri = <span style="color: #a31515">"</span><span style="background: yellow">@</span>Url.RouteUrl(<span style="color: #a31515">"DefaultApi"</span>, <span style="color: blue">new</span> {httproute=<span style="color: #a31515">""</span>, controller=<span style="color: #a31515">"Beers"</span>, id=Model.ToString()})<span style="color: #a31515">"</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; $.getJSON(getUri, <span style="color: blue">function</span> (data)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; createViewModel(data);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; });
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; });
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">function</span> createViewModel(jsonData)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">var</span> vm = {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; name: ko.observable(jsonData.Name),
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; brewery: ko.observable(jsonData.Brewery),
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; ibu: ko.observable(jsonData.Ibu)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; };
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; ko.applyBindings(vm);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue"></</span><span style="color: maroon">script</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    <span style="background: yellow">}</span>
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    <span style="background: yellow">@</span><span style="color: blue">using</span> (Html.BeginForm())
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">fieldset</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">legend</span><span style="color: blue">></span>Edit Beer<span style="color: blue"></</span><span style="color: maroon">legend</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">div</span> <span style="color: red">class</span><span style="color: blue">="editor-label"></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">label</span> <span style="color: red">for</span><span style="color: blue">="name"></span>Name:<span style="color: blue"></</span><span style="color: maroon">label</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"></</span><span style="color: maroon">div</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">div</span> <span style="color: red">class</span><span style="color: blue">="editor-field"></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">input</span> <span style="color: red">type</span><span style="color: blue">="text"</span> <span style="color: red">id</span><span style="color: blue">="name"</span> <span style="color: red">data-bind</span><span style="color: blue">="value: name"/></span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"></</span><span style="color: maroon">div</span><span style="color: blue">></span>
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">div</span> <span style="color: red">class</span><span style="color: blue">="editor-label">< /span></p> 
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">label</span> <span style="color: red">for</span><span style="color: blue">="brewery"></span>Brewery<span style="color: blue"></</span><span style="color: maroon">label</span><span style="color: blue">></span>
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"></</span><span style="color: maroon">div</span><span style="color: blue">></span>
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">div</span> <span style="color: red">class</span><span style="color: blue">="editor-field"></span>
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">input</span> <span style="color: red">type</span><span style="color: blue">="text"</span> <span style="color: red">id</span><span style="color: blue">="brewery"</span> <span style="color: red">data-bind</span><span style="color: blue">="value: brewery"/></span>
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"></</span><span style="color: maroon">div</span><span style="color: blue">></span>
    </p>
    
    <p style="margin: 0px">
      &#160;
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">div</span> <span style="color: red">class</span><span style="color: blue">="editor-label"></span>
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">label</span> <span style="color: red">for</span><span style="color: blue">="ibu"></span>Ibu<span style="color: blue"></</span><span style="color: maroon">label</span><span style="color: blue">></span>
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"></</span><span style="color: maroon">div</span><span style="color: blue">></span>
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">div</span> <span style="color: red">class</span><span style="color: blue">="editor-field"></span>
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">input</span> <span style="color: red">type</span><span style="color: blue">="text"</span> <span style="color: red">id</span><span style="color: blue">="ibu"</span> <span style="color: red">data-bind</span><span style="color: blue">="value: ibu"/></span>
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"></</span><span style="color: maroon">div</span><span style="color: blue">></span>
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">p</span><span style="color: blue">></span>
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">input</span> <span style="color: red">type</span><span style="color: blue">="button"</span> <span style="color: red">value</span><span style="color: blue">="Save"</span> <span style="color: red">id</span><span style="color: blue">="cmdSave"/></span>
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue"></</span><span style="color: maroon">p</span><span style="color: blue">></span>
    </p>
    
    <p style="margin: 0px">
      &#160;&#160;&#160; <span style="color: blue"></</span><span style="color: maroon">fieldset</span><span style="color: blue">></span>
    </p>
    
    <p style="margin: 0px">
      }
    </p></p></div> 
    
    <p>
      La vista recibe el ID de la cerveza a editar, hace la llamada via Ajax para obtener esos datos, crea el viewmodel de knockout y muestra los campos de la UI enlazados.
    </p>
    
    <p>
      Vamos a validar los IBUs. Los IBUs son una medida que indican la amargura de una cerveza. Una lager normal suele estar alrededor de los 30-35 IBUs mientras que una IPA puede alcanzar los 150. Así que vamos a meter una validación para que el valor de los IBUs sea numérico 🙂
    </p>
    
    <p>
      Para ello vamos a extender el observable ibu. Para ello debemos añadir la función que extiende el observable en ko.extenders. Extender un observable significa declarar una función que se ejecutará cuando se acceda a dicho observable. Dicha función puede hacer algo y debe devolver un observable (que es el que se terminará enlazando al campo enlazado al observable inicial). Usualmente se devuelve el propio observable original, pero se puede devolver un observable calculado que use el original de cualquier manera.
    </p>
    
    <p>
      Empecemos pues por declarar una función que valide si ibu es un número. Para ello añadimos al principio de la función createViewModel:
    </p>
    
    <div style="font-size: 10pt; font-family: consolas; background: white; color: black">
      <p style="margin: 0px">
        ko.extenders.numericval = <span style="color: blue">function</span> (target, params) {
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: blue">function</span> validate(value) {
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">return</span>&#160; value % 1 == 0;
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; }
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; validate(target());
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; target.subscribe(validate);
      </p>
      
      <p style="margin: 0px">
        &#160;
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: blue">return</span> target;
      </p>
      
      <p style="margin: 0px">
        };
      </p></p>
    </div>
    
    <p>
      En este código hacemos cuatro cosas:
    </p>
    
    <ol>
      <li>
        Definimos y añadimos el campo numericval dentro de ko.extenders. Dicho campo es una función.
      </li>
      <li>
        Dentro de dicha función definimos otra función interna que es la que realmente comprueba si un valor es entero o no.
      </li>
      <li>
        Realizamos una primera validación (del valor inicial del observable)
      </li>
      <li>
        Suscribimos la función validate al observable. Cada vez que el observable se modifique la función validate se ejecutará.
      </li>
    </ol>
    
    <p>
      Si ahora ejecutáramos el código veríamos que no ocurre nada. El código dentro de ko.extenders.numericval no se ejecuta nunca. La razón es que hemos definido la extensión del observable <em>pero nos falta extender el observable en sí</em>. Eso se consigue con el método extend. Este método recibe un objeto json con una propiedad. El nombre de la propiedad es la extensión a aplicar y el valor de la propiedad es el valor que tomará el segundo parámetro (<em>params</em>) de la extensión que hemos creado (el primer parámetro <em>target</em> es el observable que extendemos):
    </p>
    
    <div style="font-size: 10pt; font-family: consolas; background: white; color: black">
      <p style="margin: 0px">
        <span style="color: blue">var</span> vm = {
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; name: ko.observable(jsonData.Name),
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; brewery: ko.observable(jsonData.Brewery),
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; ibu: ko.observable(jsonData.Ibu).extend({ numericval: <span style="color: #a31515">""</span> })
      </p>
      
      <p style="margin: 0px">
        };
      </p></p>
    </div>
    
    <p>
      Ahora sí! Hemos definido una extensión (numericval) y la hemos aplicado al observable ibu. Le pasamos “” como parámetros (que tampoco usamos para nada).
    </p>
    
    <p>
      Pero esta extensión todavía NO hace nada. Es decir, si depuráis el código javascript veréis que se efectivamente cada vez que se modifica el observable ibu se ejecuta el método<br /> validate, pero este método no hace nada salvo devolver true o false.
    </p>
    
    <p>
      ¿Como podemos informar al usuario de que el valor es incorrecto?
    </p>
    
    <p>
      <strong>Subobservables</strong>
    </p>
    
    <p>
      Pues la solución a la pregunta anterior pasa por usar un “subobservable”. Un subobservable es un observable <em>definido dentro de otro observable</em>. Vamos pues a crear un subobservable, que llamaremos hasError. Este subobservable lo definiremos <em>dentro</em> del observable que extendamos, es decir dentro de <em>target.</em> Luego modificaremos validate() para que en lugar de devolver true o false, establezca el valor del subobservable hasError:
    </p>
    
    <div style="font-size: 10pt; font-family: consolas; background: white; color: black">
      <p style="margin: 0px">
        ko.extenders.numericval = <span style="color: blue">function</span> (target, params) {
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; target.hasError = ko.observable();
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: blue">function</span> validate(value) {
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160;&#160;&#160;&#160;&#160; target.hasError(!(value % 1 == 0));
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; }
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; validate(target());
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; target.subscribe(validate);
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: blue">return</span> target;
      </p>
      
      <p style="margin: 0px">
        };
      </p></p>
    </div>
    
    <p>
      Y ya casi estamos! Ahora nos queda un punto que es… enlazar un elemento de la UI al subobservable. ¿Y como enlazamos a un subobservable? Pues del mismo modo que enlazamos a un observable normal, salvo que ahora el nombre del observable es “observable.subobservable”. Así si queremos enlazar algo al valor del subobservable hasError, deberemos usar “ibu.hasError” como nombre:
    </p>
    
    <div style="font-size: 10pt; font-family: consolas; background: white; color: black">
      <p style="margin: 0px">
        <span style="color: blue"><</span><span style="color: maroon">div</span> <span style="color: red">class</span><span style="color: blue">="editor-field"></span>
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">input</span> <span style="color: red">type</span><span style="color: blue">="text"</span> <span style="color: red">id</span><span style="color: blue">="ibu"</span> <span style="color: red">data-bind</span><span style="color: blue">="value: ibu, css: {&#8216;input-validation-error&#8217;: ibu.hasError, &#8216;field-validation-error&#8217; : ibu.hasError}"/></span>
      </p>
      
      <p style="margin: 0px">
        <span style="color: blue"></</span><span style="color: maroon">div</span><span style="color: blue">></span>
      </p></p>
    </div>
    
    <p>
      Fijaos en el atributo data-bind del <input>. Ahora estoy usando dos bindings:
    </p>
    
    <ol>
      <li>
        value (que ya usábamos) para enlazar el valor
      </li>
      <li>
        css para aplicar clases css en función del valor del observable. El binding css toma un objeto json, en el cual los nombres de las propiedades son las clases que se aplicarán y el valor de dichar propiedades es un campo del viewmodel. Si vale true se aplica la clase y si vale false no. En este caso: <ol>
          <li>
            Se aplicará input-validation-error si ibu.hasError vale true
          </li>
          <li>
            Se aplicará field-validation-error si ibu.hasError vale true
          </li>
        </ol>
      </li>
    </ol>
    
    <blockquote>
      <p>
        <strong>Nota:</strong> En este caso he puesto los nombres de las clases entre comillas simples porque <strong>esos nombres no son identificadores javascript válidos</strong>. Si los nombres de las clases a aplicar son identificadores javascript válidos, no es necesario entrecomillarlos.
      </p>
    </blockquote>
    
    <p>
      Por supuesto podríamos añadir un mensaje de error, p.ej. en un <span> que se visualice si ibu.hasError vale true. Para ello podemos usar el binding visible que tiene knockout:
    </p>
    
    <div style="font-size: 10pt; font-family: consolas; background: white; color: black">
      <p style="margin: 0px">
        <span style="color: blue"><</span><span style="color: maroon">div</span> <span style="color: red">class</span><span style="color: blue">="editor-field"></span>
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">input</span> <span style="color: red">type</span><span style="color: blue">="text"</span> <span style="color: red">id</span><span style="color: blue">="ibu"</span> <span style="color: red">data-bind</span><span style="color: blue">="value: ibu, css: {&#8216;input-validation-error&#8217;: ibu.hasError, &#8216;field-validation-error&#8217; : ibu.hasError}"/></span>
      </p>
      
      <p style="margin: 0px">
        &#160;&#160;&#160; <span style="color: blue"><</span><span style="color: maroon">span</span> <span style="color: red">data-bind</span><span style="color: blue">="visible: ibu.hasError"</span> <span style="color: red">class</span><span style="color: blue">="message-error"></span>Error: Debe ser numérico<span style="color: blue"></</span><span style="color: maroon">span</span><span style="color: blue">></span>
      </p>
      
      <p style="margin: 0px">
        <span style="color: blue"></</span><span style="color: maroon">div</span><span style="color: blue">></span>
      </p></p>
    </div>
    
    <p>
      ¡Y listos!
    </p>
    
    <p>
      Aquí tenéis el resultado:
    </p>
    
    <p>
      <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_2782537E.png"><img title="image" style="border-left-width: 0px; border-right-width: 0px; background-image: none; border-bottom-width: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-top-width: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_17761578.png" width="244" height="160" /></a>
    </p>
    
    <p>
      Un tema que se observa si se ejecuta el código es que el observable ibu no se modifica cada vez que tecleamos, si no tan solo cuando el textbox pierde el foco. Pero como ya vimos en el post anterior, eso tiene fácil solución.&#160; Basta con usar valueUpdate:
    </p>
    
    <div style="font-size: 10pt; font-family: consolas; background: white; color: black">
      <p style="margin: 0px">
        <span style="color: blue"><</span><span style="color: maroon">input</span> <span style="color: red">type</span><span style="color: blue">="text"</span> <span style="color: red">id</span><span style="color: blue">="ibu"</span> <span style="color: red">data-bind</span><span style="color: blue">="value: ibu, valueUpdate: &#8216;afterkeydown&#8217; , </span>
      </p>
      
      <p style="margin: 0px">
        <span style="color: blue">&#160;&#160;&#160; css: {&#8216;input-validation-error&#8217;: ibu.hasError, &#8216;field-validation-error&#8217; : ibu.hasError}"/></span>
      </p></p>
    </div>
    
    <p>
      Y con eso hemos terminado. Apuntar solamente que las extensiones se “almacenan” en ko.extenders y son independientes del viewmodel (recordad que debemos “aplicarlas” usando extend). Por lo tanto podemos tener nuestras librerías de “validaciones” con knockout!
    </p>
    
    <p>
      Espero que os haya resultado interesante!
    </p>
    
    <p>
      Os dejo el ejemplo completo en my skydrive: <a title="https://skydrive.live.com/redir?resid=6521C259E9B1BEC6!174" href="https://skydrive.live.com/redir?resid=6521C259E9B1BEC6!174">https://skydrive.live.com/redir?resid=6521C259E9B1BEC6!174</a> (Nota el zip es KoDemoVI.zip, aunque la solución y la carpeta luego se llamen KoDemoV). Para probar la edición basta con ir a /Home/Edit/{id}, pasando el id de la cerveza.
    </p>
    
    <p>
      Saludos!
    </p>