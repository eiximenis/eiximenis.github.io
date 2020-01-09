---
title: Ficheros de configuración por entorno… ¿Sí, no o todo lo contrario?
description: Ficheros de configuración por entorno… ¿Sí, no o todo lo contrario?
author: eiximenis

date: 2019-07-12T08:48:31+00:00
geeks_url: /?p=2400
geeks_ms_views:
  - 113
categories:
  - devops
  - opinion

---
Andaba yo ayer gastándome el dedo intentando refrescar mi Twitter, extrañado [por qué no veía nuevos tweets][1]. Total, que tras unos intentos infructuosos, repasé algunos tweets antiguos de mi timeline, mientras esperaba el bus que me llevaría de la T2 del Aeropuerto del Prat, donde aterricé, a la T1 donde tenía el coche. Y así di con este tweet de [Quique][2]:  .

<blockquote class="twitter-tweet" data-width="550" data-dnt="true">
  <p lang="en" dir="ltr">
    If you have one JSON (or equivalent) file per ENV in your project... I'm sorry but you're not doing well 🤷🏽‍♂️ I don't care how much articles you read about this. Configure ENVs in your CD, if you don't have CD create one 😬 read about CI and CD and enjoy!
  </p>
  
  <p>
    &mdash; Quique Fdez Guerra (@CKGrafico) <a href="https://twitter.com/CKGrafico/status/1149271018374123520?ref_src=twsrc%5Etfw">July 11, 2019</a>
  </p>
</blockquote>


  
Y bueno... iba yo a responder, pero vi que la respuesta daba para mucho más que un tweet o sea que, aquí va 😉
  
<!--more-->


  
**¿Donde debe residir la configuración?**
  
En su tweet, Quique recomienda el uso de un sistema de CI/CD, no seré yo quien le contradiga en ese punto: **un sistema de CI/CD es algo que deberías considerar incluso en tus _pet projects_**. Pero tener un sistema de CI/CD no es para nada excluyente a tener ficheros de configuración por entorno.
  
Primero, **yo parto de la idea de que todo lo que se pueda tener en el repositorio es mucho mejor que tenerlo en cualquier otro sitio**. Por eso prefiero las builds de YAML de Azure Devops antes que las builds gráficas con cajitas (y por eso me pongo las manos en la cabeza sobre porque narices han tardado tanto a añadirlas, arrastrando a mucha gente a prácticas que no son las mejores, pero eso es algo típico en Microsoft).
  
Por el mismo motivo **prefiero la configuración en código, en el repositorio**, antes que en el sistema de CI/CD. [Daniel Fornells][3] contestaba al tweet de Quique:

<blockquote class="twitter-tweet" data-width="550" data-dnt="true">
  <p lang="en" dir="ltr">
    Partly disagree.<br />How about having multiple CD? Moving those configs (no secrets) to repo let you track history and change everywhere by single commit.
  </p>
  
  <p>
    &mdash; Daniel Fornells (@danifornells) <a href="https://twitter.com/danifornells/status/1149291377085353986?ref_src=twsrc%5Etfw">July 11, 2019</a>
  </p>
</blockquote>


  
Y tiene toda la razón. Pero **la clave no es mantener el histórico** (como bien le contesta Quique tu herramienta de CI/CD puede hacer eso). La **clave es poder actualizar código y configuración de forma conjunta en un único commit**. Si debo actualizar el código, hacer el _commit_ y luego actualizar la configuración en el entorno de CI/CD tengo discrepancias: si quiero construir una versión antigua de mi código, que pueda tener etiquetada... ¿como puedo asegurar que se me aplique la configuración que correspondía a dicha versión? No, **que la herramienta de CI/CD me permita mantener el histórico no es la solución, si ese histórico no va atado a la versión correspondiente del código que esté en el repositorio. **Y la mejor manera es asegurar que la configuración se encuentra en el repositorio.
  
De hecho, **es exactamente la misma razón por la cual preferimos la infraestructura como código**: porque así está sincronizada. La **configuración forma parte de tu código** y debería ser mantenida con la misma herramienta que mantienes este.
  
