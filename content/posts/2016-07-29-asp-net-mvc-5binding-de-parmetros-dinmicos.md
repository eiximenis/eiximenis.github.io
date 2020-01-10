---
title: ASP.NET MVC 5–Binding de parámetros dinámicos

author: eiximenis

date: 2016-07-29T08:21:05+00:00
geeks_url: /?p=1792
geeks_ms_views:
  - 2490
categories:
  - asp.net MVC
---

Estando trabajando en un proyecto con ASP.NET MVC 5 surgió la “necesidad” (impulsada por la pereza) de tener un controlador MVC (ojo, no WebApi) que recibiese datos en JSON via POST y que devolviese una vista parcial. Hasta ahí nada raro (eso se soporta de serie desde MVC3), donde la pereza intervino es que queríamos que el parámetro del controlador fuese dynamic en lugar de un tipo en concreto. Y eso, en MVC5 no está soportado. Veamos por qué y como podemos solucionarlo 😉

<!--more-->

**Nota:** Todo lo dicho en este post afecta solo a MVC5. En WebApi y en MVC6 (ASP.NET Core) las cosas funcionan distinto.

## Por qué no se enlazan parámetros dinámicos en MVC5

Para ello tenemos que entender como funciona el proceso de model binding en MVC5. He hablado, en este mismo blog, varias veces sobre ello, así que el proceso resumido sería:

1. Los Value Providers analizan la petición y dejan los datos de la petíción en un sitio común. Cada value provider analiza una parte de la petición (cuerpo, query string. path info, cookies, cabeceras, etc). Podemos tener varios value providers que analicen la misma parte de la petición (p. ej. el cuerpo) pero dependiendo del formato. Es decir, un value provider analiza el cuerpo si los datos son en JSON, otro distinto si son XML, etc.
2. El model binder analiza el parámetro que recibe el controlador y basándose en el tipo de parámetro recoje los valores necesarios dejados por los value providers. La clave: el model binder se basa en el tipo del parámetro recibido por la acción del controlador.

Si tenemos un parámetro de una clase, (llamémosla BeerVM) que tiene dos propiedades Id y Name y una acción de un controlador recibe como parámetro un objeto BeerVM, el model binder preguntará a los value providers por los valores “Id” y “Name” pues esas son las propiedades del objeto. Es indiferente si esos valores han venido vía query string, path info, en el cuerpo o donde sea (siempre que haya un value provider que lea de allí).

Entendiendo esto se puede ver claramente porque MVC5 es incapaz de enlazar un parámetro dynamic: ¿qué propiedades tiene el tipo dynamic? De hecho, debido a como funciona realmente dynamic, lo que el model binder ve es un “object” así que no enlaza ninguna propiedad. Ojo, no es que el parámetro no se enlace (en ese caso valdría null) es que no lo hace ninguna propiedad… por que ¡realmente no tiene ninguna!

