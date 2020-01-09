---
title: ASP.NET Historia de una optimización. ¡Cuidado con la Sesión!
description: ASP.NET Historia de una optimización. ¡Cuidado con la Sesión!
author: eiximenis

date: 2015-02-18T13:51:52+00:00
geeks_url: /?p=1694
geeks_visits:
  - 1798
geeks_ms_views:
  - 1244
categories:
  - Uncategorized

---
En un cliente en el que he estado últimamente tenían un problema de rendimiento en su aplicación ASP.NET. El problema era más o menos que “cuando un usuario está buscando algo, entonces la aplicación se queda bloqueada”. Por supuesto el primer paso fue determinar que significaba “bloqueada” ya que es una palabra un tanto ambigua… Uno de los aspectos que conoce todo el mundo que trata con problemas reportados por usuarios finales es que muchas veces (por no decir casi siempre) el problema está descrito entre mal y peor.

Al final que la aplicación se bloqueaba significaba que el usuario no podía navegar a ningún otro sitio. Debo contextualizar que es una aplicación web que consta de una pagina principal con un menú a modo de “escritorio” y cada opción que selecciona el usuario se abre en una ventana nueva de navegador. Así los usuarios terminan teniendo varias ventanas del navegador abiertas y van haciendo cosillas (búsquedas, ediciones, lo que sea que hagan) en cada una de ellas. Pues bien lo que ocurría es que mientras en alguna ventana se estaban buscando datos, cuando se pulsaba en cualquier otra opción, esta se quedaba esperando y no cargaba hasta que finalizaba la búsqueda de la otra ventana.

Lo primero que hice fue verificar que no estuviesen haciendo ellos un bloqueo por código (tenían algunos singletons donde se guardaban ciertos datos) pero no vi nada sospechoso. Había descartado cualquier bloqueo de BBDD porque un análisis previo realizado había confirmado que no habían bloqueos, por lo que en este aspecto estaba tranquilo. El siguiente punto fue ver en que momento se bloqueaban las otras peticiones. Ahí tuve que invertir un poco de tiempo ya que el proyecto consta de varias soluciones de VS y es un proyecto en webforms bastante complejo. Al final pude configurar mi sistema para depurar parte de la web y observé que ni llegaba al Page_Load del formulario. Eso era interesante pero como el <a href="https://msdn.microsoft.com/en-us/library/ms178472%28v=vs.85%29.aspx" target="_blank" rel="noopener noreferrer">ciclo de vida de Webforms</a> es inescrutable tenía que asegurarme que no se quedase en algún punto _anterior_ al Page_Load (lease algún evento Init o PreInit perdidos por ahí) de cualquier formulario base que pudiese haber. Después de navegar un poco por el código fuente (es una aplicación con una jerarquía de formularios bastante… interesante) llegué a la conclusión de que no. De que las peticiones ni tan siquiera habían entrado en el pipeline de webforms. Se quedaban atascadas antes.

Finalmente me dio por monitorizar el estado de peticiones del proceso de trabajo desde la consola de IIS y llegué a ver lo siguiente:

<img title="Captura" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 10px 10px 0px 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="Captura" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/Captura_5F00_41A91E59.png" width="1208" height="137" />

Tenía dos peticiones en marcha y una tercera que estaba bloqueada **esperando acceso a la sesión**. Y es que **en asp.net solo se permite una petición concurrente por usuario que tenga acceso de escritura a la sesión**. Si la petición requiere solo de acceso a lectura no hay problema, pero nunca entraran dos peticiones concurrentes que tengan permisos de escritura en la sesión (por usuario). Esto es muy importante porque **no se trata de si realmente se lee o se escribe en la sesión. Se trata de si _se tienen permisos para escribir o leer_ en la sesión.**

De las dos peticiones en marcha que tenía una era a un servicio web que no usaba sesión y la otra era la del buscador. El problema básico de la aplicación es que todas las peticiones tenían permisos de lectura y escritura en la sesión, por lo que todas las peticiones se encolaban una tras otra y nunca había dos peticiones (a dos formularios .aspx) en paralelo. Eso en la operativa normal de la aplicación no se notaba y pasaba desapercibido, pero cuando había un buscador en marcha se notaba simplemente porque el buscador podía tardar bastante tiempo en responder (del orden de segundos). Por lo tanto si un usuario mientras esperaba que el buscador le devolviese resultados (recuerda que el buscador está en otra ventana) volvía a la ventana principal y intentaba lanzar otra operación, esta operación se quedaba sin poder cargarse hasta que finalizaba el buscador. Además al abrirse en ventana nueva, la sensación del usuario era de una ventana nueva en blanco que no hacía nada.

Un workaround rápido fue **declarar que los buscadores tuviesen solo acceso de lectura a la sesión**. Como todos los buscadores derivaban de un formulario padre para búsquedas fue fácil y rápido añadir el atributo EnableSessionState=”ReadOnly” en la directiva @Page de dicho formulario. Y problema solucionado…

…o más bien parche aplicado, porque la solución real pasaría por la inversa: declarar que el acceso habitual a la sesión es de “solo lectura” (añadiendo <pages enableSessionState="ReadOnly" /> en el web.config) y declarar acceso de lectura y escritura solo en aquellos formularios que quieran escribir en la sesión (usando EnableSessionState=”true” en la directiva @Page). De hecho sería incluso mejor deshabilitar en el web.config el acceso a sesión (colocando false en el enableSessionState) y colocar explícitamente los valores ReadOnly y true a cada formulario que requiera acceder a la sesión (en modo de solo lectura o con permisos totales). Pero ese es un refactoring mucho más complejo, claro.

Ten presente pues **que si tus peticiones declaran que pueden acceder en modo lectura y escritura a la sesión** nunca se ejecutarán de forma concurrente (para un mismo usuario). Quizá no te des por aludido porque tu aplicación no abre varias ventanas pero… ¿haces varias llamadas AJAX de forma simultánea? Si es así… ¿qué permisos tienen al respecto de la sesión?.

Si en lugar de webforms usas ASP.NET MVC recuerda que puedes aplicar el atributo [SessionState] para indicar que un controlador requiere un acceso a la sesión distinto del que esté indicado por defecto en el web.config (o que no requiere acceso en absoluto).

¿Mi recomendación? Evita usar la sesión en todo lo que puedas. Recuerda que en ASP.NET los datos de autenticación **no** se guardan en la sesión (tienen su propia cookie separada). Pero si al final te decides a usarla, desactívala por defecto en el web.config y actívala explícitamente en todas aquellas páginas/controladores que la requieran. Y cuando la actives actívala siempre en modo ReadOnly a no ser que realmente debas escribir en ella, claro 😉

Con esos sencillos pasos conseguirás evitar que se te queden peticiones enganchadas esperando por una sesión que a lo mejor ni necesitan!

Saludos!