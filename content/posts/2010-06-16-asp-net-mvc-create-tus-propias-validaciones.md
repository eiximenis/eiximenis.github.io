---
title: 'ASP.NET MVC: Create tus propias validaciones'
author: eiximenis

date: 2010-06-16T12:02:00+00:00
geeks_url: /?p=1518
geeks_visits:
  - 7930
geeks_ms_views:
  - 4743
categories:
  - Uncategorized

---
Una de las noverdades de ASP.NET MVC 2 es que lleva integrado el uso de <a target="_blank" href="http://msdn.microsoft.com/en-us/library/dd901590(VS.95).aspx" rel="noopener noreferrer">Data Annotations</a> para permitirnos validar los modelos. <a target="_blank" href="http://www.asp.net/mvc/tutorials/validation-with-the-data-annotation-validators-cs" rel="noopener noreferrer">En ASP.NET MVC 1 también era posible</a> pero no era un proceso tan integrado como con la nueva versión.

Mediante Data Annotations podemos indicar un conjunto de reglas que deben cumplir las propiedades de nuestros modelos. Así para indicar que el campo _Login_ es obligatorio basta con decorar la propiedad correspondiente:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> UserData<br />{<br />    [Required(ErrorMessage=<span style="color: #006080">"El nombre de usuario NO puede estar vacío"</span>)]<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Login { get; set; }<br />}</pre>
  
  <p>
    </div> 
    
    <p>
      Y luego dejar que el sistema de ASP.NET MVC 2 haga &ldquo;la magia&rdquo;: cuando el model binder deba reconstruir el objeto <em>UserData</em> a partir de los datos de la request, si no existe el dato para la propiedad Login, automáticamente nos pondrá el ModelState a inválido:
    </p>
    
    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">[HttpPost()]<br /><span style="color: #0000ff">public</span> ActionResult Index(UserData data)<br />{<br />   <span style="color: #0000ff">if</span> (!ModelState.IsValid) {<br />       <span style="color: #008000">// Hay errores en el modelo (data.Login está vacío)</span><br />       <span style="color: #0000ff">return</span> View(data);<br />   }<br /><br />   <span style="color: #008000">// El modelo es correcto... realizar tareas</span><br />}<br /></pre>
      
      <p>
        </div> 
        
        <p>
          Pero bueno... lo normal es que los atributos que vienen de serie se te queden &ldquo;cortos&rdquo; y que tarde o temprano necesites crear tus propias validaciones... y este es el motivo de este post.
        </p>
        
        <p>
          Vamos a realizar un ejemplo sencillo: crearemos una validación que indique si una propiedad string contiene una representación válida de un número entero. Vamos a soportar notación decimal, hexadecimal (prefijada por 0x), octal (prefijada por 0) y binaria (prefijada por 0b). Así:
        </p>
        
        <ul>
          <li>
            1234567890 es una cadena válida (decimal)
          </li>
          <li>
            0xaa112d es una cadena válida (hexadecimal prefijada por 0x)
          </li>
          <li>
            01239 es una cadena inválida (prefijo 0 indica octal y 9 no es carácter octal).
          </li>
          <li>
            0b00112101 es una cadena inválida (prefijo 0b indica binario y el carácter 2 no es binario).
          </li>
        </ul>
        
        <p>
          Vamos a ello?
        </p>
        
        <p>
          <strong>1. Creación de la validación en el servidor</strong>
        </p>
        
        <p>
          Para crear una validación en el servidor debemos crear un nuevo <em>atributo</em> que herede de <a target="_blank" href="http://msdn.microsoft.com/en-us/library/system.componentmodel.dataannotations.validationattribute.aspx" rel="noopener noreferrer">ValidationAttribute</a> y que contenga él código de la validación sobrecargando el método <em>IsValid</em>:
        </p>
        
        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> DeveloperIntegerAttribute : ValidationAttribute<br />{<br />    <span style="color: #0000ff">protected</span> <span style="color: #0000ff">override</span> ValidationResult IsValid(<span style="color: #0000ff">object</span> <span style="color: #0000ff">value</span>, ValidationContext validationContext)<br />    {<br />        <span style="color: #0000ff">bool</span> valid = <span style="color: #0000ff">false</span>;<br />        <span style="color: #0000ff">if</span> (<span style="color: #0000ff">value</span> <span style="color: #0000ff">is</span> <span style="color: #0000ff">string</span> && <span style="color: #0000ff">value</span> != <span style="color: #0000ff">null</span>)<br />        {<br />            <span style="color: #0000ff">string</span> sval = <span style="color: #0000ff">value</span> <span style="color: #0000ff">as</span> <span style="color: #0000ff">string</span>;                <br />            <span style="color: #0000ff">if</span> (sval.StartsWith(<span style="color: #006080">"0x"</span>) || sval.StartsWith(<span style="color: #006080">"0X"</span>))<br />            {<br />                valid = CheckChars(sval.Skip(2), <span style="color: #006080">"0123456789abcdefABCDEF"</span>);<br />            }<br />            <span style="color: #0000ff">else</span> <span style="color: #0000ff">if</span> (sval.StartsWith(<span style="color: #006080">"0b"</span>) || sval.StartsWith(<span style="color: #006080">"0B"</span>))<br />            {<br />                valid = CheckChars(sval.Skip(2), <span style="color: #006080">"01"</span>);<br />            }<br />            <span style="color: #0000ff">else</span> <span style="color: #0000ff">if</span> (sval.StartsWith(<span style="color: #006080">"0"</span>))<br />            {<br />                valid = CheckChars(sval.Skip(1), <span style="color: #006080">"01234567"</span>);<br />            }<br />            <span style="color: #0000ff">else</span><br />            {<br />                valid = CheckChars(sval, <span style="color: #006080">"0123456789"</span>);<br />            }                <br />        }<br />        <span style="color: #0000ff">return</span> valid ? ValidationResult.Success : <span style="color: #0000ff">new</span> ValidationResult(ErrorMessage);<br />    }<br /><br />    <span style="color: #0000ff">private</span> <span style="color: #0000ff">bool</span> CheckChars(IEnumerable&lt;<span style="color: #0000ff">char</span>&gt; str, <span style="color: #0000ff">string</span> validchars)<br />    {<br />        <span style="color: #0000ff">return</span> str.All(x =&gt; validchars.Contains(x));<br />    }<br />}</pre>
          
          <p>
            </div> 
            
            <p>
              Como podéis ver el código es realmente sencillo: debemos comprobar que el parámetro &ldquo;value&rdquo; satisface nuestras condiciones y en este caso devolver <em>ValidationResult.Success</em> y en caso contrario devolver un <a target="_blank" href="http://msdn.microsoft.com/en-us/library/system.componentmodel.dataannotations.validationresult.aspx" rel="noopener noreferrer">ValidationResult</a> con el mensaje de error asociado.
            </p>
            
            <p>
              Ahora ya podemos aplicar nuestro nuevo flamante atributo [DeveloperInteger] a nuestras clases de modelo:
            </p>
            
            <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
              <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> FooModel<br />{<br />    [Required]<br />    [DeveloperInteger(ErrorMessage=<span style="color: #006080">"Numero no correcto (se aceptan prefijos 0 - octal, 0x - hexa, 0b - binario y ninguno para decimal"</span>)]<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> DeveloperNumber { get; set; }<br /><br />    [Required]<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> OtraCadena {get; set;}<br />}</pre>
              
              <p>
                </div> 
                
                <p>
                  Y listos! Con esto la validación ya está integrada en el sistema de ASP.NET MVC. Podemos comprobarlo si creamos una vista para crear datos de tipo FooModel:
                </p>
                
                <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #006080">&lt;</span>%@ Page Language="C#" Inherits="System<span style="color: #cc6633">.Web</span><span style="color: #cc6633">.Mvc</span><span style="color: #cc6633">.ViewPage</span><span style="color: #006080">&lt;</span>CustomValidations<span style="color: #cc6633">.Models</span><span style="color: #cc6633">.FooModel</span><span style="color: #006080">&gt;</span>" %<span style="color: #006080">&gt;</span><br /><span style="color: #006080">&lt;</span>!DOCTYPE <span style="color: #0000ff">html</span> PUBLIC "-<span style="color: #008000">//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd"&gt;</span><br /><span style="color: #006080">&lt;</span><span style="color: #0000ff">html</span> xmlns="http:<span style="color: #008000">//www.w3.org/1999/xhtml" &gt;</span><br /><span style="color: #006080">&lt;</span><span style="color: #0000ff">head</span> runat="server"<span style="color: #006080">&gt;</span><br />    <span style="color: #006080">&lt;</span><span style="color: #0000ff">title</span><span style="color: #006080">&gt;</span>Index<span style="color: #006080">&lt;</span>/<span style="color: #0000ff">title</span><span style="color: #006080">&gt;</span><br /><span style="color: #006080">&lt;</span>/<span style="color: #0000ff">head</span><span style="color: #006080">&gt;</span><br /><span style="color: #006080">&lt;</span><span style="color: #0000ff">body</span><span style="color: #006080">&gt;</span><br />    <span style="color: #006080">&lt;</span>% using (Html<span style="color: #cc6633">.BeginForm</span>()) {%<span style="color: #006080">&gt;</span><br />        <span style="color: #006080">&lt;</span>%: Html<span style="color: #cc6633">.ValidationSummary</span>(true) %<span style="color: #006080">&gt;</span><br />        <span style="color: #006080">&lt;</span><span style="color: #0000ff">fieldset</span><span style="color: #006080">&gt;</span><br />            <span style="color: #006080">&lt;</span><span style="color: #0000ff">legend</span><span style="color: #006080">&gt;</span>Fields<span style="color: #006080">&lt;</span>/<span style="color: #0000ff">legend</span><span style="color: #006080">&gt;</span>            <br />            <span style="color: #006080">&lt;</span><span style="color: #0000ff">div</span> class="editor-<span style="color: #0000ff">label</span>"<span style="color: #006080">&gt;</span><br />                <span style="color: #006080">&lt;</span>%: Html<span style="color: #cc6633">.LabelFor</span>(model =<span style="color: #006080">&gt;</span> model<span style="color: #cc6633">.DeveloperNumber</span>) %<span style="color: #006080">&gt;</span><br />            <span style="color: #006080">&lt;</span>/<span style="color: #0000ff">div</span><span style="color: #006080">&gt;</span><br />            <span style="color: #006080">&lt;</span><span style="color: #0000ff">div</span> class="editor-field"<span style="color: #006080">&gt;</span><br />                <span style="color: #006080">&lt;</span>%: Html<span style="color: #cc6633">.TextBoxFor</span>(model =<span style="color: #006080">&gt;</span> model<span style="color: #cc6633">.DeveloperNumber</span>) %<span style="color: #006080">&gt;</span><br />                <span style="color: #006080">&lt;</span>%: Html<span style="color: #cc6633">.ValidationMessageFor</span>(model =<span style="color: #006080">&gt;</span> model<span style="color: #cc6633">.DeveloperNumber</span>) %<span style="color: #006080">&gt;</span><br />            <span style="color: #006080">&lt;</span>/<span style="color: #0000ff">div</span><span style="color: #006080">&gt;</span><br />            <span style="color: #006080">&lt;</span><span style="color: #0000ff">div</span> class="editor-<span style="color: #0000ff">label</span>"<span style="color: #006080">&gt;</span><br />                <span style="color: #006080">&lt;</span>%: Html<span style="color: #cc6633">.LabelFor</span>(model =<span style="color: #006080">&gt;</span> model<span style="color: #cc6633">.OtraCadena</span>) %<span style="color: #006080">&gt;</span><br />            <span style="color: #006080">&lt;</span>/<span style="color: #0000ff">div</span><span style="color: #006080">&gt;</span><br />            <span style="color: #006080">&lt;</span><span style="color: #0000ff">div</span> class="editor-field"<span style="color: #006080">&gt;</span><br />                <span style="color: #006080">&lt;</span>%: Html<span style="color: #cc6633">.TextBoxFor</span>(model =<span style="color: #006080">&gt;</span> model<span style="color: #cc6633">.OtraCadena</span>) %<span style="color: #006080">&gt;</span><br />                <span style="color: #006080">&lt;</span>%: Html<span style="color: #cc6633">.ValidationMessageFor</span>(model =<span style="color: #006080">&gt;</span> model<span style="color: #cc6633">.OtraCadena</span>) %<span style="color: #006080">&gt;</span><br />            <span style="color: #006080">&lt;</span>/<span style="color: #0000ff">div</span><span style="color: #006080">&gt;</span><br />            <span style="color: #006080">&lt;</span><span style="color: #0000ff">p</span><span style="color: #006080">&gt;</span><br />                <span style="color: #006080">&lt;</span><span style="color: #0000ff">input</span> type="submit" value="Create" /<span style="color: #006080">&gt;</span><br />            <span style="color: #006080">&lt;</span>/<span style="color: #0000ff">p</span><span style="color: #006080">&gt;</span><br />        <span style="color: #006080">&lt;</span>/<span style="color: #0000ff">fieldset</span><span style="color: #006080">&gt;</span><br />    <span style="color: #006080">&lt;</span>% } %<span style="color: #006080">&gt;</span><br />    <span style="color: #006080">&lt;</span><span style="color: #0000ff">div</span><span style="color: #006080">&gt;</span><br />        <span style="color: #006080">&lt;</span>%: Html<span style="color: #cc6633">.ActionLink</span>("Back to List", "Index") %<span style="color: #006080">&gt;</span><br />    <span style="color: #006080">&lt;</span>/<span style="color: #0000ff">div</span><span style="color: #006080">&gt;</span><br /><span style="color: #006080">&lt;</span>/<span style="color: #0000ff">body</span><span style="color: #006080">&gt;</span><br /><span style="color: #006080">&lt;</span>/<span style="color: #0000ff">html</span><span style="color: #006080">&gt;</span></pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Este código es el código <em>estándard</em> que genera Visual Studio si añadimos una vista tipada para <em>Crear</em> objetos FooModel. Fijaos en el uso de ValidationMessageFor para mostrar (en caso de que el modelo no esté correcto) los mensajes de error correspondientes.
                    </p>
                    
                    <p>
                      Y finalmente como siempre en el controlador, el par de métodos para mostrar la vista para entrar datos y para recoger los datos y mirar si el modelo es válido:
                    </p>
                    
                    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> ActionResult Index()<br />{<br />    <span style="color: #0000ff">return</span> View();<br />}<br /><br />[HttpPost()]<br /><span style="color: #0000ff">public</span> ActionResult Index(FooModel data)<br />{<br />    <span style="color: #0000ff">if</span> (!ModelState.IsValid)<br />    {<br />        <span style="color: #0000ff">return</span> View(data);<br />    }<br />    <span style="color: #0000ff">else</span><br />    {<br />        <span style="color: #008000">// modelo es correcto</span><br />        <span style="color: #0000ff">return</span> RedirectToAction(<span style="color: #006080">"Hola"</span>);<br />    }<br />}</pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          Más fácil imposible, no??? 🙂
                        </p>
                        
                        <p>
                          <strong>2. Validación en Javascript</strong>
                        </p>
                        
                        <p>
                          Crear validaciones en servidor es muy fácil, pero ahora lo que se lleva es validar los datos en el cliente. Esto hace nuestra aplicación más amigable ya que le evitamos esperas al usuario (no enviamos datos <em>incorrectos</em> al servidor).
                        </p>
                        
                        <blockquote>
                          <p>
                            <strong>Nota</strong>: Se ha repetido innumerables veces pero no está de más decirlo de nuevo: Las validaciones en cliente <strong>no pueden sustituir nunca</strong> a las validaciones en servidor. El objetivo de validar en cliente no es garantizar la seguridad ni la consistencia de los datos, és <strong>únicamente</strong> proporcionar mejor experiencia de usuario. Siempre debe validarse en servidor, siempre!
                          </p>
                        </blockquote>
                        
                        <p>
                          Para validar en cliente debemos realizar tres pasos: Habilitar la validación de cliente en la vista, activarla en servidor y crear el código javascript. Veamos cada uno de esos pasos.
                        </p>
                        
                        <p>
                          El primero es el más fácil, basta con añadir la llamada al método EnableClientValidation() que está en el HtmlHelper. Este método es de la Microsoft Ajax Library así que debéis referenciarla con tags <script>:
                        </p>
                        
                        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">&lt;head runat=<span style="color: #006080">"server"</span>&gt;<br />    &lt;script src=<span style="color: #006080">"../../Scripts/MicrosoftAjax.js"</span> type=<span style="color: #006080">"text/javascript"</span>&gt;&lt;/script&gt;<br />    &lt;script type=<span style="color: #006080">"text/javascript"</span> src=<span style="color: #006080">"../../Scripts/MicrosoftMvcValidation.js"</span>&gt;&lt;/script&gt; <br />    &lt;title&gt;Index&lt;/title&gt;<br />&lt;/head&gt;<br />&lt;body&gt;<br />    &lt;% Html.EnableClientValidation(); %&gt;</pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              De esta manera veréis p.ej. que automáticamente ya nos valida si los campos de texto están vacíos (porque usábamos [Required]). Obviamente nuestra validación propia, no la realiza en javascript... veamos como podemos hacerlo.
                            </p>
                            
                            <p>
                              Primero debemos <em>activar</em> la validación en servidor, y eso se consigue creando una clase derivada de <em><a target="_blank" href="http://msdn.microsoft.com/es-es/library/ee470840.aspx" rel="noopener noreferrer">DataAnnotationsModelValidator<TAttr></a></em> donde TAttr es el tipo del atributo que tiene la validación, en nuestro caso <em>DeveloperIntegerAttribute</em>. En esta clase debemos redefinir el método <em>GetClientValidationRules</em> para devolver la lista de validaciones en cliente a ejecutar (métodos javascript a llamar).
                            </p>
                            
                            <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                              <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> DeveloperIntegerValidator : DataAnnotationsModelValidator&lt;DeveloperIntegerAttribute&gt;<br />{<br />    <span style="color: #0000ff">private</span> <span style="color: #0000ff">string</span> message;<br />    <span style="color: #0000ff">public</span> DeveloperIntegerValidator(ModelMetadata metadata, ControllerContext cc, DeveloperIntegerAttribute attr)<br />        : <span style="color: #0000ff">base</span>(metadata, cc, attr)<br />    {<br />        message = attr.ErrorMessage;<br />    }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">override</span> IEnumerable&lt;ModelClientValidationRule&gt; GetClientValidationRules()<br />    {<br />        var rule = <span style="color: #0000ff">new</span> ModelClientValidationRule()<br />        {<br />            ValidationType = <span style="color: #006080">"devinteger"</span>,<br />            ErrorMessage = message<br />        };<br />        <span style="color: #008000">// Aquí podríamos añadir parámetros a la función javascript:</span><br />        <span style="color: #008000">// rule.ValidationParameters.Add("parametro", valor);</span><br />        <span style="color: #0000ff">return</span> <span style="color: #0000ff">new</span>[] { rule };<br />    }<br />}</pre>
                              
                              <p>
                                </div> 
                                
                                <p>
                                  Fijaos que devolvemos una colección de objetos <a target="_blank" href="http://msdn.microsoft.com/en-us/library/system.web.mvc.modelclientvalidationrule.aspx" rel="noopener noreferrer">ModelClientValidationRule</a> que representan los métodos javascript a llamar para realizar nuestra validación (podemos asociar más de uno).
                                </p>
                                
                                <p>
                                  Un paso adicional que debemos realizar es indicar que la clase <em>DevelolperIntegerValidator</em> gestiona las validaciones en cliente para <em>DeveloperIntegerAttribute</em> y para ello añadimos la siguiente línea en global.asax (p.ej. en Application_Start):
                                </p>
                                
                                <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                                  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">DataAnnotationsModelValidatorProvider.RegisterAdapter(<br />    <span style="color: #0000ff">typeof</span>(DeveloperIntegerAttribute), <span style="color: #0000ff">typeof</span>(DeveloperIntegerValidator));</pre>
                                  
                                  <p>
                                    </div> 
                                    
                                    <p>
                                      Ahora ya sólo nos queda crear nuestro método javascript. Realmente <strong>no tenemos</strong> una función llamada <em>devinteger</em> sino que accedemos a Sys.Mvc.ValidatorRegistry.validators y establecemos una entrada llamada <em>devinteger</em> cuyo valor es un método que recoge los parámetros de validación (si hubiese, en nuestro caso no hay) y devuelve <strong>otra función</strong> que es la que realiza la validación...
                                    </p>
                                    
                                    <p>
                                      Sí, parece complicado pero tampoco lo es tanto:
                                    </p>
                                    
                                    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                                      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">Sys.Mvc.ValidatorRegistry.validators[<span style="color: #006080">"devinteger"</span>] = <span style="color: #0000ff">function</span> (rule) {<br />    <span style="color: #008000">// Si tuvieramos un parametro llamado parametro lo recogeríamos aqui:</span><br />    <span style="color: #008000">// var parameter = rule.ValidationParameters["parametro"];</span><br />    <span style="color: #008000">// Debemos devolver la función que realiza la validación</span><br />    <span style="color: #0000ff">return</span> <span style="color: #0000ff">function</span> (value, context) {<br />        <span style="color: #0000ff">var</span> svalue = <span style="color: #006080">""</span> + value;<br />        <span style="color: #0000ff">var</span> chars = <span style="color: #006080">"0123456789"</span>;<br />        <span style="color: #0000ff">var</span> radix = 10;<br />        <span style="color: #0000ff">var</span> start = 0;<br />        <span style="color: #0000ff">if</span> (svalue.substr(0, 2) == <span style="color: #006080">"0x"</span> || svalue.substr(0, 2) == <span style="color: #006080">"0X"</span>) { chars = <span style="color: #006080">"01234567890abcdefABCDEF"</span>, start = 2; }<br />        <span style="color: #0000ff">else</span> <span style="color: #0000ff">if</span> (svalue.substr(0, 2) == <span style="color: #006080">"0b"</span> || svalue.substr(0, 2) == <span style="color: #006080">"0B"</span>) { chars = <span style="color: #006080">"01"</span>; start = 2; }<br />        <span style="color: #0000ff">else</span> <span style="color: #0000ff">if</span> (svalue.search(0, 1) == <span style="color: #006080">"0"</span>) { chars = <span style="color: #006080">"01234567"</span>; start = 1; }<br /><br />        <span style="color: #0000ff">return</span> (<span style="color: #0000ff">function</span> (str, chars) {<br />            <span style="color: #0000ff">var</span> ok = <span style="color: #0000ff">true</span>;<br />            <span style="color: #0000ff">for</span> (<span style="color: #0000ff">var</span> i = 0; i &lt; str.length; i++) {<br />                <span style="color: #0000ff">var</span> <span style="color: #0000ff">char</span> = str.charAt(i);<br />                ok = chars.indexOf(<span style="color: #0000ff">char</span>) != -1;<br />                <span style="color: #0000ff">if</span> (!ok) <span style="color: #0000ff">break</span>;<br />            }<br />            <span style="color: #0000ff">return</span> ok;<br />        })(svalue.substring(start), chars) ? <span style="color: #0000ff">true</span> : rule.ErrorMessage;<br />    };<br />};<br /></pre>
                                      
                                      <p>
                                        </div> 
                                        
                                        <p>
                                          Fijaos en el <em>return function (value, context)</em>. Aquí devolvemos la función anónima que realmente realiza la validación. Dicha función recibe dos parámetros: value, que es el valor a evaluar y context que es un contexto de validación. El código dentro de esta función anónima debe devolver <em>true</em> si el value valida satisfactoriamente y la cadena de error en caso contrario.
                                        </p>
                                        
                                        <p>
                                          Y listos! Ya tenemos la validación por javascript en cliente! Si alguien sabe alguna manera mejor de realizar dicha validación en javascript que me lo diga (sí, javascript no es mi fuerte). Yo intenté usar parseInt, pero dado que <a target="_blank" href="http://msdn.microsoft.com/es-es/library/1kc6b02f(VS.80).aspx" rel="noopener noreferrer">parseInt</a> valida los carácteres válidos hasta que encuentra el primer inválido no me servia (para mi 0x1j es inválido y no el número 1).
                                        </p>
                                        
                                        <p>
                                          Y listos! Ya tenemos la validación en cliente para nuestro validador propio... Que os parece? Fácil, no???? 🙂
                                        </p>
                                        
                                        <p>
                                          Un saludo!
                                        </p>