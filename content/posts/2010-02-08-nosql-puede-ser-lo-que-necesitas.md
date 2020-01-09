---
title: NoSQL… ¿puede ser lo que necesitas?
description: NoSQL… ¿puede ser lo que necesitas?
author: eiximenis

date: 2010-02-08T12:36:00+00:00
geeks_url: /?p=1493
geeks_visits:
  - 2511
geeks_ms_views:
  - 1479
categories:
  - Uncategorized

---
Ultimamente se oye hablar cada vez más de BBDD no relacionales o tal y como se las conoce ahora &ldquo;NoSQL&rdquo;. En <a target="_blank" href="/controlpanel/blogs/posteditor.aspx/www.dosideas.com" rel="noopener noreferrer">dosideas</a> publicaron un <a target="_blank" href="http://www.dosideas.com/base-de-datos/657-nosql-el-movimiento-en-contra-de-las-bases-de-datos.html" rel="noopener noreferrer">interesante post al respecto de los sistemas NoSQL</a>. La idea es renunciar a algunos de los principios (y funcionalidades) de las bases de datos _tradicionales_ (relacionales) a cambio de obtener mayores velocidades en el acceso a datos.

Cuando nos adentramos en este mundo, debemos dejar de _pensar en tablas_, ya que nuestros datos dejarán de estar guardados en formato relacional. Aunque existen varios _formatos_ en los cuales se guardan nuesteos datos parece ser que los más comunes son _(clave,valor)_ o usar _documentos_ que son en cierto modo una extensión de la (clave, valor). Si os pasáis por el <a target="_blank" href="http://en.wikipedia.org/wiki/NoSQL" rel="noopener noreferrer">artículo de la wikipedia sobre NoSQL</a> hay varios enlaces a distintos sistemas NoSQL. A mi me gustaría hablaros de uno con el que he hecho algunas pruebas: <a target="_blank" href="http://www.mongodb.org" rel="noopener noreferrer">MongoDB</a>.

**Montando el entorno...**

Para empezar a usar el entorno, basta con <a target="_blank" href="http://www.mongodb.org/display/DOCS/Downloads" rel="noopener noreferrer">descargarnos los binarios</a>. La versión más reciente estable es la 1.2.2. MongoDB usa el esquema de numeración de versiones par, donde las versiones estables siempre son pares y las de desarrollo son impares (así actualmente en desarrollo ya existe la 1.3, que cuando se estabilice pasará a ser 1.4). Para instalar MongoDB basta con descomprimir el zip donde más os plazca 🙂

MongoDB está escrita en C++ y viene con una librería (.lib) y varios headers para ser usada directamente. Por suerte existe una API C# para MongoDB que os podéis descargar desde [http://github.com/samus/mongodb-csharp][1] (podéis descargaros los binarios (MongoDB.Linq.dll y MongoDB.Driver.dll) o bien el código fuente (una solución VS2008 que genera los dos assemblies mencionados).

Una vez tengáis instalado MongoDB y los dos assemblies del driver para C#... estamos listos para empezar!

Para poner en marcha el servidor de MongoDB basta con ir donde hayáis descomprimido MongoDB y lanzar el comando:

<span style="font-family: Courier New; font-size: x-small;">mongod &#8211;dbpath <data_path></span>

donde _<data_path>_ es el directorio de datos que quereis usar.

**El concepto de documentos...**

MongoDB se define como base de datos de _documentos_, entendiendo como un documento una estructura de datos que es una colección de elementos &ldquo;clave, valor&rdquo;, donde las claves son cadenas y los elementos cualquier cosa que se quiera. Aunque esto puede parecerse una tabla (donde las claves sean los nombres de los campos) se diferencia del concepto de tabla en que por un lado no tiene esquema fijo (una clave _puede_ _o no_ aparecer en un documento) y en que los valores van más allá de los admitidos generalmente por los campos de las bases de datos relacionales (p.ej. podemos guardar colecciones de _otros_ documentos como valores). El hecho que no haya esquema fijo hace estas bases de datos NoSql ideales para el desarrollo de soluciones que manejan datos poco estructurados.

**Algunas operaciones básicas...**

Para conectarnos a la BBDD basta con instanciar un objeto del tipo Mongo y llamar al método Connect. Una vez hayamos finalizado debemos llamar a Disconnect:

