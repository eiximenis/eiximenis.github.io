---
title: 'ASP.NET SimpleMembership: Una vuelta de tuerca más'
description: 'ASP.NET SimpleMembership: Una vuelta de tuerca más'
author: eiximenis

date: 2012-11-25T23:36:05+00:00
geeks_url: /?p=1618
geeks_visits:
  - 5226
geeks_ms_views:
  - 1715
categories:
  - Uncategorized

---
Cuando, allá por el 2005, aparecía ASP.NET 2.0 una de las novedades que incorporaba era el sistema de Membership. Dicho sistema pretendía ser una solución unificada para solucionar problemas de autenticación y autorización de usuarios. La idea era muy simple: en lugar de que los controles web (recordad, estamos en 2005, MVC no existe) accedan a la BBDD directamente (o el proveedor de autorización usado, como Active Directory) para comprobar logins y passwords se usa una capa intermedia que centralice dichos accesos. Dicha capa se estandariza y pasa a formar parte de la API del sistema para que cualquier control la pueda usar. Es esta capa lo que conocemos con el nombre de Membership (aunque realmente incluye el proveedor propiamente de Membership (usuarios), el de Roles y el de perfiles). Es curioso como, a tenor de las preguntas en los foros de MSDN en español, todavía hay hoy, más de siete años después de la salida de ASP 2.0 mucha confusión al respecto de Membership. De serie ASP.NET venía (viene) con un proveedor de Membership para SQL Server y otro para Active Directory (autenticación windows). Si requieres que la información de tus usuarios esté en otro sistema (p.ej. MySql) debes crearte un proveedor propio (lo que llamamos _custom membership provider_), que realmente es crear una clase que derive de <a href="http://msdn.microsoft.com/es-es/library/system.web.security.membershipprovider(v=vs.80).aspx" target="_blank" rel="noopener noreferrer">MembershipProvider</a>.

A lo largo del tiempo el sistema de Membership ha mostrado sus problemas. Los más evidentes son:

  1. Una interfaz muy hinchada. El proveedor de Membership intenta dar solución “a todo lo posible en temas de Membership”. Eso significa que si tienes que crearte un proveedor propio vas a tener que implementar muchos métodos, aunque no tengan sentido en tu aplicación. P.ej. si en tu aplicación los usuarios no pueden estar bloquados da igual. Deberás implementar los métodos de bloquear y desbloquear usuarios porque están declarados en el Membership. O al revés… si tus usuarios NO tienen password (p.ej. usas OAuth) da igual, porque Membership requiere un password por cada usuario. 
  2. Un esquema SQL fijo. Antes he comentado que ASP.NET incorpora de serie un Membership provider para SQL Server. Pues bien, eso es cierto siempre y cuando use el esquema prefijado del Membership provider. Si quiero que mis usuarios se almacenen en mi tabla “Usuarios” ya no podré usar el Memebrship provider por defecto, porque este espera que las tablas tengan unos nombres y columnas predeterminados. 

Con el tiempo el Membership ha ido sufriendo ciertas mejoras, todas ellas en tiempos más o menos recientes ciertamente, para intentar solucionar algunos de los problemas. La más importante es la aparición de los <a href="http://www.variablenotfound.com/2011/09/aspnet-universal-providers.html" target="_blank" rel="noopener noreferrer">Universal Providers, de los cuales ya escribió José M. Aguilar en su blog</a>. Estos intentan evitar que tengamos que escribir nuestro propio Membership provider si queremos usar otra BBDD que no sea Sql Server. Las últimas releases de los Universal Providers solucionan el problema que José María menciona en su blog (que no consiguió hacerlo funcionar con MySql) y funcionan con cualquier BBDD ya que usan EF para conectarse (por lo que cualquier BBDD que tenga un proveedor de EF será válida y eso incluye, entre otras, a MySql). De regalo, los Universal Providers nos traen una simplificación del esquema SQL usado, lo que es _otro_ avance 😉 Pero ojo, aunque ahora tenemos un esquema simplificado, seguimos estando atados al esquema de tablas que requieren los universal providers, así que si queremos usar nuestro propio esquema de BBDD entonces seguimos estando obligados a escribirnos nuestro propio Membership provider…