![Controlador no recibe datos](https://geeks.ms/etomas/wp-content/uploads/sites/154/2016/07/binding.png)

En la imágen se puede observar como, a pesar de enviar la petición en JSON con los datos (a la derecha le petición generada por Postman), en el controlador no recibimos los datos, ya que el parámetro “data” es tratado como un object. De hecho si hicieramos data.id el código compilaría (data es dynamic) pero en ejecución nos daría un error.

## Como podemos enlazar parámetros dynamic

Veamos una posible solución. Tendrá sus limitaciones, pero servirá para el escenario básico: enlazar un parámetro dynamic con el contenido de todo el cuerpo de la petición (algo parecido a lo que hace WebApi). Otra limitación es que solo soportaremos JSON (application/json), aunque el sistema sería extensible a otros formatos.

Para ello vamos a crearnos un value provider adicional junto con un model binder específico para dynamic. Quizá te preguntes por qué necesitamos el value provider, cuando ASP.NET MVC ya tiene uno que lee el cuerpo en el caso de application/json, ¿verdad? La respuesta es que si queremos aprovechar el value provider que tiene ya ASP.NET MVC el model binder que necesitamos para los parámetros dynamic no se puede implementar. Recuerda siempre que el enlazado de los parámetros va gobernado por el model binder. Es este quien pregunta a los value providers por las propiedades necesarias (una a una). No hay manera de preguntarle a los value providers que propiedades han leído. Solo podemos preguntarles por una propiedad concreta. Si no sabemos por cual preguntar… mal vamos.

La solución pues va a consistir en un value provider específico que lea el cuerpo y lo guarde con un nombre específico (solo conocido por él y nuestro model binder). Además el value provider ya guardará el cuerpo deserializado como un dynamic (es lo más sencillo). Nuestro model binder será muy “tonto”: se limitará a preguntar a los value providers si existe el valor con este nombre concreto y si existe (existirá solamente si nuestro nuevo value provider ha intervenido) asignará su valor al modelo. Vayamos por parte y empecemos por el value provider.

Lo primero, por supuesto, es crear la factoría:

```cs
public class DynamicValueProviderFactory : ValueProviderFactory
{
    public override IValueProvider GetValueProvider(ControllerContext controllerContext)
    {
        var ctype = controllerContext.HttpContext.Request.ContentType;
        if (ctype.StartsWith("application/json"))
        {
            return new DynamicJsonValueProvider(controllerContext);
        }
 
        return null;
    }
}
``` 

No tiene mucha complicación: Se limita a mirar que el content-type de la petición sea “application/json” y en este caso devuelve nuestro value provider. En cualquier otro caso, devolvemos null porque no tenemos un value provider para convertir a dynamic otros formatos.

El value provider es realmente simple:

```cs
public class DynamicJsonValueProvider : IValueProvider
{
    private ControllerContext _controllerContext;
    private dynamic _bodyData;
 
    public DynamicJsonValueProvider(ControllerContext context)
    {
        _controllerContext = context;
    }
 
    public bool ContainsPrefix(string prefix)
    {
        return prefix == "__dynamic_data";
    }
 
    public ValueProviderResult GetValue(string key)
    {
        if (_bodyData == null)
        {
            ReadBodyData();
        }
        if (key == "__dynamic_data")
        {
            return new ValueProviderResult(_bodyData,
                _bodyData.ToString(), CultureInfo.CurrentCulture);
        }
 
        return null;
    }
 
    private void ReadBodyData()
    {
        var req = _controllerContext.HttpContext.Request;
        using (var stream = req.InputStream)
        using (var reader = new StreamReader(stream))
        {
            stream.Seek(0, SeekOrigin.Begin);
            var data = reader.ReadToEnd();
            _bodyData = JObject.Parse(data);
        }
    }
}
```

Básicamente lo que hace en el método `ReadBodyData` es usar JSON.NET para leer el contenido del cuerpo de la petición y guardarlo en un JObject (JObject es una clase que es dynamic-friendly, eso es, puede ser consultada usando una referencia dynamic). Este value provider guarda en una clave específica (“__dynamic_data”) el JObject con el resultado de deserializar el cuerpo de la petición.

Con esto, cuando haya una petición con “application/json” nuestro value provider intervendrá (además del value provider por defecto que tiene MVC5).

El siguiente paso es crear el model binder. Y realmente más sencillo no puede ser:

```cs
public class DynamicModelBinder : IModelBinder
{
    public object BindModel(ControllerContext controllerContext, ModelBindingContext bindingContext)
    {
        var dresult =
            bindingContext.ValueProvider.GetValue("__dynamic_data");
        if (dresult != null)
        {
            dynamic info = dresult.RawValue;
            return info;
        }
        return null;
    }
}
```

Directamente pregunta por la clave `__dynamic_data`. Si existe, significa que nuestro value provider ha actuado. En este caso, nuestro Model Binder recoje el valor de dicha clase y lo devuelve (y así este será el valor asignado al parámetro del controlador).

Finalmente nos queda añadir nuestro value provider y nuestro model binder en la configuración de MVC5:

```cs
ValueProviderFactories.Factories.Add(new DynamicValueProviderFactory());
ModelBinders.Binders.Add(typeof(object), new DynamicModelBinder());
```

Cuando añadimos un model binder debemos asociarlo a un tipo. Usar typeof(dynamic) no es correcto, porque dynamic no existe en ejecución. Dado que los dynamic son objects, lo asociamos al tipo object que eso es lo que ve realmente el CLR.

Y ahora sí. Ya podemos repetir la misma petición con Postman y…

![Ahora se reciben los datos](https://geeks.ms/etomas/wp-content/uploads/sites/154/2016/07/binding2.png)

Como se puede ver en la imágen ¡nuestro parámetro dynamic está enlazado correctamente!

Una cosa interesante es que podemos enlazar parámetros dinámicos y no dinámicos sin problemas. Si tenemos una clase `BeerVM` con las propiedades `id` y `name`, podemos tener un método como el siguiente:

```cs
public ActionResult DetailsBoth(dynamic data1, BeerVM data2)
{
    return Json(new { Ok = true });
}
```

Y ambos parámetros se nos enlazarán (con los mismos valores por supuesto) si repetimos la petición, como se puede comprobar en la siguiente imágen:

![Enlazando paramétros dinámicos y no dinámicos](https://geeks.ms/etomas/wp-content/uploads/sites/154/2016/07/binding3.png)

Bueno, espero que este post os haya resultado interesante… La verdad es que pensándolo bien, quizá la pereza para definir un viewmodel para nuestra acción no compensa tener que montar esto, pero oye… ¿y lo qué nos hemos divertido?

Saludos!



