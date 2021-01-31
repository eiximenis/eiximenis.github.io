---
title: "AKS, affinity zones y volúmenes"
author: eiximenis
description: "Vamos a contar cuatro cosillas que he visto sobre AKS y affinity zones al usar volúmenes"
date: 2021-01-31
draft: false
categories:
  - aks
  - kubernetes
tags: ["azure", "backup", "k8s"]
---

Es posible crear un AKS cuyos nodepools tengan más de una zona de afinidad. Esto asegura que distintos nodos del cluster están físicamente separados en zonas distintas dentro de la misma región, lo que añade redundancia: si una zona se cae, el cluster sigue funcionando. En la documentación se mencionan algunas limitaciones (no todas las regiones lo soportan u el tamaño de MV del pool debe estar disponible en todas las zonas de afinidad).

> Eso sí, sólo puedes establecer las zonas de afinidad del nodepool cuando lo creas, **una vez creado no se pueden modificar**

Sobre el papel eso es fantástico: todo lo que sea añadir tolerancia a fallos es bueno, pero ¡ojo! que viene con un precio a pagar, y es que **solo se pueden enlazar discos de la misma zona de afinidad en la que está un nodo**. Eso significa que, si tienes un _pod_ que se ejecuta en un nodo que está en la zona de afinidad 1, todos sus _pods_ solo podrán enlazar discos que estén en esa misma zona de afinidad. Los nodos están marcados con las etiquetas `topology.kubernetes.io/region` y `failure-domain.beta.kubernetes.io/region` indicando la región y las etiquetas  `topology.kuberbetes.io/zone` y `failure-domain.beta.kubernetes.io/zone` indicando la zona de afinidad (hay dos etiquetas para lo mismo, porque la de `failure-domain` es vieja y está obsoleta, pero se mantiene por compatibilidad).

## Zonas de afinidad y PVs autoaprovisionados

¿Qué sucede cuando creas un PV autoaprovisionado, a partir de un PVC? Recuerda que, en este caso, el PV es creado **automáticamente por el PVC**. Para comprobar qué ocurre he creado 4 PVCs y he esperado a que se hayan creado los 4 PVs subyacentes:

```
> kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS      REASON   AGE
pvc-6a66df7a-e248-4a92-b02b-63693ebcb939   5Gi        RWO            Delete           Bound    default/pvc3          managed-premium            84s
pvc-ad543861-d4aa-4b94-9174-eccb4367af74   5Gi        RWO            Delete           Bound    default/pvc1          default                    84s
pvc-c3a2db5e-b014-43df-961a-ef46fdfe4309   3Gi        RWO            Delete           Bound    default/pvc4          managed-premium            84s
pvc-ff0f10c0-d996-42a3-908a-143bdeb38749   8Gi        RWO            Delete           Bound    default/pvc2          default                    84s
```

En mi grupo de recursos `MC_*` tengo efectivamente los discos ya creados (recuerda, la creación del PV y por lo tanto del disco de Azure es tan buen punto se crea el PV asociado al PVC). Pero, en qué zona de afinidad está cada disco?

```powershell
$aks = "<nombre-del-aks>"
$rg = "<rg-del-aks>"
$rgmc=$(az aks show -n $aks -g $rg -ojson | ConvertFrom-Json).nodeResourceGroup
az disk list -g $rgmc --query '[][name,zones[0]]'
```

La salida en mi caso es:

```json
[
  [
    "kubernetes-dynamic-pvc-6a66df7a-e248-4a92-b02b-63693ebcb939",
    "2"
  ],
  [
    "kubernetes-dynamic-pvc-ad543861-d4aa-4b94-9174-eccb4367af74",
    "1"
  ],
  [
    "kubernetes-dynamic-pvc-c3a2db5e-b014-43df-961a-ef46fdfe4309",
    "2"
  ],
  [
    "kubernetes-dynamic-pvc-ff0f10c0-d996-42a3-908a-143bdeb38749",
    "1"
  ]
]
```

En este caso dos de los discos se han creado en la zona de afinidad 1 y dos más en la zona de afinidad 2. La información de la zona de afinidad del disco se encuentra también en el PV asociado, ya que este declara una `nodeAffinity` para asegurar que los pods que usen dicho PV se ejecuten en un nodo que esté en esta misma zona de afinidad:

