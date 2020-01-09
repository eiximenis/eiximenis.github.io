---
title: Como independizar tu capa de lógica de tu capa de presentación…
author: eiximenis

date: 2009-09-17T18:36:00+00:00
geeks_url: /?p=1466
geeks_visits:
  - 6981
geeks_ms_views:
  - 3100
categories:
  - Uncategorized

---
A raiz del [siguiente post][1] del excelente [blog de Oskar][2], [Julio Trujillo][3] comentó en un comentario (copio literalmente) &ldquo;_Sería interesante una explicación de como convertir Forms a WPF o al menos como poder diseñar una capa que permita conectar la capa de negocio a una de WPF o Forms indistintamente_&rdquo;. A este comentario respondí yo con unas cuantas ideas, pero luego Julio pidió a ver si podiamos exponer las &ldquo;buenas prácticas&rdquo; e incluso un ejemplo... Julio, no respondí a tu comentario, simplemente porque el tema es demasiado para un simple comentario, y se merece al menos un post... y estaba sacando tiempo 😉

Creo que el comentario de Julio, encerraba dos preguntas en una: cómo se puede convertir _fácilmente_ nuestras aplicaciones winforms a wpf y por otro como reaprovechar el máximo código posible. Voy a exponer unas cuantas ideas que quizá os pueden ayudar pero que se resumen en dos: usad una arquitectura n-layer (por cierto que nosotros hablamos sólo de &ldquo;n-capas&rdquo; pero [no confundais n-layer con n-tier][4]), por un lado y el patrón [separated presentation][5] por el otro...

**Arquitectura n-layer**

En esta arquitectura, básicamente, distribuimos nuestro código en n-capas lógicas, donde n suele tener un valor cercano a 3: la separación _clásica_ es presentación, lógica y datos. Aunque se pueden meter más (o menos, igual podemos prescindir de la capa de datos) capas lógicas (como p.ej. [lógica de presentación][6]).

Una arquitectura n-layer nos ayudará a reaprovechar nuestro código de la capa lógica con independencia de la capa de presentación que tengamos... Para ello basta seguir dos reglas que nos indicarán si vamos por buen camino:

  1. Nuestra capa de lógica debe estar en un proyecto separado (una librería de clases). Puede haber (y generalmente habrá) una referencia desde el proyecto que sea la capa de presentación al proyecto que es la capa de lógica **pero nunca debe aparecer** una referencia a la capa de lógica que vaya hacia la capa de presentación.
  2. El proyecto que contiene nuestra capa lógica no debe tener nunca ninguna referencia a ningún ensamblado de .NET que dependa, directa o indirectamente, de Winforms o WPF... P.ej. si te aparece una referencia a System.Windows.Forms... mal, porque estás ligando tu capa de lógica a una _tecnología de presentación_.

La comunicación desde la capa de presentación a la capa lógica _puede_ ser acoplada: lógico, disponemos de una referencia en la capa de presentación a la capa lógica y por lo tanto podemos instanciar cualquier clase pública de la capa de lógica y llamar a sus métodos directamente.

La comunicación desde la capa lógica a la capa de presentación **debe** ser desacoplada: no tenemos otra opción, dado que no podemos tener ninguna referencia a la capa de presentación desde la capa lógica. Aquí tenemos tres alternativas válidas:

  1. Comunicación pasiva: es decir, la capa de lógica se limita a devolver toda la información que la capa de presentación solicita mediante los valores de retorno de los métodos. Así, la comunicación la inicia siempre la capa de presentación.
  2. Eventos, o cualquier mecanismo similar (como [commands][7]): Cuando la capa de lógica quiere informar de algo a la capa de presentación, lanza eventos a los que está suscrita la capa de presentación y esta actúa en consecuencia. Esto permite que la capa lógica realice envíos de información a la capa lógica sin ser necesario que esta última inicie la comunicación.
  3. Utilizar interfaces junto con un patrón _[service locator][8]_. Esta alternativa precisa el uso de [un contenedor IoC][9] (como [Unity][10] o [Windsor][11]), así como la aparición de un _tercer_ assembly destinado a contener las interfaces. Usando esta aproximación, las clases de la capa de presentación implementan todas ellas las interfaces (definidas en el assembly aparte) y la capa de lógica obtiene referencias a los objetos de la capa de presentación usando las interfaces y el contenedor de IoC. Esta alternativa permite una conversación &ldquo;tu-a-tu&rdquo; con la capa de presentación.

