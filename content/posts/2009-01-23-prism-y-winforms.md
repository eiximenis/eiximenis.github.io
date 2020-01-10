---
title: PRISM y Winforms
author: eiximenis

date: 2009-01-23T14:14:00+00:00
geeks_url: /?p=1434
geeks_visits:
  - 2810
geeks_ms_views:
  - 1337
categories:
  - Uncategorized

---
Los que sigais mi blog ya habreis visto que últimamente comento algunas cosillas sobre [PRISM](http://www.codeplex.com/CompositeWPF), la librería para crear aplicaciones compuestas en WPF.

<!--more-->

En este post, pero no quiero hablar de PRISM y WPF, sino sobre si es posible aprovechar PRISM para la creación de aplicaciones compuestas usando Winforms. Recordad que en Winforms ya tenemos una solución completa para la creación de aplicaciones compuestas: [CAB](http://msdn.microsoft.com/en-us/library/aa480450.aspx) y [SCSF](http://msdn.microsoft.com/en-us/library/aa480482.aspx). [Ezequiel](https://geeks.ms/blogs/ejadib/) publicó hace tiempo un [post lleno de enlaces sobre CAB y SCSF](https://geeks.ms/blogs/ejadib/archive/2007/04/14/composite-ui-application-block-por-donde-empiezo.aspx). Echadle una ojeada si os interesa el tema.

Aunque tengamos CAB y SCSF para crear nuestras aplicaciones compuestas en Winforms es lícito que nos planteemos si PRISM es una solución que se adapta a nuestras necesidades: Es más ligero que CAB y sus partes están menos acopladas. P.ej. si usamos CAB estamos atados forzosamente al ObjectBuilder, mientras que con PRISM el contenedor IoC que queramos usar lo podemos escojer. Además, los que hayan programado en CAB sabrán que tiene algunos comportamientos _extraños_, que aumentan mucho su curva de aprendizaje... si a todo esto le sumamos que PRISM es la _futura_ guia de aplicaciones composite, quizá nos interese ver si podemos empezar a aplicarla **ya** en nuestras aplicaciones Winforms.

Voy a intentar dar una respuesta en este post, pero antes que nada un _disclaimer:_ las aplicaciones winforms que creemos usando PRISM tendrán dependencias a assemblies de WPF, por lo que serán aplicaciones winforms sólo para el framework 3. Es posible usar PRISM sin ninguna dependencia a WPF, pero se pierde parte de su funcionalidad.

**1. El punto de partida**

El punto de partida, como muchas cosas en esta vida, la da Google. Buscando por Winforms y PRISM uno llega a un post Brian Noyes: [Composite Extensions for Windows Forms](http://www.softinsight.com/bnoyes/2008/10/13/CompositeExtensionsForWindowsForms.aspx). En este post Brian comenta el uso básico de PRISM para Winforms. Partiremos de su post y lo extendremos un poco.

Asumo que os habeis leído su post, y que os habeis descargado de su blog el proyecto. Si no, yo he puesto una copia **tal cual** (ver enlaces al final del post). Para compilar el proyecto de Brian, primero debeis compilar la solución CompositeApplicationLibrary.sln (dentro del directorio CAL), para generar PRISM. Para poder compilarlos necesitareis tener los assemblies de Unity (Microsoft.Practices.Unity.dll y Microsoft.Practices.ObjectBuilder2.dll). Unity os lo podeis descargar de la [página de Unity en Codeplex](http://www.codeplex.com/unity).

Una vez tengais los assemblies de PRISM generados podeis abrir la solución CompositeExtensions.sln y usais los assemblies de PRISM para colocar las referencias que falten. Compilais la solución y ya tendreis las extensiones de Brian para PRISM. 

**2. Explorando el punto de partida**

Las extensiones de Brian para PRISM, generan dos assembiles nuevos (más dos adicionales de tests unitarios):

  1. CompositeExtensions.Unity.dll: Contiene un bootstrapper nuevo (SimpleUnityBootstrapper) que nos permite crear aplicaciones winforms. 
  2. CompositeExtensions.dll: Contiene un port a winforms del sistema de eventos de PRISM. 

Ambas extensiones están totalmente libres de cualquier referencia a WPF. Para desarrollar una aplicación con las extensiones de Brian, simplemente es necesario crear un Bootstrapper derivado de SimpleUnityBootstrapper, registrar en Unity el control que queramos utilizar como &ldquo;contenedor&rdquo; de nuestras vistas, y en los módulos recuperar el&nbsp; control &ldquo;contenedor&rdquo; y añadir en él los controles hijos. El post de Brian lo explica paso por paso y hay una aplicación de demo (muy simple) que os recomiendo que la mireis bien. Porqué ahora vamos a empezar a extender el trabajo de Brian.

**3. Añadiendo soporte para regiones**

Las extensiones de Brian están bien, pero no hay soporte para el concepto de regiones... Supongo que es deliberado, ya que las interfaces y clases que debemos usar tienen dependencias contra WPF, pero supongamos que eso no nos importa y vayamos a ver como añadir soporte para regiones windows forms en PRISM.

**3.1. Nuestro Region Adapter**

La clave es tener nuestro propio RegionAdapter que sea capaz de interaccionar con un contenedor Winforms (en este caso un Control). Crear un RegionAdapter es muuuuy fácil: basta con derivar de la clase RegionAdapter<T> (siendo T el tipo de contenedor) y redefinir CreateRegion() y Adapt(). En el primer método debemos devolver el tipo de region PRISM que queremos. En el segundo debemos &ldquo;adaptar&rdquo; los contenidos de la región al contenedor usado.

Veamos el código y estará todo mucho más claro:

```cs
public class ControlRegionAdapter : RegionAdapterBase<Control>
{
    protected override IRegion CreateRegion()
    {
        return new AllActiveRegion();
    }
    protected override void Adapt(IRegion region, Control regionTarget)
    {
        region.ActiveViews.CollectionChanged += delegate
        {
            regionTarget.Controls.Clear();
            foreach (object co in region.ActiveViews)
            {
                if (co is Control)
                {
                    regionTarget.Controls.Add((Control)co);
                }
            }
        };
    }
}
```

En el método CreateRegion devolvemos una AllActiveRegion, que es una región de PRISM que entiende que todas sus vistas son activas.

En el método Adapt, hacemos que cada vez que cambie la colección de ActiveViews de la región, nos coloque las vistas activas dentro de la colección Controls del contenedor. En este punto (al usar CollectionChanged) es cuando nos aparece la referencia contra el assembly de WPF WindowsBase.dll.

**3.2 Indicar a PRISM que use nuestro nuevo Region Adapter**

Para indicar a PRISM que use un Region Adapter deteminado, debemos usar la clase RegionAdapterMappings, y añadir el mapping correspondiente. Un mapping le indica a PRISM que RegionAdapter usar para cada tipo de contenedor de región.

En este punto deberemos **modificar** las extensiones de Brian: él no usa el concepto de regiones, así que no crea ningún RegionAdpaterMappings inicial. Para hacerlo deberemos modificar la clase SimpleUnityBootstrapper. En el método ConfigureContainer, dentro del if (_useDefaultConfiguration) añadimos la línea:

```cs
RegisterTypeIfMissing(typeof(RegionAdapterMappings),
   typeof(RegionAdapterMappings), true);
```

Con esto hacemos que al crear el contenedor Unity, se cree un singleton de tipo RegionAdapterMappings. La clase RegionAdapterMappings está dentro del assembly Microsoft.Practices.Composite.Wpf.dll de PRISM, por lo que debereis añadir la referencia.

El siguiente paso es añadir un método protected virtual en la misma clase SimpleUnityBootstrapper:

```cs
protected virtual RegionAdapterMappings ConfigureMappings() 
{
    return Container.Resolve<RegionAdapterMappings>();
}
```

Y finalmente lo llamamos desde el método Run del propio SimpleUnityBootstrapper. Justo después de la llamada a ConfigureContainer(), llamamos a ConfigureMappings(). Con ello hemos modificado el bootstrapper inicial de Brian para que cree un RegionAdapterMappings (vacío) y nos de un punto de extensión (ConfigureMappings) donde nosotros podamos añadir nuestros propios mappings.

Ahora en nuestra clase bootstrapper podemos hacer un override del método ConfigureMappings y añadir nuestro mapping para que use la clase ControlRegionAdapter que hemos definido antes:

```cs
protected override RegionAdapterMappings ConfigureMappings()
{
    RegionAdapterMappings mappings = base.ConfigureMappings();
    mappings.RegisterMapping(typeof(Control), new ControlRegionAdapter());
    return mappings;
}
```

Con esto indicamos a PRISM que use nuestro Region Adapter cuando añadamos elementos a una región cuyo contenedor sea un Control.

**3.3 Crear el RegionManager**

Para poder crear Regiones, necesitamos crear un RegionManager. Para ello, lo más fácil és modificar, de nuevo, el SimpleUnityBootstrapper de Brian para que nos cree un RegionManager. Otra vez dentro del método ConfigureContainer, dentro del mismo if (_useDefaultConfiguration) añadimos la línea para que nos registre el RegionManager:

```cs
RegisterTypeIfMissing(typeof(IRegionManager), 
   typeof(RegionManager), true);
```

Ahora ya tenemos un RegionManager de PRISM listo para usar... Ya sólo nos queda crear una región.

**3.4 Crear una región**

En WPF se pueden definir las regiones usando XAML, pero en winforms no tenemos nada parecido, así que vamos a hacerlo programáticamente. Por suerte el RegionManager tiene un método AttachNewRegion que nos va a servir para ello.

Primero, modificaremos de nuevo el SimpleUnityBootstrapper de Brian para añadir un método virtual:

```cs
protected virtual void AttachInitialRegions() { }
```

Y lo llamamos desde el método Run, justo _después_ de la llamada a ConfigureMappings que añadimos antes.

Ahora, podemos volver a nuestro bootstrapper y hacer el override para crear la región inicial:

```cs
protected override void AttachInitialRegions()
{
    base.AttachInitialRegions();
    this.Container.Resolve<IRegionManager>().
        AttachNewRegion(this.Shell.MainRegionContainer,"Main");
}
``` 

Asumid que this.Shell es una referencia al formulario principal de la aplicación y que la propiedad MainRegionContainer me devuelve un Control que es el contenedor de la región.

**4. Un módulo... para hacer algo**

Las aplicaciones PRISM se componen de módulos (a la práctica objectos que implementan IModule) que colaboran entre ellos. Cada módulo se encarga de implementar parte de la aplicación, generalmente añadiendo vistas a las regiones existentes o bien añadiendo regiones nuevas.

Vamos a crear un módulo realmente simple, que añada una vista a nuestra región.

Para ello, añadimos una clase nueva que implemente IModule. El código puede ser como el siguiente:

```cs
public class Module1 : IModule
{
    private IUnityContainer container;
    private IRegionManager regionManager;

    public Module1(IUnityContainer ctl, IRegionManager rm)
    {
        this.container = ctl;
        this.regionManager = rm;
    }

    public void Initialize()
    {
        this.RegisterViewsAndServices();
        this.regionManager.Regions["Main"].
            Add(container.Resolve<IView1>());
    }

    private void RegisterViewsAndServices()
    {
        this.container.RegisterType<IView1, View1>();
    }
}
```

En el constructor recibimos el contenedor Unity y el RegionManager a usar. Los módulos los crea PRISM y ambos parámetros del constructor son suministrados via Dependency Injection por Unity.

El método Initialize() es el único método que declara IModule, y es donde tenemos que hacer todo el trabajo. En este caso llamamos a un método propio (RegisterViewsAndServices) que indica que cuando alguien pida un objeto IView1, devuelva un View1. Finalmente, accedemos al RegionManager y añadimos una instancia nueva de IView1.

Que son IView1&nbsp; y View1? IView1 es una interfaz, los métodos de la cual no tienen importancia (de hecho, en mi ejemplo está vacía). View1 es la vista a añadir: un UserControl cuyo contenido puede ser el que se desee.

Finalmente sólo queda hacer que nuestro bootstrapper cargue el módulo. Para ello usamos el StaticModuleEnumerator para cargar e inicializar el módulo:

```cs
protected override IModuleEnumerator GetModuleEnumerator()
{
    return new StaticModuleEnumerator().
        AddModule(typeof(Module1));
}
```  

Y listos! Con ello nuestra aplicación PRISM funcionando en Windows Forms está lista!

Saludos!