… al menos, hasta la aparición del Simple Membership!

Así pues ahora en ASP.NET tenemos _dos_ modelos de Membership, el clásico que data de 2005 y cuya última iteración serían los universal providers y el nuevo conocido por SimpleMembership. La ventaja de SimpleMembership respecto al Membership clásico es que se integra con cualquier esquema de BBDD existente… Pero, **ojo! SimpleMembership tan solo funciona con cualquier BBDD de la familia de SQL Server**. Es decir, con cualquier versión de SQL Server (incluyendo Express), SQL Azure, SQL Compact y la última iteración de SQL Server Express conocida como LocalDb.

De todos modos que nadie se lleve a engaño: SimpleMembership _deriva_ del Membership clásico y eso en el sentido más literal. Por un lado en WebMatrix.WebData.dll nos encontramos con:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: blue">public</span> <span style="color: blue">abstract</span> <span style="color: blue">class</span> <span style="color: #2b91af">ExtendedMembershipProvider</span> : MembershipProvider
  </p></p>
</div>

Y luego con:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: blue">public</span> <span style="color: blue">class</span> <span style="color: #2b91af">SimpleMembershipProvider</span> : ExtendedMembershipProvider
  </p></p>
</div>

Así que queda claro, no? Por cierto que un vistazo al código fuente de SimpleMembershipProvider también nos aclara el porque solo funciona bajo la familia de BBDD de Sql Server:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: blue">if</span> (!SimpleMembershipProvider.CheckTableExists(db, SimpleMembershipProvider.OAuthMembershipTableName))
  </p>
  
  <p style="margin: 0px">
    &#160; db.Execute(<span style="color: #a31515">"CREATE TABLE "</span> + SimpleMembershipProvider.OAuthMembershipTableName + <span style="color: #a31515">" (Provider nvarchar(30) NOT NULL, ProviderUserId nvarchar(100) NOT NULL, UserId int NOT NULL, PRIMARY KEY (Provider, ProviderUserId))"</span>, <span style="color: blue">new</span> <span style="color: blue">object</span>[0]);
  </p></p>
</div>

Este código precedente está dentro del método interno CreateTablesIfNeeded y como podéis ver usa directamente sintaxis de SQL Server. Códigos similares a esto se repiten en otros métodos de la clase SimpleMembership.

Llegados a este punto debemos tener claro que podemos usar SimpleMembership en cualquier sitio donde se espere el Membership clásico (p.ej. controles Webforms) pero al revés obviamente no. Actualmente las APIs que usan SimpleMembership son Web Pages 2 (es decir WebMatrix 2) o bien el template de “Internet Application” de MVC4. En este post nos centraremos tan solo en ASP.NET MVC4.

**SimpleMembership en ASP.NET MVC 4**

Si creamos un proyecto nuevo de ASP.NET MVC4 del tipo “Internet Application” VS2012 nos genera un AccountController que va a usar el SimpleMembership (en lugar de generarnos un AccountController que use el código del Membership tradicional), además de generarnos toda la infraestructura necesaria para ponerlo en marcha.

Si miráis el código del web.config veréis que NO hay en ningún momento ninguna referencia a la configuración del proveedor de Membership (etiqueta <membership> del web.config). De hecho **no hay ninguna referencia a SimpleMembership** en el web.config. Y es que SimpleMembership se “habilita por defecto” siempre y cuando se referencie el ensamblado WebMatrix.WebData.dll. La clave está en el siguiente código de la clase WebMatrix.WebData.ConfigUtil.cs:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: blue">private</span> <span style="color: blue">static</span> <span style="color: blue">bool</span> IsSimpleMembershipEnabled()
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px"
>
    &#160;&#160;&#160;&#160;&#160; <span style="color: blue">string</span> str = ConfigurationManager.AppSettings[WebSecurity.EnableSimpleMembershipKey];
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160; <span style="color: blue">bool</span> result;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160; <span style="color: blue">if</span> (!<span style="color: blue">string</span>.IsNullOrEmpty(str) && <span style="color: blue">bool</span>.TryParse(str, <span style="color: blue">out</span> result))
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">return</span> result;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160; <span style="color: blue">else</span>
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: blue">return</span> <span style="color: blue">true</span>;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p></p>
</div>

