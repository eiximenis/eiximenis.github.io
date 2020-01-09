---
title: 'RA000 (o MSB3217): Cannot Register assembly Foo.dll (Method does not have an implementation).'
author: eiximenis

date: 2010-01-20T12:25:47+00:00
geeks_url: /?p=1489
geeks_visits:
  - 1454
geeks_ms_views:
  - 1137
categories:
  - Uncategorized

---
Hola! Un post cortito, sobre un error que me he encotrado… Al compilar un proyecto, marcado para interoperabilidad COM VS.NET se me ha quejado con el siguiente error:

<font face="Courier New">c:WINDOWSMicrosoft.NETFrameworkv3.5Microsoft.Common.targets(3019,9): error MSB3217: <u>Cannot register assembly</u> "C:TeamserverPhoenixRefactoringCoreDevelopmentCore-WI5825-SIO4binDebugPhoenixContainer.dll". Method &#8216;GetDefaultIWorkspace&#8217; in type &#8216;CaixaPenedes.Phoenix.Core.CompositeUI.ShellUserControl&#8217; from assembly &#8216;Core, Version=1.0.0.0, Culture=neutral, PublicKeyToken=77c76132715b70fa&#8217; <u>does not have an implementation</u>.</font>

Yendo al directorio bin/Debug y ejecutar regasm PhoenixContainer.dll daba el mismo error.

La situación era la siguiente:

  1. Tengo dos **soluciones** distintas, una de las cuales compila (entre otros) el ensamblado Core.dll, donde está la clase ShellUserControl que supuestamente tiene el método no implementado. Por supuesto el método está implementado. La otra solución compila PhoenixContainer.dll, el ensamblado que me da el error. 
  2. PhoenixContainer.dll tiene una referencia (entre otros a Core.dll). 
  3. Ambas soluciones compilan **contra el mismo** directorio de salida.

Buscando por ahí, he visto que más gente tenía el mismo error… <a href="http://adamserrata.blogspot.com/2008/12/regasm-and-gac-fun-msb3217-and-ra0000.html" target="_blank" rel="noopener noreferrer">en esta página dan con la que parece ser la causa principal</a>: que al ejecutar regasm, los ensamblados que regasm usen **no** sean los mismos contra los que se ha compilado el proyecto. Generalmente eso puede ser debido a versiones incorrectas en la GAC.

Este no era mi caso: Yo tenia Core.dll y PhoenixContainer.dll en el mismo directorio y no uso la GAC, así que es imposible que me estuviese pillando alguna otra Core.dll.

Finalmente he decidido poner en marcha <a href="http://msdn.microsoft.com/en-us/library/e74a18c4.aspx" target="_blank" rel="noopener noreferrer">fuslogvw</a> y que me mostrase todos los "bind failures”, es decir todas las veces que el CLR intenta cargar un ensamblado, y por la razón que sea no lo encuentra. Y touché: Ha aparecido un bind failure: Regasm.exe intentaba cargar Microsoft.Practices.Composite.UI.dll (este ensamblado forma parte de <a href="http://msdn.microsoft.com/en-us/library/aa480450.aspx" target="_blank" rel="noopener noreferrer">CAB</a>). El ensamblado Core.dll tiene una referencia contra Microsoft.Practices.Composite.UI.dll, pero **con copy local a false**, puesto que usamos un directorio “compartido” donde hay varios ensamblados _externos_ a nuestro proyecto. Sospecho que la razón por la cual Regasm.exe intenta cargar Microsoft.Practices.Composite.UI.dll (y no otros ensamblados también referenciados por Core.dll) es porque el método GetDefaultIWorkspace es público y devuelve un IWorkspace, tipo definido en este ensamblado. Lo curioso, es que este método **nunca** es utilizado desde PhoenixContainer.dll (aunque el tipo ShellUserControl sí).

Así en el bin/debug tenía Core.dll pero no Microsoft.Practices.Composite.UI.dll y esa era la causa del error: modificando la referencia con copy local a true, todo ha funcionado correctamente!

Cada vez tengo más claro que el copy local a false, sólo trae que compilaciones, a excepción que los archivos referenciados estén en la GAC…

Saludos!