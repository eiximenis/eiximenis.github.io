---
title: 'SF Mesh: Primeras impresiones'
description: 'SF Mesh: Primeras impresiones'
author: eiximenis

date: 2018-07-18T14:39:20+00:00
geeks_url: /?p=2150
geeks_ms_views:
  - 784
categories:
  - Sin categoría

---
Dentro de esta vorágine de productos relacionados con los contenedores en la que está inmersa Microsoft, ahora le toca el turno a [Service Fabric Mesh que salió hace nada en public preview][1]. En este post quiero comentaros un poco de qué va este producto y como ha ido mi (limitada) experiencia con él.
  
**Disclaimer:** Este es un post sobre un producto que está en _preview_. Lo expuesto aquí puede cambiar a medida que avance tanto el propio producto como mi experiencia en él.
  
<!--more-->


  
SF Mesh se anuncia como un producto _serverless_: eso es porque a diferencia de otros productos como AKS (Kubernetes) **no creamos clústers con los que luego operamos, si no que desplegamos aplicaciones**. La idea es que nos podemos olvidar del clúster real que está ejecutando nuestra aplicación. Ni lo vemos, ni tenemos por qué verlo. En contraste con productos como AKS, los cuales a pesar de ser administrados, tenemos siempre presente el concepto de &#8220;clúster&#8221; (y vamos, hasta podemos ver las VMs que lo componen).
  
**Service Fabric**
  
Siempre he pensado que Service Fabric es un producto que ha ido con el paso cambiado: salió antes de la fiebre de los contenedores y se ofrecía como un producto para la construcción de aplicaciones distribuídas altamente escalables. De hecho parte de los servicios de Azure están servidos por Service Fabric, lo que da muestra del potencial del producto. Digo que ha ido con el paso cambiado, porque cuando SF salió, esas arquitecturas eran una &#8220;minoría&#8221;: solo para determinados escenarios y muy lejos del _run-run_ habitual.
  
