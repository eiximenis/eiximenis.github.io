---
title: 'Ejecutar bases de datos en Kubernetes: ¿Sí, no?'
author: eiximenis

date: 2018-09-21T07:21:35+00:00
geeks_url: /?p=2187
geeks_ms_views:
  - 749
categories:
  - kubernetes
  - opinion

---
Esta es una de las preguntas que tarde o temprano cualquiera que trabaje con Docker y Kubernetes (o cualquier otro orquestrador) se termina encontrando: Tienes toda tu aplicación en contenedores, ejecutándose en Kubernetes. Sabes también que existen versiones _dockerizadas_ de las bases de datos. Así pues... ¿es conveniente ejecutar las bases de datos como contenedores adicionales en el orquestrador?
  
<!--more-->


  
La respuesta es depende 🙂
  
En **entornos de desarrollos** (y pruebas) no tengo dudas: por supuesto. Estos entornos suelen ser sencillos, por lo que un despliegue normal (un servicio y un deployment) te sirve para desplegar tu base de datos como parte del clúster. Puedes agregarle un volumen para que no se pierda el contenido de la BBDD cada vez que, por cualquier razón, el _pod_ se reinicie.
  
Muchos de esos entornos de desarrollo se paran y se ponen en marcha cuando el desarrollador quiere y la pérdida de datos no es excesivamente importante (suelen haber _seeds_ y demás). Ejecutar tus bases de datos en el clúster te elimina la dependencia de recursos externos y su gestión. **Yo abogo por usar siempre compose en desarrollo para tener, al menos, toda la infraestructura de desarrollo en contenedores en local**. Pues esto es lo equivalente, pero en entornos con orquestradores de desarrollo: tu orquestrador es todo tu entorno, incluyendo la aplicación y infraestructura necesaria. Igual puedes pensar que total, usar ARM o terraform para crear la infraestructura en el _cloud_ tampoco es tan complejo (y tienes razón) pero aunque no sea algo muy complejo, debes crear y ejecutar esos scripts. Por otro lado incurrirás en costes adicionales (de esos recursos externos). Más complejidad, más coste... no le veo las ventajas. En _on-premise_, a pesar de que puedes automatizar la creación de infraestructura, hay incluso menos dudas.
  
La **duda viene en entornos &#8220;productivos&#8221;** (no solo producción, también pre y similares): en estos entornos un despliegue sencillo de la base de datos no te sirve: necesitas tener alta disponibilidad. El como lo consigas ya depende de cada motor de base de datos, pero en cualquier caso te verás obligado a usar artefactos de Kubernetes más complejos como los volúmenes persistentes, servicios _headless_ y _statefulsets_.
  
En este punto es importante, creo, saber si puedes/quieres/vas a usar la BBDD como un servicio PaaS. Si es el caso, quizá esa sea la mejor opción: Te olvidas de toda la gestión y configuración y sin hacer nada (salvo crear el recurso) ya tienes tu BBDD con alta disponibilidad y servicios adicionales (backups, etc). ¿Pero y si usar PaaS no es una opción (o bien, por cualquier motivo, no quieres usarlo)?
  
Hay que decir que mucha gente se echa las manos a la cabeza cuando les planteas la posibilidad de ejecutar las BBDD productivas en Kubernetes: &#8220;no tiene sentido&#8221; suele ser la respuesta más habitual. Quiero dejar claro que <span style="text-decoration: underline;"><strong>Kubernetes hoy en día, tiene las herramientas para ejecutar BBDD en entornos productivos con todas las garantías</strong></span>. Si lees artículos a este respecto, asegúrate de que estén actualizados. **Kubernetes no dipuso de dichas herramientas hasta su versión 1.5 que salió en Diciembre del 2016**. Cualquier artículo anterior a fecha seguramente te dirá que Kubernetes no es válido para ejecutar bases de datos en entornos productivos.
  
Así que poder, se puede. Entonces la respuesta a si hacerlo o no ya no es puramente técnica si no que hay otros aspectos a considerar.
  
Uno de ellos es el coste: dependiendo de tus necesidades, es posible que sea más barato ejecutar la BBDD dentro de un clúster de Kubernetes que en un servidor dedicado. Por supuesto esto depende mucho de las necesidades que tenga la BBDD, pero es un factor a evaluar.
  
Otro aspecto es Devops: si tu base de datos se ejecuta en Kubernetes su despliegue queda integrado en el ciclo de vida de despliegue de la aplicación. Es decir, tenemos un modelo de despliegue unificado de tu aplicación y sus dependencias. Luego para monitorizar otro tanto de lo mismo: cuando monitorizas el clúster, ya estás monitorizando todo el entorno, incluyendo la base de datos.
  
No digo que esas razones sean lo suficientemente poderosas como afirmar que sí, que debes tener tus bbdd en K8s. De hecho creo que para muchas organizaciones quizá hoy en día siga siendo recomendable tener las BBDD en entornos propios. Solo digo que si quieres puedes tener tus BBDD productivas en Kubernetes. A ti te toca evaluar si esa es o no la mejor opción para cada caso concreto.
  
Si al final te animas y quieres montar un entorno productivo con las bases de datos en Kubernetes echa un vistazo a [Kubernetes Database][1], donde hay multitud de scripts y configuraciones para varios sistemas de bases de datos que te ayudará a dar los primeros pasos 😉

 [1]: https://github.com/kubedb