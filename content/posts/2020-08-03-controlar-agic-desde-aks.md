---
title: "Como controlar un Application Gateway desde AKS"
description: "Si quieres saber como configurar y controlar un Application Gateway a través del controlador ingress de AKS, en este post, Jorge responderá a todas tus dudas! :)"
author: Jorge Turrado
authorEmoji: 🐛 # emoji for subtitle, summary meta data
authorImage: "/images/whoami/jturrado.jpg" # image path in the static folder
authorImageUrl: "" # your image url. We use `authorImageUrl` first. If not set, we use `authorImage`.
authorDesc: | 
  Apasionado por programación y el mundo del software, empecé en esto hace muchos años con VB6 y tutoriales haciendo mi primera calculadora. Autodidacta nato, le debo prácticamente todo a la comunidad, que es de donde pude aprender todo lo que se hoy día. A lo largo de mi carrera he tocado muchos palos de diferentes ramas como la robótica, automatismo, visión artificial, servicios de alto rendimiento y actualmente he caído en la web y la nube. 

  Cuando no estoy trabajando, investigando alguna tecnología o colaborando en algún proyecto Open Source, me gusta jugar al balonmano, salir a pasear con mi pareja y mis perros, ir a nadar, ir al cine (casi como si no fuera un mago nivel 80 xD)… Y después de hacer todas esas cosas el poco tiempo que me queda lo dedico a crear contenido para ese pequeño rinconcito que tengo en internet que es mi blog.
socialOptions: # override params.toml file socialOptions
  email: ""
  facebook: ""
donationOptions:
  enable: false
date: 2020-08-03T18:00:00
draft: false
categories:
  - aks
  - k8s
---

# Como controlar un Application Gateway desde AKS