Poco después vino el hype de los microservicios, el auge de Docker y con él, la tiranía de Kubernetes. Justo en este momento, como pasa muchas veces en nuestro mundo, las arquitecturas distribuídas (mal llamadas muchas veces &#8220;de microservicios&#8221;) pasan a ser &#8220;de dominio público&#8221;. De repente da la sensación de que todo el mundo se cree Amazon o Netflix y se lanza a crear &#8220;microservicios&#8221;. Por supuesto, los de siempre para vender lo de siempre le dan volandas al tema y así llegamos a la situación actual: Las arquitecturas distribuídas se basan en contenedores, las [12 factor apps][2] se convierten en un modelo a seguir y todo eso pilla a SF fuera de ese mundo. Sí, es un orquestador, pero está fuera del mundo de los contenedores.
  
Cierto que le agregaron soportes para contenedores, pero recuerdo que cuando añadimos el soporte a SF en [eShopOnContainers][3] fue un dolor tremendo. Es cierto que parte del dolor venía de usar contenedores Windows, pero otra parte venía porque comparado con Kubernetes, SF era extremadamente limitado y/o complejo. Tampoco culparé al producto 100% de ello: nuestros conocimientos en SF no eran equivalentes a los de Kubernetes, pero nos dio la sensación de que en situaciones donde k8s ofrece soluciones estandarizadas y aceptadas por la comunidad (ingress y ingress controller sin ir más lejos), en SF te tocaba &#8220;hacértelo tu&#8221;.
  
SF sigue siendo un producto válido en sus escenarios, especialmente si vas a aprovechar sus modelos de aplicación que van más allá del propuesto por la [CNCF][4] y del modelo de 12 factor apps. En cualquier caso parece que la industria está yendo mayoritariamente por otro camino, en el cual SF no me parece que aporte demasiado.
  
**Service Fabric Mesh**
  
¿Y este SF Mesh en qué se diferencia de SF? Bueno, me centraré solo en los contenedores. De hecho **de momento SF Mesh solo soporta contenedores.**
  
Como dije la principal diferencia es que **desplegamos una aplicación**, por lo que en ningún momento &#8220;vamos a crear un clúster de Mesh&#8221;. Ese concepto desaparece. Lo primero que necesitamos es [configurar la CLI de Azure para instalar la extensión de SF Mesh][5].
  
El modelo de despliegue usando la CLI se **basa en plantillas ARM** lo que bueno... para mí es un poco _meh_,  porque si algo no tienen esas plantillas, es concisión. Pero no solo eso, el despliegue en ARM es síncrono lo que es un rollo patatero: que el despliegue no termina hasta que SF haya hecho p. ej. el pull de la imagen es algo que me irrita profundamente... especialmente en contenedores windows y sus imágenes de 15Gb. Por otro lado, al menos en el estado actual del producto, prepárate para despliegues que se quedan pillados, mensajes de errores crípticos, etc, etc.
  
**Desplegar una app usando las plantillas ARM**
  
&nbsp;
  
Una aplicación en Mesh se compone básicamente de dos elementos:

  * Una definición de red, donde definimos los _ingresses_ (es decir los accesos desde el exterior)
  * Una aplicación, vista como un conjunto de servicios, donde cada servicio se compone de uno o más contenedores (_code packages_ en la terminología de Mesh). Cada _code package_ expone uno o más _endpoints_ (vamos, puertos) y tiene su propia configuración. Cada servicio pide también los recursos que necesita para funcionar (tanta RAM, tantas CPUs).

Hasta donde he podido probar por ahora **debes desplegar tanto la red como la aplicación en la misma plantilla ARM **(todos los ejemplos los he visto, así y cuando lo he intentado separar las cosas no han funcionado).
  
Además, como los servicios forman parte de la definición del recurso de la aplicación, **todos los servicios deben ir en la misma plantilla ARM**. No sé, a mi me parece una chufa y de las gordas. Para muestra, un botón: aquí hay el código de la plantilla ARM que despliega la API de catálogo de eShopOnContainers. Esta API requiere rabbitmq y sql server para funcionar así que se incluyen también. En total son tres servicios (rabbit, sql y la API de catálogo):

<pre class="EnlighterJSRAW" data-enlighter-language="json">{
    "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json",
    "contentVersion": "1.0.0.0",
    "parameters": {
      "location": {
        "type": "string",
        "metadata": {
          "description": "Location of the resources."
        },
        "defaultValue": "eastus"
      }
    },
    "resources": [
      {
        "apiVersion": "2018-07-01-preview",
        "name": "eShopNetwork",
        "type": "Microsoft.ServiceFabricMesh/networks",
        "location": "[parameters('location')]",
        "dependsOn": [],
        "properties": {
          "addressPrefix": "10.0.0.4/22",
          "ingressConfig": {
            "layer4": [
              {
                "name": "catalogapiIngress",
                "publicPort": "5001",
                "applicationName": "eShopOnMesh",
                "serviceName": "catalogapi-svc",
                "endpointName": "catalogListener"
              }
            ]
          }
        }
      },
      {
        "apiVersion": "2018-07-01-preview",
        "name": "eShopOnMesh",
        "type": "Microsoft.ServiceFabricMesh/applications",
        "location": "[parameters('location')]",
        "dependsOn": [
          "Microsoft.ServiceFabricMesh/networks/eShopNetwork"
        ],
        "properties": {
          "description": "eShopOnContainers on mesh",
          "services": [
            {
              "type": "Microsoft.ServiceFabricMesh/services",
              "location": "[parameters('location')]",
              "name": "sqldata-svc",
              "properties": {
                "description": "SQL Server (sql-data svc)",
                "osType": "linux",
                "codePackages": [
                  {
                    "name": "mssql",
                    "image": "microsoft/mssql-server-linux:2017-CU8",
                    "endpoints": [
                      {
                        "name": "sqldataListener",
                        "port": "1433"
                      }
                    ],
                    "environmentVariables": [
                      {
                        "name": "ACCEPT_EULA",
                        "value": "Y"
                      },
                      {
                        "name": "SA_PASSWORD",
                        "value": "Pass@word"
                      }
                    ],
                    "resources": {
                      "requests": {
                        "cpu": "1",
                        "memoryInGB": "4"
                      }
                    }
                  }
                ],
                "replicaCount": "1",
                "networkRefs": [
                  {
                    "name": "[resourceId('Microsoft.ServiceFabricMesh/networks', 'eShopNetwork')]"
                  }
                ]
              }
            },
            {
              "type": "Microsoft.ServiceFabricMesh/services",
              "location": "[parameters('location')]",
              "name": "rabbitmq-svc",
              "properties": {
                "description": "RabbitMQ",
                "osType": "linux",
                "codePackages": [
                  {
                    "name": "rabbitmq",
                    "image": "rabbitmq:3-management-alpine",
                    "endpoints": [
                      {
                        "name": "rabbitListener",
                        "port": "5672"
                      },
                      {
                        "name": "rabbitManagementListener",
                        "port": "15672"
                      }
                    ],
                    "environmentVariables": [
                      {
                        "name": "ACCEPT_EULA",
                        "value": "Y"
                      },
                      {
                        "name": "SA_PASSWORD",
                        "value": "Pass@word"
                      }
                    ],
                    "resources": {
                      "requests": {
                        "cpu": "1",
                        "memoryInGB": "1"
                      }
                    }
                  }
                ],
                "replicaCount": "1",
                "networkRefs": [
                  {
                    "name": "[resourceId('Microsoft.ServiceFabricMesh/networks', 'eShopNetwork')]"
                  }
                ]
              }
            },
            {
              "type": "Microsoft.ServiceFabricMesh/services",
              "location": "[parameters('location')]",
              "name": "catalogapi-svc",
              "properties": {
                "description": "Catalog API Service.",
                "osType": "linux",
                "codePackages": [
                  {
                    "name": "catalog-api",
                    "image": "eshop/catalog.api",
                    "endpoints": [
                      {
                        "name": "catalogListener",
                        "port": "80"
                      }
                    ],
                    "environmentVariables": [
                      {
                        "name": "ASPNETCORE_ENVIRONMENT",
                        "value": "Development"
                      },
                      {
                        "name": "ASPNETCORE_URLS",
                        "value": "http://0.0.0.0:80"
                      },
                      {
                        "name": "ConnectionString",
                        "value": "Server=sqldata-svc;Database=Microsoft.eShopOnContainers.Services.CatalogDb;User Id=sa;Password=Pass@word"
                      },
                      {
                        "name": "PicBaseUrl",
                        "value": "http://localhost:5202/api/v1/c/catalog/items/[0]/pic/"
                      },
                      {
                        "name": "EventBusConnection",
                        "value": "rabbitmq-svc"
                      },
                      {
                        "name": "AzureStorageEnabled",
                        "value": "False"
                      },
                      {
                        "name": "AzureServiceBusEnabled",
                        "value": "False"
                      }
                    ],
                    "resources": {
                      "requests": {
                        "cpu": "0.5",
                        "memoryInGB": "1"
                      }
                    }
                  }
                ],
                "replicaCount": "1",
                "networkRefs": [
                  {
                    "name": "[resourceId('Microsoft.ServiceFabricMesh/networks', 'eShopNetwork')]"
                  }
                ]
              }
            }
          ]
        }
      }
    ]
  }</pre>

Es evidente que este mecanismo **no es adecuado para grandes aplicaciones**. Imagina si tienes que mantener en un único JSON la configuración de, pongamos, 10 servicios... es sencillamente imposible de mantener.
  
La idea de que la aplicación Mesh sea un &#8220;recurso de Azure&#8221; y que por lo tanto la despleguemos usando ARM no está mal conceptualmente, pero debemos tener en cuenta que una aplicación tiene necesidades de despliegue distintas a las de infraestructura: se actualiza mucho y de forma parcial. Es cierto que ARM permite actualizar, pero siempre debo usar la plantilla del recurso entera. Un ejemplo de lo que quiero decir lo podñeis ver en la propia documentación de Mesh, cuando hablan de [escalar servicios][6]. Primero nos dicen que nos instalemos una app usando una plantilla ARM llamada [mesh_rp.base.linux.json][7]. Luego nos dicen que para escalar un servicio apliquemos la plantilla [mesh_rp.scaleout.linux.json][8]. Bien, si las miras **verás que son la misma plantilla ARM donde solo cambia un campo**. De verdad, para escalar un servicio ¿tengo que volver a redesplegar un json **con toda la definición de la aplicación?** No es para nada operativo.
  
Tal y como yo lo veo hay varias posibles soluciones a este problema:

  1. **Que distintas aplicaciones de Mesh puedan compartir una red de Mesh**. Eso permitiría desplegar cada servicio como una aplicación de Mesh, pero **al compartir la red de Mesh se podrían comunicar usando los endpoints internos**. Este esquema conceptualmente se parece mucho al de Kubernetes con Helm.
  2. **Que los servicios se puedan desplegar como recursos ARM** vinculados a una aplicación. De este modo podría tener una plantilla ARM por recurso y desplegar en cada caso los que necesite.
  3. Que la CLI soporte operaciones parciales (añadir un servicio, etc) a una aplicación. Entiendo que puede ser más complejo de lo que parece, ya que se tiene que obtener el ARM que hay desplegado para terminar mandándolo de vuelta.

Por ahora creo que ninguno de esos tres escenarios está soportado, aunque por lo que he leído parece que se optará por el primero.
  
**Problemas con los despliegues**
  
Por ejemplo, en un caso me ocurre lo siguiente. Creo un despliegue y al cabo de un rato (unos 10 minutos) termina con:
  
[<img class="alignnone size-full wp-image-2153" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/07/sf-mesh-error1.png" alt="Deploy termina diciendo &quot;updating&quot; y que mire el log" width="895" height="122" />][9]
  
Lanzo el comando que me indica (_az mesh show_) para ver el estado de la aplicación y esa es la salida:

<pre class="EnlighterJSRAW" data-enlighter-language="json">{
  "debugParams": null,
  "description": "eShopOnContainers running under SF Mesh",
  "diagnostics": null,
  "healthState": "Ok",
  "id": "/subscriptions/&lt;guid&gt;/resourcegroups/sf-mesh-edu/providers/Microsoft.ServiceFabricMesh/applications/eShopOnMesh",
  "location": "eastus",
  "name": "eShopOnMesh",
  "provisioningState": "Updating",
  "resourceGroup": "sf-mesh-edu",
  "serviceNames": [
    "catalogapi-svc"
  ],
  "services": null,
  "status": "Ready",
  "statusDetails": null,
  "tags": {},
  "type": "Microsoft.ServiceFabricMesh/applications",
  "unhealthyEvaluation": null
}</pre>

Si me voy al portal de Azure puedo ver mi despliegue que ahí sigue ejecutándose:
  
[<img class="alignnone wp-image-2162 size-full" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/07/sf-mesh-deploy-pillado.png" alt="Vista del deployment en el portal de Azure: 1h 37 min y ahí sigue" width="830" height="274" />][10]
  
&nbsp;
  
¿Qué le ha ocurrido a ese despliegue? ¡Buena suerte averiguándolo!
  
El primer paso es intentar obtener los logs del contenedor, con un comando exageradamente largo (_az mesh code-package-log get &#8211;app-name catalog-api &#8211;code-package-name catalog-api -g sf-mesh-edu &#8211;service-name catalogapi-svc &#8211;replica-name 0_) pero en determinados casos este comando te lanza una excepción (_&#8216;ClientRequestError&#8217; object has no attribute &#8216;response&#8217;_). Aunque no tengo muy claro el porque de este error, existe la posibilidad que sea porque el contenedor no se ha llegado a iniciar.
  
Otra opción es mirar como está el servicio que se desplegaba. En mi caso esta aplicación desplegaba un solo servicio (catalogapi-svc) así que usé el comando _az mesh service show &#8211;name catalogapi-svc &#8211;app-name catalog-api &#8211;resource-group sf-mesh-edu _(sí, los comandos de Mesh no son cortos, la verdad). Si esto tampoco te aclara nada, puedes mirar el estado de la replica (vendría a ser lo equivalente el pod en Kubernetes): _az mesh service-replica show &#8211;service-name catalogapi-svc &#8211;app-name catalog-api &#8211;resource-group sf-mesh-edu &#8211;replica-name 0_
  
En mi caso nada de esto me ha ayudado: al intentar obtener el estado de la replica 0 (los nombres de las replicas empiezan por 0 y van subiendo) obtengo la misma excepción que cuando obtenía los logs. Mi intuición me dice que el contenedor no se ha levantado, pero no tengo ninguna pista de por qué.
  
Por otro lado, no sé cual es la política de reinicios de Mesh, y no he visto nada en la documentación que lo especifique: si un servicio no puede inciarse por cualquier error... ¿qué se supone que ocurre?
  
**Despliegue con YAMLs**
  
También existe soporte para despliegue usando YAMLs pero **por ahora este soporte solo existe dentro del [tooling de Visual Studio][11]**, no de la CLI. Nos aparece un tipo nuevo de proyecto &#8220;Service Fabric Mesh Application&#8221;, que se traduce en un proyecto con extensión _sfaproj_. Por defecto se nos incluyen dos yamls dentro del proyecto: app.yaml y network.yaml que definen la aplicación y la red. Aquí un ejemplo del network.yaml:

<pre class="EnlighterJSRAW" data-enlighter-language="json">network:
  schemaVersion: 1.0.0-preview1
  name: MeshDemoNetwork
  properties:
    description: MeshDemoNetwork description.
    addressPrefix: 10.0.0.4/22
    ingressConfig:
      layer4:
        - name: TestWebIngress
          publicPort: 20007
          applicationName: MeshDemo
          serviceName: TestWeb
          endpointName: TestWebListener</pre>

Este proyecto _sfaproj_ viene acompañado con sus propias _targets_ de msbuild (Microsoft.VisualStudio.Azure.SFApp.Targets.target), una de las cuales lo que hace es **buscar todos los proyectos que sean de SF Mesh en la misma solución**. Para ello creo que se fija en que el csproj tenga referenciado un paquete NuGet llamado _Microsoft.VisualStudio.Azure.SFApp.Targets_ y además tenga la propiedad:

<pre class="EnlighterJSRAW" data-enlighter-language="xml">&lt;IsSFAppServiceProject&gt;true&lt;/IsSFAppServiceProject&gt;</pre>

Si es el caso, entonces busca un fichero, dentro del proyecto, llamado &#8220;Services Resources\service.yaml&#8221; donde hay la definición YAML del recurso.
  
Cuando usamos la opción de _publish_ **el tooling de Visual Studio traduce todos los YAMLs al JSON de ARM**, construye las imágenes a partir del Dockerfile, las publica en un ACR y luego publica el ARM de la aplicación Mesh.
  
El tooling de Visual Studio tiene varias limitaciones:

  1. Para depurar solo soporta contenedores Windows
  2. Para publicar soporta tanto Linux como Windows, aunque si la imagen es multiarch, asume que es Windows. P. ej. si usas como imagen base la del runtime de netcore ([_microsoft/dotnet:2.1-aspnetcore-runtime_][12]) te dará un error al publicar diciendote que estás usando una imagen Windows 1803 (versión no soportada en SF Mesh), aunque tus servicios sean de tipo Linux. Eso es porque esa imagen es multiarch y existe tanto en Windows como en Linux. Debes especificar en tu Dockerfile un tag de alguna versión Linux de la imagen (como p. ej. _microsoft/dotnet:2.1-aspnetcore-runtime-alpine_).
  3. Todos los proyectos que conformen tu aplicación Mesh deben estar en la misma solución (sln)

Si publicas con VS y luego con el portal de Azure vas al grupo de recursos puedes verás el despliegue de tu aplicación:
  
[<img class="alignnone size-large wp-image-2159" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/07/deployment-vs-1024x73.png" alt="Deployment ARM hecho por VS" width="660" height="47" />][13]
  
Si lo abres verás los dos recursos ARM desplegados (la red y la aplicación):
  
[<img class="alignnone size-large wp-image-2160" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/07/deployment-vs2-1024x208.png" alt="Recursos ARM del despliegue" width="660" height="134" />][14]
  
Si vas a template verás la plantilla ARM template que ha generado el tooling de Visual Studio a partir de los ficheros YAML.
  
**Limitaciones actuales de Mesh y roadmap que han avanzado**
  
Una limitación actual de Mesh, es que los _ingresses_ de la red son nivel 4, lo que implica básicamente, que no podemos exponer dos servicios por el mismo puerto externo. Observa que en Kubernetes el controlador ingress es de nivel 7, y por lo tanto podemos exponer N servicios bajo el mismo puerto externo, diferenciándolos mediante ruta. En [esta issue][15] se menciona que están trabajando en un ingress de nivel 7 (según parece basado en [Envoy][16]).
  
Por ahora **no se pueden mezclar servicios Linux y Windows** en una misma aplicación, aunque es algo que está en el roadmap.
  
**Mi opinión**
  
Bueno... **Mesh es una buena idea pero solo es eso: una idea**. En su nivel de madurez actual sirve solo para jugar y poco más. Entiendo que en este mundo donde los anuncions van a golpe de gran evento de marketing hayan sacado la _preview_ pública, pero vamos... no está absolutamente listo para nada. **No es una beta, es que a duras penas llega a pre-alfa**. Además en mi opinión se han focalizado en cosas que no eran tan importantes (como el tooling de Visual Studio) en lugar de mejorar el _core_ del producto (que no ha cambiado tanto desde que lo empecé a probar hará cosa de 2-3 meses cuando estaba en _preview privada_).
  
Si quieres probarlo para empezar a ver como funciona, [échale un vistazo a la documentación][17], pero **no esperes a día de hoy poder desplegar algo más complejo que un Hello World**. La CLI debe mejorar bastante, ofrecer más comandos para las operaciones típicas del mantenimiento de una aplicación. También informar mejor de los errores, saber fácilmente si una réplica está corriendo o no y por qué es imprescindible.
  
**Nota: **Aquí tienes [otro post sobre Mesh][18], el entusiasmo del cual, como has podido deducir, no comparto ni de lejos. Es cierto insisto **que el producto está en preview y que solo puede que mejorar**, así que lo iremos siguiendo a ver como va evolucionando... Ya os contaré.

 [1]: https://azure.microsoft.com/en-us/blog/azure-service-fabric-mesh-is-now-in-public-preview/
 [2]: https://12factor.net/
 [3]: https://github.com/dotnet-architecture/eShopOnContainers/
 [4]: https://www.cncf.io/
 [5]: https://docs.microsoft.com/en-us/azure/service-fabric-mesh/service-fabric-mesh-howto-setup-cli
 [6]: https://docs.microsoft.com/en-us/azure/service-fabric-mesh/service-fabric-mesh-howto-app-scale-out
 [7]: https://sfmeshsamples.blob.core.windows.net/templates/visualobjects/mesh_rp.base.linux.json
 [8]: https://sfmeshsamples.blob.core.windows.net/templates/visualobjects/mesh_rp.scaleout.linux.json
 [9]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/07/sf-mesh-error1.png
 [10]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/07/sf-mesh-deploy-pillado.png
 [11]: https://docs.microsoft.com/en-us/azure/service-fabric-mesh/service-fabric-mesh-howto-setup-developer-environment-sdk
 [12]: https://hub.docker.com/r/microsoft/dotnet/
 [13]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/07/deployment-vs.png
 [14]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2018/07/deployment-vs2.png
 [15]: https://github.com/Azure/service-fabric-mesh-preview/issues/77
 [16]: https://www.envoyproxy.io/
 [17]: https://docs.microsoft.com/en-us/azure/service-fabric-mesh/
 [18]: https://www.codit.eu/blog/service-fabric-mesh-a-new-way-of-running-containerized-applications/