---
title: '[DllImport] y clases genéricas'
author: eiximenis

date: 2009-02-23T09:52:00+00:00
geeks_url: /?p=1439
geeks_visits:
  - 855
geeks_ms_views:
  - 557
categories:
  - Uncategorized

---
Un post rápido para decir sólo dos cosas:

  * DllImport y clases genéricas no se llevan bien. Meter un DllImport en una clase genérica (o derivada de alguna genérica) lanza un TypeLoadException.
  * **Más importante que la anterior**: No nos habríamos topado con el error de haber seguido las recomendaciones de uso de DllImport. Y ni siquiera podemos alegar desconocimiento de ellas, [ya que si hubiesemos usado el análisis estático de código se nos habría avisado][1].

En resumen, [ya se ha dicho varias veces por aquí,][2] pero el análisis estático de código es tu amigo... 🙂

Saludos!

 [1]: http://msdn.microsoft.com/es-es/library/ms182161.aspx
 [2]: /blogs/rcorral/archive/2009/02/10/el-cuento-de-los-tres-desarrolladores.aspx