> Esta entrada está escrita por [Jorge Turrado](https://twitter.com/JorgeTurrado), un crack tanto en lo personal como en lo profesional. Es un placer que haya elegido mi blog para publicar esa entrada. Si quieres leer más cosas publicadas por Jorge, pásate por su blog: [Fixed Buffer](https://www.fixedbuffer.com/). ¡Gracias!

Es un gusto poder estar aquí en el blog del grandísimo Eduard Tomas, el cual me ha abierto sus puertas encantado para escribir sobre un tema que personalmente me ha resultado muy interesante y útil. Os pongo en situación.

En los últimos meses he participado en un proyecto con una arquitectura típica trabajando con Kubernetes, teníamos nuestro nginx ingress controller desde el que enrutabamos el tráfico hacia los diferentes servicios y de ahí hacia sus respectivos pods. Todo esto trabajando sobre la nube de Microsoft, es decir, en AKS. En un momento dado, se tomo la decisión de poner por delante del AKS un [Application Gateway][agw] (de aquí en adelante **agw**) que enrutase el tráfico de internet hacia el cluster. El escenario final con el que íbamos a trabajar era algo similar a esto:

!["La imagen muestra un diagrama donde se indica que las peticiones HTTP llegan al Application Gateway y se enrutan hacia el AKS llegando al Nginx ingress controller, y de ahí al servicio y a sus pods"][layout1]

> Por si no conocéis Application Gateway, es un servicio de Azure que hace de "proxy inverso" y permite ciertas configuraciones de seguridad. Esto es muy útil a la hora de exponer ciertas partes de una red virtual privada hacia internet teniendo un control sobre lo que esta expuesto. En este caso un AKS, pero también podrían ser diferentes máquinas virtuales,etc...

Por hacer un breve resumen, las peticiones llegan al agw y desde ahí se reenvían al cluster, nginx recibe la petición y la enruta hacia el servicio correspondiente, y este último hacia el pod que le toca. Hemos añadido una capa más de seguridad a nuestro cluster al colocarlo detrás de un agw, pero hemos añadido un salto de red más que si nuestro escenario expusiera directamente el nginx. Aunque esto puede ser totalmente irrelevante en la gran mayoría de los casos, el salto extra esta ahí y tiene su coste. 

Esto tiene una mejor solución (sino no estaría escribiendo esto), y consiste en utilizar el propio _ingress controller_ que ofrece Microsoft para trabajar directamente contra el agw. Este _ingress controller_ se llama "**A**pplication **G**ateway **I**ngress **C**ontroller" (de ahora en adelante **agic**). La manera de funcionar de este controlador consiste en analizar los objetos ingress que le pertenecen y realizar despliegues directamente sobre el agw. Con esto lo que vamos a conseguir es que el tráfico se enrute desde el agw directamente hacia los pods, evitándonos así 2 saltos de red. El nuevo planteamiento es algo similar a esto:

!["La imagen muestra un diagrama donde se indica que las peticiones HTTP llegan al Application Gateway y se enrutan directamente a los pods"][layout2]

> El [código fuente de agic][agic] esta disponible en GithHub para poder echarle una ojeada.

## Prerrequisitos

Contar esto es muy fácil, pero yo soy más de los que piensa que es mejor ver que oir, vamos a poner en marcha agic desde 0. Para eso, vamos a necesitar de varias cosas. Por supuesto un AKS y un Application Gateway (¿a qué nadie se lo imaginaba?), pero además también vamos a necesitar que ambos estén conectados a una red privada virtual (de aquí en adelante vnet) y dar permisos a agic para ver y editar el agw. 

> No es necesario que AKS y agw esten en la misma vnet, pueden estar desplegados en diferentes vnets sin ningún problema, pero en ese caso será necesario hacer los [peerings][peering] correspondientes entre las redes para que las peticiones puedan viajar desde agw hasta el cluster. 

Para los más veteranos en manejo de AKS, conoceréis que existen 2 tipos modos de manejar las identidades en un cluster de Kubernetes de Azure, utilizando un _service principal_ o una identidad manejada. Esto mismo es aplicable también a como dar permisos sobre agic, que soporta los dos escenarios. Personalmente soy de los que opinan que es mejor gestionar permisos desde una identidad manejada que desde un _service principal_, y como me gusta complicarme vida, es el modelo que vamos a utilizar aquí. Por tanto, la lista de cosas que vamos a necesitar en cuanto a infraestructura es:

* Grupo de recursos
* Red Privada Virtual
* AKS
* Application Gateway
* Asignación de permisos para las identidad manejada de AKS

Una vez que tenemos claro lo que necesitamos desplegar antes de meternos en harina dentro del cluster, vamos con ello. Lo primero que vamos a hacer es desplegar el grupo de recursos con:

```powershell
az group create --location westeurope --name agic-example
```

> **Disclaimer**: Los rangos de las redes, tamaños y nodos del cluster, etc... los he elegido muy arbitrariamente, en cada caso concreto es necesario evaluar los rangos necesarios. El único requisito clave es que el agw debe ser como mínimo *Standard_v2* y por tanto el _sku_ de la IP pui también debe ser _Standard_.

Sobre ese grupo de recursos vamos a desplegar una vnet con dos subredes (una para el AKS y otra para agw) ejecutando:

```powershell
az network vnet create -g agic-example -n agic-example-vnet --address-prefix 10.0.0.0/16
az network vnet subnet create -n aks-subnet --vnet-name agic-example-vnet -g agic-example --address-prefixes "10.0.0.0/17"
az network vnet subnet create -n agw-subnet --vnet-name agic-example-vnet -g agic-example --address-prefixes "10.0.128.0/24"
```

Lo siguiente va a ser desplegar ya el cluster de AKS, para ello basta con ejecutar:

```powershell
az aks create -g agic-example -n aks-cluster --enable-managed-identity --kubernetes-version 1.17.7 --docker-bridge-address "172.17.0.1/16" --service-cidr "10.1.0.0/16" --dns-service-ip "10.1.0.10" --vnet-subnet-id $(az network vnet subnet show -g agic-example -n aks-subnet --vnet-name agic-example-vnet --query "id") --network-plugin azure --zones 3 --node-count 3 --no-ssh-key --disable-rbac

az role assignment create --assignee $(az aks show --name aks-cluster -g agic-example --query "identity.principalId") --scope $(az network vnet subnet show -g agic-example -n aks-subnet --vnet-name agic-example-vnet --query "id") --role "Network Contributor"
```

> **Importante**: La asignación del rol "Network Contributor" es obligatoria al haber elegido el despliegue del cluster utilizando una identidad manejada en lugar de un _service principal_. En caso de que no se haga, el cluster tendrá fallos durante la operación.

Ya casí hemos terminado de desplegar infraestructura, solo nos queda la parte relativa al agw, la cual vamos a desplegar ejecutando:

```powershell
az network public-ip create -g agic-example -n agw-pip --sku Standard

az network application-gateway create --capacity 1 --http-settings-cookie-based-affinity Enabled --http-settings-port 80 --http-settings-protocol Http --location westeurope --name agw --public-ip-address $(az network public-ip show -g agic-example -n agw-pip --query "id") --resource-group agic-example --sku Standard_v2 --subnet agw-subnet --vnet-name agic-example-vnet
```

Si todo ha ido bien, deberíamos poder ir al portal de Azure y encontrarnos algo como esto:

!["La imagen muestra el grupo de recursos del portal de Azure donde se ven la vnet, el agw, la ip pública y el AKS"][infra]

## Requisitos

Después de un rato, ya tenemos la infraestructura lista para empezar a desplegar agic. Como decía antes, agic tiene 2 modos de autenticarse en Azure, utilizando un _service principal_ o útilizando una identidad manejada. El hecho de que elijamos un sistema u otro va a cambiar la manera de proceder. Si elegimos utilizar un _service principal_, vamos a tener que crearlo y asignarle los permisos. En caso de utilizar una identidad manejada, vamos a tener que crearla y asignarle los permisos. ¿Dónde cambia entonces el proceso? Pues en que si utilizamos una entidad manejada, vamos a necesitar añadir al cluster un proxy de autenticación para que esa identidad pueda trabajar. Esto lo vamos a conseguir desplegando antes de agic ese proxy que en nuestro caso va a ser [AAD-Pod-Identity][aad-pod-identity].

En resumen, vamos a necesitar:

* Una entidad manejada
* Asignación de permisos (tanto a la nueva identidad como al cluster)
* Desplegar AAD-Pod-Identity

Lo primero que vamos a hacer es crear nuestra nueva identidad manejada con el comando:

```powershell
az identity create -n identity -g agic-example
```

Ahora, son 5 las asignaciones de roles que tenemos que realizar:

1. La nueva identidad debe ser _Contributor_ de agw
2. La nueva identidad debe ser _Reader_ del grupo de recursos donde se encuentra agw
3. La identidad del cluster debe ser _Managed Identity Operator_ de la nueva identidad
4. La identidad de Kubelet debe ser contribuidor del grupo de recursos de los nodos
5. La identidad de Kubelet debe ser _Managed Identity Operator_ de la nueva identidad

> Aunque estos dos últimos puede parecer que no tienen sentido ya que la identidad del cluster ya tiene esos permisos, es necesario para que AAD-Pd-Identity pueda trabajar, para mas info os dejo la [issue en GitHub donde se habla del tema][aad-pod-indentity-roles-issue]

Teniendo claro el camino, vamos con ello. Para eso basta con ejecutar los siguientes comandos:

```powershell
# 1
az role assignment create --assignee $(az identity show -n identity -g agic-example --query "principalId") --scope $(az network application-gateway show -g agic-example -n agw --query "id") --role "Contributor"
# 2
az role assignment create --assignee $(az identity show -n identity -g agic-example --query "principalId") --scope $(az group show --name agic-example --query "id") --role "Reader"
# 3
az role assignment create --assignee $(az aks show --name aks-cluster -g agic-example --query "identity.principalId") --scope $(az identity show -n identity -g agic-example --query "id") --role "Managed Identity Operator"
# 4
az role assignment create --assignee $(az aks show --name aks-cluster -g agic-example --query "identityProfile.kubeletidentity.objectId") --scope $(az group show --name $(az aks show --name aks-cluster -g agic-example --query "nodeResourceGroup") --query "id") --role "Contributor"
# 5
az role assignment create --assignee $(az aks show --name aks-cluster -g agic-example --query "identityProfile.kubeletidentity.objectId") --scope $(az identity show -n identity -g agic-example --query "id") --role "Managed Identity Operator"
```

Ahora si que hemos terminado de desplegar todo lo que necesitamos en Azure :)

