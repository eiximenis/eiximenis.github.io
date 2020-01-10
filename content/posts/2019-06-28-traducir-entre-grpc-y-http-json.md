---
title: Traducir entre gRPC y HTTP/JSON

author: eiximenis

date: 2019-06-28T16:22:05+00:00
geeks_url: /?p=2386
geeks_ms_views:
  - 685
categories:
  - asp.net core
  - docker
tags:
  - grpc

---
Como comenté en mi [post anterior sobre gRPC][1], la traducción entre gRPC y JSON es estándard. Esto nos permite tener nuestra comunicación interna en gRPC y exponer una fachada en HTTP con JSON para aquellos clientes que (todavía) no pueden usar gRPC.
  
En este post os voy a mostrar como podemos crear dicha fachada usando Envoy ejecutándose en un contenedor Docker. ¡Vamos allá!
  
<!--more-->


  
Dado que [Envoy][2] es una pieza clave en este post, dejadme que os lo presente. Envoy es un _reverse proxy_ de alto rendimiento escrito en C++ y que tiene soporte nativo para HTTP1.1, HTTP/2 y gRPC. En el caso de este post me interesa contaros como usando Envoy podemos simplificar la traducción de gRPC a HTTP/JSON. Por favor, tened presente **que eso no traduce gRPC a REST**. Eso es extremadamente difícil porque **RPC y REST parten de paradigmas de diseño opuestos**. Cierto es que sí &#8220;_restificas_&#8221; tu API gRPC, cuando la traduzcas a HTTP/JSON se parecerá más&#8221;a una API REST, pero para eso debes diseñar tu API gRPC teniendo esto en mente.
  
Vale, como en el post anterior voy a presuponer que tienes el &#8220;Hello World&#8221; de Microsoft funcionando. Nuestro amado GreeterService. Lo único es que mi cliente no es una aplicación de consola, si no una API que expone un controlador MVC que al llamarlo, llama al servidor por gRPC**. **Para levantar el ejemplo simplemente usad &#8220;docker-compose up server client&#8221; y el cliente se expondrá en localhost:9002. Navegad a localhost:9002/hello?name=xxxx y el cliente llamará al servidor via gRPC y mostrará la respuesta. El servidor muestra un log conforme se ha llamado al servicio gRPC.
  
[<img class="alignnone size-large wp-image-2387" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/06/grpc-log-server-1024x365.png" alt="Terminal donde se ve el log del servidor gRPC y como al usar cURL, el cliente llama al sevicio gRPC y aparece el log" width="660" height="235" />][3]
  
En este caso, el servidor solo expone un endpoint gRPC, pero vamos a usar Envoy para hacer este traslado.
  
Lo primero será añadir el contenedor de Envoy a nuestro docker-compose:

<pre class="EnlighterJSRAW" data-enlighter-language="json">envoy:
  image: envoyproxy/envoy</pre>

El siguiente punto **es configurar Envoy**. Para ello necesitamos compilar el fichero proto a su formato binario (.rb). Esta compilación se realiza usando _protoc, _que está desarrollado en Go.... pero antes de usar _protoc_ debemos hacer algunas modificaciones en nuestro fichero .proto.
  
Vayamos por partes.
  
**Configurar Envoy &#8211; Parte 1: Modificar el fichero .proto**
  
El fichero proto que hemos usado **no es válido ya que le falta información. **En efecto, le falta la información de que _endpoints_ de gRPC deben ser traducidos a HTTP/JSON y como se mapean las rutas. Esto, aunque podría ser configuración propia de Envoy, [se pone en el fichero .proto][4], porque hay una extensión de gRPC lo define así. Tiene lógica: el propio servicio gRPC define en su proto &#8220;como se transforma a HTTP&#8221;. Envoy usará esa información. En mi caso la definición del servicio en el .proto queda así (pongo solo la descripción del servicio):

<pre class="EnlighterJSRAW" data-enlighter-language="null">service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {
    option (google.api.http) = {
      get: "/api/v1/hello/{name}"
    };
  }
}</pre>