El valor de WebSecurity.EnableSimpleMembershipKey es “enableSimpleMembership”. De este código se deduce que el SimpleMembership se habilitará si existe el appSetting “enableSimpleMembership” con valor a true o bien si dicho appSetting no existe.

Dicho código se llama en el siguiente método (de la clase WebMatrix.WebData.WebSecurity):

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    <span style="color: blue">internal</span> <span style="color: blue">static</span> <span style="color: blue">void</span> PreAppStartInit()
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160; <span style="color: blue">if</span> (!ConfigUtil.SimpleMembershipEnabled)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">return</span>;
  </p>
  
  <p style="margin: 0px">
    &#160; MembershipProvider currentDefault1 = Membership.Providers[<span style="color: #a31515">"AspNetSqlMembershipProvider"</span>];
  </p>
  
  <p style="margin: 0px">
    &#160; <span style="color: blue">if</span> (currentDefault1 != <span style="color: blue">null</span>)
  </p>
  
  <p style="margin: 0px">
    &#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; SimpleMembershipProvider membershipProvider = WebSecurity.CreateDefaultSimpleMembershipProvider(<span style="color: #a31515">"AspNetSqlMembershipProvider"</span>, currentDefault1);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; Membership.Providers.Remove(<span style="color: #a31515">"AspNetSqlMembershipProvider"</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; Membership.Providers.Add((ProviderBase) membershipProvider);
  </p>
  
  <p style="margin: 0px">
    &#160; }
  </p>
  
  <p style="margin: 0px">
    &#160; <span style="color: green">// Codigo omitido por brevedad...</span>
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Fijaos en lo que hace dicho código:

  1. Mira si se usa SimpleMembership 
  2. En este caso obtiene el proveedor de Membership con clave AspNetSqlMembershipProvider 
  3. Crea un SimpleMembershipProvider (vinculado al proveedor de Membership anterior) 
  4. Elimina el proveedor de Membership encontrado en el punto 2. 
  5. Añade el SimpleMembershipProvider como nuevo proveedor de Membership (recordad que SimpleMembershipProvider deriva de ProviderBase). 

Si alguien se pregunta desde donde se llama el método PreAppStartInit() se llama desde el método Start() de la clase WebMatrix.WebData.PreApplicationStartCode. Pero si buscas desde donde se llama este método Start() encontrarás que **no** se llama desde ningún sitio. ¿Y entonces? Entonces ha llegado la hora que leas este post de @haacked: [http://haacked.com/archive/2010/05/16/three-hidden-extensibility-gems-in-asp-net-4.aspx][1]. En este post se explican tres métodos de extensibilidad de ASP.NET que no son muy conocidos. De todos nos interesa el primero, el atributo _PreApplicationStartMethod_ que te permite declarar un método para ser invocado esto… antes del Application_Start. Y en efecto. el ensamblado WebMatrix.WebData usa este atributo:

<div style="font-size: 10pt; font-family: consolas; background: white; color: black">
  <p style="margin: 0px">
    [<span style="color: blue">assembly</span>: PreApplicationStartMethod(<span style="color: blue">typeof</span> (PreApplicationStartCode), <span style="color: #a31515">"Start"</span>)]
  </p></p>
</div>

¡Y así cerramos el círculo! Ya sabemos como y cuando se crea el SimpleMembership: antes de inicializar la aplicación web.

Tan solo queda aclarar que cualquier aplicación ASP.NET 4 (y eso incluye ASP.NET MVC4) incluye un proveedor de Membership llamado AspNetSqlMembershipProvider, incluso si no lo define en el web.config, ya que el machine.config contiene dicha definición.

Y así terminamos este post. Hemos visto que es el SimpleMembership, la relación que tiene con los antiguos proveedores de Membership de ASP.NET y como se carga y configura. Nos queda ver como usarlo, pero eso lo dejamos para otro post!

Un saludo!

 [1]: http://haacked.com/archive/2010/05/16/three-hidden-extensibility-gems-in-asp-net-4.aspx "http://haacked.com/archive/2010/05/16/three-hidden-extensibility-gems-in-asp-net-4.aspx"