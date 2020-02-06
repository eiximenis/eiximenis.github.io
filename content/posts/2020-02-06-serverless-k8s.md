---
title: "Serverless & Kubernetes"
author: eiximenis
description: Serverless y Kubernetes son dos de las palabrejas de moda en el mundillo cloud. El primero promete ejecutar nuestro código sin que tengamos que saber nada de la infraestructura subyacente, y el segundo nos ofrece una plataforma de ejecución para aplicaciones basadas en contenedores. Pero... ¿son compatibles? ¿Y si lo son, cómo?
date: 2020-02-06T16:30:00
categories:
  - k8s
  - serverless
---

Antes de empezar a discutir como Kubernetes y Serverless pueden estar juntos en la misma frase que no sea sacada de un _flyer_ comercial (y por lo tanto, tenga sentido) debemos hablar un poco sobre Serverless en si mismo.

## ¿Qué es Serverless?

El concepto de Serverless significa a grandes rasgos que no "ves" el servidor que ejecuta tu aplicación. Claro, servidor hay, todavía no se ha inventado el humo que ejecute software, así que en algún lado hay un servidor. Pero, la abstracción que tu proveedor de _cloud_ hace de él, te impide verlo. En un modelo Serverless, despliegas tu código y este "se ejecuta". El "dónde" es algo que no te importa. Se podría argüir que p. ej. una Web App en Azure es una implementación del modelo Serverless. Pero Web App encaja más dentro del modelo PaaS (_Platform as a Service_), ya que tenemos una visión clara del servidor. No, no tengo acceso completo a la máquina (virtual) que ejecuta mi aplicación en una Web App, pero si debo desplegar mi código en una Web App en concreto y puedo configurar ciertos elementos de esta Web App.

Por otro lado tenemos el concepto de _Functions as a Service_ (FaaS) que es una implementación concreta del modelo Serverless. Bajo FaaS mi aplicación se compone de un conjunto de funciones que se ejecutan en base a unos disparadores (_triggers_). Es un modelo de desarrollo basado en eventos, en el cual esos eventos disparan esos disparadores que ejecutan las funciones correspondientes. Una petición http puede ser uno de esos eventos, lo mismo que un mensaje en una cola, añadir un registro en una base de datos o subir un fichero en un sistema de almacenamiento. FaaS es muy popular en el _cloud_, ya que los proveedores de cloud han integrado muchos de sus servicios con este modelo y el despliegue es muy sencillo.

