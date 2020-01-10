---
title: Gestión de Builds y Releases con Docker y Azure Devops
author: eiximenis

date: 2019-10-25
geeks_url: /2019/10/25/gestion-de-builds-y-releases-con-docker-y-azure-devops
categories:
  - devops
  - docker
---

El sistema de Builds y Releases de Azure Devops es extremadamente flexible, pero se basa en una premisa: la build publica cierto artefacto binario que la Release recoje e instala en los distintos entornos (stages en la terminología de Azure Devops).

<!--more-->

Eso para escenarios tradicionales, funciona de maravilla: con la build usas los SDKs correspondientes para generar un binario y publicas dicho binario, que instalas con la release.

Pero cuando entra Docker en la ecuación, empiezan a aparecer ciertas fricciones…

El ejemplo más habitual de una build que use Docker en Azure Devops es que la build **termina publicando la imagen en un registro de Docker (usualmente un ACR)**.

En este caso **la release no recibe ningún artefacto de la build**, es decir la build, a nivel de Azure Devops no publica ningún artefacto. Aquí entra una primera consideración: Qué ocurre si queremos redesplegar una release antigua por cualquier razón?

En el modelo clásico, la release obtiene los binarios de la build a la que está asociada, por lo que todo funciona correctamente (dejando de lado aspectos más problemáticos como gestión de esquemas de BBDD, pero no quiero entrar en eso ahora). Pero en nuestro escenario, la release no obtiene artefaco único de la build, si no que debe recojer la imagen de Docker del registro correspondiente. Es importante pues **asegurar que la release recoje la misma imagen que generó la build**. Por suerte eso tiene fácil solución: **usar un _tag_ único de imagen vinculado por ejemplo a la variable `Build.BuildId`** y listos.

Eso funciona a las mil maravillas, aunque **tenemos una pequeña fricción**: las builds en Azure Devops tienen un tiempo de retención, es decir, pasados xxx días se eliminan del servidor. Cuando se elimina una build se eliminan sus artefactos publicados, ya que dicha build "ha desaparecido" a todos los efectos. El problema es cuando nuestra build genera side effects que afectan a elementos externos: en nuestro caso la build publica una imagen de Docker con un _tag_ en un repositorio externo. **Cuando Azure Devops elimina la build, obviamente no eliminará la imagen generada por dicha build**. Por lo tanto **en el registro de Docker nos quedarán imágenes antiguas, que ya no tienen sentido**. En un entorno de CI completo, que genere varias builds al día, se pueden terminar generando muchas imágenes. Esas imágenes seguirán ahí, en el registro externo, incluso cuando las builds se eliminen.

La solución a eso pasa por **usar los webhooks de Azure Devops para enterarse cuando una build se elimina y poder hacer la "limpieza" necesaria**, es decir eliminar la imagen del registro externo. Otra solución es ejecutar un job cada x tiempo que elimine las imágenes no usadas (aunque deberíamos asegurarnos que realmente no haya la build asociada en Azure Devops). Ambas aproximaciones son tediosas y/o pesadas y requieren infraestructura «fuera» de Azure Devops. **La solución ideal sería que el propio pipeline de build permitiese definir acciones a ejecutar cuando la build es eliminada**.

Si estás en Kubernetes y usas [Helm](https://helm.sh), la cosa se complica un poco, ya que a tu build no le basta con publicar la imagen de Docker: debes publicar también el chart. **Hay mucha gente que se salta ese punto**, y obtiene el código del chart vinculando el repositorio de Git a la release como una fuente más. Eso te permite acceder al contenido del repositorio desde la release y recojer el código fuente del chart. El problema es que no tienes garantía de que el código del chart sea el mismo que había cuando se generó la build en el caso de que reinstales una release antigua. Hay dos posible soluciones a ese problema:

1. Desde la _build_ publicas el código fuente del chart, de este modo la release lo tendrá disponible. La release puede optar por instalar el chart o bien publicarlo en un registro de charts (como ACR) si este chart debe ser usado por más gente.
2. Desde la _build_ empaquetas el chart y lo publicas en un registro de charts (como ACR que también lo permite). Este caso es análogo al de publicar la imagen de Docker a un registro y tiene los mismos problemas (terminan muchos charts publicados, y esos muchas veces son idénticos entre sí, ya que el chart no varía mucho).

En general el problema está que cuando usamos Docker la build hace realmente dos cosas: compila y publica. Y esa parte de publicación parece que correspondería más a la release, o al menos a una "pre-release" (un concepto que me acabo de sacar de la manga).

Si tenemos entornos completamente separados que **no comparten el registro de Docker** (p. ej. tienes dev y pre con un ACR distinto en cada entorno) entonces la cosa se complica: Si la build publica en el registro de Docker de dev, la release de pre fallará, porque el ACR de pre no tiene publicada la imagen (está en el ACR de dev). Sí, podrías tener una build por entorno, pero eso no es nada elegante.

Veo dos posibles soluciones a ese problema:

1. Que la _build_ **no publique en un registro de Docker**, si no que exporte el tar de la imagen (con docker save) y publique ese tar como artefacto. Lo único a tener presente es que son artefactos "grandes" (generalmente más grandes que el código compilado)
2. Que la build publique en un "ACR DE CI" y que la release recoja la imagen y la "republique" en el ACR correspondiente. Esa idea personalmente me gusta más: cada entorno en runtime tiene su propio ACR, y el "ACR DE CI" es un recurso compartido que se usa solo para tener un punto donde recojer las imágenes de Docker para republicarlas.

¿Y vosotros? ?Qué tácticas usáis y qué problemas os habéis encontrado al usar pipelines de Azure Devops con Docker?