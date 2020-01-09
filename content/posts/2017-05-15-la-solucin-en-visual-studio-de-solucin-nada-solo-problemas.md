---
title: 'La “solución” en Visual Studio: de solución nada, solo problemas'
author: eiximenis

date: 2017-05-15T07:22:22+00:00
geeks_url: /?p=1876
geeks_ms_views:
  - 3570
categories:
  - opinion

---
Desde sus inicios Visual Studio contiene el concepto de solución como grupo de proyectos. Es más, no puedes abrir un solo proyecto siempre debes abrir una solución (si abres un proyecto VS crea una solución que contiene el proyecto de forma automática).
  
Como idea, hace años no estaba mal. Y que quede claro: en según que contextos puede seguir siendo válida. Pero en muchos otros, solo aporta problemas: el desarrollo de software ha cambiado mucho, las formas en como usamos los IDEs no son iguales, pero VS sigue anclado a este obsoleto sistema de gestionar proyectos.
  
<!--more-->


  
**El rol de la solución**
  
La solución juega el rol central de “agrupador de proyectos”. Es habitual tener varios proyectos dentro de una misma solución. El sistema funciona muy bien cuando la agrupación sigue el esquema de “una aplicación + proyectos adicionales”. P. ej. un proyecto (con una aplicación web MVC o una WPF, o una Xamarin, o…) y varias librerías de clases.
  
También por extensión **podemos tener varias aplicaciones que comparten librerías de clases en una misma solución**. P. ej. unas librerías compartidas entre una aplicación Xamarin y una MVC: entonces solemos metemos en una solución la aplicación MVC, la Xamarin y las librerías de clases. **No es ideal** porque p. ej. un desarrollador web no tiene porque tener el workload de Xamarin instalado, pero bueno: eso nos permite compartir el código fácilmente. **Se puede optar por tener dos soluciones** (una con la aplicación Xamarin y otra con la MVC y ambas con las librerías de clases) **pero entonces empiezan las fricciones:** si agregamos un proyecto nuevo que es necesario por algunos de los proyectos compartidos debemos agregarlo en las todas las soluciones que tengamos. Eso es bastante tedioso, y no debería ser necesario ya que **la referencia está a nivel de csproj, no a nivel de solución**. Pero si el proyecto referenciado no está en la solución, VS no lo cargará y no hablemos ya de que lo compile.
  
Además **una solución no puede contener “sub-soluciones”** (lo que mitigaría eso, ya que podríamos tener una sub-solución de “SharedLibs” y usar esta sub-solución en las dos soluciones principales). Y súmale a todo esto que **el formato del fichero sln es horrendo con unos GUIDs** que no se sabe muy bien que son y qué hacen… Quien haya tenido el placer de participar en un merge donde en dos ramas distintas se hayan añadido proyectos a la misma solución sabrá lo doloroso que puede ser.
  
Efectivamente: **todas esas fricciones se solucionan si en lugar de referencias a proyectos tenemos referencias binarias** (ya sea a la antigua usanza (directamente a los ensamblados) o bien usando nuget).  Pero, usar referencias binarias cuando estás desarrollando todas las partes de la aplicación es un peñazo.
  
**Como ha cambiado el desarrollo**
  
Honestamente si echo la vista atrás, **años atrás, no tenía demasiadas fricciones con el concepto de solución**.
  
Pero **a medida que los proyectos crecían y pasábamos a usar soluciones con 150 o más proyectos, entonces es cuando Visual Studio empezaba a crujir**. Y a pesar de que con cada nueva versión prometían que “esa sí, esa de verdad, esa es la buena, esa es la que carga soluciones enormes sin morir”, la realidad es que con soluciones grandes VS2017 sigue crujiendo (bueno, este cruje incluso en soluciones pequeñas :P).
  
Pero **en estos casos, lo que hacíamos era “fraccionar” la solución en soluciones más pequeñas y sustituír las referencias “entre proyectos” por referencias  binarias** (o más tarde, en algunos casos, por nugets internos). La razón es que en una solución de 200 proyectos rara vez tocas los 200, así que tampoco los necesitas todos juntos. Añade _overhead_ en algunos casos, claro: si tienes que tocar alguno de los proyectos que no está en tu solución debes abrir (en otro VS, porque VS no abre dos soluciones) la solución correspondiente, compilar y (depende de como lo tengas todo configurado) copiar las dlls al directorio “de libs compartido” o bien publicar el nuget nuevo y actualizar paquetes en el otro proyecto.
  
