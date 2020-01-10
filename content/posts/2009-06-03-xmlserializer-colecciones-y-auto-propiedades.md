---
title: XmlSerializer, colecciones y auto-propiedades…

author: eiximenis

date: 2009-06-03T09:06:30+00:00
geeks_url: /?p=1453
geeks_visits:
  - 2970
geeks_ms_views:
  - 1312
categories:
  - Uncategorized

---
Que XmlSerializer es una clase _curiosa_ es evidente, hay multitud de maneras de controlar la serialización de un objeto y varios trucos más o menos _ocultos_ (os recomiendo el [blog de jmservera][https://iremote.blogspot.com] que tiene algunos posts interesantes)…

… Lo que quiero comentaros ahora es un caso que me encontré el otro día (valeeee… ayer), en concreto con las auto-propiedades que se incorporaron en C# 3.0.

<!--more-->

En la msdn se dice que XmlSerializer es capaz de serializar propiedades `ICollection` o `IEnumerable` que sean read-only (de hecho la regla [CA2227 del análisis estático](https://msdn.microsoft.com/en-us/library/ms182327.aspx) se basa en esto). Pues bien, esto es cierto **si entendemos como read-only que no haya setter… ni tan siquiera privado**.

Es decir, esto NO funciona:

```cs
class Program
{
    static void Main(string[] args)
    {
        XmlSerializer xml = new XmlSerializer(typeof(Foo));
        using (FileStream fs = new FileStream(@"C:tempfoo.xml",
            FileMode.Open, FileAccess.ReadWrite, FileShare.None))
        {
            xml.Serialize(fs,new Foo());
            fs.Close();
        }

    }
}
public class Foo
{
    public Foo()
    {
        this.Data = new List<Bar>();
        this.Data.Add (new Bar());
        this.Data.Add (new Bar());
    }

    public List<Bar> Data { get; private set; }
    int OtherData;
}
public class Bar {}
```

Si intentamos serializar (o deserializar da igual) recibimos una System.InvalidOperationException (con un mensaje "_No se puede generar una clase temporal (result=1). error CS0200: No se puede asignar la propiedad o el indizador &#8216;ConsoleApplication6.Foo.Data&#8217; (es de sólo lectura)_".

Si transformamos la clase Foo al estilo de C# 2.0 sin usar auto-propiedades, el código funciona, siempre que no pongamos el setter privado…

… como mínimo curioso, no?

Es evidente que cuando se hizo XmlSerializer nadie pensó en un setter de propiedad privado (ya que entonces lo usual era no poner setters privados en propiedades read-only), pero con la aparición de las auto-propiedades en C# 3.0 estos son cada vez más frecuentes… así que igual habría que actualizar XmlSerializer, porque que me obliguen a usar el estilo de C# 2.0 para poder serializar las propiedades de colección no es que me guste especialmente.

Y quizá, ya puestos a pedir, C# debería incorporar auto-propiedades read-only sin necesidad de poner el setter privado y que tuviesen la misma semántica que las variables readonly: sólo podrían ser inicializadas en el constructor. Esto permitiría también que el CLR realizara determinadas optimizaciones…

Saludos!! 😉

