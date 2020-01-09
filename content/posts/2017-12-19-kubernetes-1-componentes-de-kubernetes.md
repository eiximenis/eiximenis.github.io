---
title: 'Kubernetes (1): Componentes de Kubernetes'
author: eiximenis

date: 2017-12-19T11:10:29+00:00
geeks_url: /?p=1945
geeks_ms_views:
  - 1500
categories:
  - docker
  - kubernetes

---
Bueno, con este post empiezo una serie de posts sobre [Kubernetes][1]. Tengo varios posts en borrador, pero creo que antes de publicarlos puede estar bien una pequeña introducción a Kubernetes: qué es. Y de esto va este post.
  
<!--more-->


  
**¿Qué es Kubernetes?**
  
A grandes rasgos: un orquestador de contenedores. Es decir un sistema cuya misión es ejecutar un conjunto de contenedores en una o más máquinas, ofreciendo herramientas para la gestión y el escalado de dichos contenedores.
  
**Nunca he instalado un Kubernetes on-premise** y espero no tener que hacerlo nunca. Digo esto porque hablamos de &#8220;Kubernetes&#8221; como un todo, pero Kubernetes es más bien &#8220;un conjunto de herramientas&#8221; integradas entre ellas. No sé como es &#8220;instalar un clúster de Kubernetes&#8221; pero no debe ser nada sencillo. Por suerte **los tres grandes proveedores de cloud** ofrecen Kubernetes como servicio manejado: Azure ofrece [AKS][2], Google ofrece [GKE][3] y Amazon tiene [EKS][4]. Todos ellos servicios que permiten crear de forma más o menos rápida y sencilla un clúster Kubernetes.
  
Si quieres probar en local también hay una solución: [MiniKube][5]. Con MiniKube puedes ejecutar tu propio Kubernetes (de un solo nodo) en una MV (VirtualBox o HyperV). MiniKube se encarga de todo, de descargar la imagen, montar la máquina y hasta configurar _kubectl_ (la herramienta CLI para gestionar Kubernetes).
  
Por lo general en un clúster de Kubernetes hay dos tipos de máquinas:

  * master: Su objetivo es tomar decisiones que afectan al conjunto del clúster (p. ej. escalado de contenedores). No suelen ejecutar contenedores (aunque no está prohibido que lo hagan). Un clúster de Kubernetes contiene al menos un nodo master, aunque pueden haber varios.
  * node: Ejecutan contenedores (como se ha dicho antes, técnicamente un master es _también_ un _node_, salvo que muchas veces se configura para no ejecutar contenedor alguno).

Esto es desde el punto de máquinas. Veamos ahora los componentes de software más importantes (no están listados todos) que cada tipo de máquina ejecuta. Así, pues, instalar un clúster de Kubernetes es instalar cada uno de esos sistemas y configurarlos:

  1. [etcd][6]: Se usa etcd en los nodos _master_ para guardar los datos referentes a todo el clúster.
  2. kube-apiserver: Kubernetes ofrece una API REST para operar con el clúster. Este componente que se ejecuta en los nodos _master _es el que sirve dicha API.
  3. kube-controller-manager: Es un _daemon_ que se ejecuta en los nodos _master_ y que se encarga de gestionar &#8220;los controladores&#8221;. Un controlador en Kubernetes es un proceso en segundo plano que ejecuta determinadas tareas ordinarias para el funcionamiento del clúster,
  4. cloud-controller-manager: Es otro _daemon_ que se ejecuta en los nodos _master_ y que se encarga de gestionar &#8220;los controladores de cloud&#8221;. Esos controladores tienen dependencias con proveedores de cloud y estan separados de los normales para permitir evoluciones separadas. La idea es que cada proveedor de cloud (que quiera soportar Kubernetes) termine evolucionando dichos controladores.
  5. kubelet: Se ejecuta en los nodos _node_ y tiene una sola función: dada una lista de contenedores a ejecutar en una máquina, garantizar que estos se estan ejecutando.
  6. kube-proxy: Se ejecuta en los nodos _node_ y es un proxy sencillo que puede hacer _forwarding_ a nivel de TCP/UDP.
  7. docker: Obviamente se ejecuta en los nodos _node_ y es el encargado, real, de ejecutar los contenedores. Otro motor soportado por Kubernetes como alternativa a Docker, es [rkt.][7]
  8. [supervisord][8]: kubelet se encarga de asegurarse que los contenedores se ejecutan. Pero, ¿quien se encarga de verificar que _kubelet_ y _docker_ están ejecutándose? Pues este pequeño proceso que corre en todos los nodos _node_.
  9. [fluentd][9]: Se ejecuta en los nodos _node_ y ofrece servicios de _log_ a nivel de clúster.

