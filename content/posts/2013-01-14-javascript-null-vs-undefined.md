---
title: '[Javascript] – null vs undefined'

author: eiximenis

date: 2013-01-14T18:13:30+00:00
geeks_url: /?p=1624
geeks_visits:
  - 2781
geeks_ms_views:
  - 1972
categories:
  - Uncategorized

---
Buenas un post rapidito!! 😉

El otro día en un curso sobre patrones Javascript que estaba impartiendo surgió una de las dos(*) eternas dudas sobre Javascript. Uno de los alumnos (probablemente el único que estaba despierto) me preguntó “_¿y cual es la diferencia exacta entre null y undefined?_”.

Es curioso la desinformación que existe sobre este tema, cuando en realidad es muy simple. Por supuesto buscando por internet aparecen multitud de páginas sobre el tema, ¡pero es que he visto varias que están mal! P. ej. en una página donde se hablaba precisamente de esto, se decía:

  1. _Una función siempre tiene que retornar un valor._ 
  2. _Si algo es **undefined** no tiene valor por tanto una función nunca puede retornar **undefined** (me refiero a funciones propias de JavaScript)_ 

Esas dos afirmaciones son falsas (a no ser que con el paréntesis final intente aclarar algo que no llego a entender) y además tienen que ver con el llamado “doble error de null” o el hecho de que un error anula a otro y todo parece correcto cuando en realidad… ¡hay dos errores!

Para empezar podemos ver que una función javascript puede perfectamente devolver undefined. Para una demostración rápida me voy a basar en el REPL que tiene Node.js:

![](https://geeks.ms/wp-content/uploads/old-web-files/etomas/image_thumb_75AB808B.png)

Lo cierto es que es práctica común y <u>errónea</u> usar código como el siguiente para comprobar p.ej. si una variable (o propiedad) ha sido definida o un parámetro ha sido pasado:

```js
if (login == null) {}
``` 

Esta simple línea contiene **dos errores que se anulan uno al otro**:

  1. El primer es usar null en lugar de undefined.
  2. El segundo es usar la comparación coercitiva (`==`) en lugar de la comparación estricta (`===`). 

El código correcto debería ser:

```js
if (login === undefined) {}
```

En javascript se usa `undefined` para indicar que **algo no existe**. Que no exista no significa que no tenga valor. Para indicar que algo existe pero no tiene valor se usa `null`. Piensa p. ej. que en javascript los argumentos de las funciones son opcionales: si quien llama a la función omite el argumento, este toma el valor de undefined. Pero a lo mejor quien nos llama a la función nos ha pasado el argumento pero sin valor: entonces será null. Lo mismo ocurre con las propiedades de los objetos, que en javascript como buen lenguaje dinámico pueden existir o no sobre la marcha.

El problema es que `undefined !== null` pero `undefined == null` y esa es la causa de toda la confusión. Cuando hacemos `if (x == null)` **la comparación coercitiva intenta hacer conversiones de tipos** y puede convertir entre el tipo `undefined` a `null`.

Realmente la comparación coercitiva (`==`) es una herramienta poderosísima y debería usarse con mucho cuidado. Por norma general deberías usar siempre `===` (y su contrario `!==`) para comparar. De esta manera te evitarías muchos errores y comportamientos extraños. Y echar mano del doble igual solo cuando es estrictamente necesario.

Saludos!

(*): Por cierto que la otra eterna duda (que también salió, como no) tuvo que ver sobre el volátil significado de `this` en javascript y también escribiré sobre ello.