Enrutamos el método _SayHello_ a la ruta &#8220;/api/v1/hello/{name}&#8221;, donde  {name} se enlaza a la propiedad _name_ de la _HelloRequest_.
  
Para que eso funcione **hay que importar el fichero _annotations.proto_,** para ello en el fichero .proto debes también añadir la línea:

<pre class="EnlighterJSRAW" data-enlighter-language="null">import "google/api/annotations.proto";</pre>

Puedes añadirla al principio, justo después de la línea &#8220;syntax&#8221;.
  
**¡No recompiles el servidor de gRPC!** Lo que hemos agregado no afecta al servidor, pero si lo intentas compilar vas a recibir errores. Luego hablamos de eso. Ahora toca generar el fichero proto compilado (.pb) a partir del .proto. Para ello vamos a crearnos un contenedor de Docker.
  
**Configurar Envoy &#8211; Parte 2: Crear un contenedor para compilar el fichero .proto**
  
Esto es fácil. Yo he creado una carpeta _ProtoCompiler_ y he metido un Dockerfile en ella:

<pre class="EnlighterJSRAW" data-enlighter-language="generic">FROM golang:1.13beta1-buster as ProtoCompilerBase
WORKDIR /protoc
RUN apt-get update && apt-get install unzip
RUN curl -OL https://github.com/google/protobuf/releases/download/v3.2.0/protoc-3.2.0-linux-x86_64.zip
RUN unzip protoc-3.2.0-linux-x86_64.zip -d protoc3 && mv protoc3/bin/* /usr/local/bin/ &&  mv protoc3/include/* /usr/local/include/
WORKDIR /gapis
RUN git clone https://github.com/googleapis/googleapis
ENV GOOGLEAPIS_DIR=/gapis/googleapis
FROM ProtoCompilerBase
WORKDIR /app
ENTRYPOINT protoc -I${GOOGLEAPIS_DIR} -I/app --include_imports --include_source_info --descriptor_set_out=/out/${PROTO_NAME}.pb /app/${PROTO_NAME}.proto</pre>

Este Dockerfile parte de una image de Go a la que le instala todas las dependencias y luego invoca &#8220;protoc&#8221;. El nombre del fichero proto se le pasa en la variable de entorno PROTO_NAME, y debe estar ubicado en el directorio /app del contenedor.
  
Luego, solo queda modificar el fichero compose:

<pre class="EnlighterJSRAW" data-enlighter-language="json">protoc:
  image: protoc-compiler
  build:
    context: ProtoCompiler
    dockerfile: Dockerfile
  environment:
    - PROTO_NAME=${PROTO_NAME:-proto}
  volumes:
    - "${PROTO_DIR}/${PROTO_NAME:-proto}.proto:/app/${PROTO_NAME:-proto}.proto"
    - "./ProtoCompiler/_out:/out"</pre>

Le pasamos la variable de entorno &#8220;PROTO_NAME&#8221; al contenedor y definimos dos _bind mounts_:

  1. El de entrada, para dejar el fichero proto en el /app del contenedor. La ubicación del fichero proto en nuestro sistema local la define la variable PROTO_DIR
  2. El de salida, para poder recojer el fichero .pb generado.

Si ahora ejecutas los comandos (por supuesto si estás en Linux usa _export_ en lugar de _set_):

<pre class="EnlighterJSRAW" data-enlighter-language="null">set PROTO_DIR=./DemoService/Protos
set PROTO_NAME=greet
docker-compose build protoc
docker-compose run protoc</pre>

Deberías tener en el directorio _ProtoCompiler/_out_ el fichero .pb generado. La siguiente imagen muestra el proceso. Observa como en __out_ no hay nada y luego de ejecutar el contenedor con &#8220;docker-compose run&#8221; nos aparece el fichero .pb.
  
[<img class="alignnone wp-image-2391 size-full" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/06/generate-sb.png" alt="Línea de comandos donde se ve que el directorio _out está vacío, se establecen las variables de entorno (PROTO_NAME y PROTO_DIR) y se ejecuta el &quot;docker-compose run protoc&quot;. Luego se muestra como el directorio _out contiene el binario (.sb)" width="506" height="546" />][5]
  
Ahora que ya tenemos el fichero .pb, ya podemos crear la configuración de Envoy.
  
**Configurar Envoy &#8211; Parte 3: Generar el yaml de Envoy**
  
Envoy se configura mediante un fichero yaml. Contar [como se configura Envoy][6] queda fuera del alcance de este post, así que voy a poner el código de un fichero de configuración y comentaré las partes más relevantes. Por supuesto, habría otras maneras, Envoy es muy potente y flexible:

<pre class="EnlighterJSRAW" data-enlighter-language="null">admin:
  access_log_path: /tmp/admin_access.log
  address:
    socket_address: { address: 0.0.0.0, port_value: 9901 }
static_resources:
  listeners:
  - name: listener1
    address:
      socket_address: { address: 0.0.0.0, port_value: 51051 }
    filter_chains:
    - filters:
      - name: envoy.http_connection_manager
        typed_config:
          "@type": type.googleapis.com/envoy.config.filter.network.http_connection_manager.v2.HttpConnectionManager
          stat_prefix: grpc_json
          codec_type: AUTO
          route_config:
            name: local_route
            virtual_hosts:
            - name: local_service
              domains: ["*"]
              routes:
              - match: { prefix: "/" }
                route: { cluster: grpc, timeout: { seconds: 60 } }
          http_filters:
          - name: envoy.grpc_json_transcoder
            config:
              proto_descriptor: "/etc/envoy/greet.pb"
              services: ["Greet.Greete"]
              print_options:
                add_whitespace: true
                always_print_primitive_fields: true
                always_print_enums_as_ints: false
                preserve_proto_field_names: false
          - name: envoy.router
  clusters:
  - name: grpc
    connect_timeout: 1.25s
    type: logical_dns
    lb_policy: round_robin
    dns_lookup_family: V4_ONLY
    http2_protocol_options: {}
    load_assignment:
      cluster_name: grpc
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: server
                port_value: 80</pre>

Vale, parece muy largo y complejo, pero en esencia estamos configurando el filtro de [_envoy.grpc\_json\_transcoder_][7] que es el encargado de traducir entre HTTP/JSON y gRPC:

<pre class="EnlighterJSRAW" data-enlighter-language="json">http_filters:
- name: envoy.grpc_json_transcoder
  config:
    proto_descriptor: "/etc/envoy/greet.pb"
    services: ["Greet.Greeter"]</pre>

Con esto indicamos que fichero .pb usar y qué servicio (en formato _package.nombreServicio_).

<pre class="EnlighterJSRAW" data-enlighter-language="null">- match: { prefix: "/" }
  route: { cluster: grpc, timeout: { seconds: 60 } }</pre>

Enrutamos todas las peticiones al _cluster grpc_. Un cluster en Envoy es (más o menos) un servicio virtual.

<pre class="EnlighterJSRAW" data-enlighter-language="null">clusters:
- name: grpc
  connect_timeout: 1.25s
  type: logical_dns
  lb_policy: round_robin
  dns_lookup_family: V4_ONLY
  http2_protocol_options: {}
  load_assignment:
    cluster_name: grpc
    endpoints:
    - lb_endpoints:
      - endpoint:
          address:
            socket_address:
              address: server
              port_value: 80</pre>

Configuramos el cluster _grpc_ para que termine llamando por HTTP puerto 80 al servicio llamado &#8220;server&#8221;.
  
Con esto, cuando nosotros llamemos a Envoy, usando HTTP/JSON, este enrutará la petición al servicio llamado _server_ por el puerto 80, usando gRPC.
  
**Configurar Envoy &#8211; Parte 4: Todo junto en docker compose**
  
Debemos añadir la configuración de Envoy en el fichero compose. Para ello le debemos pasar (usando un _bind mount_), el fichero .pb y el fichero envoy.yaml. En mi caso el fichero envoy.yaml lo tengo en una carpeta llamada Envoy. Así que agrego eso al fichero docker-compose.override.yml:

<pre class="EnlighterJSRAW" data-enlighter-language="null">envoy:
  volumes:
    - "./ProtoCompiler/_out/${PROTO_NAME:-proto}.pb:/etc/envoy/${PROTO_NAME:-proto}.pb"
    - "./Envoy/envoy.yaml:/etc/envoy/envoy.yaml"
  ports:
    - 9100:51051</pre>

Observa que Envoy se expone por el puerto 9100. Así que ahora ya solo queda hacer un &#8220;_docker-compose up server envoy_&#8221; y verificar que **usando el puerto 9100 podemos acceder via HTTP/JSON a nuestro servicio gRPC**. Podemos usar curl para ello, tecleando p. ej. _curl http://localhost:9100/api/v1/hello/eiximenis_ y observando como se invoca el servicio gRPC y recibimos la respuesta en HTTP/JSON.
  
[<img class="alignnone size-large wp-image-2393" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/06/envoy-grpc-1024x608.png" alt="Dos consolas donde en una hay el curl mencionado (con la respuesta en json) y en la otra los logs del servidor, donde se observa como se llama al servicio grpc" width="660" height="392" />][8]
  
**¡Perfecto! ¡Ya has traducido de gRPC a HTTP/JSON usando Envoy!** ¿Qué te parece? ¡Más fácil imposible!
  
**Los errores al compilar el servidor gRPC**
  
Si usas el fichero .proto que hemos modificado en el servidor y lo intentas recompilar obtendrás errores:
  
[<img class="alignnone size-full wp-image-2394" src="https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/06/errors-compiling-new-proto.png" alt="Los errores que aparecen al compilar el proto modificado (captura pantalla errores de VS)" width="956" height="108" />][9]
  
Los errores son un &#8220;File not found&#8221; y un error que el Import falla. Eso es **porque en local no tengo el fichero annotations.proto** que estamos importando.
  
Si usas protoc directamente la solución es clonar el repositorio <https://github.com/googleapis/googleapis> y establecer la variable de entorno GOOGLEAPIS_DIR al directorio donde has clonado el repo. Pero eso **no funciona en dotnet build/Visual Studio**. Eso es porque el paquete Grpc.Tools establece determinados directorios como ficheros para importar cuando invoca protoc.exe. Hay una [issue al respecto][10], donde se comenta **que la solución pasa por incorporar el fichero annotations.proto** al proyecto. **Pero eso dista de ser lo deseable (yo no lo he probado). **Es posible hacer una compilación invocando a mano (usando <Exec>) protoc, y quizá hay alguna configuración de Grpc.Tools que lo permita. **No lo he investigado y lo desconozco**. **Si tengo novedades al respecto ya lo publicaré en el blog**.
  
Bueno, lo dejamos aquí por hoy. En el próximo post veremos como desplegar eso en un Kubernetes usando envoy como _side car_ container 🙂
  
**PD:** Os dejo un fichero zip con el código fuente final. **IMPORTANTE**: En este fichero .zip, el .proto tiene las líneas adicionales comentadas. La idea es **que una vez hayáis compilado el servidor y el cliente y antes de generar el .pb**, descomentéis esas líneas. Si las dejo descomentadas os encontraréis con los errores de VS que comentaba al final.
  
El fichero [zip lo tenéis aquí][11]. ¡Saludos!
  
&nbsp;

 [1]: http://geeks.ms/etomas/2019/06/27/grpc-y-no-grpc-todo-junto-en-el-mismo-proyecto/
 [2]: https://www.envoyproxy.io/
 [3]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/06/grpc-log-server.png
 [4]: https://cloud.google.com/service-infrastructure/docs/service-management/reference/rpc/google.api#http
 [5]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/06/generate-sb.png
 [6]: https://www.envoyproxy.io/docs/envoy/latest/configuration/configuration
 [7]: https://www.envoyproxy.io/docs/envoy/latest/configuration/http_filters/grpc_json_transcoder_filter
 [8]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/06/envoy-grpc.png
 [9]: https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/06/errors-compiling-new-proto.png
 [10]: https://github.com/grpc/grpc/issues/18214
 [11]: https://1drv.ms/u/s!Asa-selZwiFlg_5PtDCc79CHA8hn4Q