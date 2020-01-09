---
title: Cambiar el esquema con ASP.NET Identity
author: eiximenis

date: 2013-12-23T21:52:54+00:00
geeks_url: /?p=1660
geeks_visits:
  - 4683
geeks_ms_views:
  - 4868
categories:
  - Uncategorized

---
Una de las grandes novedades (probablemente la de mayor calado si obviamos la revolución de OWIN) de la nueva versión de ASP.NET es ASP.NET Identity, que sustituye al viejuno Membership (que apareció allá en 2005). ASP.NET Identity está diseñado para dar solución a muchos de los problemas de los que Membsership acaecía.

Una de las preguntas más recurrentes en los foros de la MSDN y fuera de ellos es como usar Membership con un esquema de base de datos propio. Esto básicamente implica crearte un custom memberhip provider lo que, sin ser excesivamente complicado, te hace bueno… derivar de una clase con un porrón de métodos abstractos, la mitad de los cuales puede que ni apliquen en tu caso y que vas a dejar llenos de NotImplementedException. Pesado, feo y además un NotImplementedException siempre demuestra un fallo de diseño en algún punto del sistema (en este caso en el Membership).

ASP.NET Identity ha sido diseñado, entre otras cosas, para solucionar este problema: que los cambios de esquema de la BBDD no impliquen tantos problemas. En este post veremos como adaptar ASP.NET Identity a un esquema de BBDD propio, utilizando, eso sí, el proveedor de ASP.NET Identity para Entity Framework. Y es que otras de las características de ASP.NET Identity es que puede trabajar con varios proveedores de datos (p. ej. se podría hacer un proveedor noSQL para MongoDb) aunque, obviamente (no le pidamos peras al olmo) implementar un proveedor nuevo no es algo trivial. MS ha implementado uno para EF que es el que mucha gente va a utilizar, así que centrémonos en él. Dicho proveedor reside en el paquete Microsoft.AspNet.Identity.EntityFramework y se incorpora por defecto cuando seleccionamos la opción “Individual User Accounts” al crear un proyecto ASP.NET en VS2013.

Si no hacemos nada, el esquema por defecto que se nos crea es el siguiente:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_1FDAF0CE.png" width="244" height="135" />][1]

Miremos ahora alguna de las tablas, p. ej. AspNetUsers:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_03CCC294.png" width="504" height="129" />][2]

Bien, veamos ahora como podemos añadir un campo adicional a esta tabla.

**Añadiendo un campo adicional a la tabla de usuarios**

Honestamente este es la parte fácil (y la que encontrarás en la mayoría de tutoriales sobre ASP.NET Identity). Pero antes de ver como hacerlo déjame contarte un par de cosillas sobre como está montado el proveedor de ASP.NET Identity para EF. Dicho proveedor se basa en EF Code First y en varias clases que implementan las interfaces necesarias para ASP.NET Identity. P. ej. ASP.NET Identity trabaja con la interfaz IUser y el módulo de EF utiliza la clase IdentityUser. La interfaz IUser es muy simple, solo define dos campos (Id y UserName ambos de tipo string). La clase IdentityUser implementa dicha interfaz y añade varios campos más (como PasswordHash y SecurityStamp). Podríamos decir que la tabla AspNetUsers se genera a partir de dicha clase.

Nosotros no podemos modificar dicha clase y añadir campos, así que el proveedor de ASP.NET Identity para EF ha optado por ofrecernos otro mecanismo. Dicho proveedor para trabajar necesita un contexto de EF (es decir un DbContext), pero dicho DbContext debe derivar de IdentityDbContext<T> donde el tipo T derive de IdentityUser. Así pues para añadir un campo simplemente debemos:

  1. Crear una clase X derivada de IdentityUser y añadir el campo necesario.
  2. Derivar de IdentityDbContext<X> donde X es la clase creada en el punto anterior.

El template de VS2013 nos crea una clase llamada ApplicationUser que ya hereda de IdentityUser. Dicha clase está vacía y está pensada para que nosotros añadamos los campos necesarios. También nos crea un contexto de EF (llamado ApplicationDbContext) que hereda de IdentityDbContext<ApplicationUser>. Es decir, nos da casi todo el trabajo hecho 🙂