Pesado, sí. Admisible… bueno, a todo se acostumbra uno.
  
Cambiemos todavía más el paradigma. Ahora queremos empezar a romper nuestro monolito, que estamos en la era de los containers. **Rompemos nuestra API en varios proyectos MVC Core**. **Cada uno de esos proyectos Core tiene sus propias librerías adicionales, pero también hay un conjunto de librerías compartidas entre ellos**: logging, librerías de DTOs, infraestructura varia.
  
Observa que **no estamos aplicando microservicios** ni nada parecido. Microservicios requiere una cultura de devops que no tenemos en este punto. **Simplemente queremos desplegar nuestra aplicación en varios containers** (o servidores) por separado porque queremos aprovecharnos de las ventajas: actualizaciones más o menos independientes (dependiendo de hasta que nivel nos atemos con las librerías comunes) y también escalado “por partes” de la aplicación. Y por supuesto porque en un futuro siempre podremos adoptar la cultura pura de Microservicios y terminar separando del todo los desarrollos y tener pipelines de CI/CD completamente independientes.
  
En este caso, **imaginate que tenemos una aplicación con 10 web apis, y varios clientes (escritorio, xamarin, etc)**. Entre esa docena (aprox) de aplicaciones (puesto que cada web api es un aplicación de por si) comparten librerías de clases (algunas usadas por dos web apis, otra por una y un cliente, otras por todos, etc). Al final el resultado es algo como 10 contenedores Docker que ejecuto donde quiero y mis clientes Xamarin o escritorio. **Y una solución que los contiene  a todos**. Y el terror.
  
**¿Como mitigar esto?**
  
**La principal razón de usar las soluciones es por tener el código todo cargado en Visual Studio y para poder tener referencias entre proyectos** (y no binarias/nuget). Si un proyecto no está en la solución VS no lo compilará y si este proyecto es usado (en forma de referencia a proyecto) por otro proyecto, este otro proyecto tampoco compilará.
  
Para mi, lo siguiente sería lo ideal (empieza mi carta a los reyes):

  1. Poder cargar uno o varios csproj en VS. No me parece mal tener un fichero sln que sea una colección de csprojs, pero nada más que eso. Simplemente un “shortcut” para que VS cargue N csprojs de golpe.
  2. Las referencias deberían poder ser a otro csproj y binarias/nuget a la vez.
  3. Si un csproj tiene una referencia a otro csproj, VS debería compilar este proyecto referenciado, aun cuando no esté cargado.
  4. Si una referencia de un csproj es binaria/nuget y a otro csproj a la vez, VS debería permitirme de forma fácil o bien usar la referencia binaria o bien compilar el csproj referenciado.

Para mi estas modificaciones habilitarian muchos escenarios adicionales:

  1. Cargo solo el csproj en el que estoy trabajando (p. ej. la app Xamarin) y sé que todas las dependencias se compilan.
  2. En cualquier momento puedo cargar uno o varios de los csprojs referenciados
  3. Otros equipos pueden proporcionar “referencias estables” en forma de binarios/nugets y yo puedo compilar mi código contra esas referencias estables (si no quiero/puedo compilar las referencias como csprojs adicionales)

El fichero principal del trabajo es el csproj, no la solución: abro un csproj y a partir de este abro csprojs adicionales, cuando yo lo estime necesario. Por supuesto algunas funciones de VS (como find usages y similares) deberían ser configurables y buscar en los csprojs abiertos, en las referencias directas o bien en todas las referencias.
  
Con esto evitamos el fichero sln (que deja de ser necesario y en todo caso, de existir, cada uno podría tener los suyos, sin necesidad de subirlos al control de código fuente).
  
Por supuesto estoy seguro que la realidad no es tan sencilla como este post, y que hay algunos temas más que se deberían tratar, pero **sí creo que es necesario que se empieze a pensar en como mejorar el formato de la solución de VS,** porque se está quedando obsoleto.
  
¿Y tú… qué experiencias tienes con las soluciones en VS? ¿Como sería para tí la “solución” ideal?
  
Gracias!
  
**Nota:** Edito para añadir una respuesta de Lluis Sanchez via twitter, que comenta puntos realmente interesante: <https://twitter.com/slluis/status/864065394868441088>  🙂