```
> kubectl describe pv pvc-ad543861-d4aa-4b94-9174-eccb4367af74
Name:              pvc-ad543861-d4aa-4b94-9174-eccb4367af74
Labels:            failure-domain.beta.kubernetes.io/region=westeurope
                   failure-domain.beta.kubernetes.io/zone=westeurope-1
Annotations:       pv.kubernetes.io/bound-by-controller: yes
                   pv.kubernetes.io/provisioned-by: kubernetes.io/azure-disk
                   volumehelper.VolumeDynamicallyCreatedByKey: azure-disk-dynamic-provisioner
Finalizers:        [kubernetes.io/pv-protection]
StorageClass:      default
Status:            Bound
Claim:             default/pvc1
Reclaim Policy:    Delete
Access Modes:      RWO
VolumeMode:        Filesystem
Capacity:          5Gi
Node Affinity:
  Required Terms:
    Term 0:        failure-domain.beta.kubernetes.io/region in [westeurope]
                   failure-domain.beta.kubernetes.io/zone in [westeurope-1]
Message:
Source:
    Type:         AzureDisk (an Azure Data Disk mount on the host and bind mount to the pod)
    DiskName:     kubernetes-dynamic-pvc-ad543861-d4aa-4b94-9174-eccb4367af74
    DiskURI:      /subscriptions/eabfc9fb-9bcb-433a-a8b8-4931ca652a93/resourceGroups/mc_velerotest_velerotest_westeurope/providers/Microsoft.Compute/disks/kubernetes-dynamic-pvc-ad543861-d4aa-4b94-9174-eccb4367af74
    Kind:         Managed
    FSType:
    CachingMode:  ReadOnly
    ReadOnly:     false
Events:           <none>
```

Observa como la _Node Affinity_ declara que este PV debe estar en un nodo que tenga la etiqueta `failure-domain.beta.kubernetes.io/zone` al valor `westeurope-1` (la zona de afinidad 1, que es la que corresponde al disco asociado a este PV).

Si ahora creo un pod que use dicho PV (a través del PVC asociado (`pvc1` en el ejemplo anterior)) el _Scheduler_ de Kubernetes se asegurará de que dicho pod se ejecute en un nodo que esté en la misma zona de afinidad. El resultado es que el pod queda asignado al nodo `aks-agentpool-75116936-vmss000000` que es el que está en la zona de afinidad 1:

```
> kubectl get po -o wide
NAME                         READY   STATUS    RESTARTS   AGE    IP            NODE                                NOMINATED NODE   READINESS GATES
nginxpvc1-5856997b55-mhjtq   1/1     Running   0          112s   10.240.0.22   aks-agentpool-75116936-vmss000000   <none>           <none>
```

¿Qué pasaría si, por cualquier motivo, este nodo no pudiera ejecutar el pod? Pues muy sencillo, se quedaría en _pending_:

```
> kubectl delete deploy nginxpvc1
deployment.apps "nginxpvc1" deleted
> kubectl taint node aks-agentpool-75116936-vmss000000 donothig:NoSchedule
node/aks-agentpool-75116936-vmss000000 tainted
> kubectl apply -f .\deploy-pvc1.yaml
deployment.apps/nginxpvc1 created
> kubectl get po
NAME                         READY   STATUS    RESTARTS   AGE
nginxpvc1-5856997b55-dxfdd   0/1     Pending   0          58s
```

Y si lanzas un `kubectl describe` en la sección de `EVENTS` te queda claro que el pod no puede ejecutarse porque el scheduler no puede encontrar ningún nodo:

```
Events:
  Type     Reason            Age   From               Message
  ----     ------            ----  ----               -------
  Warning  FailedScheduling  2m    default-scheduler  0/3 nodes are available: 1 node(s) had taint {donothig: }, that the pod didn't tolerate, 2 node(s) had volume node affinity conflict.
```

El `describe` deja claro que de los 3 nodos del cluster, hay 2 que tienen un "volume node affinity conflict" (o sea, no pueden ejecutar pods que usen el PV indicado) y otro tiene un _taint_. Así pues no hay nodos disponibles y el pod se quedará en _pending_...

## Zonas de afinidad y PVs estáticos

Si creas tu el PV vinculado a un disco pre-existente debes poner un `nodeAffinity` en dicho PV, ya que si no lo haces el scheduler puede desplegar el pod en cualquier nodo, y si el nodo está en otra zona de afinidad, no se podrá montar el disco. Veamos un ejemplo. Tengo un disco llamado `testvhd` creado en la zona de afinidad 3. Defino un PV asociado a dicho disco:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  finalizers:
  - kubernetes.io/pv-protection
  name: testvhd