## Desplegar AAD-Pod-Identity en el cluster

Ya ha llegado la hora de trabajar en el cluster y vamos a empezar con AAD-Pod-Identity ya que es requisito para agic con el modelo de identidad manejada. Existen varias maneras de desplegar AAD-Pod-Identity en el cluster, pero para esta entrada vamos a optar por la que personalmente me parece más fácil, que es utilizando Helm directamente. Lo primero que vamos a necesitar es obtener el contexto para _kubectl_, esto lo vamos a conseguir simplemente utilizando _az aks get-cred
entials_

```powershell
az aks get-credentials -n aks-cluster -g agic-example
```

A partir de aquí, ya hemos sincronizado el contexto y basta con ejecutar:

```powershell
helm repo add aad-pod-identity https://raw.githubusercontent.com/Azure/aad-pod-identity/master/charts
helm install --set mic.leaderElection.namespace=aad-pod-identity \
             --set nmi.micNamespace=aad-pod-identity \
             --namespace aad-pod-identity --create-namespace
              aad-pod-identity aad-pod-identity/aad-pod-identity
```

> Los valores que configuramos simplemente son para que nuestro aad-pod-identity se despliegue en un namespace concreto en vez de el valor por defecto. Si por el contrario los valores por defecto son suficientes, bastaría con ejecutar `helm install aad-pod-identity aad-pod-identity/aad-pod-identity`

