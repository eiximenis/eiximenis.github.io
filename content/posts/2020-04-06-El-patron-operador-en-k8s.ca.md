---
title: "El patró operador a Kubernetes"
author: eiximenis
description: Som-hi! Anem a començar la sèrie sobre com crear operadors per a Kubernetes. En aquest primer post veurem què és un operador de Kubernetes i quins usos té.
date: 2020-04-06T12:00:00
language: ca
series:
  - Operadors de Kubernetes
categories:
  - k8s
---

Abans de parlar sobre el com podem crear operadores, millor que veguem què és un operador, per a què el podem fer servir i quins avantatges ens aporta respecte altres mecanismes similars. Som-hi!

Un **operador és una forma de desplegar i mantenir aplicacions _per a_ Kubernetes**. La clau, aquí, és "_per a_". O sigui, no estem parlant de desplegar una aplicació _en un_ Kubernetes. Estem parlant de desplegar una aplicación _per a_ Kubernetes, i això implica que farem servir `kubectl` no nomès per al desplegament, si no també pel manteniment i la configuració. És a dir, l'aplicació s'instal·la, es manté i es configura fent servir la API de Kubernetes i totes les eines associades (començant per `kubectl`).

El concepte d'operador fou introduit per la gent de [CoreOS](https://coreos.com/) y es fonamenta en dos altres conceptes claus de Kubernetes, que hem d'entendre per poder comprendre com funcionen els operadors:

* Recursos: Un _recurs_ en Kubernetes és un objecte que defineix un estat desitjat. Així, el recurs `ReplicaSet` permet definir el nombre de instàncies d'un determinat _pod_ que es vol tenir sempre en execució.
* Controladors: Un _controlador_ per la seva banda s'encarrega de monitoritzar un _recurs_ i assegurar-se que l'estat real del clúster és compatible amb l'estat actual. En cas que això no sigui així, ha de prendre les accions necessàries per reconciliar els dos estats. Per exemple, el controlador de `ReplicaSet` s'encarrega de garantir que el nombre de _pods_ reals en execució és sempre el nombre definir en el recurs `ReplicaSet` associat, creant i destruint _pods_ sempre que calgui.

## Controladors a Kubernetes

Els recursos els gestionem amb els fitxers YAML, però els controlaors son codi que s'executa en el clúster. A vista d'ocell podem entendre un controlador com un "bucle etern" que realitza les següents accions:

1. Consultar l'estat actual del clúster
2. Si l'estat actual és el desitjat, no fer res.
3. Però si no, utilitzar la API de Kubernetes per a modificar l'estat del clúster

Una representació, un pel més acurada, es pot veure en el següent diagrama:

