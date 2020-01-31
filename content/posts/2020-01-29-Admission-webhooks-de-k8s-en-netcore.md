---
title: Admission webhooks de Kubernetes con netcore
author: eiximenis
description: Los admission webhooks son uno de los mecanismos de personalización más potentes que ofrece Kubernetes. Permite añadir reglas mediante las cuales determinados objetos (pods, deployments, ...) pueden ser aceptados o no en el clúster e incluso ser automáticamente modificados.
date: 2020-01-29T18:20:00
categories:
  - netcore
  - k8s
---

Si has usado Kubernetes un poco, seguro que conoces el concepto de _sidecar container_: Un contenedor que se ejecuta en el mismo _pod_ que el contenedor principal y que ofrece servicios adicionales. Es muy habitual en implementaciones de [Service Mesh](https://en.wikipedia.org/wiki/Service_mesh) tales como [Istio](https://istio.io/). También [dapr](https://github.com/dapr/dapr) se basa en un _sidecar_ así como [Devspaces](https://docs.microsoft.com/es-es/azure/dev-spaces/) sin ir más lejos, por poner solo tres ejemplos.

Todo eso viene a cuento, porque cuando usas uno de esos sistemas, tus _deployments_ son modificados automáticamente por el sistema para añadir el _sidecar_ container. Es decir, el YAML que Kubernetes recibe no es el YAML que tu instalas, y el responsable es precisamente un [admission webhook](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/) que suele ser la forma usada en esos casos para realizar esas modificaciones.

## Qué és un admission webhook

Como su nombre indica es un "webhook" es decir, un _endpoint_ http al que Kubernetes llamará cuando ocurra un determinado evento, tal como que se va a crear un _pod_ o modificar un servicio. Dichos webhooks se suelen ejecutar en el clúster como contenedores, por lo que **pueden ser desarrollados en cualquier lenguaje**. Por lo general, todos los ejemplos los verás en [Go](https://golang.org/), ya que este es el lenguaje de facto para Kubernetes y tiene una librería impresionante para interactuar con el clúster. Pero, como son contenedores, los puedes crear en cualquier lenguaje.

Para instalar un webhook, a grandes rasgos, solo debes hacer dos cosas:

* Desplegar el contenedor en el clúster (eso incluye, básicamente, el _deployment_ y el servicio tal y como harías en cualquier otro caso)
* Configurar el clúster para que use el nuevo webhook. Eso se hace a través de un objeto propio de Kubernetes de tipo `ValidatingWebhookConfiguration` o `MutatingWebhookConfiguration` en función de si nuestro webhook solo valida o también modifica los datos.

Efectivamente un _admission webhook_ puede o bien:

* Validar que el objeto que se va a crear/modificar es válido según determinadas reglas y aceptar/rechazar dicha acción
* Mutar un objeto creado/modificado (p. ej. para añadir un _sidecar_ container)

En el primer caso hablamos de un "validating webhook" y en el segundo de un "mutating webhook". Aunque técnicamente, los "mutating webhooks" también pueden actuar como "validating webhooks" ya que tienen la potestad de denegar objetos, es mejor tener esas responsabilidades separadas. Además Kubernetes llama primero a los "mutating webhooks" y luego a los "validating webhooks". Así de este modo un "validating webhook" siempre recibe la versión "real final" (ya modificada por los "mutating webhooks") para validarla y decidir si la acepta o no. 

Ambos webhooks reciben el mismo tipo de peticiones. Un objeto de tipo `AdmissionReview` que tiene el formato tal y comp sigue:

```json
{
  "kind": "AdmissionReview",
  "apiVersion": "admission.k8s.io/v1beta1",
  "request": {
    "uid": "baa65061-426d-11ea-85e9-00155d012c20",
    "kind": {
      "group": "",
      "version": "v1",
      "kind": "Pod"
    },
    "resource": {
      "group": "",
      "version": "v1",
      "resource": "pods"
    },
    "namespace": "default",
    "operation": "CREATE",
    "userInfo": {
      "username": "docker-for-desktop",
      "groups": [
        "system:masters",
        "system:authenticated"
      ]
    },
    "object": { ... },
    "oldObject": null,
    "dryRun": false
  }
}
```

>**Nota**: Actualmente hay dos versiones de `AdmissionReview` (la `v1beta1` y la `v1`). El webhook puede elegir cuales acepta y en caso de aceptar todas, Kubernetes envía siempre la primera versión disponible.

Tenemos pues información sobre la acción (`CREATE` en ese ejemplo), el usuario que la ha realizado, el espacio de nombres y en `object` está todo el objeto que en este caso se va a crear. Así, si se crea un _pod_ aquí tendrás toda la definición del _pod_ (en json). Por lo tanto, el webhook puede inspeccionar este objeto y decidir si acepta o no esa acción. Interesantes son los atributos `kind` y `resource`. El primero tiene el tipo del recurso (lo que vendrá serializado en el campo `object`) mientras que el segundo indica el recurso que está siendo modificado. A veces `kind` y `resource` coinciden, pero no siempre. P. ej. si se escala un _deployment_, en `resource` tendrás el _deployment_ que se escala, pero en `kind` lo que tendrás el objeto `Scale` (de `autoascaling/v1`) asociado.

Un _admission webhook_ debe responder con un código HTTP 200 y en el cuerpo de la respuesta (`Content-Type: application/json`) debe ser un objeto `AdmissionReview` (sí, el mimso tipo recibido, aunque se usan otros campos). Si es un "validating webhook" basta con una respuesta como la siguiente:

```json
{
  "apiVersion": "admission.k8s.io/v1",
  "kind": "AdmissionReview",
  "response": {
    "uid": "Valor copiado de request.uid",
    "allowed": true|false
  }
}
```

Con eso, se acepta o no la petición recibida.

Los "mutating webhook" pueden modificar la petición recibida y para ello usan los siguientes campos de `response`:

* `patchType`: Indica el tipo de modificación. A día de hoy **solo se soporta `JSONPatch`**.
* `patch`: Las modificaciones a realizar

El campo `patch` es el divertido: Se trata de un array de modificaciones [JSONPatch](http://jsonpatch.com/) codificado en BASE64.

## Un ejemplo en .Net Core

Todo eso está muy bien, pero basta de cháchara y veamos un ejemplo en .Net Core. **Para mantener el código bajo mínimos he usado [FeaterHttp](https://github.com/davidfowl/FeatherHttp) de [David Fowler](https://twitter.com/davidfowl)** que permite crear aplicaciones ASP.NET Core de forma muy sencilla. Este es el código de `Program.cs`:

```cs
public class Program
{
    public static async Task Main(string[] args)
    {
        var builder = WebApplication.CreateBuilder(args);
        var app = builder.Build();
        var validatorWebhook = new Webhook(Validate.Run);
        app.MapPost("/validate", validatorWebhook.CheckAndRun);
        app.MapFallback(async ctx => {
            Console.WriteLine("Called url: " + ctx.Request.Path);
        });
        await app.RunAsync();
    }
}
```

Como _FeatherHttp_ no está en el feed de nuget oficial, debo usar un `nuget.config` propio:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
    <packageSources>
        <clear />
        <add key="featherhttp" value="https://f.feedz.io/davidfowl/featherhttp/nuget/index.json" />
        <add key="NuGet.org" value="https://api.nuget.org/v3/index.json" />
    </packageSources>
</configuration>
```

No hay clase `Startup` ni nada. Simplemento enruto las llamadas HTTP POST a `/validate` a un `DelegateRequest` definido y creo un fallback (solo para depuración porque no se llama nunca).

La clase `Webhook` define el método `CheckAndRun` que es el `DelegateRequest`:

```cs
public class Webhook
{
    private readonly Func<dynamic, HttpContext, Task> _action;
    public Webhook(Func<dynamic, HttpContext, Task> action)
    {
        _action = action;
    }

    public async Task CheckAndRun(HttpContext ctx)
    {
        var ctype = ctx.Request.ContentType.ToLowerInvariant();
        if (ctype != "application/json")
        {
            Console.WriteLine($"Error. Invalid ContentType: {ctype}");
            return;
        }

        using (var reader = new StreamReader(ctx.Request.Body, Encoding.UTF8))
        {
            var json = await reader.ReadToEndAsync();
            dynamic data = JObject.Parse(json);
            await _action(data, ctx);
        }
    }
}
```

Esta clase solo valida que el content-type de la petición sea `application/json`. Si lo es, deserializa el JSON recibido a un `dynamic` y lo pasa al delegado que se ha especificado en el constructor. Solo comentar que uso [Newtonsoft.Json](https://www.newtonsoft.com/json) para deserializar el JSON, porque [`System.Text.Json` no soporta `dynamic` todavía](https://github.com/dotnet/corefx/issues/38007).

Finalmente nos queda el código propio de validación, en mi caso ubicado en el método `Validate.Run`:

```cs
public static class Validate
{
    public static async Task Run(dynamic data, HttpContext ctx)
    {
        dynamic pod = data.request["object"];
        string uid = data.request.uid.ToString();
        string image = pod.spec.containers[0].image.ToString();
        var name = pod.metadata.name.ToString();
        Console.WriteLine($"Pod {name} has image {image}");
        var tokens = image.Split(new char[] { ':' }, StringSplitOptions.RemoveEmptyEntries);
        if (tokens.Length < 2 || tokens[1] == "latest")
        {
            Console.WriteLine("latest images are not allowed.");
            await ctx.Response.GenerateResponse(uid, allowed: false);
            return;
        }

        await ctx.Response.GenerateResponse(uid, allowed: true);
    }
}
``` 

Simplemente accedo al campo `spec.containers[0].image` del objeto recibido (un _pod_) y miro si el tag es `latest`. En caso de que sea `latest` denego la petición. Simple, pero como ejemplo ya sirve :P Me apoyo en el método de extensión `GenerateResponse`:

```cs
static class HttpResponseExtensions
{
    public static async Task GenerateResponse(this HttpResponse response, string uid, bool allowed)
    {
        response.ContentType = "application/json";
        var content = new
        {
            apiVersion = "admission.k8s.io/v1beta1",
            kind = "AdmissionReview",
            response = new
            {
                uid = uid,
                allowed = allowed
            }
        };
        await response.WriteAsync(JsonConvert.SerializeObject(content));
    }
}
```

Y ya, no hay más código. Ya tengo mi webhook listo... Creo una imagen de Docker y... ¡a instalarlo!

## Instalando el mutating webhook

Eso en teoría es sencillo: solo debo crear un servicio, un _deployment_ y el objeto `ValidatingWebhookConfiguration`. Pero, hay **un pequeño temilla**: los _admission webhooks_ solo se pueden llamar via HTTPS. Y sí... eso implica un certificado :)

A ver, hay varias maneras de generar este certificado. Por lo general os encontraréis que mucha gente usa un script bash, para generar un certificado ya sea autofirmado o bien firmado por la propia CA del clúster (usando un `CertificateSigningRequest`). Ambos sistemas funcionan, pero hay una alternativa que es **usar Helm para generar un certificado** y de este modo todo se puede desplegar via Helm. Para ello haremos uso de [las funciones criptográficas de Helm](http://masterminds.github.io/sprig/crypto.html), tal y como se menciona en [este post](https://medium.com/nuvo-group-tech/move-your-certs-to-helm-4f5f61338aca). Así, vamos a crear un CA, un certificado propio y lo usaremos. La plantilla de Helm es como sigue:

```yaml
{{- $altNames := list ( printf "%s.%s" (include "samplewh.name" .) .Release.Namespace ) ( printf "%s.%s.svc" (include "samplewh.name" .) .Release.Namespace ) -}}
{{- $ca := genCA "samplewh-ca" 365 -}}
{{- $cert := genSignedCert ( include "samplewh.name" . ) nil $altNames 365 $ca -}}

apiVersion: admissionregistration.k8s.io/v1beta1
kind: ValidatingWebhookConfiguration
metadata:
  name: {{ template "samplewh.name" . }}.eiximenis.dev
  namespace: {{ .Release.Namespace }}
  labels:
    app: {{ template "samplewh.name" . }}
webhooks:
  - name: {{ template "samplewh.name" . }}.eiximenis.dev
    failurePolicy: Ignore
    rules:
      - operations: ["CREATE"]
        apiGroups: [""]
        apiVersions: ["v1"]
        resources: ["pods"]
    clientConfig:
      service:
        name: {{ template "samplewh.name" . }}
        namespace: {{ .Release.Namespace }}
        path: "/validate"
      caBundle: {{ $ca.Cert | b64enc }}
---
apiVersion: v1
kind: Secret
type: kubernetes.io/tls
metadata:
  name: {{ template "samplewh.name" . }}-tls-secrets
  labels:
    app: {{ template "samplewh.name" . }}
    chart: {{ template "samplewh.chart" . }}
    heritage: {{ .Release.Service }}
    release: {{ .Release.Name }}
  annotations:
    "helm.sh/hook": "pre-install"
    "helm.sh/hook-delete-policy": "before-hook-creation"
data:
  tls.crt: {{ $cert.Cert | b64enc }}
  tls.key: {{ $cert.Key | b64enc }}
```

La plantilla crea dos objetos: el propio `ValidatingWebhookConfiguration` y un secreto para guardar el certificado TLS. Del primero me interesa comentaros el campo `webhooks` que contiene una lista de los webhooks a configurar. Para cada webhook configuramos (en `rules`) sobre qué objetos y operaciones aplica este webhook (p. ej. en mi caso al crear un _pod_ usando la API `v1`) y luego usando `clientConfig` le indicamos donde está dicho webhook. En este caso está en un servicio (`service`) ejecutándose en Kuberntes y con el nombre indicado. Y en el campo `caBundle` le debemos indicar el certificado TLS.

En mi caso el servicio se llama `samplewh` y el certificado debe usar este CN, así como los siguientes nombres alternativos: `samplewh.default` (donde `default` es el espacio de nombres del servicio) y `samplewh.default.svc`. Esos nombres alternativos son los que se colocan en la variable `$altNames` del chart.

Guay... Ahora solo nos falta una cosa... 

## Leer el certificado TLS desde Kestrel y configurar HTTPS

Aquí tenemos dos acciones a realizar. La primera es **que Kestrel no soporta por configuración usar certificados PEM**, ya que espera en su lugar certificados PKCS#12 (vamos, ficheros `.pfx`). [Con openssl es fácil pasar de uno a otro](https://www.ssl.com/how-to/create-a-pfx-p12-certificate-file-using-openssl/). Para que esa conversión ocurra sin que nadie tenga que lanzar script alguno, la he puesto en un _init container_ de Kubernetes. Al iniciar el _pod_ dicho contenedor simplemente ejecuta `openssl` y convierte el certificado de `PEM` a `PKCS#12` y este certificado en `.pfx` es el que se pasa a Kestrel. Para ello, el _init container_ simplemente ejecuta un fichero `.sh` que recibe a través de un _config map_:

```sh
apk update && apk add --no-cache openssl 
openssl pkcs12 -export -out /var/lib/pfx/cert.pfx -inkey /var/lib/secrets/tls.key -in /var/lib/secrets/tls.crt -passout pass:Passw0rd
echo 'pfx file generated:'
ls /var/lib/pfx
```

Desde el chart se crea un _config map_ con dicho fichero:

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: {{ include "samplewh.name" . }}-cm
data:
  entrypoint.sh: |-
{{ .Files.Get "entrypoint.sh" | indent 4}}
```

Y dicho config map se mapea como volúmen al _init container_ que de este modo puede acceder y ejecutarlo:

```yaml
volumes:
- name: tls-secrets
  secret:
    secretName: {{ template "samplewh.name" . }}-tls-secrets   
- name: entrypoint-vol
  configMap:
    name: {{ include "samplewh.name" . }}-cm
    defaultMode: 0777
- name: pfx
  emptyDir: {}
initContainers:
- name: {{ .Chart.Name }}-pfx-conv
  image: alpine
  command: ["/bin/sh"]
  args: ["-c", "/init/entrypoint.sh"]
  volumeMounts:
  - name: pfx
   mountPath: /var/lib/pfx
  - name: entrypoint-vol
  mountPath: /init
  - name: tls-secrets
  mountPath: /var/lib/secrets  
```

El volúmen `pfx` está montado en ambos contenedores (el _init container_ y el que ejecuta el webhook) y es donde el primero deja el fichero `.pfx` que lee el segundo.

Finalmente paso las siguientes variables de entorno a Kestrel para cargar el certificado `.pfx` y habilitar HTTPS:

```yaml
- name: Kestrel__Certificates__Default__Path
  value: /var/lib/pfx/cert.pfx
- name: Kestrel__Certificates__Default__Password
  value: Passw0rd
- name: Kestrel__Endpoints__Https__Url
  value: https://*:443
```

Así establecemos la ruta del fichero `.pfx`, la contraseña (la establece el _init container_ al hacer la conversión) y el puerto para Https.

¡Y listos! Ya tenemos nuestro _admission webhook_:

![Salida de consola donde se ve que un pod con imagen "latest" es denegado](/images/posts/2020-01-29-demo.png)

Como se puede ver en la imagen, el _pod_ que tiene la imagen con la etiqueta `latest` es denegado por el _admission webhook_ y Kubernetes no nos permite crearlo. Por otro lado, el otro _pod_ que tiene cualquier otra etiqueta si que se crea sin problemas.

¡Espero que os haya resultado interesante!

PD: Tenéis todo el código en [este repo de GitHub](https://github.com/eiximenis/AdmissionWebhook-netcore-demo).