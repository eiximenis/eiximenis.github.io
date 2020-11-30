---
title: "Escenarios corporativos en AKS"
author: eiximenis
description: "Vamos a ver en este post varias características de AKS ideales para entornos corporativos"
date: 2020-11-30
draft: false
categories:
  - kubernetes
  - aks
tags: ["k8s", "aks"]
---

A medida que ha ido pasando el tiempo, AKS ha ido incorporando cada vez más mecanismos para ir cubriendo escenarios necesarios en entornos corporativos. Ese post muestra una lista y algunos ejemplos de esas características :)

## Clusters privados

Desde hace mucho tiempo es posible tener los nodos _worker_ en una vnet propia y privada, lo que impide acceso exterior a esos nodos. Los nodos _master_ (los que ejecutan el _control plane_) nunca han sido accesibles, pero sí **lo es el Api Server**. Al final el Api Server es lo que usa `kubectl`, lo que se traduce en que, si bien no tenemos acceso público (ni privado) a los nodos _master_, si hay acceso público al Api Server: cualquiera que sepa la URL del Api Server podrá acceder a nuestro cluster. Por supuesto el acceso está securizado por certificado, pero que esté ahí accesible abre posibles vectores de ataque, así como el problema con el que nos topamos si un fichero de configuración de `kubectl` es "encontrado" por alguien no autorizado.

Para evitar ese problema, AKS tiene dos opciones:

