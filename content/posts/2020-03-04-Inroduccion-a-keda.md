---
title: "Introducción a KEDA"
author: eiximenis
description: Para ejecutar workloads serverless en Kubernetes, se necesita poder escalarlos adecuadamente. Kubernetes viene con todas las herramientas necesarias para ello, pero deben configurarse y no es nada sencillo. KEDA viene a rellenar ese hueco.
date: 2020-03-04T18:00
draft: true
categories:
  - k8s
  - serverless
tags:
  - Keda
---

KEDA significa _Kubernetes Event Driven Autoscaler_ y es un proyecto iniciado por _Microsoft_ y _Red Hat_ para facilitar el uso de workloads serverless ejecutándose en Kubernetes.

## Serverless en Kubernetes SIN Keda

El problema **no es ejecutar workloads serverless (FaaS para ser más concretos)**: A fin de cuentas si puedes crear una imagen Docker de tu FaaS la puedes desplegar en Kubernetes y este la va a ejecutar. Vamos a ver un ejemplo usando [Azure Functions](https://azure.microsoft.com/es-es/services/functions), aunque más adelante en esta serie hablaremos de [OpenFaas](https://www.openfaas.com/) y [AWS Lambda](https://aws.amazon.com/es/lambda/) entre otras.


