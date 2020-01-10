---
title: 'ASP.NET MVC3: Validación remota'

author: eiximenis

date: 2011-01-14T10:22:00+00:00
geeks_url: /?p=1550
geeks_visits:
  - 5856
geeks_ms_views:
  - 1891
categories:
  - Uncategorized

---
Muy buenas!

Una de las novedades que nos trae ASP.NET MVC3, con respecto a MVC2 es poder usar _fácilmente_ la validación remota: eso es, desde _cliente_ llamar a un método del servidor que nos diga si un dato (entrado p.ej. en un campo de texto es válido o no). Y cuando digo _fácilmente_ me refiero a _fácilmente, muy fácilmente_.

<!--more-->

Vamos a ver un ejemplo: para ello vamos a modificar la aplicación de ejemplo que crea MVC3 para que al darnos de alta, consulte si el usuario ya existe y si es el caso no nos deje. Os recuerdo que esa validación es Ajax, eso significa: el campo de texto pierda el foco se realizará la petición (en background) al servidor que comprobará si el usuario entrado ya existe y si es así mostrará un error en el campo de texto asociado.

Vamos pues, a verlo paso a paso 😉

**1. Viendo que nos genera MVC3**

Para empezar creamos un nuevo proyecto ASP.NET MVC3. Cuando os pida el template, usad el de _Internet Application_ (no uséis el _Empty)_. De esa manera MVC3 nos crea el esqueleto de la aplicación inicial:

[<img style="background-image: none; border-right-width: 0px; margin: ; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_61EB0D0E.png" width="215" height="244" />][1]

