---
title: 'Katana: Cortando el framework'

author: eiximenis

date: 2013-07-15T13:40:00+00:00
geeks_url: /?p=1647
geeks_visits:
  - 4850
geeks_ms_views:
  - 2698
categories:
  - Uncategorized

---
Si eres desarrollador en tecnologías Microsoft y especialmente desarrollador web, acúerdate de esas dos palabras: <a target="_blank" href="http://katanaproject.codeplex.com/" rel="noopener noreferrer">Proyecto Katana</a>. Este proyecto representa el futuro a corto plazo de todas las tecnologías de desarrollo web de Microsoft y, no me cabe duda de ello, el futuro a medio plazo del propio .NET.

No dejes que esa introducción te confunda: Katana no es una nueva tecnología, ni una nueva plataforma, ni un lenguaje nuevo, ni tan siquiera realmente una API nueva. Es simplemente un cambio de filosofía. Es como Microsoft entiende que debe ser la evolución de ASP.NET para poder seguir dando respuestas al, cada vez más, cambiante mundo del desarrollo de aplicaciones web. Katana se circumscribe sólo a ASP.NET, pero no me cabe duda de que al final todo .NET irá en la misma dirección.

¿Y qué dirección es esa? Para saber donde vamos hemos de saber de donde venimos...

**Hace 12 años (año arriba, año abajo)...**

Microsoft sacó .NET y con él una &ldquo;nueva versión&rdquo; de ASP. Conocida primero como ASP+, luego ASPX para adoptar la terminología de ASP.NET al final. La intención de MS con ASP.NET era doble. Por un lado que los desarrolladores de &ldquo;toda la vida&rdquo; en tecnologías web de Microsoft (que en un 95% usaban ASP) empezaran a usar la nueva versión. Pero también que esa nueva versión pudiese ser atractiva y usable rápidamente por la gran mayoría de los desarrolladores que usaban tecnologías Microsoft, lo que era equivalente a decir VB6 en aquella época.

Es este segundo objetivo el que hace que Microsoft apueste por el modelo de Webforms para que &ldquo;desarrollar para web sea igual a hacerlo para escritorio&rdquo;. No olvides eso: el assembly principal de ASP.NET (System.Web.dll) está totalmente acoplado a IIS.

ASP.NET se integró dentro de lo que se pasó a llamar (y se llama) .NET Framework: Un framework **monolítico** del que ASP.NET era tan solo una parte y que se actualizaba cada cierto tiempo (entre 1 y 2 años y pico), básicamente cada vez que había una nueva versión de Visual Studio.

En medio de todo este tiempo varios cambios en el paradigma del desarrollo web a los que ASP.NET no podía responder con prontitud por su modelo de distribución y actualización, hicieron que alguien en Redmond se diese cuenta de que alguna cosa se tenía que hacer.

**Hace 4 años (año arriba, año abajo)...**

Veía la luz ASP.NET MVC. Era un nuevo framework, basado en ASP.NET que venía a complementar a Webforms. Desde este momento había dos APIs para desarrollar aplicaciones web en tecnologías Microsoft: Webforms y ASP.NET MVC. Ambas funcionaban bajo toda la plataforma de ASP.NET (y el .NET Framework subyacente que era el 3.5SP1 por aquel entonces).

Pero la novedad principal de ASP.NET MVC no era que fuese una nueva API si no el hecho de que **se distribuía independientemente del .NET Framework**. Era la primera vez que una API de Microsoft veía la luz fuera del &ldquo;paraguas&rdquo; aglutinador del .NET Framework. Esto suponía la posibilidad para Microsoft de evolucionar ASP.NET MVC mucho más ágilmente. Para que te hagas una idea: junto con el .NET Framework 4, salió ASP.NET MVC2. Después salieron MVC3 y MVC4 sin que hubiese ninguna versión más del Framework, hasta que salió Visual Studio 2012 y el .NET Framework 4.5.

ASP.NET MVC marcaba una nueva tendencia (que luego ha seguido Entity Framework) en donde se intenta huir de un framework monolítico para pasar a otro &ldquo;framework&rdquo; formado por varios módulos que se actualizan de forma separada.

Pero recuerda: ASP.NET MVC seguía funcionando bajo System.Web.dll y por lo tanto estaba &ldquo;atada&rdquo; a IIS (y también al .NET Framework, ya que System.Web.dll forma parte del .NET Framework).

**Hace tres año (año arriba, año abajo)...**

La aparición de ASP.NET MVC3 supuso otro hito importante en esta historia, no por MVC3 en si mismo, si no por un componente que le acompañaba: NuPack.

NuPack, luego llamado NuGet, era una extensión de Visual Studio que permitía (permite) instalar paquetes de forma muy sencilla. Estos paquetes pueden ser cualquier cosa, usualmente librerías de terceros (pero también templates de proyectos, etc). Era como tener <a target="_blank" href="http://pear.php.net/" rel="noopener noreferrer">PEAR</a> pero en .NET.

No sé si cuando salió alguien fue consciente de la importancia que adquiriría NuGet para el desarrollo en tecnologías Microsoft...

