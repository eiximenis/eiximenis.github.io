---
title: PRISM y AvalonDock
author: eiximenis

date: 2009-01-20T12:56:00+00:00
geeks_url: /?p=1432
geeks_visits:
  - 2223
geeks_ms_views:
  - 1263
categories:
  - Uncategorized

---
Hola a todos!

Conocéis PRISM? Viene a ser, salvando las distancias, la CAB de WPF: es decir un conjunto de buenas prácticas para la creación de aplicaciones compuestas en WPF y una librería que implementa dichas buenas prácticas. Si desarrollais aplicaciones en WPF es obligatorio echarle un vistazo. Pasaos por la [página de PRISM en codeplex][1].

Por otro lado, [AvalonDock][2] es una muy buena librería que proporciona soporte para interfaces _dockables_ usando WPF que simula al estilo de docking de Visual Studio. 

Estoy desarrollando una aplicación usando ambas librerías y me he encontrado con un _problemilla_: al añadir una vista usando PRISM dentro de un contenedor de AvalonDock aparece un error. A ver, que me explico un poco mejor... 🙂

PRISM usa el concepto de &ldquo;regiones&rdquo; para dividir el espacio de la ventana de la aplicación. En cada &ldquo;región&rdquo; se pueden incrustar una o más vistas (objetos que se representan visualmente). Por ejemplo podemos mapear una región de PRISM a un ItemsControl y cada vista que añadamos aparecerá dentro de este ItemsControl. Para mapear una región PRISM a un control se usa XAML:

```xml
<ItemsControl Grid.Row="0" cal:RegionManager.RegionName="HelpZone" />
```

Por su lado AvalonDock se basa en proporcionar un contenedor especial (el DockingManager) dentro del cual se insertan otros contenedores especiales que contienen el contenido a mostrar... el cual tiene que ser un objeto de unas clases especiales, llamadas DockableContent o DocumentContent. Estas clases son las que _realmente_ contienen el contenido real. P.ej. para mostrar una cadena dentro de una ventana _dockable_ usando AvalonDock necesito el siguiente código XAML:

```xml
<ad:DockingManager Name="mainDockingManager" Grid.Row="1">
  <ad:DocumentPane>
     <ad:DockableContent>Hola AvalonDock</ad:DockableContent>
  </ad:DocumentPane>
</ad:DockingManager>
```

La clase DocumentPane contiene tantos DockableContent como ventanas _dockables_ se quieran tener.

Al mezclar PRISM y AvalonDock es cuando surgen los primeros problemas. Si definimos una región dentro del DocumentPane:

```xml
<ad:DockingManager Name="mainDockingManager" Grid.Row="1">
  <ad:DocumentPane cal:RegionManager.RegionName="MainZone" />
</ad:DockingManager>
```

Cuando añadamos una vista, no dará una excepción: `DocumentPane can contain only DockableContents or DocumentContents!`

Esto ocurre porque PRISM intenta asociar como contenido del DocumentPane la vista que directamente le hemos indicado, que será un UserControl o algún otro objeto pero no un DockableContent o un DocumentContent.

Lo que hemos de conseguir es que PRISM, de forma transparente para nosotros, nos cree un DockableContent (o un DocumentContent) y nos lo añada al contenido del DocumentPane al cual está mapeado nuestra región de PRISM. Por suerte,&nbsp; en PRISM tenemos el concepto de _RegionAdapter_, que como su nombre indica es una clase que &ldquo;adapta&rdquo; los contenidos de una región de PRISM al contenedor real (nuestro DocumentPane). Vamos a ver como podemos implementar un RegionAdapter para DocumentPane.

El código quedaría más o menos así:

```cs
public class DockableRegionAdapter : RegionAdapterBase<DocumentPane>
{
    protected override IRegion CreateRegion()
    {
        return new AllActiveRegion();
    }

    protected override void Adapt(IRegion region, 
    DocumentPane regionTarget)
    {
        region.ActiveViews.CollectionChanged += delegate
        {
            var childs = new Dictionary<object, DockableContent> ();
            foreach (var child in regionTarget.Items)
            {
                if (child is DockableContent)
                {
                    childs.Add(((DockableContent)child).Content, 
                        (DockableContent)child);
                }
            }
            regionTarget.Items.Clear();
            foreach (object ci in region.Views)
            {
                DockableContent dc = childs.ContainsKey(ci) 
                    ? childs[ci]
                    : new DockableContent() { Content = ci };
                    regionTarget.Items.Add(dc);
            }
        };
    }
}
```

Los dos métodos que hay que redefinir cuando se crea un RegioAdapter de PRISM son `CreateRegion` y `Adapt`. En el primero simplemente hemos de devolver el tipo de región que queremos. En este caso simplemente creo un objeto AllActiveRegion, que es un región de PRISM que todas las vistas que tenga las considera activas.

El segundo método es Adapt, y es donde se hace todo el trabajo. Cada vez que cambie la colección ActiveViews, básicamente borro el contenido del DocumentPane de AvalonDock y lo añado de nuevo, creando un DockableContent nuevo para cada vista que haya en la región. El código que rellena el diccionario _childs_ es para reutilizar aquellos DockableContent que se hubiesen creado anteriormente.

Finalmente solo nos queda informar a PRISM que tenemos un RegionAdapter nuevo. Para ello redefinimos el método `ConfigureRegionAdapterMappings` del Bootstrapper para añadir nuestro RegionAdapter vinculado a contenedores de tipo `DocumentPane`:

```cs
protected override RegionAdapterMappings 
    ConfigureRegionAdapterMappings()
{
    RegionAdapterMappings mappings = 
        base.ConfigureRegionAdapterMappings();
    mappings.RegisterMapping(typeof(DocumentPane), 
        new DockableRegionAdapter());
    return mappings;
}
```

Y listos! Ahora al añadir una vista a la región de PRISM, se crea automáticamente un DockableContent y se añade al DocumentPane que contiene la región, lo que añade una ventana _dockable_ con la nueva vista en la interfaz de usuario.

No puedo asegurar que sea la mejor implementación posible pero... a mi me funciona 😉

&iexcl;Bien por PRISM!

 [1]: http://www.codeplex.com/CompositeWPF
 [2]: http://www.codeplex.com/AvalonDock
 [3]: http://11011.net/software/vspaste