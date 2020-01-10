---
title: La sencillez de una interfaz compleja

author: eiximenis

date: 2009-03-12T11:11:00+00:00
geeks_url: /?p=1442
geeks_visits:
  - 3219
geeks_ms_views:
  - 1258
categories:
  - Uncategorized

---
Hace algún tiempecillo escribí un artículo para el e-zine de [raona][1], que enviamos a distintos clientes. En el artículo esbozaba los patrones básicos para diseñar interfaces de usuario compuestas. Posteriormente me surgió la idea de que una ampliación de dicho artículo, donde se mostrasen ejemplos en [PRISM][2] y WPF de estos conceptos podría ser interesante. Afortunadamente en [DotNetMania][3] pensaron lo mismo y es por ello que en la revista de este marzo hay un artículo con este mismo título.

<!--more-->

Lo que ahora sigue és el **<span style="text-decoration: underline;">artículo original</span>**, el que escribí para el e-zine. Aunque el de DotNetMania describe las mismas ideas, ambos artículos tienen poco a ver (tanto en contenido, como en enfoque como en extensión). Como creo que el artículo del e-zine tambiém tiene su interés, me he tomado la libertad de postearlo aquí 🙂

PD: Por si os interesa, los distintos e-zines que vamos sacando en raona, los podeis consultar en la [sección de ezines de raona][4].

### La sencillez de una interfaz compleja (versión e-zine).

Cada vez las aplicaciones son más y más complejas, y los usuarios son más y más exigentes. Los desarrolladores debemos afrontar este doble reto siendo capaces de proporcionar mejores y más complejas aplicaciones, en cada vez menos tiempo. Evidentemente este es un proceso global, que afecta a todo el desarrollo de una solución de software, pero me gustaría centrarme en los retos concretos que esto implica cuando hablamos de interfaces de usuario. No en vano, la interfaz de usuario es el punto de conexión entre el usuario y la aplicación. Es un elemento crítico y de su implementación depende en buena medida el éxito final de toda la aplicación.

**Interfaces complejas**

Las aplicaciones actuales demandan interfaces de usuario complejas a nivel técnico pero que a la vez sean sencillas de utilizar y de mantener. Que se adapten a cualquier tipo de usuario, que tanto usuarios expertos como inexpertos se sientan a gusto utilizándola. En resumen que la experiencia del usuario al utilizar la interfaz sea plena y que nuestra aplicación sea usable por cualquier usuario.

El término _interfaz compleja_, no quiere referirse al aspecto visual o usable si no al hecho de que cada vez es más complejo y difícil realizar interfaces que permitan conjugar las crecientes posibilidades de la aplicación con la sencillez de uso y la usabilidad. Este es el reto que debemos solucionar.

**Nuevas tecnologías, nuevas posibilidades**

Con la aparición de WPF, Microsoft ha dotado a los desarrolladores de un montón de nuevas posibilidades para la construcción de interfaces. El hecho de disponer de un lenguaje declarativo (XAML) para definir las interfaces permite la separación entre dos roles básicos hoy en día: los diseñadores y los desarrolladores. Pero separar desarrolladores y diseñadores es sólo un primer paso (un primer gran paso) pero no soluciona la problemática de fondo: el modo en cómo están diseñadas y desarrolladas las interfaces de usuario.

El mecanismo habitual de desarrollar interfaces de usuario en .NET, viene influenciado de los tiempos de Visual Basic. Es un patrón que consiste en una clase contenedora (llamada comúnmente formulario) que contiene a los controles y toda la lógica asociada a dichos controles. Por norma general el formulario se suscribe a los eventos de los controles a los que contiene y realiza la lógica apropiada en las funciones gestoras de dichos eventos. La ventaja de este modelo de programación es que es sencillo, intuitivo y fácil de aprender. Las desventajas son que al tener unidas la lógica con la presentación se vuelve más complejo el mantenimiento tendiendo a _colapsar_ en grandes proyectos y se dificulta la separación del desarrollo entre varios equipos.

