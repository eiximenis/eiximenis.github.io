---
title: Docker para el desarrollador asp.net (i)
author: eiximenis

date: 2016-11-21T10:52:01+00:00
geeks_url: /?p=1826
geeks_ms_views:
  - 2441
categories:
  - asp.net 5
  - asp.net vNext
  - docker

---
Buenas! Vamos a **empezar una serie de posts** dedicadas a _Docker_ desde el punto de vista de un desarrollador asp.net. **Empezaremos por lo más básico pero nos iremos adentrando un poco en el mundo de Docker**. El objetivo es que terminemos teniendo unos conocimientos _medios_ que nos permitan entender que es Docker, como funciona, qué ventajas tiene y como usarlo (y cuando) en arquitecturas más complejas donde haya más de un contenedor. Pero… **empecemos por el principio**.
  
<!--more-->

**¿Qué es Docker?**
  
Docker es un _sistema de contenedores_. La analogía clásica de los contenedores es que son “como” máquinas virtuales pero mucho más ligeras (de hecho una máquina virtual puede (y suele) ejecutar varios contenedores a la vez). Al igual que una máquina virtual un contenedor tiene su propio entorno, sus propias librerías y su propio sistema de ficheros, y está relativamente aislado del resto de contenedores.
  
Pero a diferencia de una máquina virtual, un contenedor **ejecuta un solo proceso**. Cuando el proceso que ejecuta el contenedor termina, el contenedor “se para”.
  
Otra diferencia fundamental es que una máquina virtual puede (y a menudo hace) ejecutar un sistema operativo distinto del que ejecuta el _host_. Eso **no es así** en los contenedores, o al menos **no** es **exactamente así:**  un contenedor puede ejecutar solo procesos de la misma plataforma que su _host_. Así si el _host_ es un Linux ARM, el contenedor puede ejecutar solamente procesos ejecutables en un Linux ARM. Si el _host_ es un Linux x64 el contenedor puede ejecutar solamente procesos ejecutables en un Linux x64. Y así, sucesivamente. A diferencia de una máquina virtual, que está completamente aislada del resto del mundo (a pesar de que puedan existir otras máquinas virtuales en un mismo _host_) todos los contenedores de un _host_ comparten el kernel del SO nativo. Realmente no se instala un SO en un contenedor, el SO como tal viene dado por el _host_.
  
De todos modos, cuando empieces con Docker verás que existen lo que se llaman imágenes base. Así p. ej. podemos instalar la imágen base de _fedora_ o de _ubuntu_ en un contenedor. Eso puede parecer una contradicción con lo dicho anteriormente pero no lo es: tanto _fedora_ como _ubuntu_ son distribuciones de Linux que se diferencian en las librerías que llevan instaladas, pero el kernel del sistema operativo es el kernel de Linux, el mismo kernel en ambas distribuciones (y en el resto de distribuciones Linux). Es por ello que podemos tener una máquina con ubuntu ejecutando un contenedor fedora: porque en ambos casos se trata de Linux. No vas a poder tener una máquina Linux ejecutando un contenedor Windows. Ni tampoco una máquina Windows ejecutando un contenedor Linux.
  
Y ahora la “mala”noticia: Docker existe (a día de hoy y de forma estable) solo para Linux.
  
**Docker for Windows**
  
Bueno… ¡no todo está perdido! Es posible ejecutar contenedores Docker en un _host_ Windows siempre y cuando este _host_ Windows tenga algún mecanismo para ejecutar un SO Linux. Y este mecanismo es, obviamente, una máquina virtual. Y eso es, precisamente, lo que utiliza “Docker for Windows”. Debemos tener presente que cuando hablamos de Docker hablamos conjuntamente de dos elementos distintos:

  1. **Las herramientas de cliente**: esas herramientas CLI (línea de comandos) nos permiten administrar nuestros contenedores. Existen **versiones nativas de las herramientas CLI** de Docker para windows.
  2. **El sistema de ejecución de contenedores**: ese sistema es el “corazón” de Docker y es el que ejecuta los distintos contenedores en nuestra máquina. Esta es la parte que solo existe para Linux.

Cuando instalamos Docker for Windows, se nos instalan las herramientas de cliente y luego, automáticamente, una máquina virtual HyperV que ejecuta Linux. Esa máquina es la que ejecutará todos los contenedores Docker que tengamos. También las herramientas de línea de comandos se configuran para comunicar de forma transparente para nosotros con dicha máquina virtual.
  
**Contenedores Windows**
  
Ya hemos visto que no es posible en Docker tener contenedores Windows. ¿Qué significa eso? Recuerda que los contenedores usan realmente el kernel de la máquina _host_. En el caso de Docker for Windows la  máquina _host_ es la MV HyperV que ejecuta Linux, por lo que **solo podemos tener contenedores Linux**. Eso para el **desarrollador de asp.net tiene una implicación muy grande:** Básicamente significa que te olvides de Docker… a no ser que o bien te metas en berenjenales con Mono o bien uses asp.net core que sí que puede correr en Linux y por lo tanto en un contenedor Docker.
  