Por supuesto las tres alternativas pueden combinarse.

Esto independiza nuestra capa de lógica (y de datos) de la presentación usada. Si alguna vez queremos migrar nuestra aplicación, p.ej. de winforms a wpf, sólo deberemos reescribir la capa de presentación... lo que según como esté diseñada puede ser un trabajo asumible o una obra de titanes.

**Separated Presentation**

Bajo este nombre se agrupan multitud de patrones ([MVC][12], [MVP][13], [MVVM][14]), pero todos ellos coinciden en lo básico: separa **siempre** tu código que _muestra_ los datos (el formulario) del código que indica cuando y como debe mostrarlos.

Lamentablemente, desde los tiempos de Visual Basic, Microsoft nos acostumbra a desarrollar usando el &ldquo;patrón formulario&rdquo;, que básicamente consiste en crear un form, rellenar-lo de controles y asociar eventos a funciones tipo MyButton_Click() que contendrán todo el código. Este modelo es rápido, fácil de entender y produce buenos resultados... hasta que uno debe empezar a cambiar cosas. A lo mejor al cabo de unos meses en tu aplicación le aparece _otro_ formulario que se parece _mucho_ a un formulario ya existente pero a lo mejor muestra un par de controles más o se comporta ligeramente distinto... O bien haces copy-paste del primer formulario en uno nuevo y lo modificas (funciona pero luego vas a mantener dos formularios parecidos) o empiezas a meter _ifs_ en el primer formulario hasta que todo se vuelve un galimatías.

Créeme: Olvida el &ldquo;patrón formulario&rdquo; cuanto antes. Sirve para _pequeñas_ aplicaciones que no precisen un mantenimiento excesivo, pero para aplicaciones mayores usa alguna variante de _separated presentation_.

Si nos centramos en el caso concreto de Winforms y WPF, una de las mejores elecciones es MVP: si **ya sabes** que tu aplicación deberá ser migrada (o bien debe ser multi-presentación) entonces MVP te ahorrará bastante trabajo: &ldquo;básicamente&rdquo; sólo deberás rediseñar los formularios y podrás aprovechar el modelo y los presenters. Yo por ejemplo suelo usar _siempre_ MVP cuando desarrollo en Winforms (no por temas de migración, sino porque considero que es el patrón que aplica mejor) y cuando estoy en WPF uso MVVM o MVP dependiendo de la ocasión.

Reconozco que si se debiera migrar una aplicación WPF creada usando MVVM a winforms habría bastante trabajo, pero de todos modos muchas veces si se migra una aplicación a una tecnología de presentación nueva es para aprovechar las nuevas capacidades (me cuesta imaginarme una migración de WPF a winforms, mientras que al revés puede ser más habitual).

En fin, migrar una aplicación a otra tecnología de presentación **siempre cuesta trabajo**, pero si separamos bien las capas (n-layer) y organizamos bien nuestra capa de presentación (separated presentation) tendremos mucho de ganado...

Saludos!!!!

 [1]: /blogs/oalvarez/archive/2009/09/09/controles-wpf-en-winforms.aspx
 [2]: /blogs/oalvarez/default.aspx
 [3]: http://yodesarrollador.com/
 [4]: http://icomparable.blogspot.com/2008/10/arquitectura-n-tier-o-arquitectura-n.html
 [5]: http://martinfowler.com/eaaDev/SeparatedPresentation.html
 [6]: http://www.exforsys.com/tutorials/application-development/n-tier-architecture-presentation-logic-layer.html
 [7]: http://en.wikipedia.org/wiki/Command_pattern
 [8]: http://martinfowler.com/articles/injection.html#UsingAServiceLocator
 [9]: http://weblogs.asp.net/sfeldman/archive/2008/02/14/understanding-ioc-container.aspx
 [10]: http://www.codeplex.com/unity
 [11]: http://www.castleproject.org/container/index.html
 [12]: http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller
 [13]: http://msdn.microsoft.com/en-us/magazine/cc188690.aspx
 [14]: http://msdn.microsoft.com/en-us/magazine/dd419663.aspx