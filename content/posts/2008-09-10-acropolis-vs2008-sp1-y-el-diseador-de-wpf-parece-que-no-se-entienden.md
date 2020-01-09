---
title: Acropolis, VS2008 SP1 y el diseñador de WPF parece que no se entienden
author: eiximenis

date: 2008-09-10T12:26:00+00:00
geeks_url: /?p=1419
geeks_visits:
  - 1266
geeks_ms_views:
  - 959
categories:
  - wpf

---

MMmm... pues eso 🙂

Los síntomas eran los siguientes: En un proyecto WPF, al cargar un archivo xaml, el diseñador se quejaba con el mensaje: &#8220;Index was out of range: Must be non-negative&nbsp; and less than the size of the collection&#8221;. Luego daba un número de línea y posición que no decían nada en absoluto.

El proyecto compilaba y se ejecutaba correctamente, simplemente el diseñador se negaba a mostrar la clase. He de decir que yo había cargado antes este proyecto, sin ningún problema! 

Tras intentar entender que podía estar pasando, deducí que el error estaba en el ItemTemplate de un ListBox que había en el XAML. Bueno... deducí esto básicamente porque VS2008 me lo subrayaba todo en azul claro 😛

En concreto había varias lineas que parecian no gustarle a VS2008, p.ej:

``` xml
<Border Style=”{StaticResource RacePitBorderStyle}”>
```

Siendo `RacePitBorderStyle` un estilo definido usando `<Style>` en el propio XAML (quito el código dentro de `<Style>` para hacerlo más claro). 

``` xml
<Control.Resources>
    <Style x:Key=”RacePitBorderStyle” TargetType=”Border”>
    </Style>
    <wofconverters:RetratSourceConverter x:Key=”RetratSourceConverter”/>
</Control.Resources>
```

Lo primero que observé, fue que si modificaba el `StaticResource` del Border para usar DynamicResource, este error desaparecía (ni idea de por que), pero luego me daba otro en la línea:
    
``` xml
<Image Source=”{Binding Path=Retrat,Converter={StaticResource RetratSourceConverter}}” Width=”32″ Height=”32″></Image>
```    
    
Ahora se quejaba de un error de casting (sorry, no tengo el mensaje exacto).
    
Ya con la mosca tras la oreja, me puse a googlear y por suerte encontré a <a href="http://forums.microsoft.com/Forums/ShowPost.aspx?PostID=3072901&SiteID=1" mce_href="http://forums.microsoft.com/Forums/ShowPost.aspx?PostID=3072901&SiteID=1">alguien a quien le pasaba lo mismo</a>. El Comentaba que el error se lo ocasionaba el tener instalado Acropolis... tal y como yo lo tenía!

Así pues desinstalé Acropolis, y ole! El diseñador ya cargaba de nuevo mi proyecto (incluso usando el StaticResource, tal y como estaba antes).
En fin... 😉

Saludos!
    
PD: Y SP1 que pinta aquí? Seguramente nada, aunque juraría que antes, con VS2008 sin SP1 y con Acropolis el proyecto me funcionaba... pero hacía bastantes días que no cargaba este proyecto y no lo puedo asegurar. En todo caso, con SP1 sin Acropolis funciona bien 🙂
