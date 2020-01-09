---
title: Azure Functions – Serverless Backends
author: eiximenis

date: 2016-09-27T06:23:44+00:00
geeks_url: /?p=1815
geeks_ms_views:
  - 1661
categories:
  - azure

---
**Últimamente se oye bastante hablar de _serverless backend_.** Esta expresión puede sugerir implementaciones de servidor sin… servidor. Pero no, la realidad es que dicha expresión define una arquitectura en la cual no nos preocupamos para nada del servidor. Existir, existe, pero es totalmente transparente (o casi) para nosotros.

Es una **astracción superior a la que ofrece un sistema PaaS**. Tomemos un ejemplo de PaaS en Azure, como las _web apps_. Efectivamente, con una _web app_ tenemos un nivel de abstracción relativamente alto: nos olvidamos de instalar un servidor en una máquina virtual y nos olvidamos de muchas tareas de administración. Con muy poco esfuerzo por nuestra parte podemos tener nuestra aplicación o API HTTP desplegada en una aplicación web con capacidad de escalar según sea necesario. Pero, a pesar de esas facilidades, seguimos viendo claramente un _servidor_: desplegamos en la _webapp_, configuramos la _webapp_, escalamos la _webapp_. El servidor sigue estando presente, solo que es mucho más sencillo de configurar que el sistema equivalente en IaaS u on-premise. 

En un sistema tipo _serverless backend_, la única tarea realmente indispensable que hay que hacer es subir código y este se ejecuta de forma casi inmediata. No hay apenas configuración. No vemos cual es el servidor subyacente que ejecuta el código. El concepto de servidor “desaparece”: es nuestro código y se ejecuta. No hay que preocuparse de nada más. PaaS vino para simplificarnos la vida respecto IaaS y el _serverless backend_ viene para simplificarla todavía más. 

En Azure este paradigma se encarna mediante las _Azure Functions_ que son exactamente lo que su nombre indica: **funciones de código que se suben a Azure y se empiezan a ejecutar.** 

#### Creando una función de Azure

La creación **no podría ser más sencilla, podemos codificar nuestra función directamente desde el portal de Azure**. 

Lo primero que debemos hacer es agregar una _Function App_. No es más que un contenedor donde vamos a tener todas nuestras funciones. Para ello desde el portal de Azure pulsamos agregar un nuevo elemento, tecleamos “_functions_” en el buscador y nos aparecerá la opción: 