**Hace un año (año arriba, año abajo)...**

Salió ASP.NET MVC4 y el hito final de esta pequeña historia. No por MVC4 si no por un componente que le acompañaba (otra vez) y que parecía que formaba parte de MVC4 cuando realmente poco tenía que ver con ella: WebApi.

La gran novedad de WebApi es que **no tenía ninguna dependencia contra System.Web.dll**. Es decir, WebApi podía evolucionar de forma completamente independiente de ASP.NET (eso no era del todo cierto en ASP.NET MVC aunque se distribuyera via NuGet) y además al no depender de System.Web.dll implicaba que no se estaba atado a IIS por lo que se podía hospedar WebApi en otros procesos (p. ej. una aplicación de línea de comandos).

La idea de WebApi es pues la idea final: una parte del framework sin dependencias externas, que evoluciona a su propio ritmo y que se distribuye automáticamente via NuGet.

**Hoy (día arriba, día abajo)...**

El proyecto Katana es quien ha tomado el relevo de esta filosofía que se inaugura con WebApi. La idea del proyecto Katana es tener un conjunto de componentes cada uno de los cuales pueda evolucionar independientemente y por supuesto sin ninguna dependencia a System.Web.dll.

Katana es un conjunto de componentes, implementados por Microsoft y que son componentes <a target="_blank" href="http://owin.org/" rel="noopener noreferrer">OWIN</a>. Estos componentes van desde hosts, servidores hasta componentes funcionales (p. ej. hay una versión de SignalR &ldquo;katanizada&rdquo; o la propia WebApi). Recuerda la clave: cada componente podrá evolucionar a su propio ritmo y podrá ser sustuído por otro en cualquier momento. Olvídate de decir que &ldquo;esta web es ASP.NET 3.5&rdquo; y que todo el mundo sepa que es esto. Tu web ser&agrave; una combinación de distintos módulos, cada uno con su propia versión. Desde NuGet seleccionarás cual de los módulos quieres usar (p.ej. un nuevo módulo de autenticación).

Y a futuro... extiende esta visión a todo el framework. Cierra los ojos e imagina que ya no hay un .NET Framework 6 ó 7 o el que toque y que en su lugar tu proyecto usará System.Collections en una versión y también System.Net y el entity framework que toque y nada más. Incluso, porque no, seleccionar que CLR (o CLRs) quieres atacar. Yo al menos lo veo así (aunque desde que predije el fracaso del iPad tengo claro que como pitoniso no me ganaré la vida).

**OWIN**

Antes he mencionado que Katana es (será) básicamente un conjunto de componentes OWIN y me he quedado tan ancho. Pero... ¿qué es básicamente OWIN? Pues inspirándose de nuevo en la comunidad de Ruby (en concreto en <a target="_blank" href="http://rack.github.io/" rel="noopener noreferrer">Rack</a>) la comunidad de .NET ha creado una abstracción entre el servidor web y los componentes del framework que se use. La idea es que:

  1. Se puedan (y usar) crear nuevos componentes de forma sencilla y fácil
  2. Sea más sencillo portar aplicaciones entre hosts.

No entraré en detalles sobre OWIN ahora (lo dejo para un post siguiente) pero básicamente sus puntos clave son:

  1. El servidor Web OWIN es el encargado de rellenar los datos OWIN (<a target="_blank" href="http://owin.org/spec/owin-1.0.0.html#_3.2._Environment" rel="noopener noreferrer">environment</a> según la especificación) a partir de la petición. A la práctica estos datos son un diccionario (IDictionary<string,object>). Las cadenas usadas como claves están definidas en la propia especificación de OWIN.
  2. Los componentes deben implementar el <a target="_blank" href="http://owin.org/spec/owin-1.0.0.html#ApplicationDelegate" rel="noopener noreferrer">application delegate (AppFunc)</a>. Básicamente la idea es que los componentes reciben el diccionario anterior y devuelven una Task con la definición de la tarea a realizar.

Estos dos puntos parecen una chorrada, pero quédate con un par de ideas al respecto:

  * OWIN está diseñado con visos de ser asíncrono (de ahí que los componentes devuelvan un Task). La idea es parecida a la de nodejs: procesar asíncronamente aumenta el rendimiento en aquellas peticiones que básicamente dependen de E/S que son la mayoría.
  * Los componentes OWIN son encadenables de forma sencilla.

**Conclusión**

El modelo de un framework monolítico podía ser válido hace, pongamos, siete años, pero cada vez es necesario poder evolucionar más rápidamente.

Evolución rápida implica que sea al margen del .NET Framework. ASP.NET MVC abrió el camino y el proyecto Katana es la culminación de esa idea: tener todas las APIs y componentes de ASP.NET (y algunos más) pero sin dependencias contra System.Web.dll (y por lo tanto con el .NET Framework y con IIS). Para la implementación de Katana, Microsoft se ha apoyado en OWIN, una iniciativa de la comunidad .NET, que define una interfaz entre servidores web y frameworks de desarrollo web modulares.

En próximos posts iremos ampliando la información sobre OWIN y Katana.

Saludos!