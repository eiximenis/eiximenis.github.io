---
title: 'Marker Interface: ¿Patrón o Anti-patrón?'

author: eiximenis

date: 2018-01-19T13:14:31+00:00
geeks_url: /?p=1984
geeks_ms_views:
  - 1257
categories:
  - 'C#'
  - patrones

---
Llamamos _marker interface_ a una interfaz vacía. Sí, sí sin métodos ni propiedades ni nada. A pesar de que te pueda parecer una tontería tiene sus usos. Vamos hablar un poco de este patrón y sus usos y por qué es en cierta manera un anti-patrón, aunque no siempre, porque en esa vida, como todo, todo depende...
  
<!--more-->


  
Empecemos por el principio. Una _marker interfaz_ es una interfaz vacía. Algo como así:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">interface SensitiveEntity {}</pre>

¿Y eso para que narices sirve? Bueno, pues en general nos permite establer relaciones _is-a_ &#8220;ficticias&#8221;. Las llamo &#8220;ficticias&#8221; porque dado que _SensitiveEntity_ no define comportamiento alguno cualquier objeto de cualquier tipo es realmente una _SensitiveEntity_. Bueno, al menos conceptualmente, pero no para el compilador. Para el compilador la relación solo existe si hay una implementación de la interfaz, pero observa que cualquier clase puede implementar esta interfaz.
  
¿Y cual es el uso típico de esa interfaz? Pues código parecido al siguiente:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">static string GetSerializationString(object o)
{
    var json = JsonConvert.SerializeObject(o);
    if (o is SensitiveEntity)
    {
        json = EncryptJson(json);
    }
    return json;
}</pre>

La idea es que si el objeto que queremos serializar es una _SensitiveEntity_ entonces encriptamos el json resultante. En caso contrario no es necesario.
  
Al final usamos la _marker interface_ solo para consultar esta relación _is-a_ para tomar una acción o no. No nos interesa para nada el comportamiento de _SensitiveEntity_.
  
**Por qué se puede considerar un anti-patrón**
  
Como siempre lo que es un anti-patrón depende del lenguaje. Lo que expongo a continuación es válido para C# (y también para Java). Desde el punto de vista académico de OO, el uso de una interfaz vacía no tiene sentido. Pero el mundo académico es una cosa y la realidad otra muy distinta.
  
El motivo principal por el cual podemos considerar el _marker interface_ un anti-patrón es simplemente que **hay una alternativa pensada precisamente para esos casos**. A ver, la realidad es que estamos usando esta interfaz para añadir metadatos a un tipo. En este caso queremos añadir el metadato de que este tipo contiene datos sensibles. Que este metadato implique modificar la jerarquía de tipos de nuestro programa no es algo que debiera ocurrir.
  
Tanto Java como C# (como otros lenguajes) tienen ya mecanismos explícitos para añadir metadatos a tipos. En Java lo llaman _annotations_ y en C# lo llamamos _atributos_:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">class SensitiveEntityAttribute: Attribute { }
[SensitiveEntity]
class MyEntity { }</pre>

Este código muestra la declaración del atributo _SensitiveEntityAttribute_ y como lo aplicamos a una clase usando la notación de corchetes. Con eso agregamos los metadatos correspondientes al tipo _MyEntity_. Eso no presupone nada a nivel tipos, simplemente indica que el tipo _MyEntity_ tiene unos metadatos (cuyo tipo es _SensitiveEntityAttribute_).
  
Ahora en nuestro _GetSerializationString_ podemos consultar todos los metadatos del tipo y mirar si alguno de ellos es del tipo que deseamos:

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">static void WriteObject(object o)
{
    var json = JsonConvert.SerializeObject(o);
    if (o.GetType().GetCustomAttributes(typeof(SensitiveEntityAttribute), inherit: false)
        .Any())
    {
        json = EncryptJson(json);
    }
}</pre>

Al margen de que eso no implica nada a nivel de jerarquía de tipos, hay otra ventaja en usar atributos. Si volvemos al ejemplo en que usábamos una interfaz, si una clase A implementa la interfaz, no hay manera alguna de que alguna subclase de A deje de implementarla. Eso **significa que usando _marker interface_ los &#8220;metadatos&#8221; (representados como implementaciones de interfaces) son heredados automáticamente por todas las clases derivadas**. Eso usando atributos no ocurre, o dicho de otro modo se puede decidir si se quiere que los atributos se hereden o no (observa el valor del parámetro _inherit_ en el código anterior).
  
Por esos motivos se puede decir que el uso típico de una _marker interface_ en C# es un anti-patrón y deberías considerar usar atributos en su lugar.
  
**Pero hay un caso que...**
  
... en el que si veo lícito usar una _marker interface_. Y es, por supuesto, porque el lenguaje no nos permite hacerlo de otro modo. En el caso de C# es cuando quieres consultar esos metadatos en tiempo de compilación (generalmente usando una restricción sobre un tipo genérico):

<pre class="EnlighterJSRAW" data-enlighter-language="csharp">interface Immutable { }
class A : Immutable { }
class ImmutableList&lt;T&gt; : IEnumerable&lt;T&gt;
    where T: Immutable
{
}</pre>

La clase _ImmutableList_ está diseñada para contener objetos immutables. Eso le permitiría ciertas optimizaciones a la hora de calcular p. ej. si la lista se ha modificado o no. Para **enfatizar esto** forzamos a que solo se puedan añadir objetos que implementen _Immutable_ a dicha lista (mediante una restricción de genéricos). Con esto obtenemos dos ventajas:

  1. Nuestro código está más documentado
  2. Cuando un desarrollador quiere usar _ImmutableList_ con un tipo suyo, ese tipo debe implementar _Immutable_. Aunque eso no implica que deba hacer modificación alguna en el tipo, **al menos si que tomará consciencia de que este tipo debería ser inmutable**.

Por supuesto implementar la interfaz no convierte al tipo en inmutable, y cualquier tipo puede implementar la interfaz, pero en este caso la interfaz aporta un pequeño plus de alerta para los desarrollados y, sobre todo, ayuda a documentar el código.
  
¿Es estrictamente necesario? No. La clase _ImmutableList_ podría prescindir de esta restricción del tipo genérico y estaría implementada exactamente igual. A nivel funcional esa interfaz totalmente superflua, por lo que aquí ya, cada uno, debe valorar si vale la pena o no.
  
¡Un saludo!