Podemos comprobar que esto ha funcionado ejecutando:

```powershell
kubectl get pods -n aad-pod-idenity
```

Lo que debería devolvernos algo parecido a esto:

```powershell
NAME                                    READY   STATUS    RESTARTS   AGE
aad-pod-identity-mic-6b49f5c66b-fffn5   1/1     Running   0          30m
aad-pod-identity-mic-6b49f5c66b-tgxk5   1/1     Running   0          30m
aad-pod-identity-nmi-gj729              1/1     Running   0          30m
aad-pod-identity-nmi-l5r6z              1/1     Running   0          30m
aad-pod-identity-nmi-rtqqj              1/1     Running   0          30m
```

## Desplegar Application-Gateway-Ingress-Controller

Ahora sí que hemos llegado al punto final, ya lo tenemos todo listo para poder desplegar agic y que funcione correctamente. Para poder desplegarlo simplemente tenemos que ejecutar:

```powershell
helm repo add application-gateway-kubernetes-ingress https://appgwingress.blob.core.windows.net/ingress-azure-helm-package/
helm install --set appgw.subscriptionId=<Suscripción> \
             --set appgw.resourceGroup=agic-example \
             --set appgw.name=agw \
             --set armAuth.type=aadPodIdentity \
             --set armAuth.identityClientID=$(az identity show -n identity -g agic-example --query "clientId") \
             --set armAuth.identityResourceID=$(az identity show -n identity -g agic-example --query "id") \
             --namespace agic --create-namespace \
             agic application-gateway-kubernetes-ingress/ingress-azure
```

> Los valores que estamos configurando son bastante explícitos en cuanto a su significado y que es lo que hay que asignar, pero por las dudas, dejo el [enlace a la descripción de los valores][helm-values].

Sí todo ha ido según se espera, deberíamos obtener una respuesta como esta al ejecutar

```powershell
kubectl get pods -n agic
```
```powershell
NAME                                  READY   STATUS    RESTARTS   AGE
agic-ingress-azure-78b9f88b5d-6q2v9   1/1     Running   0          63s
```