Mucha gente confunde Serverless con FaaS y es cierto que muchas veces ambos conceptos se utilizan como puros sinónimos. Pero estrictamente FaaS es una implementación concreta del modelo más genérico de Serverless. Per no todas las aplicaciones Serverless deben diseñarse y desplegarse como un conjunto de funciones. Otros modelos de Serverless que tenemos en Azure por ejemplo podrían ser [ACI](https://azure.microsoft.com/es-es/services/container-instances/) (ejecución de contenedores), Logic Apps o el (eternamente en preview) [Service Fabric Mesh](https://docs.microsoft.com/es-es/azure/service-fabric-mesh/service-fabric-mesh-overview). Estos tres servicios cumplen con la premisa de Serverless de no ver qué servidor es el que ejecuta el código que despliego.

## Serverless Kubernetes

La primera pregunta a hacernos es si existe o puede existir el concepto de "Serverless Kubernetes". Hay que tener presente que Kubernetes ya me abstrae de los servidores subyacentes: no tengo necesidad de acceder a las máquinas que conforman el Kubernetes para poder desplegar una aplicación en él y mantenerla. Pero a nivel operativo si que veo los distintos nodos. E incluso en un Kubernetes manejado (como puede ser [AKS](https://azure.microsoft.com/es-es/services/kubernetes-service/), [EKS](https://aws.amazon.com/es/eks/) o [GKE](https://cloud.google.com/kubernetes-engine)) hay que decidir qué nodos añadir y sus características.

En este contexto cuando hablamos de "Serverless Kubernetes" nos referimos a un "Kubernetes sin nodos": simplemente despliego una aplicación en este Kubernetes y listos. El propio modelo de aplicación de Kubernetes (a través de las peticiones y límites de recursos) me ofrece un mecanismo para que pueda especificar qué recursos necesita un _pod_ para ejecutarse, y debería ser responsabilidad del proveedor de este "Kubernetes sin nodos" encontrar donde ejecutar el _pod_. 

En Azure este concepto existe a través de la integración entre ACI y AKS. Recordemos que ACI es un modelo Serverless de ejecutar contenedores. El equipo de AKS ha integrado ACI en AKS de forma que es posible configurar un AKS paa qué ejecute los _pods_ en ACI, en lugar de en las máquinas virtuales tradicionales. Esta integración se conoce como ["nodos virtuales"](https://docs.microsoft.com/en-us/azure/aks/virtual-nodes-portal).

Claro, que no es Azure el único proveedor que ofrece esa solución. AWS tiene también un modelo Serverless de ejecución de contenedores: [Fargate](https://aws.amazon.com/es/fargate/). Es posible integrar EKS con Fargate y terminar con una solución parecida a la de los nodos virtuales en Azure. 

Todas esas integraciones están basadas en un proyecto open source, llamado [Virtual Kubelet](https://github.com/virtual-kubelet/virtual-kubelet). Este proyecto permite, a grande rasgos, integrar otros sistemas de computación que son vistos como nodos por Kubernetes. Así, el _virtual kubelet_ crea un "nodo fictício" de forma que todo lo que se despliegue en este nodo, será ejecutado por el sistema de computación asociado.

Por lo tanto, a día de hoy podemos afirmar que **existen soluciones de Serverless Kubernetes**, en el sentido de que no tengo unos nodos explícitos si no que la ejecución de las aplicaciones desplegadas en este Kubernetes se deriva a otros sistemas tales como ACI o Fargate.

## Serverless EN Kubernetes

Claro que esta es la segunda parte de la discusión. **¿Podemos desplegar una aplicación serverless en Kubernetes?**. La verdad es que el despliegue de aplicaciones en Kubernetes está más pensado desde el punto de infraestructura que de aplicación (conceptos como los de "servicio" o "pod" son un claro ejemplo), pero la idea es: ¿Puedo decirle a Kubernetes "ejecuta este código"? Y me olvido de configurar todos los objetos de Kubernetes asociados.

Pues también es posible hacer eso. Existen proveedores FaaS para Kubernetes. La idea de esos proveedores es que, nosotros al desplegar la aplicación, desplegamos simplemente el código de dichas funciones y es el proveedor el encargado de empaquetar nuestro código en un contenedor, crear los objetos de Kubernetes asociados (p. ej. un _deployment_) y ejecutar nuestro código.

Por un lado tenemos productos como [OpenFaaS](https://www.openfaas.com/) o [Knative](https://cloud.google.com/knative/) entre muchos otros que permiten el despliegue de aplicaciones FaaS en Kubernetes o nos dan las herramientas para integrar otros motores. De hecho, usando Knative, la gente de AWS, ha integrado Lambda con EKS, de forma que podemos ejecutar una aplicación FaaS basada en Lambda en EKS. Microsoft tampoco se ha quedado atrás, y es posible ejecutar Azure Functions en AKS. Hay que tener presente que en estos casos se conserva la capacidad de autoescalar en base a eventos externos. Si tenemos una función ejecutándose en Kubernetes que se dispara cuando llega un mensaje a una cola, nos puede interesar escalarla horizontalmente en base al número de mensajes que se reciban. Para esta parte es interesante el proyecto [KEDA](https://keda.sh/) que permite autoescalar contenedores basados en eventos.

Todos esos productos cubren aspectos distintos y aunque se solapan en mayor o menor medida, es posible combinarlos.

Claro que, igual te preguntas qué sentido tiene ejecutar mis aplicaciones FaaS en Kubernetes, en lugar de hacerlo con el sistema de ejecución de mi proveedor de _cloud_. Hay varias razones donde esto te podría interesar, pero en general se trata de que tus aplicaciones FaaS conviven con el resto de aplicaciones "tradicionales" que se están ejecutándose en el clúster. Y si además usas infraestructura en Kubernetes (y/o pensada para Kubernetes como p. ej. [KubeMQ](https://kubemq.io/)) tus aplicaciones FaaS la comparten con el resto de aplicaciones. Por lo general se trata de qué si has adoptado Kubernetes como tu plataforma de ejecución, no tengas que "dejarla de lado" por el mero hecho de querar adoptar el paradigma de FaaS.

## Serverless EN Serverless Kubernetes

Ambos conceptos los podemos combinar, por supuesto. Y podemos tener una aplicación Serverless (FaaS) ejecutándose en un "Serverless Kubernetes" (o Kubernetes "sin nodos"). De este modo, despliego el código de mi función en Kubernetes y esta termina ejecutándose en un sistema Serverless tal como ACI o Fargate: lo mejor de ambos mundos.

![Esquema de Serverless EN Serverless Kubernetes](/images/posts/2020-02-06-serverless-k8s.png)

La figura anterior muestra un ejemplo: Se despliega una función en un clúster que tiene OpenFaaS, y este genera los objetos Kubernetes necesarios, que son enviados a la API de Kubernetes. Estos objetos son enviados a un "nodo virtual" que (a través del _virtual kubelet_) termina ejecutando el contenedor en un recurso externo, tal como ACI o Fargate.

En resúmen, que sí: es posible tener "Serverless" y "Kubernetes" juntos en la misma frase y que tenga todo el sentido del mundo.

¡Saludos!