Cabe mencionar que Windows Server 2016 incluye soporte nativo para Docker (a través de un acuerdo entre Docker y Microsoft). En este caso sí que hablamos de contenedores Windows, pero está todavía en _preview_ y además requiere Server 2016, así que nos olvidaremos de este sistema.
  
**Contenedores sin estado?**
  
Leerás por ahí que “los contenedores Docker no tienen estado”. Eso es cierto, aunque hay que entender a que se refiere exactamente. Para ello debemos introducir un **concepto nuevo: la imagen**.
  
La relación entre contenedores e imágenes vendría a ser, para entendernos, la misma que entre la de objetos y clases: para ejecutar un contenedor antes debemos tener una imagen que describa ese contenedor. Podemos tener varios contenedores ejecutándose a partir de una misma imagen. La imagen describe todo lo que el contenedor contiene, desde el sistema de ficheros hasta las variables de entorno y el proceso a ejecutar.
  
Bien, si paramos y reiniciamos un mismo contenedor, **los cambios realizados en este (ficheros creados, etc) son persistentes**. Ahora bien, si borramos el contenedor y ejecutamos otro, a partir de la misma imágen, entonces este otro contenedor no tiene ninguno de los cambios realizados en el contenedor anterior. A eso es a lo que nos referimos cuando decimos que los contenedores son _stateless_. No los contenedores en sí, si no las imágenes.
  
En general es interesante que los contenedores **sean realmente _stateless_**_._ Eso nos permite eliminarlos y recrearlos (a partir de la imágen) y no tener que mantenerlos . Es cierto que un contenedor parado no ocupa recursos ni de RAM ni de procesador del _host_ pero sí que ocupa espacio en disco. Si los contenedores son realmente _stateless_ los podemos borrar completamente y siempre podemos reiniciarlos a partir de la imagen en cualquier momento. Eso significa trasladar el estado a algún elemento externo (como una BBDD o un sistema de ficheros externo). Vamos, como ya puedes intuír eso encaja muy bien con la web, que también es _stateless_ por naturaleza.
  
**¿Cuanta memoria tiene mi contenedor? ¿Cuantos cores?**
  
Si te estás haciendo esas preguntas es que estás pensando más en máquinas virtuales que en contenedores. A un contenedor de Docker no se le asigna un tamaño de RAM  ni un determinado número de cores: usa la misma RAM y los mismos cores que tenga el SO _host_ (en el caso de Docker for Windows, los cores y RAM asignadas a la MV HyperV).
  
Así en ningún sitio reservarás X MB de RAM para un determinado contenedor y Y MB para otro. Simplemente los contenedores usarán toda la RAM que necesiten del _host_, del mismo modo que lo hace un proceso convencional. Del mismo modo tampoco le asignarás un espacio en disco, ni un tamaño de VHD.
  
**¿Es posible ejecutar el Docker nativo de Linux en W10 a través de bash for Windows?**
  
Pues… a día de hoy que yo sepa **no.** A pesar de que el subsistema de Linux que ofrece Windows 10 tiene muchas posibilidades, no es un Linux completo. En mi versión actual de Windows 10 (14393.351) he sido capaz de instalar Docker para Linux: las CLI se instalan correctamente, pero no funciona ningún comando relativo a la gestión de contenedores.
  
Parece que en la versión actual el subsistema Linux de W10 (WSL) viene con el kernel 3.4 que **no es compatible** con Docker (requiere 3.8 como mínimo):
  
[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/11/image_thumb.png" alt="image" width="644" height="260" border="0" />][1]
  
En la captura se puede ver como el Ubuntu 14.04 que viene con bash for windows ejecuta el kernel 3.4.0+ que no es compatible con Docker. El que se use esta versión de kernel supongo que es porque futuras versiones del kernel requieren características que no están soportadas por WSL (porque Ububtu 14.04 debería llevar el kernel 3.13, [según la documentación oficial][2]). No sé si es posible modificar el kernel y actualizarlo pero dudo que incluso si se pudiese, que Docker funcionara: WSL ofrece, por el momento, soporte para _userspace_ y no para _kernelspace_ que supongo debe requerir Docker.
  
Ahora bien, como hemos visto, a través de bash for Windows, el cliente CLI de Docker si que se instala. Así, **si se está ejecutando Docker for Windows es posible usar la CLI de Linux para manejarlo**. No deja de ser curioso que a través de un subsistema Linux en Windows 10 nos comuniquemos con una MV HyperV de Linux ejecutándose en el propio Windows 10:
  
[<img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" src="http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/11/image_thumb-1.png" alt="image" width="644" height="283" border="0" />][3]
  
En la captura se puede ver como ejecutamos un contenedor (el _hello world_) desde el CLI de Linux, conectándonos a _localhost_ (que es Docker for Windows). Curioso, pero bueno, ahí queda 😉
  
Bueno, dejemoslo aquí por hoy. En el siguiente post veremos como podemos publicar un _hello world_ de aspnet core en Docker.

 [1]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/11/image.png
 [2]: https://wiki.ubuntu.com/Kernel/Support
 [3]: http://geeks.ms/etomas/wp-content/uploads/sites/154/2016/11/image-1.png