El fichero AccountModels.cs tiene distintos _Viewmodel_s que usan las acciones del controlador Account. La acción que da de alta un usuario es Register que está definida en el controlador Account:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">[HttpPost]<br /><span style="color: #0000ff">public</span> ActionResult Register(RegisterModel model)<br />{<br />    <span style="color: #008000">// Código...</span><br />}</pre>
  
  <p>
    </div> 
    
    <p>
      La acción usa la clase RegisterModel que es uno de los <em>viewmodel</em>s definidos en AccountModels.cs:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> RegisterModel<br />{<br />    [Required]<br />    [Display(Name = <span style="color: #006080">"User name"</span>)]<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> UserName { get; set; }<br /><br />    [Required]<br />    [DataType(DataType.EmailAddress)]<br />    [Display(Name = <span style="color: #006080">"Email address"</span>)]<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Email { get; set; }<br /><br />    [Required]<br />    [ValidatePasswordLength]<br />    [DataType(DataType.Password)]<br />    [Display(Name = <span style="color: #006080">"Password"</span>)]<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Password { get; set; }<br /><br />    [DataType(DataType.Password)]<br />    [Display(Name = <span style="color: #006080">"Confirm password"</span>)]<br />    [Compare(<span style="color: #006080">"Password"</span>, ErrorMessage = <span style="color: #006080">"The password and confirmation password do not match."</span>)]<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> ConfirmPassword { get; set; }<br />}</pre>
      
      <p>
        </div> 
        
        <p>
          <strong>2. Habilitando la validación remota</strong>
        </p>
        
        <p>
          La validación remota en MVC3 se habilita, como el resto de validaciones, usando DataAnnotations, concretamente con el atributo <a href="http://msdn.microsoft.com/en-us/library/system.web.mvc.remoteattribute(v=VS.98).aspx" target="_blank" rel="noopener noreferrer">Remote</a>. En nuestro caso queremos habilitar la validación remota sobre la propiedad UserName. Para ello añado el atributo Remote:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">[Required]<br />[Display(Name = <span style="color: #006080">"User name"</span>)]<br />[Remote(<span style="color: #006080">"CheckUserAvailability"</span>, <span style="color: #006080">"Account"</span>, ErrorMessage = <span style="color: #006080">"This user name is not allowed."</span>)]<br /><span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> UserName { get; set; }</pre>
          
          <p>
            </div> 
            
            <p>
              El primer parámetro es la acción que va a validar el campo, la segunda es el controlador. Finalmente ponemos el mensaje de error (igual que las otras validaciones).
            </p>
            
            <p>
              <strong>3. Creando el código de validación</strong>
            </p>
            
            <p>
              Finalmente tenemos que crear la acción del controlador:
            </p>
            
            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">[OutputCache(Location = OutputCacheLocation.None, NoStore = <span style="color: #0000ff">true</span>)]<br /><span style="color: #0000ff">public</span> ActionResult CheckUserAvailability(<span style="color: #0000ff">string</span> username)<br />{<br />    var validUserName = Membership.FindUsersByName(username).Count == 0;<br />    <span style="color: #0000ff">return</span> Json(validUserName, JsonRequestBehavior.AllowGet);<br />}</pre>
              
              <p>
                </div> 
                
                <p>
                  Fijaos lo simple que es: Usamos el membership provider para ver si existe algún otro usuario con el mismo nombre, y devolvemos un booleano (codificado en json). El uso de [<a href="http://msdn.microsoft.com/es-es/library/system.web.mvc.outputcacheattribute.aspx" target="_blank" rel="noopener noreferrer">OutputCache</a>] es para evitar que MVC me cachee los resultados de las peticiones.
                </p>
                
                <p>
                  Y listos! Con eso ya hemos terminado.
                </p>
                
                <p>
                  Podemos ejecutar la aplicación, registrar un usuario y luego intentar darnos de alta de nuevo y veremos el error:
                </p>
                
                <p>
                  <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_401F347D.png"><img style="background-image: none; border-right-width: 0px; margin: ; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_6D34214B.png" width="244" height="110" /></a>
                </p>
                
                <p>
                  <strong>4. Y que pasa “en la trastienda”?</strong>
                </p>
                
                <p>
                  Si inspeccionamos las peticiones veremos que <em>cada vez que cambia el texto</em> se realiza una petición Ajax. Esta vez en lugar de <a href="http://getfirebug.com/" target="_blank" rel="noopener noreferrer">firebug</a> he usado las <em><a href="http://msdn.microsoft.com/en-us/library/dd565628(v=vs.85).aspx" target="_blank" rel="noopener noreferrer">Developer Tools</a></em> de IE9 (pulsar F12 para que os aparezcan):
                </p>
                
                <p>
                  <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_3E0235B4.png"><img style="background-image: none; border-right-width: 0px; margin: ; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_136654D7.png" width="244" height="43" /></a>
                </p>
                
                <p>
                  Estas son las peticiones que se realizan si entro “eiximenis” en el campo de nombre de usuario. Si las contáis <em>veréis que faltan peticiones</em>. Eso es simplemente he pulsado dos teclas&#160; muy rápido y las peticiones supongo que no se encolan
                </p>
                
                <p>
                  Si miramos los detalles de una petición vemos lo siguiente:
                </p>
                
                <p>
                  <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_128DEEED.png"><img style="background-image: none; border-right-width: 0px; margin: ; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_263AD881.png" width="244" height="86" /></a>
                </p>
                
                <p>
                  Fijaos en el primer campo (Request) me indica la URL de la petición. En mi caso es /Account/CheckUserAvailability?UserName=eiximenis
                </p>
                
                <p>
                  Es decir se pasa el valor por querystring (eso no es problema alguno para MVC que soporta el binding de datos en el querystring desde siempre).
                </p>
                
                <p>
                  <strong>5. Y qué código me genera eso en el navegador?</strong>
                </p>
                
                <p>
                  Recordáis que MVC3 apuesta fuertamente por el <a href="http://geeks.ms/blogs/etomas/archive/2010/11/12/saca-tus-scripts-de-tu-c-243-digo-html.aspx" target="_blank" rel="noopener noreferrer">unobstrusive javascript</a>? Pues eso es lo que nos genera:
                </p>
                
                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">data-val</span><span style="color: #0000ff">="true"</span> <br /><span style="color: #ff0000">data-val-remote</span><span style="color: #0000ff">="This user name is not allowed."</span> <br /><span style="color: #ff0000">data-val-remote-additionalfields</span><span style="color: #0000ff">="*.UserName"</span> <br /><span style="color: #ff0000">data-val-remote-url</span><span style="color: #0000ff">="/Account/CheckUserAvailability"</span> <br /><span style="color: #ff0000">data-val-required</span><span style="color: #0000ff">="The User name field is required."</span> <br /><span style="color: #ff0000">id</span><span style="color: #0000ff">="UserName"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="UserName"</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">=""</span> <span style="color: #0000ff">/&gt;</span><br /></pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Precioso no? Nada de código javascript mezclado por ahí que “ensucie” el html: sólo atributos.
                    </p>
                    
                    <p>
                      Los atributos data-val-remote-* son los que controlan la validación remota. Para que eso funcione, es necesario que los scripts siguientes estén referenciados:
                    </p>
                    
                    <ol>
                      <li>
                        jquery-1.4.4.min.js
                      </li>
                      <li>
                        jquery.validate.min.js
                      </li>
                      <li>
                        jquery.validate.unobtrusive.min.js
                      </li>
                    </ol>
                    
                    <p>
                      Yo los tengo siempre colocados en mi página Master principal (Views/Shared/_Layout.cshtml si usas Razor).
                    </p>
                    
                    <p>
                      <strong>6. Controlando algunos aspectos “avanzados”</strong>
                    </p>
                    
                    <p>
                      Vamos a ver como controlar algunos aspectos más de la validación remota. P.ej. te puede interesar que la llamada a la acción de validación (CheckUserAvailability) se realice via POST y no via GET. Para ello simplemente usa la propiedad HttpMethod del atributo Remote:
                    </p>
                    
                    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">[Remote(<span style="color: #006080">"CheckUserAvailability"</span>, <span style="color: #006080">"Account"</span>,<br />    HttpMethod = <span style="color: #006080">"POST"</span>, ErrorMessage = <span style="color: #006080">"This user name is not allowed."</span>)]</pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          Eso añade el atributo <strong>data-val-remote-type="POST"</strong> al campo HTML generado lo que fuerza que la petición sea usando POST.
                        </p>
                        
                        <p>
                          Otra cosa interesante es que podemos <strong>enviar más de un parámetro </strong>a la acción quen realiza la “validación remota”. P.ej. imaginad que a CheckUserAvailability le queremos también pasar el password que ha introducido el usuario (vale, no tiene lógica en ese ejemplo, pero imaginadlo igual :p).
                        </p>
                        
                        <p>
                          Para ello podemos usar la propiedad <em>AdditionalFields</em> del atributo Remote:
                        </p>
                        
                        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">[Remote(<span style="color: #006080">"CheckUserAvailability"</span>, <span style="color: #006080">"Account"</span>, HttpMethod = <span style="color: #006080">"POST"</span>,<br />    AdditionalFields = <span style="color: #006080">"Password"</span>,<br />   ErrorMessage = <span style="color: #006080">"This user name is not allowed."</span>)]<br /><span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> UserName { get; set; }</pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              Ahora si miramos de nuevo lo que nos manda la petición veremos que junto al campo UserName nos manda el valor del campo Password:
                            </p>
                            
                            <p>
                              <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_56EDE02C.png"><img style="background-image: none; border-bottom: 0px; border-left: 0px; margin: ; padding-left: 0px; padding-right: 0px; display: inline; border-top: 0px; border-right: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_3CAD7708.png" width="244" height="67" /></a>
                            </p>
                            
                            <p>
                              Tened presente que el valor de Password puede ser vacío (el usuario puede no haber introducido nada allí todavía). Pero lo importante de esto es que las validaciones remotas <strong>no están limitadas a validar UN SOLO campo</strong>.
                            </p>
                            
                            <p>
                              Y con eso terminamos el post de las validaciones remotas en ASP.NET MVC3… Como podeis ver, es un método sumamente potente y sumamente fácil!
                            </p>
                            
                            <p>
                              Un saludo! 😉
                            </p>

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_4459320F.png