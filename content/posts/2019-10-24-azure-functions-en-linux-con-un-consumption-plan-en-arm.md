---
title: Azure Functions en Linux con un Consumption Plan en ARM
author: eiximenis
date: 2019-10-24
geeks_url: /?p=2400
categories:
  - azure
  - serverless
---

Buenas! Esos días he tenido que desplegar una Azure Function en Python. A día de hoy solo se pueden desplegar en Azure Functions cuyo service plan sea Linux (no están soportadas en Windows).

En mi caso quería una plantilla ARM que soportase la creación del service plan y la function app tanto en consumption como en app service plan, para tener la máxima flexibilidad.

<!--more-->

Crear un service plan en Linux no es muy complicado:

```json
{
    "type": "Microsoft.Web/serverfarms",
    "apiVersion": "2018-02-01",
    "kind": "functionapp",
    "name": "[variables('hosting_plan_name')]",
    "location": "[resourceGroup().location]",
    "sku": {
      "name": "[parameters('funcAppSkuName')]",
      "tier": "[parameters('funcAppSkuTier')]"
    },
    "properties":  {
      "kind": "functionapp"
      "reserved": true,
      "name": "[variables('hosting_plan_name')]"
    }
```

Aquí la clave es `"reserved": true`, Eso genera el service plan para Linux. Si no lo pones, se genera un service plan para Windows.

Los parámetros `funcAppSkuName` y `funcAppSkuTier` contienen el nombre (S1, P1, …) y el nivel (Standard, Premium,…) del tier a usar. Así si quieres crear un service plan S2, pues `funcAppSkuName` es S2 y `funcAppSkuTier` es Standard.

Pero, si intentas desplegar este ARM con los valores `Y1` y `Dynamic` te encontrarás que te da un error indicando que Dynamic es un valor incorrecto para la SKU. El error es bastante claro, pero p. ej. si creas una function app con un consumption plan "a mano" desde el portal y luego analizas con resources.azure.com lo que se ha creado, verás como efectivamente esos son los valores del recurso:

!(Código ARM que devuelve el portal)[https://geeks.ms/etomas/wp-content/uploads/sites/154/2019/10/linux-consumption-plan.png]

Pero da igual, no los pongáis que no va. La razón es muy sencilla: no se puede crear explícitamente un consumption plan en Linux. El consumption plan se crea "automáticamente" cuando se crea una functionapp en Linux, que no tenga asociado un service plan (eso mismo ocurre en Windows, pero en Windows sí se pueden crear consumption plans explícitamente).

Por lo tanto la solución pasa por crear el service plan solo si la functionapp se debe desplegar bajo un modelo de service plan (no de consumption). Yo, para soportar ambos escenarios he usado un ARM parecido al siguiente (muestro solo los recursos):

```json
{
    "type": "Microsoft.Web/serverfarms",
    "condition":"[not(equals(parameters('funcAppSkuName'), 'Y1'))]",
    "apiVersion": "2018-02-01",
    "kind": "functionapp",
    "name": "[variables('hosting_plan_name')]",
    "location": "[resourceGroup().location]",
    "sku": {
      "name": "[parameters('funcAppSkuName')]",
      "tier": "[parameters('funcAppSkuTier')]"
    },
    "properties":  {
      "kind": "functionapp",
      "reserved": true,
      "name": "[variables('hosting_plan_name')]"
    }
},
{
    "apiVersion": "2015-08-01",
    "type": "Microsoft.Web/sites",
    "name": "[variables('func_app_name')]",
    "location": "[resourceGroup().location]",
    "kind": "functionapp,linux",
    "dependsOn": [
      "[resourceId('Microsoft.Web/serverfarms', variables('hosting_plan_name'))]",
      "[resourceId('Microsoft.Storage/storageAccounts', variables('storage_name'))]"
    ],
    "properties": {
      "reserved": true,
      "serverFarmId": "[if (equals(parameters('funcAppSkuName'), 'Y1'), json('null'), resourceId('Microsoft.Web/serverfarms', variables('hosting_plan_name')))]",
      "siteConfig": {
        "appSettings": [
          {
            "name": "appsetting1",
            "value": "value_appsetting1"
          },
        ],
        "alwaysOn": "[if (equals(parameters('funcAppSkuName'), 'Y1'), json('false'), json('true'))]"
      }
    }
}
```

El ARM es sencillo pero hace un par de cosas interesantes:

* Solo crea el serviceplan si el valor de funcAppSkuName no es Y1. «Y1» es el valor para los planes consumption, así que en ester caso asumimos que no queremos crear el recurso. Para ello uso el atributo [condition](https://docs.microsoft.com/es-es/azure/architecture/building-blocks/extending-templates/conditional-deploy) de ARM.
* alwaysOn se establece a true solo si no creamos un consumption plan (las AF que están en consumption plan no permiten «Always On». Para ello lo establezco a true o false basándome en el valor de funcAppSkuName, usando la [sentencia if de ARM](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/template-functions-logical#if).
* Establece el valor de serverFarmId de la functionApp solo si no creamos un consumption plan. Si no establecemos este valor, es cuando Azure deduce que nuestra AF debe correr bajo un consumption plan y lo crea automáticamente.

De esta manera se puede usar el mismo ARM para crear AF Linux que se ejecuten bajo un App Service Plan (creado por el propio ARM) o bien bajo un consumption plan (creado automáticamente por Azure).

Por si os sirve 😉