Una de las claves para desarrollar interfaces de usuario complejas y que resulten mantenibles es desacoplar la presentación de la lógica de dicha presentación. WPF presenta algunas novedades que ayudan a conseguir dicho desacople:

  1. Commands: Permite desacoplar los eventos de los controles de la gestión de dichos eventos y unificar en un mismo punto la gestión de distintos eventos de distintos controles pero que realizan la misma acción (p.ej. un botón de una toolbar y una entrada de menú). 
  2. Routed Events: Permiten tratar en un solo contenedor cualquier evento generado por cualquier de los controles &ldquo;hijos&rdquo; (estén en el nivel de profundidad que estén). 

Estas novedades aunque interesantes, por si solas no son suficientes para desacoplar totalmente la presentación y su lógica. Para ello hace falta un cambio de paradigma, un nuevo modelo.

**Composite Application Library**

Composite Application Library (CAL) también conocida por su nombre en código PRISM, es un framework lanzado por la gente de &ldquo;Patterns & Practices&rdquo; para ayudar en la construcción de aplicaciones complejas utilizando WPF.

CAL se basa en varios patrones para ayudar a conseguir un desacople total entre la presentación y la lógica de presentación. A continuación comentaremos tres de los patrones más relevantes de CAL.

**Inversion of Control (IoC)**

Este patron desacopla una clase en concreto de otra clase de las clases que utiliza, es decir de sus referencias:

