---
title: "Ejecutar contenedores bajo demanda con ACI y Azure Functions"
author: eiximenis
description: "La combinación de Azure Functions y ACI nos da un sistema sencillo para ejecutar contenedores bajo demanda con un modelo 100% serverless"
date: 2020-09-15T18:00:00
draft: false
categories:
  - azure
tags:
  - serverless
  - azurefunctions
  - aci
---

Este post nace a partir de la necesidad de **tener un sistema sencillo, pero 100% serverless para ejecutar tareas bajo demanda en Azure**. Esas tareas se pueden ejecutar cada cierto tiempo (programadas) o bien como respuesta a algún evento. 

Es obvio que en este modelo, Azure Functions encaja como anillo al dedo, pero había dos requisitos que eran importantes contemplar:

1. Esas tareas **pueden ser escritas con cualquier lenguaje de desarrollo** desde .NET Core hasta PHP, pasando por Go o Rust.
2. La duración de esas tareas puede ser arbitrariamente larga (desde segundos hasta una hora o más).

Ambos puntos "chocan" con el uso de Azure Functions: por un lado no todos lenguajes están soportados en Azure Functions y por el otro lado, **a no ser que nos vayamos a un hosting de App Service, la duración de Azure Functions está limitada**. Es cierto que muchos _workloads_ que duran x tiempo se pueden refactorizar para que en lugar de una sola tarea que dure ese tiempo se ejecuten N tareas encadenadas (o en paralelo) donde cada tarea dure menos del límite de tiempo de Azure Functions (10 minutos máximo en el caso de usar un plan de consumo), pero obviamente eso requiere rearquitecturar esos _workloads_, lo que no era posible.

Pues una aproximación sencilla y que funciona bastante bien es **crear una Azure Function que cree un ACI que ejecute la tarea**. Eso mantiene la duración de la Azure Function en un tiempo muy bajo (pocos segundos), por lo que podemos seguir usando un plan de consumo y la ejecución real de la tarea se difiere en un ACI que se crea al momento y que, se puede destruir una vez ejecutada la tarea.

Es un modelo 100% serverless, ya que los ACI solo nos son facturados mientras están ejecutando las tareas.

## Creación de la Azure Function

