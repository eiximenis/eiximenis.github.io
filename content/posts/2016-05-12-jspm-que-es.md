---
title: ¿Qué es jspm?
author: eiximenis

date: 2016-05-12T10:00:59+00:00
categories:
  - js
---

[Jspm](http://jspm.io/) es **otro gestor de paquetes para JavaScript**. Por lo tanto antes de nada es lícito preguntarnos... ¿necesitábamos otro gestor de paquetes?
En JavaScript tenemos al menos dos de dominantes: Por un lado [bower](http://bower.io/), que es (o fue) el gestor de librerías JavaScript de cliente. Es decir, te querías instalar jQuery o Angular? Pues lo instalabas desde Bower. Bower tenía en cuenta las dependencias (p. ej. que Backbone depende de Underscore) y las descargaba automáticamente. No es perfecto y adolece de ciertas limitaciones (la más importante la imposibilidad de tener una misma librería en más de una versión a la vez) pero fue un avance muy importante en el desarrollo web. Su uso sigue muy extendido, aunque actualmente se está imponiendo la tendencia de usar npm en su lugar.
[npm](https://www.npmjs.com/) es el gestor de paquetes de NodeJs. Su uso inicial se limitaba a proyectos desarrollados para node, pero en los últimos tiempos la popularización de herramientas como WebPack y Browserify (que ofrecen _polyfills_ para los módulos CommonJS de npm) hace que npm se esté usando también en desarrollo para cliente. Hoy en día es habitual cargar jQuery o Angular como módulo CommonJS (usando npm) y a través de WebPack o Browserify poder usarlos en un navegador.

¿Qué ofrece jspm para que merezca ser tenido en cuenta? En primer lugar soporta la carga de paquetes desde distintos orígenes. Uno es npm, es decir podemos instalar cualquier paquete de npm y otro es GitHub, es decir podemos instalar paquetes directamente desde cualquier repositorio de GitHub sin que estén publicados en el directorio de npm. Si se usa el plugin [jspm-git](https://github.com/Orbs/jspm-git) es posible usar cualquier repositorio Git como orígen de JSPM lo que es interesante en ciertos entornos donde se tiene el paquete publicado en un Git interno (de todas formas yo no he usado este paquete así que poco más puedo decir). De todos modos el soporte para distintos orígenes es interesante pero no es lo más relevante de jspm. Lo más relevante es **la integración con SystemJS**.
[SystemJs](https://github.com/systemjs/systemjs) es un cargador universal de módulos JavaSript. Carga módulos en cualquier formato (ES6, CommonJS, AMD, globals). Es como [RequireJs](http://requirejs.org/) pero con la diferencia de que no solo soporta AMD si no también todos los demás formatos.

Jspm vs Webpack
---------------
Webpack ofrece todo lo que tiene jspm y algunas cosas más, salvo que no es un gestor de paquetes. Es decir, con Webpack podemos crear _bundles_, cargar módulos (CommonJS y AMD, el soporte para ES6 se puede obtener con un plugin) y procesar todo tipo de ficheros (CoffeeScript, 
, imágenes, LESS...) para crear nuestros _bundles_. Es una herramienta muy potente y a la vez difícil de configurar.

Las dos diferencias principales entre jspm y Webpack serian:

1. jspm es un gestor de paquetes y Webpack no. Webpack es un _bundler_
2. Con jspm (por defecto) no se usan _bundles_, lo que significa que si tenemos 20 librerías JavaScript se realizan 20 peticiones HTTP. Que eso sea un problema o no, depende de la disponibilidad de HTTP/2.

¿Qué ocurre con módulos que no sean JavaScript (p. ej. CSS) o que se puedan transpilar a JavaScript (p. ej. importar un componente en JSX)? En Webpack hay una pléyade de _loaders_ que se encargan de todas esas tareas (transpilar JSX a JavaScript, pasar de LESS a CSS, minimifcar, etc). Cada uno de esos loaders debe ser configurado de forma correcta, lo que no siempre es sencillo. ¿Qué ofrece jspm? ¿Tiene algo?
De nuevo jspm opta por la simplicidad: nos permite decidir qué transpilador vamos a usar (Babel, TypeScript o Traceur, siendo Babel la opción por defecto). Además existen plugins que extienden el cargador de módulos usado. La idea es que podemos importar módulos que no sean JavaScript, como CSS o ficheros JSX:

``` javascript
import './styles/site.css!'
```

Observa la admiración (!) final, que indica al cargador de módulos que este fichero no es un fichero JavaScript. En este caso se mirará si existe registrado un plugin para el cargador de módulos que soporte esta extension (css). Los plugins se instalan como dependencias mediante el propio jspm.
Recuerda que se usa un transpilador. Los módulos que carguemos pueden pasar por el transpilador, de ahí que podamos soportar p. ej. JSX (ya que Babel los soporta).
La ventaja principal es que jspm es mucho más sencillo de configurar que Webpack, la diferencia es que Webpack tiene mucho más soporte en el tipo de transformaciones que es capaz de realizar (si jspm gana tracción esta diferencia debería ir disminuyéndose).

Jspm vs Browserify
------------------
Básicamente lo mismo que en párrafo anterior. De hecho Browserify se compara con Webpack, aunque hay diferencias filosóficas entre ambos: el primero lo que pretende es que cualquier módulo de nodejs sea usable desde el navegador y el segundo no está tan interesado en la compatibilidad con nodejs.
Al margen de eso, lo dicho para Webpack vale para Browserify. Ambos productos son básicamente _bundlers_ que se extienden para soportar cualquier tipo de fichero (no solo JS) y realizar transformaciones. Mientras que jspm es un gestor de paquetes que integra un cargador de módulos y un transpilador. La diferencia es filosófica, aunque al final podamos tener los mismos resultados.

Jspm vs npm + Systemjs
----------------------
Si vas a usar Systemjs en tu aplicación, no tiene sentido que uses npm en lugar de jspm. La razón es jspm ya te integra Systemjs de forma automática, por lo que te olvidas de instalarlo y configurarlo. Y dado que con jspm puedes instalar paquetes desde npm no pierdes nada.
Si usas requirejs en lugar de Systemjs entonces igual te puedes plantear dar el paso de requirejs a Systemjs. No es muy complicado y una vez hecho, puedes empezar a usar jspm de forma automática. Por el camino ganas el soporte de módulos CommonJS y ES6 y te olvidas de usar herramientas de conversión como r.js.

Instalación de paquetes
-----------------------
Para instalar paquetes jspm se usa el comando `jspm install` a imagen y semejanza del mismo comando npm. Lo único es que podemos indicar un orígen (por defecto solo soporta npm y github, pero otros orígenes se pueden añadir via plugins). Es decir podemos hacer `jspm install npm:react` y nos instalará el paquete 'react' de npm. 
**Si no indicamos orígen** se buscará el paquete en el registro propio de jspm y se usará el orígen correspondiente. El registro de jspm indica el orígen (npm o github) deseado por el paquete. Si el paquete no se encuentra en el registro de jspm entonces jspm dará un error. El registro de jspm es bastante limitado, pero no es problema: basta con indicar el orígen explícitamente si un paquete no se encuentra. Recuerda que puedes instalar cualquier paquete npm  y de github.

Un ejemplo de jspm
------------------

Vamos a ver como podemos crear un proyecto con jspm. El primer paso es instalar jspm. Para ello podemos usar el comando `npm install jspm --save-dev` (jspm es un paquete de npm). Con eso instalamos las herramientas de líneas de comandos (CLI) de jspm. Es recomendable instalar jspm de forma local en lugar de forma global (para evitar que futuras actualizaciones de jspm puedan romper el proyecto. Eso mismo aplica a cualquier CLI instalada con npm como broswerify o gulp).
Si hemos instalado jspm de forma global ya lo tendremos en el path. En caso contrario estará en ./node_modules/.bin. Nos podemos crear un script en el package.json para ejecutar jspm desde npm:

```json
  "scripts": {
    "jspm": "jspm"
  },
```

Y usar `npm run jspm` para invocar jspm o podemos invocarlo desde ./node_modules/.bin.
Una vez que tenemos jspm instalado nos "olvidamos" de npm y usamos jspm. Lo primero es inicializar el poryecto. Para ello usamos el comando `jspm init` (`npm run jspm init` si usamos un script de npm). Eso nos preguntará varias cosas (la carpeta donde dejar los paquetes de jspm, si queremos usar un transpilador, etc). Nos creará un fichero de configuración (`config.js`) y nos modificará el package.json con una sección nueva (jspm) donde hay todo lo referente a jspm. También descargará los módulos necesarios (el transpilador usado, system.js y todo lo que necesite jspm).

Tras ello tendrás un package.json parecido al siguiente (solo se muestra una parte):

```json
  "devDependencies": {
    "jspm": "^0.16.34"
  },
  "jspm": {
    "devDependencies": {
      "babel": "npm:babel-core@^5.8.24",
      "babel-runtime": "npm:babel-runtime@^5.8.24",
      "core-js": "npm:core-js@^1.1.4"
    }
  }
```

Obseva como la única dependencia de npm es el propio jspm, mientras que en la sección "jspm" ya tenemos varias dependencias (según el transpilador que hayamos elegido).

Bien, vamos a crear un par de módulos de ES6 que no hagan apenas nada, pero bueno... nos serviran para probar jspm. Crea un fichero math.js y mete lo siguiente:

```javascript
export default class Math
{
    static add(a,b) {
        return a+b;
    }
}
```

Y crea otro llamado init.js con el siguiente código:

```javascript
import Math from './math.js';
console.log(Math.add(10, 20));
```

Vale, básicamente hemos creado un módulo ES6 (Math) que exporta una clase con un método estático, y otro módulo (init) que usa el módulo anterior.
Ahora vamos a invocar a init.js desde un fichero html, gracias a System.js (que lo tenemos disponible por estar usando jspm).
Para ello creamos el fichero init.html:

```htm
<!DOCTYPE html>
<html>
    <head>
        <title>jspm demo</title>
        <script src="jspm_packages/system.js"></script>
        <script src="config.js"></script>
    </head>
    <body>
        <script>System.import('./init.js');</script>
    </body>
</html>
```

Observa tres detalles:
1. Cargamos system.js que jspm nos ha descargado automáticamente en jspm_packages
2. Cargamos el fichero config.js que nos ha creado jspm cuando hemos usado el comando `jspm init`. Este fichero contiene la configuración de que transpilador usamos y donde encontrar los módulos descargados con jspm.
3. Usamos `System.import` para cargar nuestro fichero JavaScript inicial.

Ahora, si cargas el fichero init.html en el navegador, debes ver un 30 en la consola de JS.

¿Qué ocurre en el navegador? Pues que no hay ningún _bundle_: se cargan todos los archivos JavaScript por separado
![Peticiones de jspm]({{ site.url }}/img/2016-05-12/jspm_requests.png)

Ah, y una cosa... **Esto te funciona en navegadores que no tengan ES6**. Es decir, jspm nos ha configurado el transpilador que hemos indicado (babel en nuestro caso).

Añadiendo otro tipo de recursos
-------------------------------
Vamos a ver como podemos importar ficheros que no sean JavaScript. Para ello ya debemos usar plugins, pues está fuera de la funcionalidad base de jspm. Por suerte tenemos [un plugin para CSS](https://github.com/systemjs/plugin-css) así que vamos a instalarlo. Para ello debemos usar el comando `jspm install css --save-dev`. Ahora vamos a crear un fichero body.css con una CSS sencilla:

```css
body {
   background: red;
}
```

y modificamos nuestro fichero init.html para importar la hoja de estilos contenida en body.css. Para ello agregamos el siguiente código (antes del cierre del head):
```html
<script>System.import('./body.css!');</script>
```
Observa que importamos los CSS como si fuesen JavaScript, con la salvedad de la admiración final. La admiración le dice a System.js que no use el cargador de JS para cargar este recurso si no que use el cargador correspondiente. Podemos indicar el cargador a usar detrás de la admiración (podríamos haber hecho
`System.import('./body.css!css');`, pero si el nombre del cargador es el mismo que el de la extensión del recurso, no es necesario indicarlo (como es nuestro caso).
Con eso... ya tenemos la CSS cargada y aplicada. De momento, que yo sepa, **no se puede importar una CSS desde otra CSS**. La idea no es sustituir SASS (o LESS) si no dar un mecanismo para cargar CSS. Pero si quieres hay un [plugin de SASS](https://github.com/mobilexag/plugin-sass) y [uno de LESS](https://github.com/Aaike/jspm-less-plugin), así que nada te impide usarlos! La idea es la misma: creas el fichero .scss o .less principal y lo importas con System.import. Más fácil... ¡imposible!

En resúmen
----------
jspm es, antes de nada, un gestor de paquetes. Pero eso por si solo, probablemente no justificaría su existencia. Lo interesante es la integración con System.js para cargar módulos JavaScript y mediante plugins cualquier otro recurso. También el uso de un transpilador (Babel, TypeScript o Traceur) viene incorporado de serie. No es algo que no pudiésemos hacer antes, pero jspm lo intenta simplificar al máximo. Ese es su objetivo. Por supuesto puedes usar jspm en conjunto con grunt y/o gulp para seguir automatizando ciertas tareas (o usar scripts npm, como prefieras).
Si ya estás usando Webpack y/o Broswerify entonces no tienes por qué sustituirlos por jspm. Si ya has pagado el precio (en coste de configuración e integración) del uso de esas herramientas no hay necesidad de que las tires para atrás... pero no estaría mal que le echaras un ojo a jspm para futuros proyectos y veas si te convence y se adapta a lo que tu necesitas!