<div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">var srv = <span style="color: #0000ff">new</span> Mongo();<br />srv.Connect();<br /><span style="color: #008000">// Operaciones con MongoDB</span><br />srv.Disconnect();</pre>
  
  <p>
    </div> 
    
    <p>
      Una vez estamos conectados al servidor debemos escojer la base de datos a utilizar. Esto lo podemos hacer con el método getDB:
    </p>
    
    <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
      <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">var db = srv.getDB(<span style="color: #006080">"MyAppDB"</span>);</pre>
      
      <p>
        </div> 
        
        <p>
          Una vez tenemos la base de datos ya podemos operar con ella. Lo que en una base de datos relacional son tablas con registros aquí son <em>colecciones</em> con <em>documentos</em>. A diferencia de una tabla relacional una MISMA colección puede tener documentos con distinto esquema (distintas claves):
        </p>
        
        <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
          <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;"><span style="color: #008000">// Obtenemos la coleccion 'users'</span><br />var iusers = db.GetCollection(<span style="color: #006080">"users"</span>);<br /><span style="color: #008000">// Creamos un usuario con login y pwd</span><br />Document user = <span style="color: #0000ff">new</span> Document();<br />user.Add(<span style="color: #006080">"login"</span>, <span style="color: #006080">"edu"</span>);<br />user.Add(<span style="color: #006080">"pwd"</span>, <span style="color: #006080">"mypassword"</span>);<br /><span style="color: #008000">// Insertamos el documento</span><br />iusers.Insert(user);<br /><span style="color: #008000">// Creamos otro documento. Este con login y pwd_hash</span><br />user = <span style="color: #0000ff">new</span> Document();<br />user.Add(<span style="color: #006080">"login"</span>, <span style="color: #006080">"edu2"</span>);<br />user.Add(<span style="color: #006080">"pwd_hash"</span>, <span style="color: #006080">"tH23H13"</span>);<br /><span style="color: #008000">// Insertamos el documento en la MISMA colección</span><br />iusers.Insert(user);<br /><span style="color: #008000">// Obtenemos todos los elementos de la colección</span><br />var allUsers = iusers.FindAll();<br /><span style="color: #0000ff">int</span> numUsers = allUsers.Documents.Count();</pre>
          
          <p>
            </div> 
            
            <p>
              <strong>Un ejemplo</strong>
            </p>
            
            <p>
              Vamos a ver un ejemplo de uso de MongoDB... Ahora que <a target="_blank" href="/members/lfranco/default.aspx" rel="noopener noreferrer">Lluís</a> nos está haciendo una <a target="_blank" href="/blogs/lfranco/archive/2010/02/03/usando-asp-net-membrership-en-winforms-1-n.aspx" rel="noopener noreferrer">clase maestra sobre el membership provider</a>, vamos a ver como podríamos implementar nuestro membership provider para que vaya contra MongoDB en lugar de contra una base de datos relacional. No voy a mostrar todo el código, sólo un par de extractos pero os dejo al final del post el enlace en formato zip con la solución de visual studio.
            </p>
            
            <p>
              &nbsp;
            </p>
            
            <p>
              &nbsp;
            </p>
            
            <p>
              &nbsp;
            </p>
            
            <p>
              &nbsp;
            </p>
            
            <p>
              El primer paso es definir que base de datos de MongoDB vamos a utilizar. No me he calentado mucho la cabeza: los membership providers tienen una propiedad ApplicationName que está pensada para eso. La idea es que un mismo proveedor puede manejar datos de distintas aplicaciones. El campo ApplicationName permite saber cual es la aplicación que se está manejando. Yo asumo que la BBDD de MongoDb se llamará igual que la aplicación:
            </p>
            
            <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
              <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">var srv = <span style="color: #0000ff">new</span> Mongo();<br />srv.Connect();<br />var db = srv.getDB(<span style="color: #0000ff">this</span>.ApplicationName);</pre>
              
              <p>
                </div> 
                
                <p>
                  Otro punto importante es no olvidarnos de llamar a Disconnect() cuando hemos terminado de trabajar con Mongo. La mejor manera de hacer esto, dado que la clase Mongo no implementa IDisposable es con try...finally:
                </p>
                
                <div id="codeSnippetWrapper" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; width: 97.5%; font-family: 'Courier New', courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; cursor: text; border: silver 1px solid; padding: 4px;">
                  <pre id="codeSnippet" style="text-align: left; line-height: 12pt; background-color: #f4f4f4; margin: 0em; width: 100%; font-family: 'Courier New', courier, monospace; direction: ltr; color: black; font-size: 8pt; overflow: visible; border-style: none; padding: 0px;">var srv = <span style="color: #0000ff">new</span> Mongo();<br /><span style="color: #0000ff">try</span><br />{<br />    srv.Connect();<br />    <span style="color: #008000">// Operaciones con MongoDb</span><br />}<br /><span style="color: #0000ff">finally</span><br />{<br />    srv.Disconnect();<br />}</pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      El membership provider que he creado <strong>no</strong> implementa todas las funciones, pero sí un grupo suficientemente ámplio para que sea <em>usable</em>: Es capaz de validar usuarios, añadir usuarios y borrar usuarios. P.ej. esta es una captura de pantalla de la aplicación de configuración de ASP.NET usando este proveedor:
                    </p>
                    
                    <p>
                      <a href="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_469131AB.png"><img height="164" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_694181F4.png" alt="image" border="0" title="image" style="border-bottom: 0px; border-left: 0px; display: inline; border-top: 0px; border-right: 0px" /></a>
                    </p>
                    
                    <p>
                      Nada más... os dejo el enlace al código con un zip que incluye una solución de visual studio con el proveedor y una aplicación asp.net que lo utiliza (una página con un control login). Si os interesa... echadle una ojeada! 😉
                    </p>
                    
                    <p>
                      <a target="_blank" href="http://cid-6521c259e9b1bec6.skydrive.live.com/self.aspx/BurbujasNet/ZipsPosts/MongoClient.zip" rel="noopener noreferrer">Enlace del fichero .zip</a> (en mi skydrive).
                    </p>
                    
                    <p>
                      Saludos!
                    </p>

 [1]: http://github.com/samus/mongodb-csharp "http://github.com/samus/mongodb-csharp"