---
title: Gestionando las dependencias entre módulos cargados on-demand en PRISM
description: Gestionando las dependencias entre módulos cargados on-demand en PRISM
author: eiximenis

date: 2009-01-22T14:33:00+00:00
geeks_url: /?p=1433
geeks_visits:
  - 944
geeks_ms_views:
  - 870
categories:
  - Uncategorized

---
Una aplicación PRISM se compone de varios módulos que colaboran entre ellos. Un módulo PRISM simplemente es un objeto que implementa la interfaz IModule. En un mismo assembly pueden haber tantos módulos PRISM como se desee.

PRISM ofrece dos métodos para la carga de los módulos: O bien se cargan todos al principio de la aplicación, o bien se cargan on-demand (es decir, cuando se necesitan). La primera opción es la más simple, pero en algunos casos nos interesa ir cargando los módulos cuando se necesiten (bien porque hay muchos módulos posibles o bien porque la inicialización de estos módulos es un poco pesada).

Igualmente los módulos _pueden_ tener dependencias entre ellos: si el módulo A depende del módulo B, indica que el módulo B **debe** estar cargado cuando se cargue el módulo A. En caso contrario PRISM lanzará una excepción con el mensaje &ldquo;**A module declared a dependency on another module which is not declared to be loaded**&rdquo;.

Los módulos se cargan mediante dos clases: el IModuleEnumerator, que enumera los módulos existentes y el IModuleLoader que los carga e inicializa. Existen varias implementaciones de esta clases y nosotros nos podemos crear las nuestras. En función del IModuleEnumerator que usemos debemos usar un mecanismo u otro para indicar que el módulo no se carga por defecto, sino que se cargará on-demand. P.ej. si usamos el DirectoryLookupModuleEnumerator (que enumera todos los módulos de todos los assemblies de un directorio en particular), debemos decorar el módulo con el atributo Module con la propiedad StartupLoaded a false:

```cs
[Module(ModuleName = ModuleNames.MARKETPLACE_MODULE, 
StartupLoaded = false)]
public class MarketModule : IModule
{
}
```

Para cargar un módulo on-demand debemos obtenerlo mediante el `IModuleEnumerator` y cargarlo mediante el IModuleLoader (usando el método Initialize):

```cs
ModuleInfo mi = Container.Resolve<IModuleEnumerator>().
   GetModule(ModuleNames.MARKETPLACE_MODULE);
Container.Resolve<IModuleLoader>().Initialize(new ModuleInfo[] { mi });
```

(`Container` es la propiedad que me da acceso al contenedor IoC usado, que me permite obtener el IModuleEnumerator y el `IModuleLoader`).

De forma similar a como indicamos que un módulo se cargará on-demand podemos especificar que un módulo depende de otro. La forma exacta de hacerlo depende de nuevo del IModuleEnumerator usado. Si usamos el DirectoryLookupModuleEnumerator debemos decorar la clase módulo con el atributo ModuleDependency indicando de que módulo depende dicho módulo:

```cs
[Module(ModuleName = ModuleNames.MARKETPLACE_MODULE, 
StartupLoaded = false)]
[ModuleDependency(ModuleNames.CARDS_MODULE)]
public class MarketModule : IModule
{
}
```

El módulo MARKETPLACE\_MODULE depende del módulo CARDS\_MODULE: el segundo debe estar cargado antes de cargar el primero. Así pues es de esperar que el IModuleLoader cuando cargue el módulo MARKETPLACE\_MODULE cargue también el módulo CARDS\_MODULE...

... pues no. El IModuleLoader no cargará automáticamente el módulo CARDS_MODULE, en su lugar si no está cargado lanzará la excepción previamente comentada.

Vosotros debeis saber que dependencias tiene cada módulo y aseguraros que cada módulo está cargado. Es decir, en mi caso yo debo cargar CARDS\_MODULE antes que MARKETPLACE\_MODULE, o como muy tarde a la vez:

```cs
ModuleInfo mc = Container.Resolve<IModuleEnumerator>().
   GetModule(ModuleNames.CARDS_MODULE);
ModuleInfo mm = Container.Resolve<IModuleEnumerator>().
   GetModule(ModuleNames.MARKETPLACE_MODULE);            
Container.Resolve<IModuleLoader>().
   Initialize(new ModuleInfo[] { mc, mm });
```

Por suerte es posible un workaround para no tener que ir buscando todas las dependencias: hacerse un método de extensión sobre IModuleEnumerator, y que devuelva un array de ModuleInfo: el módulo junto con todas sus dependencias:

```cs
namespace Microsoft.Practices.Composite.Modularity
{
    public static class ModuleEnumeratorExtensions
    {
        public static ModuleInfo[] GetModuleWithDependencies(
           this IModuleEnumerator moduleEnumerator, string moduleName)
        {
            List<ModuleInfo> moduleInfoList = new List<ModuleInfo>();
            ModuleInfo module = moduleEnumerator.GetModule(moduleName);
            moduleInfoList.Add(module);
            if (module.DependsOn != null)
            {
                foreach (string dependencyName in module.DependsOn)
                {
                    if (!moduleInfoList.Exists(
                         existingModule => existingModule.ModuleName == 
                         dependencyName))
                    {
                        moduleInfoList.AddRange(
                         GetModuleWithDependencies(
                          moduleEnumerator, dependencyName));
                    }
                }
            }
            return moduleInfoList.ToArray();
        }
    }
}
```

Y para cargar un módulo junto con todas sus dependencias:

```cs
ModuleInfo[] mis = Container.Resolve<IModuleEnumerator>().
   GetModuleWithDependencies(ModuleNames.MARKETPLACE_MODULE);
Container.Resolve<IModuleLoader>().Initialize(mis);
```

Es una de esas cosas que uno se pregunta porque no lo habrán añadido de serie... 🙂

Nota: La información de este post está sacada de este [post de Mariano Converti][http://blogs.southworks.net/mconverti/2008/11/13/how-to-load-modules-on-demand-that-have-dependencies-using-composite-wpf-prism/]. Como es habitual, todo el mérito para él... [Su blog sobre PRISM](http://blogs.southworks.net/mconverti/category/composite-wpf/) es de imprescindible consulta!