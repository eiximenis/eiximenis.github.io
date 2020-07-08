---
title: "Unit testing de métodos privados"
author: eiximenis
description: "Esa es una discusión que aparece de vez en cuando: ¿como testeamos los métodos privados?"
date: 2020-07-08T18:00:00
draft: false
categories:
  - general
---

Este es uno de esas cuestiones de las que se ha escrito mucho, así que este post tampoco va a descubrir la sopa de ajo, pero bueno... si os sirve para algo esas son **mis impresiones** sobre las pruebas unitarias en métodos privados. Este post viene motivado por un tweet que lanzó [Juanjo](https://twitter.com/kastwey):

{{< tweet 1280798868859469825 >}}

A partir de ahí se originó un mini-hilo donde intervení diciendo que la necesidad de probar métodos privados generalmente solía emerger de un mal diseño (asociado usualmente a la baja cohesión) y [fuí interpelado](https://twitter.com/jc_quijano/status/1280823433971064832) a desarrollar el argumento, así que eso intentaré :)

## Cohesión

La cohesión del código es, sin duda, uno de los conceptos que cuesta más de entender: se trata de medir cuan "relacionado" está nuestro código dentro un módulo (pongamos clase). Queremos que el código **esté altamente cohesionado** ya que eso hará que sea más mantenible y fácilmente entendible que si la cohesión es baja. No hay que confundir la cohesión con el acoplamiento. El acoplamiento nos indica cual es el nivel de interdependencia entre dos módulos (pongamos clases). A diferencia de la cohesión, el acoplamiento queremos que sea bajo. Ese es el objetivo: **alta cohesión y bajo acoplamiento**.

El código que tiene una baja cohesión tiende a romper el [principio de responsabilidad única](https://es.wikipedia.org/wiki/Principio_de_responsabilidad_%C3%BAnica). No hay una "metrica de cohesión", no tenemos nada como la [complejidad ciclomática](https://es.wikipedia.org/wiki/Complejidad_ciclom%C3%A1tica), no podemos sacar un número que nos diga "tu código está un 27% cohesionado". Es un elemento más intangible que otra cosa, pero está ahí y hay que prestarle atención. 

Hay varios tipos de cohesión (es decir varios tipos de criterios que se pueden seguir para agrupar código en un módulo), siendo la "cohesión funcional" el mejor tipo de cohesión: cuando todos los métodos de la clase contribuyen a una única tarea. Otras cohesiones serían la "cohesión por casualidad" que es la que se da cuando agrupamos funciones "sin ton ni son" (caso típico de las clases Utils que son sacos donde va a parar todo aquello que no sabemos donde meter) o la cohesión secuencial (operaciones que se agrupan porque ocurren uno tras otro, en plan "leer fichero, procesar datos, guardar datos"). En fin, es todo un mundo.

## Métodos privados y testing

A ver, empecemos diciendo que **no hay ninguna regla que te diga que deben haber tests para todos los métodos**. Ese es un malentendido que tiene mucha gente, que confunde la cobertura con la existencia de un test: lo que queremos es tener una alta cobertura, de acuerdo, pero eso no implica que necesitemos probar todos los métodos con tests independientes. Lo que necesitamos es garantizar que en la suite de tests que tenemos, el código ejecutado por esos tests, ejecutará también los métodos privados. Básicamente hay dos (más uno) tipos de métodos (ya sean privados o no):

* Devuelven un resultado
* No devuelven nada, pero generan un efecto observable desde fuera (un _side effect_).
* Devuelven un resultado y además generan un efecto observable desde fuera (un _side effect_) (esos intenta evitarlos).

> Un método que no genera _side effects_ es siempre preferible: es una pieza de código fácilmente testeable y reemplazable. Deberías intentar que el máximo múmero posible de tus métodos sean de ese tipo (aunque al final desarrollamos software para que haga _algo visible_, así que al final siempre tendrás _side effects_).

Si un método privado devuelve un resultado, ese resultado será usado desde un método público. Si el resultado es incorrecto, el comportamiento del método público lo será también. En este caso un conjunto de tests sobre el método público nos prueba el privado. Por otro lado si el método privado genera un _side effect_ (usualmente modificando un campo de la clase), habrá una secuencia de llamadas a métodos públicos donde se parta de un estado inicial X, se modifique (por gracia del método privado) a otro estado Y (medible) que puede usarse para calcular un resultado. Si el estado final es correcto (o lo es el resultado medido) es que el paso de X a Y ha sido correcto, por lo que el método privado ha funcionado. De nuevo, a través de un conjunto de llamadas de métodos públicos podemos probar el método privado.

Por tanto: **siempre puedes probar un método privado con uno o más tests que llamen a uno o más métodos públicos**. No deberías tener nunca ninguna necesidad de probar métodos privados directamente.

Ahora bien, ¿cual es la necesidad por la que mucha gente quiere probar el método privado directamente? Pues en muchos casos es que "el test usando el método público es muy complicado/necesita muchos mocks/no me queda claro el assert". Todas esas cuestiones suelen revelar problemas en el código.

## El test sobre el método público requiere de muchos Mocks o es complejo

Esto suele suceder cuando el método público tiene demasiadas dependencias:

```csharp
public class Offer {
  public void Save() {
    var finalprice = ApplyDiscounts();
    _db.Save(this.Data)
  }
  private void ApplyDiscounts() {
    // Lógica chunga pelotera de descuentos que modifica this.Data.Price
  }
}
```

Aquí nos puede interesar probar el método `ApplyDiscounts` y hacerlo a través de `Save` es dificil: el método `ApplyDiscounts` genera un _side effect_  (modificar `this.Data.Price`) que luego es usado en el método `Save` de lo que sea `_db`. Para verificar que este método funciona bien necesitamos montar un pifostio de Mocks o bien usar un test de integración con una BBDD real. Ante esas disyuntivas es fácil cortar por lo sano, convertir "ApplyDiscounts" en público y listos. Pero nunca deberíamos ceder a esa tentación. ¡Y ojo! **no estoy diciendo que el método `ApplyDiscounts` no deba ser público, pero si lo es no será en esa forma**.

Ante esos problemas, deberíamos ver como podemos refactorizar este código para simplificarnos la vida con los tests. Una solución puede ser la siguiente:

```csharp
public class Offer {
  public void Save() {
    var discount = PriceCalculator.GetDiscount(this);
    this.Data.Price -= discount;
    _db.Save(this.Data)
  }
}

public class PriceCalculator {
  public static decimal GetDiscount(Offer offer) {
    // Lógica chunga pelotera de descuentos que DEVUELVE el descuento
  }
}
```

¡Ah! El método ha terminado siendo público, pero en este caso ha aterrizado en otra clase y además ahora es un método que NO genera _side effect_ si no que devuelve un resultado (¡bien!). Ahora podemos probarlo de forma mucho más fácil y sencilla. Este código es incluso mejorable, haciendo p. ej. que el método `GetDiscount` no dependa de `Offer` si no solo de las partes necesarias (p. ej. el cliente y las líneas de la oferta).

Claro, puedes pensar que he hecho trampas: tanto dar la chapa con que los métodos privados deben probarse a través de los públicos, y a las primeras de cambio... ¡convierto un privado en un público!. Cierto, **pero no ha sido cambiando `private` por `public`**: el método ha aterrizado en otra clase y además antes generaba _side effects_ y ahora no. Sin duda ese segundo código es más mantenible y testeable que el primero. Y **eso lo he visto sobre todo en el momento de hacer tests. Es por eso que NO PUEDO DIFERIR LOS TESTS AL FINAL**. No estoy diciendo que debas hacer [TDD](https://es.wikipedia.org/wiki/Desarrollo_guiado_por_pruebas), pero si que deberías ir escribiendo tests a medida que vas escribiendo tu código. Si escribes un montón de código y luego los tests, te puedes encontrar que los tests te "obliguen" a modificar tu estructura de código, lo cual es más complejo cuando más código hayas tirado.

Este mismo problema, a veces en lugar de expresarse como dependencias con Mocks aparece con que los tests sobre el método público son complejos:

```csharp
public class Offer {
  public string GetDescription() {
    if (!_discountsApplied) ApplyDiscounts();
    return $"The offer has {_items.Count} items with a total of {_price} {_currency}";
  }
    private void ApplyDiscounts() {  
    // Lógica chunga pelotera de descuentos que modifica this.Data.Price
  }
}
```

Es el **mismo problema que antes** (y por lo tanto, la misma solución), salvo que en este caso en lugar de ser Mocks lo que tienes es un test sobre `GetDescription` que es complejo (y potencialmente frágil): debes recuperar la cadena y parsearla a mano para obtener el precio. Ya, da pereza solo de pensarlo, ¿no?

En este caso los métodos `ApplyDiscounts` y `Save` eran dos métodos poco cohesionados. Es cierto que ambos dependían de un mismo "paquete de datos" (campos de la oferta), pero realmente poco tenían en común (el primero no dependía de nadie, el segundo de la base de datos). Es hasta normal que aterricen en clases distintas (observa que en mi caso, las clases `Offer` y `PriceCalculator` están altamente acopladas, pero ese es otro problema).

Quizá no te convenza el hecho de que nos aparezca una clase `PriceCalculator` con un solo método... ¿Eso está bien? Mira, no voy a responderte yo a esa pregunta. Dejaré que lo haga un tal [John Carmack](https://es.wikipedia.org/wiki/John_Carmack):

{{< tweet 53512300451201024 >}}

**Uno de los mayores errores de Java fue "obligar" a que todo método formase parte de una clase**. Luego para más _inri_ cuando Microsoft sacó C# no aprovechó para corregir esta aberración (porque lo es). A veces lo que necesitas es una función. Solo una función y nada más. Java ha hecho que mucha gente vea mal esas clases estáticas con una sola función estática porque es como "hacer trampa": la OOP no debe permitir eso. Habría mucho que hablar ahí sobre lo que es o no es OOP, así que déjate de chorradas: si la solución elegante es una sola función hazla. ¿Qué C# y Java nos obligan a ponerla en una clase estática? Pues hazlo. Sin preocupaciones.

Por supuesto, no siempre que tienes un método privado esto es un indicador de un problema de tu código. El indicador es más "la dificultad" para probar ese método privado a través de los métodos públicos. No el método privado en sí. Un ejemplo:

```csharp
public class TextProcessor
{
    private readonly StringBuilder _sb;
    public bool ReplaceMode { get; private set; }
    public int CurentPos { get; private set; }
    public string Text => _sb.ToString();

    public TextProcessor()
    {
        _sb = new StringBuilder();
        CurentPos = 0;
        ReplaceMode = false;
    }

    public void ProcessKey(ConsoleKeyInfo consoleKeyInfo)
    {
        var key = consoleKeyInfo.Key;
        var special = ProcessSpecialKey(key);
        if (!special && consoleKeyInfo.KeyChar != '\0') 
        {
            InsertChar(consoleKeyInfo.KeyChar);
        }
    }
    public void RemoveChars(int pos, int count = 1)
    {
        if (pos < 0)
        {
            return;
        }
        if (pos >= _sb.Length)
        {
            pos = _sb.Length;
        }
        if (count > _sb.Length - pos)
        {
            count = _sb.Length - pos;
        }
        _sb.Remove(pos, count);
        DecreasePosition(count);
    }    
    private bool ProcessSpecialKey(ConsoleKey key)
    {
        switch (key)
        {
            case ConsoleKey.Backspace:
                RemoveChars(CurentPos - 1);
                return true;
            case ConsoleKey.Delete:
                RemoveChars(CurentPos);
                return true;
            case ConsoleKey.LeftArrow:
                DecreasePosition();
                return true;
            case ConsoleKey.RightArrow:
                IncreasePosition();
                return true;
            case ConsoleKey.End:
                GoToEnd();
                return true;
            case ConsoleKey.Home:
                GoToBegin();
                return true;
            case ConsoleKey.Insert:
                ReplaceMode = !ReplaceMode;
                return true;
        }

        return false;
    }

    private void GoToEnd()
    {
        CurentPos = _sb.Length;
    }

    private void GoToBegin()
    {
        CurentPos = 0;
    }

    private void InsertChar(char value)
    {
        if (ReplaceMode && CurentPos < _sb.Length)
        {
            _sb[CurentPos] = value;
        }
        else
        {
            _sb.Insert(CurentPos, value);
        }
        IncreasePosition();
    }
    private void DecreasePosition(int count = 1)
    {
        CurentPos -= count;
        if (CurentPos < 0) CurentPos = 0;
    }

    private void IncreasePosition(int count = 1)
    {
        CurentPos += count;
        if (CurentPos > _sb.Length) CurentPos = _sb.Length;
    }
}
```

Si en esta clase quiero probar si el método `DecreasePosition` puedo probarlo fácilmente a través de un test que:

1. Llame a `ProcessKey` y le pase una tecla normal
2. Llame a `ProcessKey` pasándole `ConsoleKey.LeftArrow`
3. Verifique que `CurentPos` sea 0.

Ojo, que **este test se apoya en otros tests existentes**. P. ej. tendré otro test que solo llamarà una vez a `ProcessKey` con una tecla normal y verificará que `CurrentPos` es 1. Y podría tener un tercero que verifique que nada más crear el objeto `CurrentPos` es 0. De este modo, sé (por el tercer test) que la posición empieza en 0, y sé (por el segundo test) que al procesar una tecla normal la posición se incrementa. Estos resultados de esos dos tests anteriores son los axiomas con los que se apoya el test.

De hecho, he dicho antes que "quiero probar si el método `DecreasePosition`", pero **esa es una mala forma de pensar**. Yo no quiero probar realmente eso. Yo quiero probar si `TextProcessor` responde bien a todas las teclas y que responda bien a `ConsoleKey.LeftArrow` es una prueba más. Por lo tanto, es mejor pensar en términos de funcionalidad de la clase y no de métodos privados. ¿Responde bien `TextProcessor` a la tecla de borrar? O a la Insertar? Debo ir creando tests para cada caso, y esos tests se basarán solo en la interfaz de la clase. Idealmente no tengo ni que saber qué métodos privados hay (o esa información debería ser irrelevante para generar mis tests). Ya... pero **¿si no hago un test diseñado para probar `DecreasePosition` como sé que la estoy testeando?** Pues muy fácil: **por el informe de cobertura**. Si después de ejecutar los tests el informa de cobertura me dice que `DecreasePosition` está cubierta, pues listos. Vamos, es que me da igual cual es el test (o tests) que ha generado esta cobertura: yo he probado la funcionalidad de la inferfaz pública de mi clase, y eso ha llevado indirectamente a probar los métodos privados.

## Conclusión

En definitiva, no deberías diseñar tests "para probar" un método privado, si no funcionalidades de la interfaz pública de la clase. Si tienes la necesidad de "probar métodos privados" porque "hacer los tests sobre los métodos públicos" eso, muchas veces, es indicador de una mala organización de tu código (generalmente baja cohesión, ruptura del SRP).

Por supuesto, entiendo que cada caso es un mundo y que el mundo real es más complejo que un post en un blog, pero espero que esas ideas te hayan ayudado. Por supuesto, si tienes casos concretos a comentar, deja un comentario y ¡lo vemos entre todos!