Como digo, aunque no estan todos (faltan algunos que se ejecutan en los nodos _master_), los principales serían esos. Se puede ver que hay una mezcla de componentes propios con componentes open source externos. Hay que tener presente esto cuando hablemos de Kubernetes: Nos referimos a él como &#8220;un solo producto&#8221; pero realmente es más que eso.
  
**Como crear un Kubernetes en local**
  
Vamos a ver ahora como crear un clúster de Kubernetes en local. Como digo, ni idea de como se instala _on-premise_, así que para instalar nuestro primer cluster usaremos MiniKube 🙂
  
Lo primero es [instalar MiniKube][10] y [kubectl][11] en nuestra máquina (ambos son binarios que basta con descargar y poner en una carpeta que esté en el PATH). Una vez hecho deberías abrir una línea de comandos como administrador y teclear:

<pre class="EnlighterJSRAW" data-enlighter-language="shell">minikube start --kubernetes-version v1.8.0 --vm-driver hyperv --hyperv-virtual-switch [adaptador-red]</pre>

En este caso MiniKube levantará un Kubernetes (1.8.0) usando hyperv. Si no usas la opción (&#8211;vm-driver) se usa VirtualBox: el propio MiniKube se encargará de descargar la ISO, crear la máquina virtual y ponerla en marcha.
  
**Nota: **Si usas hyperv, en el parámetro &#8211;hyperv-virtual-switch debes indicar un adaptador de red que tenga conectividad desde el host (en otro caso fallará al intentar conectarse con la MV usando ssh).
  
Cuando MiniKube haya terminado **tendrás un Kubernetes en local y la herramienta kubectl configurada**. Puedes comprobarlo haciendo:

<pre class="EnlighterJSRAW" data-enlighter-language="shell">kubectl config get-contexts</pre>

Y te debería salir un listado parecido al siguiente:

<pre class="EnlighterJSRAW" data-enlighter-language="generic">CURRENT   NAME                  CLUSTER               AUTHINFO
*         minikube              minikube              minikube</pre>

(Si tuvieras más clústers aparecerían ahí. El asterisco indica el clúster contra el que kubectl actúa).
  
¡Fantástico! Ya tienes tu Kubernetes corriendo en local. Kubernetes tiene un panel de control (opcional, pero si usas v1.8.0 en MiniKube viene instalado) al que puedes acceder tecleando `kubectl proxy` y luego yendo a `http://localhost:8001/ui`. Esto te debería ver el panel de control de Kubernetes:
  
[<img class="alignnone size-medium wp-image-1946" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/12/kubernetes-control-300x149.png" alt="" width="300" height="149" />][12]
  
El siguiente punto será empezar a desplegar contenedores en él, y lo veremos en el siguiente post de esta serie 🙂

 [1]: https://kubernetes.io/
 [2]: https://azure.microsoft.com/en-us/services/container-service/
 [3]: https://cloud.google.com/kubernetes-engine/
 [4]: https://aws.amazon.com/eks/
 [5]: https://github.com/kubernetes/minikube
 [6]: https://github.com/coreos/etcd
 [7]: https://coreos.com/rkt/
 [8]: http://supervisord.org/
 [9]: https://www.fluentd.org/
 [10]: https://kubernetes.io/docs/tasks/tools/install-minikube/
 [11]: https://kubernetes.io/docs/tasks/tools/install-kubectl/
 [12]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2017/12/kubernetes-control.png