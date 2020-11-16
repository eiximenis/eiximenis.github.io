---
title: "De StatefulSets y headless services"
author: eiximenis
description: "Qué relación hay entre los statefulsets y los headless services y algunos temas al respecto del DNS"
date: 2020-11-15
draft: false
categories:
  - kubernetes
tags: ["k8s"]
---

Los tipos más habituales de servicios en Kubernetes son los llamados _ClusterIP_ que tienen una IP privada solo accesible desde el interior del clúster. Asumiendo que CoreDNS está instalado (que es lo habitual, vamos), esos servicios generan una entrada DNS con su nombre, bajo el cual engloban todos los _pods_ que cumplan con el selector del servicio. Luego, tenemos los servicios de tipo _NodePort_ que además de todo lo anterior son accesibles mediante una llamada a los nodos del clúster con el puerto especificado y son pues la forma de exponer servicios al exterior. Y, finalmente, tenemos los servicios de tipo _LoadBalancer_, quienes, además de todo lo anterior, se usan para exponer el servicio a través de un balanceador (externo al clúster) y que suelen ser usados en Kubernetes que se ejecutan en la nube.

Luego, en otro orden de cosas, tenemos los servicios _ExternalName_ cuya utilidad es la de proporcionar un registro `CNAME` a un recurso que, por lo general, es exerno al clúster. Eso permite acceder a recursos externos (como bases de datos) a través del nombre de un servicio interno, lo que es útil en escenarios de migración o de varios entornos.

Y, por último, tenemos los servicios _headless_. Esos servicios son especiales, porque **no tienen ninguna IP** (el valor de `spec.ClusterIP` es `None`), aunque (a diferencia de los _ExternalName_) si que tienen _endpoints_ asociados (es decir, pueden seleccionar _pods_). La razón de su existencia es **cuando nos interesa acceder a pods de forma individual**, lo que suele ocurrir en las llamadas "aplicaciones con estado" donde puede ser necesario acceder a un pod en concreto dentro del conjunto. El ejemplo canónico de "aplicaciones con estado" son las bases de datos, que se suelen configurar en algún escenario tipo "primario/secundario", donde ambas bases de datos (en este caso pods) pueden servir lecturas, pero solo el primario puede gestionar escrituras. En este caso, para leer datos, nos da igual que pod nos atienda, pero para escribir necesitamos sí o sí, que nos atienda el primario.

## StatefulSets

Esas "aplicaciones con estado" tienen además otros requerimientos, como por ejemplo que se requiere que cuando se pongan en marcha los pods se creen en un orden establecido. Así, primero puede ser necesario levantar el primario y luego el secundario. Tenemos pues un caso que **no podemos tratar con un _deployment_ ya que ese trata a todos sus pods por igual y (entre otras cosas) los levanta todos a la vez y no les asigna identidad alguna**. Lo mismo aplica a la hora de eliminar pods: si tenemos un escenario de 4 pods (1 primario y 3 secundarios) y escalamos a 3 pods, debemos estar seguros que se eliminará un secundario. De nuevo un _deployment_ no nos garantiza eso, ya que para el _ReplicaSet_ asociado al _deployment_ todos los pods son idénticos y si debe eliminar uno, eliminará uno al azar.

Para gestionar esas necesidades especiales tenemos el _StatefulSet_. El _StatefulSet_ juega el mismo rol que el _deployment_, pero con casuísticas especiales para esas "aplicaciones con estado":

1. Los pods creados **obtienen un nombre predictivo**. Así en lugar de los sufijos "aleatorios" que genera el _deployment_, un _StatefulSet_ agrega sufijos con la cardinalidad del pod (`-0`, `-1`, etc). Ese nombre les da identidad a los pods.
2. Los pods se crean en un orden indicado (primero el `-0`, luego el `-1`, etc), y hasta que el anterior no está listo, el siguiente no se crea (eso último es opcional y se puede cambiar usando `statefulset.spec.podManagementPolicy`).
3. Los pods se destruyen en el orden inverso al creado.
4. El _StatefulSet_ puede proporcionar una _PVC_ a los pods creados (algo que el _deployment_ no puede hacer)

El punto 4 es opcional, **no es obligatorio que los pods gestionados por un _StatefulSet_ usen un PVC**.

Al margen de eso, no hay muchas otras diferencias entre un _deployment_ y un _StatefulSet_. Al igual que el _deployment_ **todos los pods de un _StatefulSet_ comparten el valor de `pod.spec`**, lo que significa que ejecutan la misma imagen con idéntica configuración. La diferencia está en que un pod puede saber si es el primero simplemente mirando el valor de su hostname (para ver si termina en `-0`). Eso permite aplicar configuraciones distintas y hacer que uno actúe de primario y el resto de secundario.