1. Filtrar las IPs que pueden acceder al Api Server. El Api Server sigue siendo público, pero un firewall impide el acceso a todas las IPs que no estén en una lista. Es parecido al modelo seguido por Azure SQL Server. En la documentación de AKS hay [más información](https://docs.microsoft.com/en-us/azure/aks/api-server-authorized-ip-ranges?WT.mc_id=AZ-MVP-4039791).
2. Usar un clúster privado. En este caso el Api Server obtiene una IP dentro de una vnet privada. El escenario en este caso es que los nodos _master_ siguen estando fuera de la suscripción de Azure del cliente (ese sigue sin tener acceso, ni pagar por ellos), pero se crea un [_Azure private link_](https://docs.microsoft.com/en-us/azure/private-link/private-link-service-overview?WT.mc_id=AZ-MVP-4039791) entre la red privada del Api Server y la vnet del cliente donde están los nodos _worker_.

La segunda opción es más segura que la primera, ya que el tráfico nunca abandona la red interna, pero requiere que los usuarios usen o bien una MV que pueda acceder a la IP privada del Api Server o bien una conexión de VPN configurada. Con la primera opción, pueden usar su propia máquina directamente con tal su IP esté en la lista de IPs permitidas.

## Identidad con AAD

Kubernetes nunca ha dispuesto de un mecanismo de gestión de identidad. En su lugar, se espera que los administradores lo conecten con algún sistema externo que permita obtener esas identidades. Luego, esas identidades se pueden referenciar usando [RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) y así establecer permisos a usuarios o a grupos de usuarios.

En AKS se puede usar Azure Active Directory, como mecanismo de gestión de identidad. De este modo los usuarios deberán autenticarse en AAD antes de poder acceder al clúster. Una vez autenticados con AAD se les aplicarán las reglas RBAC definidas en el clúster, de forma que cada usaurio obtendrá solo acceso a aquellos recursos que un administrador les haya especificado.

Si compartes un clúster entre varios entornos y/o equipos es casi obligatorio establecer reglas RBAC, para evitar que un desarrollador del proyecto X pueda acceder a recursos del proyecto Y, minimizando de esta manera el impacto de errores. El problema es que, sin un control de identidad asignado, es muy complicado que cada usuario obtenga su configuración. Podría llegar a conseguirse mediante el uso de [service accounts](https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/), pero eso implica un nivel de gestión muy elevado y de creación centralizada y distribución de los _kubeconfig_, que no termino de verlo. En estos contextos, necesitamos un servidor de identitdad, y usar AAD es la mejor opción en AKS.

Lo primero que necesitamos es, obviamente, ser un usuario con permisos en AAD para tareas de administración: crear usuarios, crear grupos y asignar usuarios a grupos. Para crear un clúster que se integre con AAD basta con usar dos parámetros en el comando `az aks create`:

* `--enable-aad`: Para habilitar la integración con AAD
* `--aad-admin-group-object-ids <object-id-del-grupo-de-admins>`: Para establecer un grupo de AAD cuyos usuarios son admins del clúster

Así, si quieres acceso administrativo al clúster, debes añadir tu usuario al grupo de administradores y de este modo podrás acceder a todos los recursos de éste. Por defecto el resto de usuarios no podrán acceder a nada, por lo que deberás darles permisos explícitos.

Esos permisos explícitos se los puedes dar usando el RBAC de Kubernetes. Estas reglas RBAC las puedes aplicar a usuarios en concreto (usando `kind: User` en el `RoleBinding.spec.subjects`) o a grupos (usando `kind: Group`). 

### Usando Azure RBAC

Eso está actualmente en preview, pero es posible usar el RBAC de Azure (no el de Kubernetes) para gestionar los permisos de los distintos usuarios/grupos que acceden al cluster. Actualmente eso debe habilitarse al crear el clúster, usando `--enable-azure-rbac`. Luego usamos `az role assignment` para asignar un rol a una entidad de AAD:

```bash
az role assignment create --role "Azure Kubernetes Service RBAC Admin" --assignee <AAD-ENTITY-ID> --scope <ID_AKS>/namespace/<namesapce>  # se puede usar <ID_AKS> solo si el permiso es global
```

Los roles los provee AKS, [en la documentación se describen los cuatro que vienen de serie](https://docs.microsoft.com/en-us/azure/aks/manage-azure-rbac#create-role-assignments-for-users-to-access-cluster?WT.mc_id=AZ-MVP-4039791):

* Azure Kubernetes Service RBAC Viewer: Solo lectura para la mayoría de objetos de un namespace
* Azure Kubernetes Service RBAC Writer: Como el anterior pero de lectura/escritura
* Azure Kubernetes Service RBAC Admin: Privilegios administrativos sobre un namespace o sobre todo el cluster
* Azure Kubernetes Service RBAC Cluster Admin: Acceso superadministrativo al cluster (todos los namespaces).

Podemos crear roles adicionales, usando `az role definition create` usando un fichero JSON con las [definiciones de permisos](https://docs.microsoft.com/en-us/azure/role-based-access-control/resource-provider-operations#microsoftcontainerservice?WT.mc_id=AZ-MVP-4039791) que puede hacer el rol.

La ventaja de usar Azure RBAC en lugar del RBAC propio de Kubernetes, es que toda la administración de usuarios y seguridad se realiza en Azure.

## Pod Identity

Una cosa es que los usuarios del cluster tengan identidad gestionada por AAD, pero también es posible que los propios pods la obtengan. Es decir, un pod puede ejecutarse con los permisos de una identidad de AAD. En general es bastante sencillo:

* Primero debe [instalarse pod identity en el cluster](https://azure.github.io/aad-pod-identity/docs/demo/standard_walkthrough/#1-deploy-aad-pod-identity).
* Después se crean las identidades en Azure (`az identity create`)

Una vez tenemos las identidades en Azure creadas, debemos vincularlas a los distintos pods. Para ello usamos dos CRDs:

* `AzureIdentity`: Representa una identidad de Azure dentro del clúster. Básicamente por cada identidad de Azure que queramos usar, crearemos uno de esos objetos. El campo `spec.resourceID` contiene el ID de la identidad en Azure, mientras que el campo `spec.clientID` contiene el clientid de la identidad. Finalmente existe un campo (`spec.type`) que indica el tipo de identidad (managed identity o service principal).
* `AzureIdentityBinding`: Representa el enlace entre una `AzureIdentity` y un conjunto de pods. Mediante ese objeto definimos un selector y aquellos pods que lo cumplan, usarán la identidad definida en el `AzureIdentity` referenciado por ese objeto. El campo `spec.azureIdentity` es el nombre (`metadata.name`) del objeto `AzureIdentity` referenciado, mientras que `spec.selector` es el valor que ejerce de selector.

Finalmente para que un pod use una identidad en concreto, debe tener una etiqueta llamada `aadpodidbinding` cuyo valor debe ser el definido en campo `spec.selector` del objeto `AzureIdentityBinding`.

De este modo, usando pod identity, nuestros pods pueden obtener automáticamente acceso a determinados recursos de Azure, sin necesidad de guardar contraseñas o tokens como secretos. El caso más típico es acceder a Key Vault desde un pod y a este escenario le dedicaré un post entero.

## Multi-tenant

Si despliegas varias aplicaciones en tu cluster, te puede interesar que esas estén aisladas en un modelo multitenant. Una aproximación multi-tenant en Kubernetes (AKS en este caso) debe abordarse desde varios puntos:

1. Scheduling (asignación de pods a nodos): Eso es importante para garantizar que _pods_ de según qué tenants se ejecutan en nodos específicos.
2. Reglas de seguridad de red: Evitar que _pods_ de tenants distintos puedan verse entre sí
3. Autenticación y autorización
4. Seguridad de los contenedores.

Vamos a describir, brevemente, cada punto con sus aproximaciones.

### Scheduling

Kubernetes da muchos mecanismos para sugerir, o directamente indicar, en qué nodo o tipo de nodos debe ejecutarse un _pod_:

* Asignar un tipo de nodos a un tipo de pods, para garantizar qué solo esos pods se ejecutaran en esos nodos. Eso requiere el uso de _taints_ y _tolerations_ trabajando conjuntamente.
* Forzar que un pod se ejecute en un tipo de nodos en concreto. Eso permite que _pods_ con ciertos requisitos (p. ej. peticiones de memoria altas) se ejecuten en nodos con gran cantidad de memoria. Para ello se suelen usar [nodeselectors](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#nodeselector).
* Asegurar que determinados pods nunca se ejecutan juntos (o bien lo contrario, que se prefiere que así sea). Para ello se suele usar las afinidades ([de nodo](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#node-affinity) o [entre pods](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#inter-pod-affinity-and-anti-affinity)).
* Garantizar que siempre exista un porcentaje de pods disponibles de un cierto _workload_. Para ello se usan PDBs ([pod disruption budgets](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/#pod-disruption-budgets)). Esos PDBs se aplican siempre y cuando un evento de "disrupción (eliminación de pods)" (ya sea voluntario o involuntario) afecte a los pods.

### Networking

Usar [networking policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/) para evitar que pods de distintos equipos/namespaces/proyectos se puedan mandar tráfico entre ellos. También para restringir el tráfico de entrada o salida de los pods. Para soportar esas políticas de red en AKS puedes usar o bien Azure policies o bien [Calico](https://docs.projectcalico.org/getting-started/kubernetes/). Hay algunas diferencias entre ellos como que por ejemplo el primero requiere usar Azure CNI (el networking avanzado de AKS) mientras que el segundo soporta tanto Azure CNI como kubenet (networking básico). En cualquier caso al crear el clúster puedes elegir qué opción prefieres. [Aquí hay más información](https://docs.microsoft.com/en-us/azure/aks/use-network-policies?WT.mc_id=AZ-MVP-4039791).

Otras opciones a considerar son las IPs de salida del clúster (_egress_ IPs) que te permiten especificar bajo qué IPs saldrá el tráfico de los pods hacia el exterior. Eso es importante cuando te integras con servicios de terceros que pueden requerir dar de alta esas IPs en algún tipo de lista. 

### Autenticación y autorización

Esos son los puntos que hemos visto antes:

* Usar AAD para autenticación
* Usar Kubernetes RBAC para autorización
* Usar Azure RBAC para autorización
* Usar pod identity para autorización de los pods

### Seguridad de los contenedores

Eso incluye desde usar [_pod security contexts_](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/), usar imágenes _rootless_, así como definir PSPs (pod security policies) que deben ser de obligado cumplimiento para los pods, para que esos sean aceptados.

En AKS además se puede usar el [Azure Policy Addon](https://docs.microsoft.com/en-us/azure/governance/policy/concepts/policy-for-kubernetes).

### Separación de clusters (logica vs física)

Muchas organizaciones crean "demasiados" AKS, cuando realmente AKS ofrece muchas herramientas para permitir separar un clúster físico en varios "clústeres lógicos":

* Aislamiento lógico: Usa namespaces para dividir entre entornos (dev, integración, qa, ...) y también entre equipos o proyectos. El uso de RBAC permite dar a los usuarios y a los pods mismos, solo acceso a aquellos recursos/namespaces requeridos. A su vez el uso de _networking policies_ permite restringir el tráfico de red entre pods de distintos propietarios/entornos.
* Nodepools: El soporte de nodepools permite tener distintos nodos en función de la necesidad de cada proyecto. De ese modo, un proyecto que use ML y CUDA puede ejecutarse en nodos con una GPU potente, y dejar el resto de nodos para otros proyectos que necesiten un H/W distinto. A su vez, eso permite mezclar workloads de Windows y Linux bajo el mismo cluster.

La ventaja de la separación lógica de clústeres es que te permiten aumentar la densidad de _pods_ por nodo, lo que en general redunda en una disminución de costes. El inconveniente es que, a pesar de los mecanismos de seguridad que se apliquen, el clúster en si mismo es la verdadera unidad de aislamiento. Para un nivel de aislamiento real entre tenants, hay que irse a una separación física de clústeres con lo que eso conlleva: menor densidad de _pods_ por nodo (mayor coste) y a la vez una mayor complejidad operacional.

## Conclusión

En resúmen, AKS tiene muchas opciones para clientes "corporativos", este post solo esboza las más importantes. Espero que te sea útil, en futuros posts iré desgranando la mayoría de esas opciones con ejemplos un poco más detallados :)

¡Saludos!