Para crear la Azure Function, lo más sencillo **es usar una Azure Function que use Powershell**. Yo he usado la versión `3.0.2881` de la [CLI de Azure Functions](https://docs.microsoft.com/en-us/azure/azure-functions/functions-run-local?WT.mc_id=AZ-MVP-4039791) y el wizard de crear una Azure Function para Powershell ya me ha configurado lo necesario.

Así pues al teclear `func init launcher` y seleccionar `powershell` como lenguaje, me ha creado lo necesario. Para destacar lo que es necesario, es que en el fichero `profile.ps1` se encuentre lo siguiente:

```powershell
if ($env:MSI_SECRET) {
    Disable-AzContextAutosave
    Connect-AzAccount -Identity
}
```

Eso permite que la Azure Function se autentique contra Azure usando [SMI](https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview?WT.mc_id=AZ-MVP-4039791) (luego veremos como configurarla).

Dado que usaremos el [módulo Az de Powershell](https://docs.microsoft.com/en-us/powershell/module/az.resources/?view=azps-4.6.1&WT.mc_id=AZ-MVP-4039791) hay que declarar que se quiere usar en el fichero `requeriments.ps1`. Por suerte la plantilla por defecto ya lo incluye:

```powershell
@{
    # For latest supported version, go to 'https://www.powershellgallery.com/packages/Az'. 
    'Az' = '4.*'
}
```

> El módulo Az de Powershell es el nuevo mecanismo oficial para acceder a Azure desde Powershell (y Powershell Core). Sustituye al antiguo [AzureRM](https://docs.microsoft.com/en-us/powershell/module/azurerm.resources/?view=azurermps-6.13.0&WT.mc_id=AZ-MVP-4039791) que a pesar de seguir estando soportado no recibirá ninguna actualización importante más.

Vale, ahora ya podemos añadir la función que deseemos, usando `func new` y eligiendo el disparador que se desee. En este caso, para probar, vamos a crear un disparador de tipo `HTTP Trigger` para poder llamar a la función con un simple GET. Le dais el nombre que queráis (yo la he llamado `CreateACI`) y en el fichero `run.ps1` poneis el siguiente código:

```powershell
using namespace System.Net

# Input bindings are passed in via param block.
param($Request, $TriggerMetadata)

Write-Host "Deploy ACI requested."

# Parse the parameters (name, image, tag and rg)
$name  = $Request.Query.name
if (-not $name) {
    $name = "Default"
}

$name = -join("$name-", [Guid]::NewGuid().ToString())

$rg = $Request.Query.rg
if (-not $rg) {
    $rg = "DefaultACIRg"
}

$image = $Request.Query.image
$tag = $Request.Query.tag

if (-not $tag) {
    $tag = "latest"
}

$fullImage = -join($image, ":", $tag)

# Create the RG if not exist
New-AzResourceGroup -Name $rg -Location "westeurope" -Force

# Deploy the ACI
$resaci=$(New-AzContainerGroup -ResourceGroupName $rg -Image $fullImage -RestartPolicy Never -Name $name) 2>&1

if (-not $resaci.Id) {
    Push-OutputBinding -Name Response -Value ([HttpResponseContext]@{
        StatusCode = [HttpStatusCode]::BadRequest
        Headers = @{"Content-Type" = "application/json"}
        Body = @{"message" = $resaci.Exception.Message}
    })
}
else {
    $body = "ACI $Name in RG $rg to run $fullImage was created. ID is $($x.Id)"
    # Associate values to output bindings by calling 'Push-OutputBinding'.
    Push-OutputBinding -Name Response -Value ([HttpResponseContext]@{
        StatusCode = [HttpStatusCode]::Accepted
        Headers = @{"Content-Type" = "application/json"}
        Body = @{"message" = $body}
    })
}
```

Este código es muy sencillo y hace lo siguiente:

1. Valida los parámetros (en este caso solo uso la query string), para obtener el nombre del ACI, así como el grupo de recursos donde debe desplegar el ACI y la imagen a ejecutar.
2. Luego usa [New-AzResourceGroup](https://docs.microsoft.com/en-us/powershell/module/az.resources/new-azresourcegroup?view=azps-4.6.1&WT.mc_id=AZ-MVP-4039791) para crear el grupo de recursos (si no existe).
3. Luego crea el ACI usando [New-AzContainerGroup](https://docs.microsoft.com/en-us/powershell/module/az.containerinstance/new-azcontainergroup?view=azps-4.6.1&WT.mc_id=AZ-MVP-4039791) para ejecutar la imagen deseada. El parámetro `-RestartPolicy Never` es para que ACI no reinicie el contenedor cuando termine (ya que ejecutamos tareas de un solo uso)
4. Recojemos el resultado de la creación del ACI y devolvemos un 400 o un 201 en función de si el comando para crear el ACI se ha ejecutado correctamente o no.

> Insisto para que no haya confusión: Observa el uso de `New-AzResourceGroup` en lugar del "clásico" `New-AzureRmResourceGroup` y de `New-AzContainerGroup` en lugar de `New-AzureRmContainerGroup`.

## Configurando Managed Identity

Para que nuestra Azure Function tenga permisos para crear recursos, es necesario que se ejecute bajo una identidad que tenga dichos permisos. En este caso, dado que puede crear grupos de recursos, vamos a darle permisos de "Contributor" a la suscripción.

Así, una vez creado el _Function App_ en el portal, nos vamos a la pestaña "Identity" y bajo "System assigned" activamos el "Status" a "On":

![Imagen del portal con el interruptor de "Identity" a ON](/images/posts/2020-09-15-smi.png)

Luego pulsamos el botón "Azure role assignments" y añadimos a la function app como "contributor" de la suscripción (en este caso porque creamos grupos de recursos, pero si tu Azure Function siempre crease los ACIs en el mismo grupo de recursos, entonces solo deberías darle de alta como "contributor" en ese grupo de recursos).

## Desplegar la Azure Function

Para desplegar la AF no hay que hacer nada especial: te basta con usar `func azure functionapp publish <nombre-functionapp>` para publicar la Azure Function. Eso nos dará una URL donde podemos llamar nuestra función y podemos verificar que... ¡cada llamada a la AF crea un ACI para ejecutar el contenedor!

![Imagen del portal con el ACI creado y una llamada con CURL a la AF](/images/posts/2020-09-15-ACI-created.png)

En la imagen anterior puedes ver dos llamadas con cURL a la Azure Function. La primera falla (porque el nombre del tag es incorrecto), pero la segunda si funciona y se puede ver en el portal como se ha creado el ACI correspondiente. ¡El contenedor ya se está ejecutando!

## Conclusiones

En este post se ha visto como es muy sencillo tener un modelo 100% serverless para ejecutar tareas bajo demanda en Azure. El **único requisito es que tengamos un contenedor de Docker** que ejecute esta tarea.

Por supuesto este post es solo un esbozo de lo que se puede hacer, pero básicamente con ACI puedes ejecutar cualquier contenedor, puedes inyectar configuración (variables de entorno), configurar IPs públicas (aunque en nuestro caso no tiene sentido) e incluso [montar volúmenes contra un storage](https://docs.microsoft.com/en-us/azure/container-instances/container-instances-volume-azure-files?WT.mc_id=AZ-MVP-4039791).

¡Espero que te haya resultado útil!