## Headless services y DNS

Un servicio _headless_ tiene muy pocas diferencias con un servicio ClusterIP. Para ver los ejemplos voy a crear dos pods idénticos:

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: pod1
  name: pod1
spec:
  containers:
  - image: dockercampusmvp/go-hello-world
    name: pod1
---
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: pod1
  name: pod2
spec:
  containers:
  - image: dockercampusmvp/go-hello-world
    name: pod1
```

Observa que ambos pods son idénticos y ejecutan la misma imagen. Vamos ahora a crear un servicio _ClusterIP_  y otro _headless_ para estos pods:

```
kubectl expose pod/pod1 --name cluster --port 80
kubectl expose pod/pod1 --name headless --port 80 --cluster-ip none
```

Eso crea ambos servicios con el selector `run: pod1` lo que afecta a ambos pods:

![Salida de kubectl mostrando los pods y los endpoints del servicio](/images/posts/2020-11-15-kubectl1.png)

Ahora ya podemos ejecutar una imagen de `dnsutils` y ver las entradas DNS del cluster:

```
$ kubectl run bb --image gcr.io/kubernetes-e2e-test-images/dnsutils:1.3 --restart Never -it --rm /bin/sh
If you don't see a command prompt, try pressing enter.
/ # nslookup cluster
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   cluster.default.svc.cluster.local
Address: 10.109.227.151

/ # nslookup headless
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   headless.default.svc.cluster.local
Address: 172.17.0.3
Name:   headless.default.svc.cluster.local
Address: 172.17.0.4

/ # exit
pod "bb" deleted
$
```

Aquí **podemos observar la diferencia fundamental entre un servicio _ClusterIP_ y uno _headless_**: en el primer caso su entrada DNS se corresponde a la ClusterIP del servicio (en mi ejemplo la `10.109.227.151`), mientras que en el segundo, hay tantas entradas DNS como endpoints tiene el servicio y cada entrada tiene el valor de la IP del pod asociado (las IPs `172.17.0.3` y `172.17.0.4` son las IPs de los dos pods).

Ahora bien, es una creencia común **el pensar que un servicio _headless_ genera entradas DNS adicionales por cada pod asociado**. No. Un servicio _headless_ genera una sola entrada DNS (la del propio servicio), al igual que lo hace un servicio _ClusterIP_, pero esa entrada tiene varias IPs (una por cada pod), en lugar de la IP del servicio (como ocurren en un _ClusterIP_). Ese comportamiento nos permite **lanzar una consulta a la entrada DNS del pod y obtener las IPs de los pods asociados**.

Bien, pero muchas veces **eso no es suficiente**. Por ejemplo si tenemos que establecer una cadena de conexión contra un pod en concreto de una base de datos (por ejemplo el primario), nos interesa tener un DNS directo contra ese pod y así poder poner una cadena de conexión determinada en un _secret_. Así querríamos tener entradas DNS del tipo `pod1.headless` para el primer pod y `pod.headless` para el segundo.

## Especificando hostname y subdomain de los pods

Es posible hacer que Kubernetes nos genere esos DNS, pero eso es independiente de que el servicio usado sea _headless_ o no. **Lo que ocurre es que en el contexto en que se generan esas entradas DNS, se suelen usar servicios _headless_ y eso hace que mucha gente lo confunda**. Vamos a verlo.

Básicamente para que Kubernetes genere esas entradas DNS para los pods deben cumplirse **tres condiciones**:

1. El pod debe tener definido `pod.spec.hostname` con algun nombre (preferiblemente igual a `metadata.name`, aunque no es obligatorio)
2. El pod debe tener definido `pod.spec.subdomain` **con el nombre de un servicio**
3. Debe haber un servicio que tenga a ese pod entre sus endpoints

Vamos a ver un ejemplo. Para ello modifico el _pod1_ para que quede así:

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: pod1
  name: pod1
spec:
  hostname: pod1             # Añadimos esa línea
  subdomain: cluster         # Añadimos esa línea
  containers:
  - image: dockercampusmvp/go-hello-world
    name: pod1
```

Observa que **asocio el pod al servicio `cluster`, a través de `pod.spec.subdomain`**. Si ahora ejecutamos de nuevo `dnsutils` podemos ver como la entrada para el pod `pod1` se ha generado, pero no para el `pod2`:

