---
title: Subir ficheros al servidor en ASP.NET MVC

author: eiximenis

date: 2010-09-08T13:23:05+00:00
geeks_url: /?p=1532
geeks_visits:
  - 11324
geeks_ms_views:
  - 4637
categories:
  - Uncategorized

---
Buenas! Hoy voy a responder alguna pregunta que me he encontrado en alguna vez, y es como _subir ficheros_ al servidor usando MVC2. 

<!--more-->

La verdad es que con ASP.NET MVC2 subir ficheros al servidor **es extremadamente simple**. Vamos a empezar viendo el código de una vista que permite subir un fichero al servidor, junto con una descripción adicional. La vista básicamente contiene un&#160; <form> como el siguiente:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">&lt;</span><span style="color: #800000">form</span> <span style="color: #ff0000">action</span><span style="color: #0000ff">="&lt;%: Url.Action("</span><span style="color: #ff0000">Upload</span><span style="color: #0000ff">") %&gt;"</span> <span style="color: #ff0000">enctype</span><span style="color: #0000ff">="multipart/form-data"</span> <span style="color: #ff0000">method</span><span style="color: #0000ff">="post"</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">label</span> <span style="color: #ff0000">for</span><span style="color: #0000ff">="descripcion"</span><span style="color: #0000ff">&gt;</span>Descripción del fichero:<span style="color: #0000ff">&lt;/</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text"</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="descripcion"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="descripcion"</span> <span style="color: #0000ff">/&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">br</span> <span style="color: #0000ff">/&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">label</span> <span style="color: #ff0000">for</span><span style="color: #0000ff">="fichero"</span><span style="color: #0000ff">&gt;</span>Fichero:<span style="color: #0000ff">&lt;/</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="file"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="fichero"</span> <span style="color: #ff0000">size</span><span style="color: #0000ff">="40"</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">br</span> <span style="color: #0000ff">/&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="submit"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="Enviar"</span> <span style="color: #0000ff">/&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">form</span><span style="color: #0000ff">&gt;</span></pre>
  
  <p>
    </div> 
    
    <p>
      Fijaos que es HTML puro y duro, aunque el tag <form> lo podeis generar con Html.BeginForm() si queréis. La clave es añadir el atributo <strong>enctype</strong> con el valor <em>multipart/form-data.</em> Como se menciona en la <a href="http://www.w3.org/TR/html401/interact/forms.html#h-17.13.4.2">especificación sobre formularios del W3C</a>, el valor de multipart/form-data es el que debe usarse cuando se quieran enviar al servidor datos binarios.
    </p>
    
    <p>
      El <input type=”file”> es el control HTML que nos permite seleccionar un fichero para enviar.
    </p>
    
    <p>
      Y desde el controlador? Pues sencillo, en este caso mi formulario tiene dos parámetros (<em>descripcion </em>y <em>fichero</em>), por lo que necesitaré que la acción del controlador tenga esos dos parámetros. El parámetro <em>descripcion</em> es un string, pero el parámetro fichero… que és?
    </p>
    
    <p>
      Pues bien ASP.NET MVC es capaz de ver que el parámetro <em>fichero</em> es un fichero que se ha subido al servidor y sabe <em>mapearlo</em> a un objeto de la clase <a href="http://msdn.microsoft.com/en-us/library/system.web.httppostedfilebase.aspx">HttpPostedFileBase</a>. Esta clase nos da acceso no sólo al contenido del fichero subido, sinó a más información (su content-type, su tamaño, el path completo desde donde se ha subido,…).
    </p>
    
    <p>
      El método del controlador queda pues, así de sencillo:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px" id="codeSnippet">[HttpPost]<br />public ActionResult Upload(string descripcion, HttpPostedFileBase fichero)<br />{<br />    fichero.SaveAs(Path.Combine(@"d:temp", Path.GetFileName(fichero.FileName)));<br />    return View();<br />}</pre>
      
      <p>
        </div> 
        
        <p>
          Fijaos en los dos parámetros string y HttpPostedFileBase. El método simplemente se guarda una copia del fichero subido en d:temp, pero obviamente aquí podéis hacer lo que queráis.
        </p>
        
        <p>
          Y listos! No hay que hacer nada más… qué, sencillo, no??? 🙂
        </p>
        
        <p>
          Un saludo
        </p>
        
        <p>
          PD: Esta técnica <strong>no es ajax</strong>, eso significa que mientras se está subiendo el fichero al servidor, la aplicación web no responde (el browser está haciendo la petición). Existe un mecanismo para realizar subidas de ficheros en background, aunque no es directo debido a que con XMLHttpRequest (el objeto del naveagador que hace posible ajax) no se pueden subir ficheros. Si estáis interesados en el siguiente <a href="http://aspzone.com/tech/jquery-file-upload-in-asp-net-mvc-without-using-flash/">post de John Rudolf se muestra como realizar un upload de fichero en ajax usando jQuery y el form plugin</a>!
        </p>