---
title: "Estableciendo un servidor de DNS propio en AKS"
author: eiximenis
description: "Usar un servicio manejado, como AKS, tiene muchas (muchísimas) ventajas, pero también algunos inconvenientes: algunas tareas no se realizan de la forma 'tradicional' si no que es necesario conocer la alternativa. Un ejemplo es como configurar el cluster para resolver ciertos dominios usando un servidor DNS propio."
date: 2020-06-13T12:30:00
draft: false
categories:
  - k8s
---

La situación es la siguiente: tienes un AKS y hay **determinados dominios** que deben servirse usando un servidor de DNS propio, no el que trae "AKS por defecto". Claro que... te has preguntado ¿como funciona la resolución DNS en Kubernetes en general y AKS en particular? 

## CoreDNS

Un servidor de Kubernetes suele incluir un servidor de DNS propio. Técnicamente no es una obligación, pero vamos, es lo habitual y AKS lo incluye de serie, así que lo vamos a dar por supuesto. Inicialmente se solía usar [kube-dns](https://github.com/kubernetes/dns) pero actualmente lo habitual es usar [CoreDNS](https://coredns.io/) que es el que usa AKS.

CoreDNS se ejecuta en el clúster como un pod, y se despliega usando los mecanismos habituales de Kubernetes. Se suele ejecutar en el espacio de nombres `kube-system`. Como digo, AKS lo trae instalado y configurado de serie, así que no debes hacer nada. Si ejecutas un `kubectl get pods -l "k8s-app=kube-dns" -n kube-system` te aparecerán los pods que ejecutan CoreDNS. Si sustituyes `get pods` por `get svc` te aparecerá el servicio (observa que escucha por el puerto 53). Por compatibilidad el servicio se llama  `kube-dns`, pero AKS viene con CoreDNS.

En un Kubernetes _on-premises_, CoreDNS se configura a través de un fichero, llamado `Corefile`, que se monta a partir de un volumen generado con un `ConfigMap` llamado `coredns`. Así, para configurar CoreDNS nos basta con crear este ConfigMap:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    // Contenido del fichero Corefile
```

AKS ya tiene ese ConfigMap incorporado y lo puedes observar con el comando `kubectl get cm coredns -n kube-system -o yaml`. Eso te permite observar como está configurado CoreDNS en un AKS de serie.

> Pero en **AKS no debemos tocar ese ConfigMap** (si lo hacemos el control plane de AKS lo sobreescribirá), en su lugar para lidiar con la configuración de CoreDNS debemos usar otro ConfigMap llamado `coredns-custom`. Lo que pongamos en este ConfigMap se mezclará con el que viene de serie y es ahí podemos "meter mano".

## El servidor DNS "por defecto" de AKS     

Por defecto en AKS el contenido del fichero `/etc/resolv.conf` de cada pod es similar a (salvo que en lugar de `default` habrá el espacio de nombres de cada pod):

```
nameserver 10.0.0.10
search default.svc.cluster.local svc.cluster.local cluster.local dcxprkdlaz2efcelurhplb4csc.ax.internal.cloudapp.net
options ndots:5
```

La IP `10.0.0.10` es la IP del servicio que ejecuta CoreDNS. Por otro lado la directiva `search` añade los sufijos a los nombres de dominio (que no tengan al menos 5 puntos). P. ej. si desde un pod ejecuto `ping coredns`, el log que obtengo de CoreDNS es como sigue:

```
[INFO] 10.37.18.32:42714 - 64031 "AAAA IN coredns.default.svc.cluster.local. udp 51 false 512" NXDOMAIN qr,aa,rd 144 0.000142912s
[INFO] 10.37.18.32:42714 - 63731 "A IN coredns.default.svc.cluster.local. udp 51 false 512" NXDOMAIN qr,aa,rd 144 0.000066706s
[INFO] 10.37.18.32:51066 - 60455 "A IN coredns.svc.cluster.local. udp 43 false 512" NXDOMAIN qr,aa,rd 136 0.000067306s
[INFO] 10.37.18.32:51066 - 60655 "AAAA IN coredns.svc.cluster.local. udp 43 false 512" NXDOMAIN qr,aa,rd 136 0.000070306s
[INFO] 10.37.18.32:39232 - 65138 "AAAA IN coredns.cluster.local. udp 39 false 512" NXDOMAIN qr,aa,rd 132 0.000073706s
[INFO] 10.37.18.32:39232 - 64938 "A IN coredns.cluster.local. udp 39 false 512" NXDOMAIN qr,aa,rd 132 0.000226019s
[INFO] 10.37.18.32:49379 - 54454 "AAAA IN coredns.dcxprkdlaz2efcelurhplb4csc.ax.internal.cloudapp.net. udp 77 false 512" NXDOMAIN qr,rd,ra 218 0.006296265s
[INFO] 10.37.18.32:49379 - 53954 "A IN coredns.dcxprkdlaz2efcelurhplb4csc.ax.internal.cloudapp.net. udp 77 false 512" NXDOMAIN qr,rd,ra 218 0.020023707s
[INFO] 10.37.18.32:48594 - 56153 "A IN coredns. udp 25 false 512" NXDOMAIN qr,rd,ra 131 0.00571869s
[INFO] 10.37.18.32:48594 - 56753 "AAAA IN coredns. udp 25 false 512" NXDOMAIN qr,rd,ra 131 0.007200918s
```

Observa como el hecho de resolver `coredns` intenta resolver antes `coredns.default.svc.cluster` y así para todos los prefijos. Claro, **que se añadan esos prefijos es lo que permite que podamos encontrar un servicio solo usando su nombre**, porque se añadirá el sufijo `.default.svc.cluster` que es realmente el FQDN del servicio (si se ejecuta en el espacio de nombres `default`). Los otros sufijos son necesarios para acceder a servicios de otros espacios de nombres, y finalmente está el misterioso sufijo `dcxprkdlaz2efcelurhplb4csc.ax.internal.cloudapp.net` que es el servidor interno DNS de Azure, que permite encontrar MVs que se estén ejecutando en la red. En mi ejemplo todas las respuestas son un `NXDOMAIN` porque el servicio `coredns` no se está ejecutando en el espacio de nombres `default`.

Ahora nos queda saber **qué servidor por defecto usa CoreDNS en AKS** para realizar sus consultas. Pues bien, en el ConfigMap `coredns` existe la siguiente configuración:

```
.:53 {
    forward . /etc/resolv.conf
}
```

Eso, básicamente, lo que hace es resolver todos los dominios usando el contenido de `/etc/resolv.conf`. Ahí es importante ver que coredns se ejecuta con `dnsPolicy: default` lo que básicamente significa que `/etc/resolv.conf` del contenedor es un enlace a `/etc/resolv.conf` del nodo que ejecute el contenedor.

## dnsPolicies en Kubernetes

Cuando se despliega un pod en Kubernetes se puede especificar que política de DNS (`spec.dnsPolicy`) sigue este pod:

* `default`: El pod hereda la resolución de nombres del nodo
* `ClusterFirst`: Cualquier petición de DNS es enviada al servidor upstream (es decir a CoreDNS) en lugar de usar la resolución de nombres del nodo. Este es el valor por defecto (ya, es curioso que haya una llamada `default` pero no sea la de por defecto).
* `ClusterFirstWithHostNet`: Que debe ser usada en aquellos pods que se ejecuten en `hostNetwork`
* `None`: Que significa que no hay resolución de DNS en el pod (en este caso el pod solo podrá resolver los nombres de dominio que se le pasen via `dnsConfig`).

Observa, pues que lo habitual es querer que todos los pods usen `ClusterFirst` y tener centralizada la configuración DNS en CoreDNS, pero el pod de CoreDNS debe usar `default` para terminar usando la resolución de nombres real del nodo donde se ejecuta.

## Resolución de nombres en el nodo en AKS

¿Y como está configurado un nodo en AKS, para resolver los DNS?

Pues bien, el fichero `/etc/resolv.conf` **de los nodos** es como sigue:

```
# Dynamic resolv.conf(5) file for glibc resolver(3) generated by resolvconf(8)
#     DO NOT EDIT THIS FILE BY HAND -- YOUR CHANGES WILL BE OVERWRITTEN
nameserver 168.63.129.16
search dcxprkdlaz2efcelurhplb4csc.ax.internal.cloudapp.net
```

Por un lado la directiva `search` nos agrega el sufijo interno de Azure, pero lo interesante es el `nameserver`, que apunta a una IP: `168.63.129.16`. Pues bien, [tal y como se indica aquí](https://docs.microsoft.com/en-us/azure/virtual-network/what-is-ip-address-168-63-129-16) esa IP es **una IP interna, accesible sólo desde MVs en Azure** y que se usa para varias cosas, entre ellas como servidor de DNS de Azure.

Así, al final, la petición llega a CoreDNS, pero CoreDNS usa la resolución de nombres del nodo (recuerda que se ejecuta bajo `dnsPolicy: default`), por lo que al final se termina ejecutando la resolución de nombres de Azure (bueno... como era de esperar xD).

Y ahora que ya tenemos todo ese _background_ os paso a comentar como podéis añadir que determinados dominios se resuelvan mediante un servidor DNS propio.

## Resolver subdominios específicos con un servidor DNS propio

En mi caso la necesidad que tenía era que todos los dominios tipo `*.internal.name` debían servirse usando un DNS propio del cliente. El resto de dominios se podían servir con el DNS que lleva de serie AKS (luego entraremos ahí). 

Como dije al principio, para configurar CoreDNS en AKS se debe usar el configmap `coredns-custom`. En el caso que nos ocupa el ConfigMap queda como:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns-custom
  namespace: kube-system
data:
  mycustom.server: |
    internal.name:53 {
      forward . x.x.x.x
    }
```

El nombre `mycustom.server` puede ser cualquier cosa con la condición que termine en `.server`. En este bloque usamos la sintaxis propia de CoreDNS para enviar la petición de dominio actual al DNS que está en `x.x.x.x` (esta es la IP de nuestro servidor de DNS propio). El resto de dominios se seguirán sirviendo con el servidor de DNS propio de AKS.

Simplemente debes aplicar este ConfigMap y luego reiniciar CoreDNS (`kubectl rollout restart deployment coredns -n kube-system`) y listos.

Espero que te haya sido interesante!

PD: Tienes muchas otras [opciones de configuración de CoreDNS descritas en la documentación de AKS](https://docs.microsoft.com/en-us/azure/aks/coredns-custom).