```
$ kubectl run bb --image gcr.io/kubernetes-e2e-test-images/dnsutils:1.3 --restart Never -it --rm /bin/sh
If you don't see a command prompt, try pressing enter.
/ # nslookup cluster
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   cluster.default.svc.cluster.local
Address: 10.109.227.151

/ # nslookup pod1.cluster
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   pod1.cluster.default.svc.cluster.local
Address: 172.17.0.5

/ # nslookup pod2.cluster
Server:         10.96.0.10
Address:        10.96.0.10#53

** server can't find pod2.cluster: NXDOMAIN

/ # exit
pod "bb" deleted
$
```

El servicio `cluster` sigue siendo un _ClusterIP_ (y su entrada DNS solo nos da la IP del servicio), pero gracias al uso de `pod.spec.subdomain` se genera la entrada DNS asociada al pod.

## Headless services y StatefulSets

La causa de esa confusión de qué los servicios _headless_ generan entradas DNS con la forma `nombre-pod.nombre-servicio` se debe a que los servicios _headless_ se usan asociados a los _StatefulSets_.

Es lógico que sea así: suele ser en "aplicaciones con estado" cuando nos interesa acceder a un _pod_ en concreto (por lo tanto nos puede interesar tener esas entradas DNS) y ya hemos visto que usamos _StatefulSets_ para desplegar dichas aplicaciones. Pero quien nos da esas entradas DNS no es el servicio _headless_, si no el hecho de que **el _StatefulSet_ genera automáticamente los valores correctos para `pod.spec.hostname` y `pod.spec.subdomain`**.

De hecho es posible asociar un _StatefulSet_ a un servicio _ClusterIP_ sin problemas. Así, si el valor de `statefulset.spec.serviceName` es el nombre de un servicio _ClusterIP_ ese servicio será usado:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: test
spec:
  serviceName: "cluster"
  replicas: 2
  selector:
    matchLabels:
      run: pod1
  template:
    metadata:
      labels:
        run: pod1
    spec:
      containers:
      - name: ctr
        image: dockercampusmvp/go-hello-world
        ports:
        - containerPort: 80
          name: http
```

Este _StatefulSet_ va a crear dos pods, y los va a asociar al servicio _cluster_ anterior (observa como `template.metadata` usamos `run: pod1` como selector, ya que ese es el selector del servicio _cluster_). Pues bien, borramos los pods `pod1` y `pod2`, y creamos este StatefulSet. Ahora tendremos dos pods (`test-0` y `test-1`) y ambos estaran bajo los endpoints de `cluster`:

```
$ kubectl get pods -o wide
NAME     READY   STATUS    RESTARTS   AGE    IP           NODE       
test-0   1/1     Running   0          3m6s   172.17.0.3   minikube   
test-1   1/1     Running   0          3m3s   172.17.0.4   minikube   

$ kubectl get endpoints
NAME         ENDPOINTS                     AGE
cluster      172.17.0.3:80,172.17.0.4:80   87m
headless     172.17.0.3:80,172.17.0.4:80   44m
kubernetes   192.168.49.2:8443             17d
```

Si ahora, de nuevo, ejecutamos `dnsutils` podemos ver como se crean entradas DNS para ambos pods:

```
$ kubectl run bb --image gcr.io/kubernetes-e2e-test-images/dnsutils:1.3 --restart Never -it --rm /bin/sh
If you don't see a command prompt, try pressing enter.
/ # nslookup test-0.cluster
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   test-0.cluster.default.svc.cluster.local
Address: 172.17.0.3

/ # nslookup test-1.cluster
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   test-1.cluster.default.svc.cluster.local
Address: 172.17.0.4

/ # nslookup test-0.headless
Server:         10.96.0.10
Address:        10.96.0.10#53

** server can't find test-0.headless: NXDOMAIN
```

Observa como tengo ambos pods en un DNS bajo `cluster` (el servicio _ClusterIP_) ya que ese es el servicio que he puesto en el _StatefulSet_. Y como dije antes el "culpable" de eso es el _StatefulSet_ (pongo la salida solo con los datos relevantes):

```
$ kubectl get pods -l run=pod1 -o yaml | grep -i 'hostname\|subdomain'
    hostname: test-0
    subdomain: cluster
    hostname: test-1
    subdomain: cluster
```

En resumen, que el responsable de que se generen esas entradas DNS es el propio StatefulSet, no el _headless_ service, aunque esta sea una confusión común :)

¡Un saludo!




