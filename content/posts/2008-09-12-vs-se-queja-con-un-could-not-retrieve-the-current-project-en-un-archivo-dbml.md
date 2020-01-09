---
title: VS se queja con un "Could not retrieve the current project" en un archivo .dbml
author: eiximenis

date: 2008-09-12T08:19:00+00:00
geeks_url: /?p=1420
geeks_visits:
  - 1388
geeks_ms_views:
  - 943
categories:
  - vs

---
Síntoma: tienes una solución que ayer (o hace algunos días, da igual) funcionaba y compilaba bien. Hoy lo abres y aparece un error que dice:

Build failed due to validation errors in `C:\edu\tmp\WofClientServer\WoFServer\WoFData.dbml`. Open the file and resolve the issues in the Error List, then try rebuilding the project.

Recompilar la solución no sirve para nada. Entonces si intentas abrir el archivo .dmbl, VS se queja con el `Could not retrieve the current project`.

La solución? Invocar VS desde una línea de comandos con:

**devenv /ResetSkipPkgs** y listos, todo volverá a funcionar!

La causa de esto es que en alguna carga previa de VS, el paquete (en este caso el diseñador de LINQ) no se carga bien por alguna razón y VS lo desactiva para el futuro. Aunque VS avisa de ello (aparece un warning en la ventana de output) es fácil no verlo, o incluso si es un paquete que usamos raramente, no acordarnos.

Evidentemente, en mi caso, el problema se dió con el diseñador de LINQ, pero se puede dar con cualquier paquete de VS que haya tenido algún error de carga.

Nos leemos 😉 

Más info sobre ResetSkipPkgs: <a href="http://msdn.microsoft.com/en-us/library/ms241276.aspx" title="ResetSkipPkgs command line switch" mce_href="http://msdn.microsoft.com/en-us/library/ms241276.aspx">http://msdn.microsoft.com/en-us/library/ms241276.aspx</a>