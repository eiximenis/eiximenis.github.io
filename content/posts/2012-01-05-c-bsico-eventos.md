---
title: 'C# Básico: Eventos'
author: eiximenis

date: 2012-01-05T12:52:46+00:00
geeks_url: /?p=1585
geeks_visits:
  - 12678
geeks_ms_views:
  - 4564
categories:
  - Uncategorized

---
Bueno… empieza el 2012: el último año de nuestra existencia si los maias no andaban errados (unos tíos que hacían pirámides hace miles de años no pueden equivocarse demasiado). Pero bueno… hasta que no llegue el apocalípsis, ahí estaremos! ¡Al pie del cañón!

De todos modos, para empezar nada más que un post ligerito, sobre [C# Básico][1], ya sabéis esta serie de posts donde vamos explorando cosillas, sin ningún orden en particular, básicas del lenguaje. Sí, ya sé que el ritmo de posts de la serie es abrumador (sobre un post por año) pero bueno… como dicen los ingleses: _less give an stone_.

Hoy vamos a hablar sobre los eventos, uno de los elementos de C# (realmente es de .NET en general pero aquí nos ceñiremos a C#) que más confusión causa, debido a que por un lado son una mezcla de convenciones, por otro VS nos autogenera código que nos puede hacer pensar que todo es más fácil, por otro porque _usarlos_ es trivial pero _entender_ lo que hay debajo de ellos, pues no lo es tanto… Ah sí, y por último a mucha gente le cuesta entender como funcionan porque están basados en delegates y el propio concepto de delegate no es trivial (¡aunque [ya hemos hablado en esta serie de delegates][2]!).

**1. ¿Qué es un evento?**

Bueno… antes que nada digamos que un evento es un mecanismo **estándard** de .NET. He dicho de .NET, no de winforms, o de WPF o de Silverlight. Repito: de .NET. Eso significa que **cualquier clase** puede lanzar eventos o suscribirse a ellas. No es necesario tener una aplicación gráfica para tener eventos, una triste aplicación de consola _puede_ tener eventos. Esa es una diferencia de .NET con otros frameworks o plataformas: Los eventos no son parte de un framework en concreto, son parte _integral_ de la plataforma.

En el mundo de .NET un evento es un mecanismo que tiene un objeto de una clase cualquiera de _notificar_ algo a un _conjunto de objetos cualesquiera_. El cualesquiera del final es importante: si yo, como objeto, expongo un evento _cualquier objeto de cualquier clase_ puede si desea suscribirse a este evento sin que yo tenga que hacer nada en especial. Eso los hace tremendamente útiles porque el desarrollador de la clase que lanza el evento no está obligado a conocer absolutamente nada sobre las posibles clases que se suscribirán a este evento.

**2. ¿Como se usa un evento?**

Antes de responder a esta cuestión respondamos a otra: ¿qué nos interesa decirle a quien se suscriba al evento? Por ejemplo si estamos creando una clase que informe de que se ha creado un fichero nuevo en disco, nos puede interesar pasar el nombre o la ruta del fichero. Si estamos creando una lista que informa cuando el usuario selecciona un elemento, nos puede interesar pasar el índice del elemento seleccionado, o el propio elemento. 

Imaginemos el siguiente código (de una aplicación de línea de comandos).

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">static</span> <span style="color: blue">void</span> Main(<span style="color: blue">string</span>[] args)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">var</span> igualada = <span style="color: blue">new</span> <span style="color: #2b91af">Ciudad</span>(<span style="color: #a31515">"Igualada"</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">var</span> castefa = <span style="color: blue">new</span> <span style="color: #2b91af">Ciudad</span>(<span style="color: #a31515">"Castelldefels"</span>);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">var</span> bandolero = <span style="color: blue">new</span> <span style="color: #2b91af">Persona</span>(<span style="color: #a31515">"José Miguel"</span>, igualada);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: #2b91af">Console</span>.ReadLine();
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; bandolero.Emigrar(castefa);
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Tenemos al bandolero, que estaba en la ciudad de Igualada y <strike>debido a una incomprensible decisión</strike> decide emigrar a Castelldefels. Ahora imaginad que tenemos una clase Censo que quiere recibir notificaciones cuando el censo de una Ciudad en concreto se ha modificado. Esto implica que la clase Ciudad debe informar cuando un habitante causa baja o alta en ella. Es en estos casos (cuando un objeto quiere informar a otro) que usamos un evento.

El uso de un evento se compone de dos partes:

  1. Quien lanza el evento (en nuestro caso la clase Ciudad) 
  2. Quien recibe el evento (en nuestro caso la clase Censo) 

**2.1 Suscribirse a un evento**

El que recibe el evento, debe suscribirse y para ello debe informar de que método quiere que sea llamado cuando se lance el evento. Es decir, debe proporcionar al evento un _delegate_ con el método. El código puede ser algo parecido a:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">public</span> <span style="color: blue">class</span> <span style="color: #2b91af">Censo</span>
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">private</span> <span style="color: blue">readonly</span> <span style="color: #2b91af">Ciudad</span> _ciudad;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">public</span> Censo(<span style="color: #2b91af">Ciudad</span> c)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; _ciudad = c;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; _ciudad.HabitanteDadoDeAlta += <span style="color: blue">new</span> <span style="color: #2b91af">CiudadEventHandler</span>(Ciudad_HabitanteDadoDeAlta);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    &#160;
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">private</span> <span style="color: blue">void</span> Ciudad_HabitanteDadoDeAlta(<span style="color: #2b91af">Persona</span> persona)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; <span style="color: #2b91af">Console</span>.WriteLine(<span style="color: #a31515">"{0} vive ahora en {1}"</span>, persona.Nombre, _ciudad.Nombre);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

En la segunda línea del constructor la clase Censo se _está suscribiendo_ al evento HabitandeDadoDeAlta que tiene la clase Ciudad. Y el parámetro es un delegate de tipo _CiudadEventHandler_ que “apunta” al método Ciudad_HabitanteDadoDeAlta. Aquí dos preguntas:

  * ¿Como sabe la clase Censo que el método Ciudad_HabitanteDadoDeAlta debe recibir un parámetro Persona? Pues por el delegate CiudadEventHandler. Este delegate está definido de la siguiente manera: 

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">delegate</span> <span style="color: blue">void</span> <span style="color: #2b91af">CiudadEventHandler</span> (<span style="color: #2b91af">Persona</span> persona);
  </p></p>
</div>

  * ¿Como sabemos que el evento HabitanteDadoDeAlta usa el delegate CiudadEventHandler? Pues eso se indica en la declaración del evento, que veremos ahora. 

Bien, hemos visto como _suscribirnos_ a un evento. Eso impli
  
ca declarar la función gestora (que será llamada cuando se lance el evento) con los parámetros que defina el delegate usado por el evento. Y también usar el operador += para asignar una instancia del delegate (apuntando a nuestra función gestora) al evento.

**2.2 Declarar un evento**

Veamos ahora, como _declarar_ un evento. Lo que es la declaración en sí, es muy, muy, muy, simple:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">public</span> <span style="color: blue">event</span> <span style="color: #2b91af">CiudadEventHandler</span> HabitanteDadoDeAlta;
  </p></p>
</div>

Esta línea (en la clase Ciudad) declara un evento, llamado HabitanteDadoDeAlta y que usa el delegate CiudadEventHandler. Lo declaramos público porque todo el mundo pueda acceder al evento, y así suscribirse a él (usando el operador +=).

**2.3 Lanzar un evento**

Una vez hemos declarado un evento, el siguiente paso es lanzarlo, cuando sea necesario. El código para lanzar un evento es _extremadamente simple._ En nuestro caso haríamos (en la clase Ciudad):

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    HabitanteDadoDeAlta(persona);
  </p></p>
</div>

Fijaos que usamos el evento como si fuese un método. Dado que el delegate define que hay un parámetro de tipo Persona, eso es lo que le pasamos. ¡Y listos! Con esto _todos_&#160; los que estén suscritos al evento, lo recibirán (y ejecutarán cada uno su función gestora, que recibirá los parámetros que nosotros hemos pasado). Las funciones gestoras se ejecutan una tras otra síncronamente.

Bueno… listos del todo no 🙂 La verdad es que este código fallaría si _nadie_ está suscrito al evento HabitanteDadoDeAlta, ya que entonces este vale _null_ y recibiríamos una NullReferenceException. Lo que tenemos que hacer es comprobar antes que el evento no es null (para mayor comodidad he creado una función privada de la clase Ciudad que lanza el evento):

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">private</span> <span style="color: blue">void</span> OnHabitanteDadoDeAlta(<span style="color: #2b91af">Persona</span> persona)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: blue">if</span> (HabitanteDadoDeAlta != <span style="color: blue">null</span>)
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160;&#160;&#160;&#160;&#160; HabitanteDadoDeAlta(persona);
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; }
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Este es el código típico que os encontraréis en la mayoría de sitios de ejemplos para lanzar un evento. 

> **Nota: Debo decir que este código NO es 100% correcto.** Contiene un sutil error que puede hacer que en según que casos (siempre relacionados con concurrencia) se reciba una NullReferenceException. En los foros de MSDN expliqué con bastante más detalle el tema y pongo el código correcto que debe usarse: [http://social.msdn.microsoft.com/Forums/es/vcses/thread/d28cf2c7-b024-4c1a-93aa-8dc5005ff713][3]. También [el maestro entre maestros][4] escribió hace tiempo [al respecto en su blog][5].

¡Listos! Con esto ya lo tienes todo! Ya sabes como declarar eventos, lanzarlos y suscribirte a ellos. Veamos ahora algunos detalles más…

**3. ¿Pero… las funciones gestoras no deben recibir siempre dos parámetros?**

Es mala costumbre de algunos que enseñan eventos en winforms (o incluso en WPF y Silverlight) decir algo parecido a: “_Las funciones gestoras de eventos siempre deben recibir dos parámetros. El primero es un object, que contiene el lanzador del evento, y el segundo contiene los datos del evento y es un objeto de una clase derivada de EventHandler_”.

Bueno… eso **no es más que una convención**. Una convención no es nada más que una regla que todos acordamos seguir, para hacernos la vida más fácil. Pero para nada es una obligación del sistema o de la plataforma. Es cierto que en winforms, WPF y Silverlight los eventos siguen esta convención. Por esto estarás harto de ver código parecido a:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    <span style="color: blue">private</span> <span style="color: blue">void</span> btnModificar_Click(<span style="color: blue">object</span> sender, <span style="color: #2b91af">EventArgs</span> e)
  </p>
  
  <p style="margin: 0px">
    {
  </p>
  
  <p style="margin: 0px">
    &#160;&#160;&#160; <span style="color: green">//...</span>
  </p>
  
  <p style="margin: 0px">
    }
  </p></p>
</div>

Como digo, todos los eventos que están declarados en winforms, en Silverlight y en WPF siguen esa convención (en este ejemplo concreto el evento Click está definido usando el delegate [EventHandler][6] que ya viene con .NET) y no está de más decir que tus propios eventos deberían seguirla (aunque si no lo hacen funcionará todo igual).

Por lo tanto: no, las funciones no deben siempre incluir esos dos parámetros, pero si desarrollas en winforms, WPF o Silverlight, deberías hacerlo para seguir la convención marcada.

**4. El código para suscribirme a los eventos es muy farragoso…**

Sí, es cierto, es una queja común. Por eso en VS2005 creo que fue, lo simplificaron, así que si quieres puedes hacer:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    _ciudad.HabitanteDadoDeAlta += Ciudad_HabitanteDadoDeAlta;
  </p></p>
</div>

Fíjate que es mucho más sencillo: te olvidas de hacer el new del delegate, pasas simplemente el nombre de la función gestora y listos. En este caso el compilador de C# infiere y crea el delegate por tí.

Y sí, crees que tener que crear una función para gestionar un evento, es un rollo y usas VS2008 o superior, puedes usar lambdas:

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    _ciudad.HabitanteDadoDeAlta += (p) => { <span style="color: #2b91af">Console</span>.WriteLine(<span style="color: #a31515">"{0} vive ahora en {1}"</span>, p.Nombre, _ciudad.Nombre); };
  </p></p>
</div>

Este mecanismo es el más compacto de todos, ya que NO estás obligado a declarar la función gestora, en su lugar pasas la lambda expression que la implementa. Fíjate que el compilador es capaz, incluso de inferir que el parámetro p es de tipo Persona (porque así obliga el delegate del evento). Si no te sientes cómodo con la notación de lambdas, piensa que NO estás obligado a usarla! 

Por supuesto el mecanismo de usar lambdas está recomendado cuando el código de las funciones gestoras es muy reducido, por visibilidad quieres tenerlo junto a la suscripción o bien te interesa usar closures.

Actualmente Visual Studio, incluso el 2010 compilando contra el Framework 4.0, sigue usando el primer código que hemos visto cuando debe generarte código para suscribirte a un evento… En MS tienen muy claro que lo que funciona no debe tocarse, o bien el tío que hizo esta parte de VS se ha ido y no saben como modificarla (si en MS trabajan como muchas empresas de España, probablemente será eso :P).

**5. Desuscripción de eventos**

Sí amigo… cuando dejes de estar interesado en recibir más notificaciones de eventos lanzados por un objeto debes _desuscribirte del evento_. Si no lo haces se seguirá ejecutando la función gestora que especificaste incluso cuando no te interesa para nada.

¿Cuando debes desuscribirte de un evento? Pues muy fácil, hay solo dos reglas:

  1. Cuando deja de interesarte recibir el evento (obvio) 
  2. Cuando estés suscrito a un evento que lance un objeto _que está fuera de tu ámbito de vida_ y tu vayas a “morir”. Por decir que está fuera de tu ámbito de vida, me refiero a que si es posible que el _otro_ objeto siga existiendo después de que tu “mueras”. 

El punto 2 mucha gente lo pasa por alto, y termina generando _memory leaks_. Sí, sí… también se pueden generar memory leaks en .NET.

Un ejemplo: en Winforms es normal que el formulario (Form) se suscriba a los eventos de los controles que él contiene. En este caso _no es necesario_ que el formulario se desuscriba, ya que cuando se destruya el objeto Form, todos los controles que este contiene son destruídos también. Pero, si este mismo formulario está suscrito a un evento que lanza un objeto que puede vivir después de que el Form haya sido eliminado, entonces el formulario **debe** desuscribirse de este evento, cuando vaya a ser eliminado. Esto se suele hacer en el método Dispose().

El código para desuscribirse a un evento es trivial (se usa el operador –=):

<div style="font-family: courier new; background: white; color: black; font-size: 10pt">
  <p style="margin: 0px">
    _ciudad.HabitanteDadoDeAlta -= Ciudad_HabitanteDadoDeAlta;
  </p></p>
</div>

No es posible desuscribirse a un evento al cual se haya suscrito a través de una expresión lambda, ¡o sea que mucho ojo con esto!

Y bueno… creo que ya está bien por hoy. Como siempre en los posts de esta serie de C# Básico, hemos empezado por lo elemental y hemos introducido algunas pinceladas más avanzadas. No sufras si no lo entiendes todo a la primera, hay una receta muy fácil: **¡practica y pregunta!**

¡Un saludo y feliz 2012!

 [1]: http://geeks.ms/blogs/etomas/archive/tags/c_2300_+basico/default.aspx
 [2]: http://geeks.ms/blogs/etomas/archive/2010/07/21/c-b-225-sico-delegates.aspx
 [3]: http://social.msdn.microsoft.com/Forums/es/vcses/thread/d28cf2c7-b024-4c1a-93aa-8dc5005ff713 "http://social.msdn.microsoft.com/Forums/es/vcses/thread/d28cf2c7-b024-4c1a-93aa-8dc5005ff713"
 [4]: http://geeks.ms/blogs/ohernandez/about.aspx
 [5]: http://geeks.ms/blogs/ohernandez/archive/2009/04/12/qu-233-problema-tiene-este-c-243-digo-ii.aspx
 [6]: http://msdn.microsoft.com/es-es/library/system.eventhandler(v=vs.80).aspx