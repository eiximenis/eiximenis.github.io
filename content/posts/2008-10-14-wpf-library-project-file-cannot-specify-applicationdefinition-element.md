---
title: '[WPF] Library project file cannot specify ApplicationDefinition element'

author: eiximenis

date: 2008-10-14T17:07:00+00:00
geeks_url: /?p=1423
geeks_visits:
  - 8506
geeks_ms_views:
  - 2151
categories:
  - wpf

---
Imagina la siguiente situación: Tienes un proyecto en WPF, con varias ventanas o controles WPF creados, y de repente te da por reorganizarlo todo un poco. Así, que añades un proyecto de tipo &#8220;Class Library&#8221; a la solución, y luego arrastras desde el Solution Explorer, algunas de las ventanas y/o controles al nuevo proyecto.

<!--more-->

Cuando más o menos lo tienes todo, le das a compilar y Visual Studio se queja con dos errores:

  * error MC1002: Library project file cannot specify ApplicationDefinition element.
  * error BG1003: The project file contains a property value that is not valid.

Además aunque le des doble-click en la ventana de errores, Visual Studio no está dispuesto a decirte en que línea o al menos que fichero es el causante de los dos errores.

El error se produce cuando al arrastrar los controles xaml al nuevo proyecto, Visual Studio cambia la &#8220;Build Action&#8221; de los controles que hayas arrastrado de &#8220;Page&#8221; a &#8220;ApplicationDefinition&#8221;, y una librería no puede tener ningún control o ventana xaml con &#8220;ApplicationDefinition&#8221;. Así pues, seleccionas en el &#8220;Solution Explorer&#8221; los ficheros xaml que hayas arrastrado (si arrastras más de un archivo te los cambia todos) y en propiedades, pones &#8220;Build Action&#8221; a &#8220;Page&#8221;... y listos!

Saludos! 

**PD:** El fichero que tiene la Build Action como &#8220;ApplicationDefinition&#8221; es aquel que proporciona el punto de entrada de la aplicación y por lo tanto solo es válido en ejecutables (suele ser el App.xaml).