Para no ser originales, vamos a hacer el ejemplo clásico que encontrarás en cualquier post: añadir la fecha de nacimiento. Para ello nos basta con añadir dicho campo en la clase ApplicationUser:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:567abbbd-fded-43fa-9d38-522fef322588" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">ApplicationUser</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#4ec9b0">IdentityUser</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">DateTime</span><span style="background:#1e1e1e;color:#b4b4b4">?</span><span style="background:#1e1e1e;color:#dcdcdc"> BirthDay { </span><span style="background:#1e1e1e;color:#569cd6">get</span><span style="background:#1e1e1e;color:#dcdcdc">; </span><span style="background:#1e1e1e;color:#569cd6">set</span><span style="background:#1e1e1e;color:#dcdcdc">; }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Si ahora ejecutas de nuevo la aplicación, seguramente te dará el siguiente error: _The model backing the &#8216;ApplicationDbContext&#8217; context has changed since the database was created. Consider using Code First Migrations to update the database (http://go.microsoft.com/fwlink/?LinkId=238269)_

Esto es porque se ha modificado el esquema de datos y ahora tenemos un código que no se corresponde a la BBDD. Tenemos dos opciones: o eliminar la BBDD (y perder todos los datos) o usar Migrations. Para usar Migrations tenemos que habilitarlas primero mediante el Package Manager Console, tecleando Enable-Migrations. Una vez las tengas habilitadas debemos:

  1. Añadir una migración que refleje el cambio realizado en ApplicationUser, usando Add-Migration <nombre_migracion> en el Package Manager Console
  2. Actualizar la BBDD usando Update-Database en el Package Manager Console.

En mi caso cuando he tecleado Add-Migration AddBirthday me ha generado dentro de una carpeta Migrations el archivo de migración. Dicho archivo es una clase C# que le indica a Migrations que hacer. Se genera automáticamente “comparando” el esquema de la BBDD con el esquema que tendría que haber según la clases de EF code first. Para que veas, en mi caso la clase de migración conti
  