[<img title="clip_image002" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="clip_image002" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/09/clip_image002_thumb.jpg" width="644" height="178" />][1] 

Pulsamos el botón de _Create_, rellenamos los datos (básicamente el nombre de nuestra _function app_) y ya podemos empezar. 

Con esto no hemos creado ninguna _azure function_, solo hemos creado su contenedor. Una vez esté creado ya lo veremos dentro del resource group que hayamos elegido: 

[<img title="clip_image003" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="clip_image003" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/09/clip_image003_thumb.png" width="644" height="203" />][2] 

Si abrimos la _function app_ (el icono con el rayo) se nos abre la interfaz para añadir funciones directamente desde el portal de Azure: 

[<img title="clip_image005" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="clip_image005" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/09/clip_image005_thumb.jpg" width="644" height="344" />][3] 

Lo primero que se nos pide es **que tipo de función queremos añadir**. Básicamente tenemos tres tipos posibles: 

1. _Timer_: Funciones que se ejecutan basadas en un temporizador. Se pueden ejecutar cada 5 minutos, cada día a las 15 en punto o los domingos las 8 de la mañana por poner unos ejemplos. 

2. _Data Processing_: Funciones que se ejecutan en base a _triggers_ lanzados por otros elementos de Azure. Hay una gran variedad de _triggers_ disponibles tales como que se cree un fichero en un storage, que llegue un evento via service bus o un elemento nuevo en una cola de Azure, entre otros. 

3. _Webhook + API_: Funciones que se integran con APIs externas que soporten webhooks. P. ej. Github los soporta, así que sería posible hacer una función que se ejecutara cada vez que alguien comentara una _issue_ en GitHub. 

En función del escenario elegido (y del lenguaje C# o JavaScript), veremos el código inicial de nuestra función. P. ej. la siguiente captura muestra el código para una función de tipo “_Data Processing_”: 

[<img title="clip_image007" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="clip_image007" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/09/clip_image007_thumb.jpg" width="644" height="230" />][4] 

Podemos usar el propio portal **para configurar los _triggers_** que dispararán dicha función. Para ello, pulsamos sobre la opción _Integrate_ de la barra de la izquierda, y de forma **totalmente gráfica** podremos configurar los triggers de nuestra función: 

[<img title="clip_image009" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="clip_image009" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/09/clip_image009_thumb.jpg" width="644" height="198" />][5] 

Además del _trigger_ podemos especificar **entradas opcionales** para nuestra función (p. ej. un documento en una cola de Azure, un documento de DocumentDb, …) y también **salidas opcionales** (fichero a escribir, mensaje a mandar,). Estas entradas y salidas se reciben como parámetros en nuestra función. 

Como se ha comentado el código se escribe en el portal: 

[<img title="clip_image011" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="clip_image011" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/09/clip_image011_thumb.jpg" width="644" height="247" />][6] 

Desde el propio portal podemos guardar la función, lo que automáticamente la compilará. La función se guarda siempre en un fichero run.csx (para el caso de C#) o index.js (para el caso de JavaScript (NodeJS)). 

Por supuesto **es posible usar cualquier editor para crear no solo el fichero run.csx (o index.js)** y subir luego el contenido a Azure. Al subirlo, automáticamente la función se compilará y estará lista para ejecutarse. Para subir el contenido se puede usar Git o incluso FTP. 

**La configuración** (los _triggers_ que disparan la función, o cada cuando se ejecuta, sus entradas y sus salidas) **se guarda en un fichero json**. Podemos ver su contenido si pulsamos el botón “Advanced Editor” dentro de la sección “Integrate”. La siguiente captura muestra el editor gráfico: 

[<img title
="clip_image013" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="clip_image013" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/09/clip_image013_thumb.jpg" width="644" height="212" />][7] 

Si pulsamos sobre el botón “Advanced editor” vemos el contenido del fichero json: 

[<img title="clip_image015" style="border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; border-left: 0px; display: inline; padding-right: 0px" border="0" alt="clip_image015" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/09/clip_image015_thumb.jpg" width="644" height="175" />][8] 

**Desplegar una Azure function es desplegar el fichero run.csx (o index.js) y su fichero json de configuración.** No es necesario hacer nada más. No hay que compilar nada, en caso de ser necesario la compilación ocurre en Azure. 

#### Webjobs y Azure functions

Azure functions es una evolución de los actuales _Webjobs_. Muchas de las tareas que se automatizaban en Webjobs pueden automatizarse en Azure functions. La diferencia fundamental es, básicamente, de concepto. Los Webjobs son un escenario PaaS, donde el servidor (una _Webapp_) es claramente explícito. El webjob se despliegua de forma tradiciona y la webapp que lo contiene se configura como cualquier otra. 

Una Azure function **se ejecuta a través de una webapp**. Se parece mucho a los webjobs, pero esto para nosotros es totalmente transparente. A pesar de que es posible hacerlo, no necesitamos por norma general acceder a la _webapp_ subyacente. En general, las Azure functions evolucionan los webjobs y los llevan a un nuevo nivel de simplicidad. 

#### Resumiendo

Azure functions es la implementación del paradigma _serverless backend_, en el cual el servidor queda difuminado hasta casi poder ser ignorado. Una Azure function es una porción de código (realmente una función) que se ejecuta como causa de un disparador (puede ser un temporizador, una acción sobre un elemento de Azure (tal como un mensaje en un service bus) o la llegada de un webhook para integrarnos con cualquier API que los soporte). La función puede realizar cualquier operación que sea necesaria (p. ej. comprimir imágenes subidas a Azure, periódicamente limpiar una tabla en una base de datos SQL Azure o cualquier otra tarea). La ejecución de una Azure function puede realizar una tarea que actúe como disparador de otra Azure function, dando lugar a escenarios complejos.

 [1]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/09/clip_image002.jpg
 [2]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/09/clip_image003.png
 [3]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/09/clip_image005.jpg
 [4]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/09/clip_image007.jpg
 [5]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/09/clip_image009.jpg
 [6]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/09/clip_image011.jpg
 [7]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/09/clip_image013.jpg
 [8]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/09/clip_image015.jpg