![Esquema d'un operador](/images/posts/2020-04-06-operator-diagram.png)

Aquest esquema, adaptat de la documentació oficial, mostra amb una mica més de detall com funciona un controlador (està basat en els components que defineix la llibreria [client-go](https://github.com/kubernetes/client-go)). Introdueixo aquests termes perque son rellevants en el cas que vulguis cercar més informació.

* El controlador es subscriu a determinats events de la API de Kubernetes per a monitoritzar creacions, eliminacions o actualitzacions de recursos.
* A cada tipus de recurs se li asigna un _informer_ que rebrà nomès els events del tipus de recurs indicat
* L'event es guarda en una cua de treball, mentres que l'estat del objecte es guarda en una cache
* Finalment la logica de reconciliació treballa amb l'event de la cua, la definició que està a la cache i la API de Kubernetes per realitzar les accions necessàries.

El diagrama anterior mostra un detall imporant: els operadors s'executen en _pods_, cosa que vol dir que instal·lar un operador consisteix simplement en desplegar fitxers YAML al clúster.

## Controladors vs Operadors

Tots els operadors fan servir el patró de controlador (i per tant en defineixen algun). Però no tots els controladors son operadors. Pots fer servir un controlador sense res més, quan necessitis reaccionar a un tipus de recurs, però els operadors van un pèl més enllà, ja que:

* Defineixen un o més controladors
* Defineixen un o més CRDs
* Gestionen el cicle de vida
* **Estan orientats a una aplicació**

L'últim punt és important: un operador està dedicat a una aplicació. Si aquesta aplicació requereix altres aplicacions, hauries de fer servir altres operadors, per aquestes aplicacions. Per exemple, pots tenir un operador encarregat d'instal·lar un, posem pel cas, wordpress. Però si wordpress depèn de mysql, hauries d'usar un altre operador pel mysql.

Els altres tres punts posen de manifest les diferències entre operadors i controladors: un operador és un (o mès d'un) controlador amb un conjunt de CRDs que permeten configurar i mantenir la aplicació fent servir la API de Kubernetes. I per acabar: un operador sol mantenir el cicle de vida de l'aplicació que gestiona, creant i destruint els _pods_ necessaris.

Agafem d'exemple l'[operador de Prometheus](https://github.com/coreos/prometheus-operator). Aquest operador instal·la varis CRDs, dels quals en destaquen un anomenat `monitoring.coreos.com/v1/Prometheus` que representa al propi [Prometheus](https://prometheus.io/) i un altre que és `monitoring.coreos.com/v1/ServiceMonitor` que defineix quins serveis han de ser monitoritzats (per prometheus). Si instal·les l'operador de Prometheus, no has de fer servir cap _deployment_ per instal·lar un Prometheus al teu clúster: nomès has de crear un CRD de tipus Prometheus. La imatge que vé a continuació mostra un Prometheus (de nom `k8s`) desplegar a l'espai de noms `monitoring`:

![Resultat de la comanda kubectl get prometheus](/images/posts/2020-04-06-kubectl-get-prometheus.png)

Fixa't com el propi CRD de Prometheus defineix les repliques (2 en aquest cas) i és el propi operador el que s'encarrega de crear i destruir els _pods_ que executen Prometheus (no hi ha cap _deployment_ ni _replicaset_).

## Operadors vs Helm

No se suposa que per instal·lar aplicacions a Kubernetes disposem de [Helm](https://helm.sh/)? Hi ha un cert solapament entre algunes de les responsabilitats d'un operador i les de Helm: en ambdós casos tenim mecanismes per a gestionar _workloads_ complexos. Si instal·lar la teva aplicació és un procés d'un sol pas i no requereixes que aquesta es gestioni a través d'objectes de Kubernetes, llavors Helm és el que hauries de considerar. Per altra banda, si la teva aplicació requereix aquest tipus de gestió (a través d'objectes de Kubernetes), llavors hauries de pensar en crear un operador. I tingues present que, com que desplegar un operador consisteix en desplegar els seus fitxers YAML, no hi ha res que t'impedeixi fer-ho amb Helm.

És bona pràctica instal·lar els CRDs que el teu operador defineixi, via YAML en comptes de fer-ho via codi al operador. La raó és per seguretat: desplegar un CRD requereix permisos d'àmbit de clúster, mentres que el teu operador, probablement, no necessita uns permisos tan elevats per fer la seva feina habitual. Però si instal·les els CRDs via codi, llavors has d'assignar permisos de clúster al _pod_ que executa l'operador, trencant el principi de [mínim privilegi](https://es.wikipedia.org/wiki/Principio_de_m%C3%ADnimo_privilegio). Així, pots definir tots els CRDs a un chart de Helm i que els instal·li juntament amb el _deployment_ que executi l'operador i la resta d'elements necessaris. Això sí, tingues present que has d'[indicar-li a Helm que instal·li els CRDs abans que la resta d'elements](https://helm.sh/docs/chart_best_practices/custom_resource_definitions/), ja que si no, tindràs problemes. Com fer això depèn de si uses Helm 2.x o 3.x.

## Conclusió

En aquest primer _post_ hem cobert el bàsic que necessites saber sobre operadors a Kubernetes. En els _posts_ que venen veurem com crear un operador i com desplegar-lo!