## Probando Application-Gateway-Ingress-Controller

Llegados a este punto, ya tenemos desplegado agic en nuestro cluster y ya solo nos queda probarlo. Antes de empezar con la prueba, es importante conocer que agic soporta anotaciones con configuraciones específicas para el ingress en concreto. Para más información sobre ellas, os recomiendo echarle un ojo a la [documentación][agic-annotations] ya que es un repositorio muy joven y constantemente recibe nuevas características.

Por último, vamos a desplegar una carga de trabajo simple como puede ser:

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: sample-app
  annotations:
    kubernetes.io/ingress.class: azure/application-gateway
spec:
  rules:
  - http:
      paths:
      - backend:
          serviceName: sample-app
          servicePort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-app
spec:
  selector:
    matchLabels:
      app: sample-app
  template:
    metadata:
      labels:
        app: sample-app
    spec:
      containers:
      - name: sample-app
        image: microsoft/dotnet-samples:aspnetapp
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 80
  replicas: 3
---
apiVersion: v1
kind: Service
metadata:
  name: sample-app
spec:
  selector:
    app: sample-app
  ports:
  - port: 80
    targetPort: 80
---
```

Una vez añadido al cluster, agic empezará automáticamente su preceso de actualización. Una vez termine (unos 10-20 segundos), podemos comprobar que el agw se ha configurado correctamente simplemente poniendo en el navegador la ip que hemos creado como ip pública del agw, la cual podemos obtener con el comando:

```powershell
az network public-ip show -g agic-example -n agw-pip --query "ipAddress"
```

Si todo ha ido bien, deberíamos encontrarnos con una web como esta:

!["La imagen muestra la web de ejemplo de ASP NET Core, no tiene ningún valor adicional ni contenido relevante"][result]

Con esto, hemos conseguido desplegar y validar nuestro agic :).

## Conclusión

Aunque parece un poco tedioso, agic ofrece una mejora en la gestión del tráfico de nuestras aplicaciones. Es cierto que para poder usarlo es necesario que el agw sea como mínimo un *Standard_v2* y que eso vale dinero, por lo que no es habitual montarlo para usar agic. Eso no quiere decir que si ya lo tenemos, utilizar agic sea una opción muy interesante. 

Aquí solo he planteado el utilizarlo como ingress por evitar unos saltos de red, pero la realidad es que ofrece más cosas aparte como por ejemplo el decidir que ingress se sirven por la IP pública y cuales por la privada (aunque no por las dos a la vez), o por ejemplo la posibilidad de utilizar las métricas de agw como fuente para un HPA (**H**orizontal **P**od **A**utoscaler, autoescalado horizontal). 

Y con esto me despido, ha sido todo un placer poder compartir agic con todos vosotros, y estaré encantado de vovler para contaros más cosas si Eduard me lo permite :). Por lo pronto, muchas gracias a todos por leerme y a Eduard por dejarme estar aquí.

**¡¡Hasta la próxima!!**

[agw]:https://docs.microsoft.com/es-es/azure/application-gateway/overview
[agic]:https://github.com/Azure/application-gateway-kubernetes-ingress
[peering]:https://docs.microsoft.com/es-es/azure/virtual-network/virtual-network-peering-overview
[aad-pod-identity]:https://github.com/Azure/aad-pod-identity
[aad-pod-indentity-roles-issue]:https://github.com/Azure/aad-pod-identity/blob/v1.5.5/docs/readmes/README.msi.md
[helm-values]:https://github.com/Azure/application-gateway-kubernetes-ingress/blob/master/docs/helm-values-documenation.md
[agic-annotations]:https://github.com/Azure/application-gateway-kubernetes-ingress/blob/master/docs/annotations.md

[layout1]:/images/posts/2020-08-03-agw-nginx.png
[layout2]:/images/posts/2020-08-03-agw-agic.png
[infra]:/images/posts/2020-08-03-infra.png
[result]:/images/posts/2020-08-03-result.png