[<img border="0" width="244" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/clip_5F00_image002_5F00_thumb_5F00_02A77BEA.jpg" alt="Dependencias entre clsases" height="181" style="border-top-width: 0px; display: inline; border-left-width: 0px; border-bottom-width: 0px; border-right-width: 0px" title="Dependencias entre clsases" />][5]

La clase ClassA utiliza dos servicios (ServiceA y ServiceB) y por lo tanto tiene referencias directas a ellos. Esto implica, entre otras cosas, que si queremos modificar las referencias debemos modificar el código fuente de ClassA. Para desacoplar la clase de sus referencias la solución es añadir un componente externo que se encargue de encontrar y gestionar las referencias de la clase ClassA. En función de cómo diseñemos este elemento externo hablaremos de un patrón _ServiceLocator_ o de _Dependency Injection_.

[<img border="0" width="491" src="/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/clip_5F00_image0024_5F00_thumb_5F00_68D5D66B.jpg" alt="Service locator &  Dependency Injection" height="174" style="border-top-width: 0px; display: inline; border-left-width: 0px; border-bottom-width: 0px; border-right-width: 0px" title="Service locator &  Dependency Injection" />][6]

Con ServiceLocator la clase ClassA sólo contiene una referencia al _Locator_ siendo esta clase la que obtiene y devuelve la referencia adecuada a la clase. El _Locator_ NO instancia los servicios, simplemente los localiza (los servicios han estado instanciados previamente por alguien): mientras un servicio esté registrado, el _Locator_ lo encontrará. CAL utiliza Unity para ofrecer un Service Locator, lo que nos permite instanciar y registrar servicios que luego serán utilizados en cualquier parte de la aplicación.

Con Dependency Injection, existe un _Builder_ que crea el objeto del servicio y lo &ldquo;incrusta&rdquo; en alguna propiedad específica de la clase ClassA (o bien en algún parámetro del constructor). CAL utiliza Unity para proporcionar Dependency Injection.

**Separated Presentation**

Si IoC nos permite desacoplar nuestras clases de las referencias que necesitan, el conjunto de patrones conocidos como &ldquo;separated presentation&rdquo;, nos permiten separar la presentación de la parte de la lógica que gestiona la presentación.

Existen varios patrones que implementan &ldquo;separated presentation&rdquo; siendo los más concidos MVP (Model &ndash; View &ndash; Presenter) o MVVM (Model &ndash; View- ViewModel). Debido a las capacidades de WPF se tiende a utilizar más MVVM (mientras que MVP es más frecuente en aplicaciones Windows forms p.ej), debido a que MVVM se basa en uno de los aspectos más potentes de WPF: su data binding.

El patrón MVVM se basa en tres elementos:

  * Vista (View): Contiene los elementos visuales (controles). En el caso de WPF suele estar creada por un diseñador e implementada vía XAML 
  * Modelo (Model): Contiene los datos que muestra la vista. En este punto es cuando se utiliza básicamente el data binding, para enlazar los valores de los controles de la vista a valores de distintos elementos del modelo. 
  * Modelo de Vista (ViewModel): En interfaces sencillas, la vista se puede enlazar directamente con el modelo, pero en interfaces complejas no suele ser posible. En muchos casos el Modelo puede tener multitud de información que no se puede mapear a valores de los controles o bien nuestra vista debe realizar y mantener valores &ldquo;temporales&rdquo; que no forman parte estricta del modelo. Es en estos casos donde entra en escena el &ldquo;ViewModel&rdquo;, actuando de _puente_ proporcionando transformadores para poder enlazar los controles de la vista y commands para que la vista pueda interactuar con el modelo. 

De esta manera no existe más una clase &ldquo;formulario&rdquo; que contiene los controles y toda la lógica asociada a ellos.

**Event Aggregator**

Un _Event Aggregator_ es la implementación de un modelo de eventos publish and subscribe, donde cuando alquien quiere recibir un evento se _suscribe_ a este tipo de evento en particular. Cuando alguien quiere enviar un evento lo _publica_ y el _event aggregator_ se ocupa de que todos los que se hayan suscrito a este tipo de evento lo reciban. De esta manera se desacoplan totalmente los publicadores de eventos de sus suscriptores.

**Conclusiones**

Para desarrollar interfaces complejas que a la vez sean amigables para el usuario y mantenibles para el desarrollador es necesario involucrar a los diseñadores por un lado y desacoplar la presentación de la lógica que la gestiona por otro. Gracias a XAML, WPF permite que los diseñadores se integren totalmente en el ciclo de desarrollo, y el uso de determinados patrones permite desacoplar los distintos elementos que conforman la inferfaz de usuario. CAL es un framework y una guía de estilo que basándose en dichos patrones y en las funcionalidades inherentes a WPF nos permite desarrollar aplicaciones complejas que sean mantenibles y amigables a la vez.

**Referencias**

  * Command pattern in WPF (<http://www.microsoft.com/belux/msdn/nl/community/columns/jdruyts/wpf_commandpattern.mspx>) 
  * Routed Events overview (<http://msdn.microsoft.com/en-us/library/ms742806.aspx>) 
  * Model &ndash; View &ndash; Presenter pattern (<http://msdn.microsoft.com/en-us/magazine/cc188690.aspx>) 
  * Model &ndash; View &ndash; ViewModel pattern (<http://blogs.msdn.com/johngossman/archive/2005/10/08/478683.aspx>) 
  * Inversion of Control (<http://msdn.microsoft.com/en-us/library/cc707904.aspx>) 
  * Service Locator Pattern (<http://msdn.microsoft.com/en-us/library/cc707905.aspx>) 
  * Dependency Injection (<http://msdn.microsoft.com/en-us/library/cc707845.aspx>) 
  * Dependency Injection on Martin Fowler&rsquo;s site (<http://www.martinfowler.com/articles/injection.html>) 
  * Event Aggregator (<http://martinfowler.com/eaaDev/EventAggregator.html>) 
  * Event Aggregator (<http://msdn.microsoft.com/en-us/library/cc707867.aspx>)

 [1]: http://www.raona.com/
 [2]: http://www.codeplex.com/CompositeWPF
 [3]: http://www.dotnetmania.com/
 [4]: http://www.raona.com/Notícies/ezine/tabid/110/Default.aspx
 [5]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/clip_5F00_image002_5F00_5C4307ED.jpg
 [6]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/clip_5F00_image0024_5F00_2B45E7F1.jpg