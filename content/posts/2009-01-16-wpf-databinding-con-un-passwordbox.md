---
title: '[WPF] Databinding con un PasswordBox'

author: eiximenis

date: 2009-01-16T13:32:00+00:00
geeks_url: /?p=1431
geeks_visits:
  - 1254
geeks_ms_views:
  - 1012
categories:
  - Uncategorized

---
Hola! ¿Que tal os sienta el 2009? Espero que lo mejor posible 🙂

Hoy un post cortito para comentar un problemilla y su solución.

El problemilla es que al intentar realizar DataBinding desde un PasswordBox no funciona, porque la propiedad Password, **no** es una DependencyProperty.

<!--more-->

Es decir, mientras que esto funciona y liga la propiedad Text a la propiedad Login del DataContext:

```xml
<TextBox Grid.Column="1" x:Name="txtLogin"  VerticalAlignment="Center"
         Text="{Binding Login}" />
```

esto no funciona:

```xml
<PasswordBox Grid.Row="1" Grid.Column="1" x:Name="txtPassword" VerticalAlignment="Center" 
             Password="{Binding Password}" />
```

Ya que la propiedad Password no es una DependencyProperty.

¿La solución? Añadir una propiedad enlazada que sí que sea una DependencyProperty y que tenga el mismo valor que la propiedad Password.

La solución completa la podeis encontrar en este post de Functional Fun: [WPF PasswordBox and Data binding](http://blog.functionalfun.net/2008/06/wpf-passwordbox-and-data-binding.html). **Todo el mérito es suyo**, yo sólo comparto el post, puesto que me ha parecido muy interesante.

¡Saludos!
