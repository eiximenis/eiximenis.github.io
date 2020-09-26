---
title: "Opinión: De Kubernetes y independencia del cloud"
author: eiximenis
description: "Muchas organizaciones eligen Kubernetes para conseguir una independencia del cloud... Pero, ¿Es así? Y ¿a cambio de qué?"
date: 2020-09-26T18:00:00
draft: false
categories:
  - opinion
tags:
  - cloud
  - kubernetes
  - k8s
---

Cuando una organización me dice que ha elegido [Kubernetes](https://kubernetes.io/) para desplegar en él su último proyecto, siempre pregunto cuales son las razones que han llevado a elegir Kubernetes como plataforma. Y lo pregunto porque en muchos casos, lo que se quiere desplegar en el clúster es poco más que una API y un frontend... Vamos, una web de toda la vida. ¿Cuales son los motivos para meterte en un producto con un alto nivel de complejidad como Kubernetes? En mi experiencia detecto varios y me gustaria, sin ánimo de sentar cátedra de nada, simplemente comentarlos.

## Kubernetes manejados que esconden la complejidad

Desde que Kubernetes ganó la guerra de los orquestadores de contenedores, todos los grandes _clouds_ han estado invirtiendo para ofrecer servicios de Kubernetes más o menos manejados. Actualmente Google Cloud ofrece [GKE](https://cloud.google.com/kubernetes-engine), en AWS tenemos [EKS](https://aws.amazon.com/eks/) y en Azure está [AKS](https://azure.microsoft.com/en-us/services/kubernetes-service?WT.mc_id=AZ-MVP-4039791), solo por citar los tres clouds líderes. Pero al margen de esos tres proveedores existen muchos otros proveedores que ofrecen Kubernetes manejados.

Eso hace **que crear un Kubernetes hoy en día sea tan fácil como sacar la billetera**. No es necesario conocer [kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/) ni configurar apenas nada para tener un Kubernetes listo para poder trabajar con él. Instalar un Kubernetes _on premises_ no es tarea nada sencilla, pero encima configurarlo para que funcione correctamente puede ser, a veces, una odisea. Los Kubernetes manejados funcionan muy, muy bien y los tienes listos en 5 minutos a un click de un botón. Por lo tanto **la barrera de entrada está rota**. Y eso es siempre bueno para la adopción de cualquier plataforma. Si para adoptar (o evaluar) Kubernetes las empresas hubieran que pasar por el tedioso y complejo proceso de instalar uno "a mano", muy pocas organizaciones lo usarían. Por suerte, **toda esa complejidad está oculta por esos Kubernetes manejados**. 

Si bien la existencia de Kubernetes manejados es un facilitador (en tanto que bajan la barrera de entrada) para que las organizaciones empiecen a usar el producto, por si mismos no son la razón de que las organizaciones elijan Kubernetes.

## Independencia de cloud

Muchas organizaciones eligen Kubernetes para tener "independencia del cloud". El razonamiento que hay detrás es sencillo: Si mi código se está ejecutando en EKS en AWS y más adelante quiero moverme a Google Cloud simplemente despliego mi código en un GKE y listos.

Por supuesto este "listos" no es tan sencillo, pero sí, _a priori_ **parece que si tu código se ejecuta sobre un Kubernetes, va a ser más fácil migrarlo a otro Kubernetes en otro cloud en un futuro**. Pero, me gustaría plantear algunas cuestiones adicionales.

### ¿Infraestructura como código? (IaC)

Empecemos por la primera cuestión: ¿Usas infraestructura como código? Dicho de otro modo, si te quieres ir de un cloud a otro, como vas a desplegar la infraestructura en este otro cloud? Y sí la usas... ¿la usas de forma (más o menos) agnóstica?

No sea que lo quieras tener todo en Kubernetes, y tu problema luego esté en migrar un montón de plantillas [ARM](https://docs.microsoft.com/en-us/azure/azure-resource-manager/management/overview?WT.mc_id=AZ-MVP-4039791) a [Cloudformation](https://aws.amazon.com/cloudformation). Aquí es donde entran en escena otros elementos como [Terraform](https://www.terraform.io/) o [Pulumi](https://www.pulumi.com/) (por citar dos). Y yendo un paso más allá, usar CRDs desplegados en el propio Kubernetes para definir la infraestructura asociada (como [Azure Service Operator](https://github.com/Azure/azure-service-operator) o [AWS Service Operator](https://aws.amazon.com/blogs/opensource/aws-service-operator-kubernetes-available/)).

En todo caso **necesitas un mecanismo de IaC sí o sí si quieres tener cierta independencia del cloud**. Pero uses el mecanismo que uses, siempre tendrás cierto trabajo al pasarte de un cloud a otro.

### ¿Donde está el Vendor Lockin?

Para facilitar irte de un proveedor de cloud a otro, lo que debes hacer es, básicamente, evitar el _vendor lockin_, es decir **evitar el uso de aquellos productos que tienen un SDK propio** que solo se encuentra en un proveedor. Ya que el uso de esos servicios es lo que más te impactará al canviarte de cloud.

Ya, parece una obviedad, pero muchas veces se olvida. Por ejemplo, **desplegar tu API en una [Azure Web App for Containers](https://azure.microsoft.com/en-us/services/app-service/containers?WT.mc_id=AZ-MVP-4039791)** tiene mucho _vendor lockin_ o poco?

Hombre, pues estás básicamente desplegando una imagen de Docker. Supongamos que ese código accede a una base de datos MySql y la despliegas en PaaS. En Azure tenemos [Azure Database for MySql](https://azure.microsoft.com/en-us/services/mysql?WT.mc_id=AZ-MVP-4039791). En este punto has usado dos servicios PaaS clásicos... pero ¿cuan atado estás a Azure?

Pues a ver, si te quieres ir a AWS, lo único que debes hacer es crear y configurar los servicios equivalentes a los que usas en Azure: quizá un [Elastic Beanstalk](https://aws.amazon.com/elasticbeanstalk/) para la API y un [RDS](https://aws.amazon.com/rds/) para la base de datos. Y redesplegar tu API en el Beanstalk. Y poco más... **salvo migrar los datos en la nueva base de datos, lo que debes hacer en cualquier caso**, y que en muchos casos es un dolor de cabeza importante.

No digo que sea sencillo, porque una migración de cloud no lo es nunca, pero **el impacto sobre tu código es limitado** y ese es un punto a favor.

En este escenario te aportaría algo haber usado Kubernetes (ojo, solo valorando desde el punto de vista de "conseguir independencia del cloud"). Pues, asumiendo que usas un servicio de Kubernetes manejado, todo se reduce a:

1. Crear la infraestructura en el nuevo cloud (el nuevo cluster y la nueva BBDD si es que la usabas en PaaS).
2. Migrar los datos.

**¡Debes hacer exactamente lo mismo que sin usar Kubernetes!** Lo que quiero notar es que **en este escenario (API + BBDD relacional clásica)** el _vendor lockin_ es bastante bajo. 

¿Donde tenemos vendor lockin? Pues en aquellos productos "con APIs propias de cada cloud", en aquellos productos que para usarlos debemos programar usando alguna librería (o conjuntos de APIs) propia. Ejemplos los hay a montones pero por citar algunos:

* Servicios de mensajería (tipo [SQS](https://aws.amazon.com/sqs/) o [Service Bus](https://azure.microsoft.com/en-us/services/service-bus?WT.mc_id=AZ-MVP-4039791)). Esos servicios exponen APIs propias y muchas veces características propias que no se mapean exactamente al equivalente en otro cloud.
* FaaS ([Lambda](https://aws.amazon.com/lambda/) o [Azure Functions](https://azure.microsoft.com/en-us/services/functions?WT.mc_id=AZ-MVP-4039791)). En estos casos las restricciones vienen porque el código que despliegas debe ser distinto y además está muy atado a las características de cada cloud.
* BBDD específicas ([CosmosDb](https://azure.microsoft.com/en-us/services/cosmos-db/) o [DynamoDb](https://aws.amazon.com/dynamodb/)). Cada una de ellas expone no solo APIs propias si no que en algunos casos la mejor manera de organizar la información es distinta en cada caso. 

Vale... ¿ves por donde voy no? **Si usas esos servicios, tu código y generalmente la arquitectura entera se adapta a ellos**. Eso no es malo de por sí: al final usamos un cloud para sacar el máximo provecho de él y es normal que nuestra arquitectura y código se adapte a la nube elegida.

> Para intentar evitar el _vendor lock-in_ puedes intentar abstraerte de esos servicios. P. ej. tener una interfaz `IMessageBroker` y una implementación para SQS p. ej. Al momento que te quieres mover de AWS a Azure sólo debes crear otra implementación para Service Bus. Los problemas de esa aproximación son: 1) No te puedes abstraer de todos los servicios, 2) Es muy fácil que la abstracción "fugue" (de forma que termine exponiendo características solo existentes en la plataforma cloud inicial) y 3) Debes adaptarte al "mínimo común" de todas las plataformas, lo que te impide aprovecharlas al máximo.

## Kubernetes como "cloud"

Si realmente requieres independencia del cloud, entonces debes considerar a Kubernetes "como tu propio cloud". Eso significa que **todo debería ejecutarse en Kubernetes**. Por motivos de simplicidad puedes hacer alguna excepción con servicios de bajo _vendor lockin_ (como una base de datos SQL PaaS), pero poco más. 

Luego hay otro aspecto que emerge en este punto y que se trata de cuan dispuesto estás a "atarte a Kubernetes". Porque, al final, si adoptas Kubernetes como "tu propio cloud", si lo quieres aprovechar al máximo deberás usar productos "pensados para Kubernetes", que en algunos casos se ejecutan solo en Kubernetes. De este modo trasladamos la dependencia que antes teníamos sobre una plataforma cloud en concreto, a una dependencia directa sobre Kubernetes. Y por lo general, cuando más nos "atemos a Kubernetes" más provecho le podremos sacar. ¿A qué me refiero con eso? Pues que a lo mejor en lugar de usar, no sé, un [RabbitMQ](https://www.rabbitmq.com/) te puedes plantear usar algún sistema de mensajería específicamente pensado para ejecutarse bajo Kubernetes, como [KubeMQ](https://kubemq.io/).

Una mención aparte merece el paradigma de FaaS. Asumiendo que todo debe correr en Kubernetes, eso implica que o bien nos olvidamos de usar ese paradigma o bien lo usamos adaptado a Kubernetes. Aquí es donde entran en juego productos como [OpenFaas](https://www.openfaas.com/), [Knative](https://knative.dev/) o en un orden ligeramente distinto incluso [KEDA](https://keda.sh/).

Resumiendo, **del mismo modo que adaptamos la arquitectura de nuestro sistema para aprovechar al máximo una plataforma cloud en concreto (y nos atamos a ella), lo mismo hacemos para Kubernetes**. Dejamos de estar atados a una plataforma cloud, en efecto, pero por contrapartida estamos atados a Kubernetes.

Si usamos Kubernetes como cloud tenemos una independencia total respecto la plataforma cloud real que usemos (porque Kubernetes ES nuestro cloud), incluso nos podríamos montar nuestro Kubernetes on-premises y movernos a un CPD propio. **Básicamente, lo único que realmente debemos migrar son los datos**.

> Ojo, que eso no sería del todo cierto. Por un lado algunas definiciones de Kubernetes (ingress, PVCs) pueden depender de las capacidades del clúster que ejecutemos (y pueden ser distintas en cada cloud) y por otro lado, habría que retocar el _capacity plan_ para adaptar los valores de los HPAs y los VPAs, ya que las características de los nodos variarán dependiendo del cloud. Pero bueno, nuestro código, no debería verse afectado.

## Entonces... ¿qué?

Lo primero es plantearte hasta qué punto requieres una independencia del cloud. Yo no conozco tantos casos de proyectos que se hayan migrado de un cloud a otro como para justificar alegremente esa independencia. Me recuerda a hace algunos años, cuando en .NET salieron los primeros ORMs (EF y NHibernate), que muchas empresas empezaron a obligar al uso de esos ORMs para "independizarse de la BBDD", por si en un futuro se querían pasar de, no sé, SQL Server a Oracle. Muchas de esas aplicaciones eran meros CRUDs y, al menos por lo que a EF respecta, el uso de un ORM a veces complicaba más que ayudaba. Años después puedo contar con las dedos de una mano (y me sobran varios) la cantidad de esos proyectos que han migrado de base de datos. Y los que han migrado, han descubierto con sudor y lágrimas, que reescribir la capa de datos era el menor de los problemas al afrontar una migración de ese tipo.

De todos modos el primer punto, antes de "usar Kubernetes para independizarte del cloud" debería ser analizar **qué deberías hacer para migrar de cloud**. Porque si tu aplicación es un par de APIs HTTP y una BBDD igual te das cuenta que tampoco debes hacer tanta cosa. Igual te sale mejor simplificarte la vida y usar servicios PaaS sin problemas. Y cuando te muevas de cloud, pues usas los servicios PaaS equivalentes del otro cloud. Es solo si tienes pensado usar servicios con alto _vendor lock-in_ que debes plantearte qué hacer. Pero, es que ¡ojo!, quizá te sale a cuenta usar esos servicios PaaS **y tener más o menos clara la guía de migración a otro cloud** que no enfrascarte con un Kubernetes: que no solo es desplegar en Kubernetes: luego hay que mantenerlo, monitorizarlo, configurarlo...

Kubernetes es un producto maravilloso, pero complejo. Y tiene muchos, muchísimos usos mucho más válidos que "independizarte del cloud". Si tu motivo principal por elegir Kubernetes es, precisamente, "ser independiente del cloud", plantéate si esa necesidad es realmente necesaria y cuan complejo es tu sistema. Y recuerda, que si adaptas Kubernetes "solo para ser independiente", eso no viene gratis: Kubernetes es un sistema complejo y bastante "dogmático" (traducción mía de _opinionated_), qué te va a forzar a adoptar ciertas prácticas de un cierto modo si no quieres verte abocado a un fracaso. 

Ah, y si solo tienes una API y una BBDD: no, no necesitas Kubernetes para nada. Ni para escalabilidad ni para alta disponibilidad ni para nada. No mates moscas a cañonazos.

Saludos!