**¿Todo debe ser mantenido en el repositorio?** ¡¡¡NOOOOO!!! Hay una parte importante de la configuración que NO debe estar en el repositorio. Los secretos, claro. Un &#8220;secreto&#8221; es toda aquella configuración que de ser visible por alguien más puede comprometer la seguridad del proyecto: cadenas de conexión, usuarios, contraseñas, tokens y claves varias... Todo eso **sí debe estar gestionado fuera de tu repositorio y es la herramienta de CD el mejor lugar donde tenerlos**. Por lo tanto, en mi opinión, lo ideal es:

  * La configuración no secreta, tenerla en el repositorio
  * La configuración secreta nunca en el repositorio (**no, no vale encriptar el fichero**). Debe ser externa y ser aplicada por una herramienta externa (el servidor de CD). Es cierto, añade fricción al sistema, pero la seguridad manda... Ah, y si te lo estás planteando: **incluso en repositorios privados debes evitar tener la configuración secreta**.

**Ahora bien, lo importante es tener la configuración (no secreta) en el repositorio**. ¿Eso significa que debo usar un web.config, appsettings.json o algo similar? No, no tiene por qué. **Si tu herramienta de CD te permite definir variables de configuración por entorno y hacerlo en un fichero del repo, eso es perfecto**. Pero, si esas variables, las defines en una UI y se guardan fuera del repo... considera si puedes alguna alternativa.
  
Un ejemplo concreto: En Net Core puedes usar appsettings.json para configurar tu sistema. Incluso admite ficheros por entorno (appsettings.{entorno}.json) que se combinan con el primero: así puedo tener la configuración global en el fichero appsettings.json y la específica de development en appsettings.Development.json por decir algo.
  
Ahora bien, si tengo un proceso de _build_ que me crea un contenedor Docker a partir de ese proyecto, **dicho proceso de build no debe añadir el fichero _appsettings.{entorno}.json_** a la imagen. Hacerlo crearía una imagen atada al entorno, que es contrario a la filosofía de Docker. Pero, si como herramienta de CD uso Azure Devops para desplegar esa imagen en un App Service, mediante Azure Devops, es posible leer un fichero appsettings.{entorno}.json **y desplegarlo como variables de entorno del App Service**. Y Net Core se configura también con variables de entorno. En este caso, mi configuración del entorno está en el repo (fichero appsettings.{entorno}.json) pero cuando se despliega el entorno la herramienta de CD lo transforma a variables de entorno. Si Azure Devops no permitiese eso, **sirve igualmente poder tener la definición de la _release_ en el repo**: como parte de la _release_ tendrás la definición de los entornos y sus variables (es decir, la configuración). Pero si tu sistema de CI/CD no permite eso... deberías valorar si te vale la pena usarlo.
  
Otro ejemplo, ahora con Kubernetes. En este caso, Azure Devops no me permite pasar de appsettings.{entorno}.json a variables de entorno, porque en Kubernetes debo desplegar usando sus propios ficheros yaml. Pero en este caso puedo tener dos aproximaciones:

  * Usar charts de helm, pero con **ficheros values.yaml fijos por entorno**. Luego desde la herramienta de CD despliego dichos _charts_.
  * Usar directamente ficheros YAML con los config maps (uno por entorno) y desplegar (p. ej. usando [Kustomize][4]) desde la herramienta de CD.

En ambos casos, **la configuración está en el repositorio **(en los ficheros values.yaml del chart o en las definiciones YAML de los configmaps) y tu herramienta de CD aplica esos ficheros de la forma necesaria cuando despliega un entorno.
  
Y recuerda que eso solo se aplica a la configuración &#8220;no secreta&#8221;. Los secretos deben ser gestionados aparte y ahí si que lo mejor es la herramienta de CD.
  
¿Y tú, qué opinas al respecto?

 [1]: https://www.elperiodico.com/es/extra/20190711/twitter-sufre-una-caida-a-nivel-mundial-7550442
 [2]: https://twitter.com/ckgrafico
 [3]: https://twitter.com/danifornells
 [4]: https://kustomize.io/