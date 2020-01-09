---
title: 'Una dudilla sobre C#'
description: 'Una dudilla sobre C#'
author: eiximenis

date: 2008-12-31T11:40:28+00:00
geeks_url: /?p=1430
geeks_visits:
  - 1198
geeks_ms_views:
  - 1227
categories:
  - Uncategorized

---
Hola… a punto todos para comernos las uvas????

Antes de que lo hagáis y os lanceis luego a brindar con cava por el nuevo año, y una cosa lleve a la otra y no esteis en condiciones, digamos de… pensar mucho, a ver si alguien me sabe responder una dudilla que me ha surgido hoy.

¿Porque este código no compila?

```cs
public class Foo
{
    public string Name { get { return string.Empty; } }
    public string Name() { return string.Empty ; }
}
```

Por si alguien (como yo) se pensaba que eso compilaba, pues no. Visual Studio se queja con un claro y explícito _error CS0102: The type &#8216;ConsoleApplication232.Foo&#8217; already contains a definition for &#8216;Name&#8217;_.

Alguien sabe el _porque_ de esta limitación? Es decir, porque han evitado que podamos hacer esto? Alguien tiene alguna idea?

Epa!!! Buen año a tod@s y que el 2009 os sea lo más propicio posible!!!! Y no os comáis las uvas antes de tiempo! Recordad que [el último minuto de este 2008 tiene un segundillo de más](https://es.wikipedia.org/wiki/Segundo_intercalar)!!! xD