ene el siguiente código:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:055d1e84-e4ea-4fd9-b368-86ac212402b5" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2.5em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">partial</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">class</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#4ec9b0">AddBirthday</span><span style="background:#1e1e1e;color:#dcdcdc"> : </span><span style="background:#1e1e1e;color:#4ec9b0">DbMigration</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">override</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">void</span><span style="background:#1e1e1e;color:#dcdcdc"> Up()</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        AddColumn(</span><span style="background:#1e1e1e;color:#d69d85">"dbo.AspNetUsers"</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#d69d85">"BirthDay"</span><span style="background:#1e1e1e;color:#dcdcdc">, c </span><span style="background:#1e1e1e;color:#b4b4b4">=></span><span style="background:#1e1e1e;color:#dcdcdc"> c</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">DateTime());</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        </span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">public</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">override</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">void</span><span style="background:#1e1e1e;color:#dcdcdc"> Down()</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    {</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        DropColumn(</span><span style="background:#1e1e1e;color:#d69d85">"dbo.AspNetUsers"</span><span style="background:#1e1e1e;color:#dcdcdc">, </span><span style="background:#1e1e1e;color:#d69d85">"BirthDay"</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    }</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Puedes ver que lo que hará es añadir la columna BirthDay. Insisto: dicha clase se genera automáticamente al hacer Add-Migration. Cuando ejecutes Update-Database, se aplicarán los cambios indicados.

Así que bueno… lo único que te queda es esto, ejecutar el comando Update-Database desde el Package Manager Console. Cuando lo apliques verás algo parecido a esto:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_07D9A117.png" width="504" height="90" />][3]

Listos… Si ahora miras el esquema de la tabla AspNetUsers verás que tenemos ya el nuevo campo:

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_727E7C5F.png" width="504" height="153" />][4]

Fácil y sencillo. La verdad, comparado con lo que se tenía que hacer para conseguir lo mismo con el Membership es una maravilla 😛

**Cambiar el esquema**

Vale… añadir campos es sencillo (hemos visto el ejemplo clásico que es la tabla de usuarios, para el resto de tablas es lo mismo pero con otras clases). Pero… y cambiar el esquema? Si quiero que la tabla AspNetUsers se llame simplemente Users y que PasswordHash se llame Pwd p. ej.? Que tenemos que hacer?

Pues bien, eso también es posible gracias al uso de EF Code First. En este caso vamos a tener que tocar el contexto de EF (la clase ApplicationDbContext en el caso de código generado por VS2013), en concreto redefinir el método OnModelCreating:

<div id="scid:9ce6104f-a9aa-4a17-a79f-3a39532ebf7c:a3fc3948-5bb9-4c2b-a626-eda610f598be" class="wlWriterEditableSmartContent" style="float: none; padding-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px">
  <div style="border: #000080 1px solid; color: #000; font-family: 'Courier New', Courier, Monospace; font-size: 10pt">
    <div style="background: #ddd; max-height: 300px; overflow: auto">
      <ol start="1" style="background: #1e1e1e; margin: 0 0 0 2em; padding: 0 0 0 5px; white-space: nowrap">
        <li>
          <span style="background:#1e1e1e;color:#569cd6">protected</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">override</span><span style="background:#1e1e1e;color:#dcdcdc"> </span><span style="background:#1e1e1e;color:#569cd6">void</span><span style="background:#1e1e1e;color:#dcdcdc"> OnModelCreating(</span><span style="background:#1e1e1e;color:#4ec9b0">DbModelBuilder</span><span style="background:#1e1e1e;color:#dcdcdc"> modelBuilder)</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">{</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    </span><span style="background:#1e1e1e;color:#569cd6">base</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">OnModelCreating(modelBuilder);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    modelBuilder</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Entity</span><span style="background:#1e1e1e;color:#b4b4b4"><</span><span style="background:#1e1e1e;color:#4ec9b0">IdentityUser</span><span style="background:#1e1e1e;color:#b4b4b4">></span><span style="background:#1e1e1e;color:#dcdcdc">()</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ToTable(</span><span style="background:#1e1e1e;color:#d69d85">"Users"</span><span style="background:#1e1e1e;color:#dcdcdc">)</span><span style="background:#1e1e1e;color:#b4b4b4">.</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        Property(u </span><span style="background:#1e1e1e;color:#b4b4b4">=></span><span style="background:#1e1e1e;color:#dcdcdc"> u</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">PasswordHash)</span><span style="background:#1e1
e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">HasColumnName(</span><span style="background:#1e1e1e;color:#d69d85">"Pwd"</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">    modelBuilder</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">Entity</span><span style="background:#1e1e1e;color:#b4b4b4"><</span><span style="background:#1e1e1e;color:#4ec9b0">ApplicationUser</span><span style="background:#1e1e1e;color:#b4b4b4">></span><span style="background:#1e1e1e;color:#dcdcdc">()</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">ToTable(</span><span style="background:#1e1e1e;color:#d69d85">"Users"</span><span style="background:#1e1e1e;color:#dcdcdc">)</span><span style="background:#1e1e1e;color:#b4b4b4">.</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">        Property(u </span><span style="background:#1e1e1e;color:#b4b4b4">=></span><span style="background:#1e1e1e;color:#dcdcdc"> u</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">PasswordHash)</span><span style="background:#1e1e1e;color:#b4b4b4">.</span><span style="background:#1e1e1e;color:#dcdcdc">HasColumnName(</span><span style="background:#1e1e1e;color:#d69d85">"Pwd"</span><span style="background:#1e1e1e;color:#dcdcdc">);</span>
        </li>
        <li>
          <span style="background:#1e1e1e;color:#dcdcdc">}</span>
        </li>
      </ol>
    </div></p>
  </div></p>
</div>

Y ahora el esquema que tengo en la BBDD es (mantengo mi clase ApplicationUser con la propiedad BirthDay):

[<img title="image" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_4B46C3DB.png" width="244" height="206" />][5]

De nuevo… comparado con lo que se tenía que hacer con el antiguo Membership es una maravilla.

El tipo de cambios que puedes hacer vienen limitados por EF Code First, pero en general estamos ante un modelo mucho más flexible que el anterior Membership.

En resumen, a pesar de que ASP.NET Identity tiene varias cosillas que no me gustan (o mejor dicho, realmente varias cosas que echo en falta) es sin duda un paso adelante respecto al anterior Membership. Me sigue quedando la duda de si lo que ofrece es suficiente para según que tipo de webs, pero si las funcionalidades que ofrece te son suficientes y no te importa ajustar tu esquema a algunos detalles menores (como esos Id de usuario de tipo string) es un avance muy significativo respecto a Membership.

Un saludo a todos y… ¡felices fiestas!

 [1]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_701D484E.png
 [2]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_73BDF3DC.png
 [3]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_57AFC5A2.png
 [4]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_4973DD63.png
 [5]: http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_76F78DD7.png