spec:
  accessModes:
  - ReadWriteOnce
  azureDisk:
    cachingMode: ReadOnly
    diskName: testvhd
    diskURI: /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/MC_velerotest_velerotest_westeurope/providers/Microsoft.Compute/disks/testvhd
    kind: Managed
    readOnly: false
  capacity:
    storage: 8Gi
  claimRef:
    apiVersion: v1
    kind: PersistentVolumeClaim
    name: testvhd
    namespace: default
  persistentVolumeReclaimPolicy: Retain
  storageClassName: managed-premium
  volumeMode: Filesystem
```

Este PV está vinculado al disco indicado en `diskURI` y al PVC indicado en `claimRef.name`. Falta crear el PVC por supuesto:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: testvhd
spec:
  storageClassName: managed-premium
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 8Gi
```

Una vez aplicado el PVC, el PV quedará atado:

```
> kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS      REASON   AGE
testvhd                                    8Gi        RWO            Retain           Bound    default/testvhd       managed-premium            108s
```

Ahora ya podemos desplegar un pod que use el PVC `testvhd` y ver qué ocurre. Pues bien, pueden ocurrir dos cosas:

1. Que el pod se despliegue, por pura casualidad, en un nodo que está en la zona de afinidad del disco (la 3 en mi caso). En este caso el pod se ejecutará sin problemas. Por defecto el scheduler intenta repartir los pods entre las distintas zonas de afinidad.
2. Que el pod se despliegue, por pura casualidad, en un nodo que está en cualquier otra zona de afinidad que la del disco. En este caso el pod se quedará en `ContainerCreating` y en la sección de `EVENTS` verás algo como:

```
Events:
  Type     Reason              Age                From                     Message
  ----     ------              ----               ----                     -------
  Normal   Scheduled           72s                default-scheduler        Successfully assigned default/nginxpvctestvhd-5575bcddcb-bbjrh to aks-agentpool-75116936-vmss000001
  Warning  FailedAttachVolume  72s                attachdetach-controller  Multi-Attach error for volume "testvhd" Volume is already used by pod(s) nginxpvctestvhd-5575bcddcb-tgccb
  Warning  FailedAttachVolume  15s (x7 over 49s)  attachdetach-controller  AttachVolume.Attach failed for volume "testvhd" : Retriable: false, RetryAfter: 0s, HTTPStatusCode: 400, RawError: Retriable: false, RetryAfter: 0s, HTTPStatusCode: 400, RawError: {
  "error": {
    "code": "BadRequest",
    "message": "Disk /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/MC_velerotest_velerotest_westeurope/providers/Microsoft.Compute/disks/testvhd cannot be attached to the VM because it is not in the same zone as the VM. VM zone: '2'. Disk zone: '3'."
  }
}
```

Eso es porque a diferencia de los PVs que se crean automçáticamente, nuestro PV no tenía ninguna afinidad de nodo definida, por lo que el Scheduler no puede saber a qué nodo debe situar el pod. Así, pues debes usar `nodeAffinity` en la definición del PV:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  finalizers:
  - kubernetes.io/pv-protection
  name: testvhd
spec:
  accessModes:
  - ReadWriteOnce
  azureDisk:
    cachingMode: ReadOnly
    diskName: testvhd
    diskURI: /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/MC_velerotest_velerotest_westeurope/providers/Microsoft.Compute/disks/testvhd
    kind: Managed
    readOnly: false
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: failure-domain.beta.kubernetes.io/region
          operator: In
          values:
          - westeurope
        - key: failure-domain.beta.kubernetes.io/zone
          operator: In
          values:
          - westeurope-3        
  capacity:
    storage: 8Gi
  claimRef:
    apiVersion: v1
    kind: PersistentVolumeClaim
    name: testvhd
    namespace: default
  persistentVolumeReclaimPolicy: Retain
  storageClassName: managed-premium
  volumeMode: Filesystem
``` 

De este modo, cuando el scheduler deba ubicar un pod que use dicho PV, lo ubicará en un nodo que esté en la zona de afinidad 3 y todo funcionará correctamente.

## Conclusiones

Usar zonas de afinidad aumenta la tolerancia a errores en tu AKS, pero **añade una dimensión adicional en la topología de tu cluster**: una vez un PV queda ubicado en una zona de afinidad, cualquier pod que lo use deberá ser ubicado en dicha zona de afinidad, por lo tanto, también debes balancear bien las zonas de afinidad. P. ej. quizá no es buena idea tener un solo nodo por zona, porque en este caso, si este nodo se viene abajo, los pods no podrán ser reubicados en ningún otro nodo.

Nos ha quedado pendiente ver de qué manera podemos influir en qué zona de afinidad se crea el disco cuando usamos volúmenes autoaprovisionados. Pero, ya es bastante por hoy, ¡eso lo dejamos para otro post!