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

La verdad es que con ASP.NET MVC2 subir ficheros al servidor **es extremadamente simple**. Vamos a empezar viendo el código de una vista que permite subir un fichero al servidor, junto con una descripción adicional. La vista básicamente contiene un&#160; `<form>` como el siguiente:

```html
<form action="<%: Url.Action("Upload") %>" enctype="multipart/form-data" method="post">
    <label for="descripcion">Descripción del fichero:</label>
    <input type="text" id="descripcion" name="descripcion" />
    <br />
    <label for="fichero">Fichero:</label>
    <input type="file" name="fichero" size="40">
    <br />
    <input type="submit" value="Enviar" />
</form>
```
Fijaos que es HTML puro y duro, aunque el tag <form> lo podeis generar con Html.BeginForm() si queréis. La clave es añadir el atributo <strong>enctype</strong> con el valor <em>multipart/form-data.</em> Como se menciona en la <a href="http://www.w3.org/TR/html401/interact/forms.html#h-17.13.4.2">especificación sobre formularios del W3C</a>, el valor de multipart/form-data es el que debe usarse cuando se quieran enviar al servidor datos binarios.
      
El `<input type=”file”>` es el control HTML que nos permite seleccionar un fichero para enviar.

Y desde el controlador? Pues sencillo, en este caso mi formulario tiene dos parámetros (<em>descripcion </em>y <em>fichero</em>), por lo que necesitaré que la acción del controlador tenga esos dos parámetros. El parámetro <em>descripcion</em> es un string, pero el parámetro fichero… que és?

Pues bien ASP.NET MVC es capaz de ver que el parámetro <em>fichero</em> es un fichero que se ha subido al servidor y sabe <em>mapearlo</em> a un objeto de la clase <a href="http://msdn.microsoft.com/en-us/library/system.web.httppostedfilebase.aspx">HttpPostedFileBase</a>. Esta clase nos da acceso no sólo al contenido del fichero subido, sinó a más información (su content-type, su tamaño, el path completo desde donde se ha subido,…).
    
El método del controlador queda pues, así de sencillo:

```csharp
[HttpPost]
public ActionResult Upload(string descripcion, HttpPostedFileBase fichero)
{
    fichero.SaveAs(Path.Combine(@"d:temp", Path.GetFileName(fichero.FileName)));
    return View();
}
```         

Fijaos en los dos parámetros string y HttpPostedFileBase. El método simplemente se guarda una copia del fichero subido en d:temp, pero obviamente aquí podéis hacer lo que queráis.
          
Y listos! No hay que hacer nada más… qué, sencillo, no??? 🙂

 Un saludo

PD: Esta técnica <strong>no es ajax</strong>, eso significa que mientras se está subiendo el fichero al servidor, la aplicación web no responde (el browser está haciendo la petición). Existe un mecanismo para realizar subidas de ficheros en background, aunque no es directo debido a que con XMLHttpRequest (el objeto del naveagador que hace posible ajax) no se pueden subir ficheros. Si estáis interesados en el siguiente <a href="http://aspzone.com/tech/jquery-file-upload-in-asp-net-mvc-without-using-flash/">post de John Rudolf se muestra como realizar un upload de fichero en ajax usando jQuery y el form plugin</a>!