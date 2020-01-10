---
title: 'Angular y React (1/n): Empezando'

author: eiximenis

date: 2014-10-22T18:11:45+00:00
geeks_url: /?p=1686
geeks_visits:
  - 1908
geeks_ms_views:
  - 1000
categories:
  - Uncategorized

---
¡Buenas! Empiezo con esta una serie de posts, que no se lo larga que será, comparando (en el buen sentido, nada de buscar un ganador ni un perdedor) un poco Angular con React.

Antes que nada el típico disclaimer: Angular y React **no cubren los mismos aspectos del desarrollo web**. Sí Angular cubre todo el espectro MVC, MVVM o como quieras llamarlo, React cubre solo la V: las vistas. Es pues, **de funcionalidad más limitada que Angular**. Así React puede combinarse con otras librerías para obtener un framework MVC completo. Hay quien lo ha hecho con Backbone (lógico, Backbone se puede combinar con _cualquier_ cosa) pero lo habitual es hacerlo con Flux. Pero bueno… hasta hay quien lo ha hecho con… ¡Angular!

Si no conoces nada de Angular, tranquilo que durante esta serie de posts, iremos explicando lo que sea necesario y lo mismo aplica a React. De todos modos para Angular <a href="https://twitter.com/XaviPaper" target="_blank" rel="noopener noreferrer">Xavier Jorge Cerdá</a> y <a href="https://twitter.com/_pedrohurtado" target="_blank" rel="noopener noreferrer">Pedro Hurtado</a> están escribiendo <a href="http://www.louesfera.com/2014/08/13/tutorial-angularjs-introduccion/" target="_blank" rel="noopener noreferrer">un tutorial en Louesfera</a>. Échale un vistazo. De React cuesta mucho más encontrar documentación en formato tutorial en castellano…

**Hello world**

Por supuesto, vamos a empezar con el Hello World, un ejemplo que generalmente es tan soso que no dice nada sobre lo que se quiere analizar, pero bueno… las tradiciones son tradiciones.

Ahí va el Hello World de Angular:

```html
<!DOCTYPE html>
<html ng-app="hello-app">        
<head lang="en">
    <meta charset="UTF-8">
    <title>Hello Angular</title>
    <script src="bower_components/angular/angular.js"></script>
    <script>
          angular.module('hello-app', []).controller('HelloController',function HelloController($scope) {
            $scope.name = "eiximenis";
        });
    </script>
</head>
<body>
    <div ng-controller="HelloController">
      Hello {{name}}
    </div>
</body>
</html>
```

Al ejecutar esta página se mostrará “Hello eiximenis” por pantalla.

Es un código muy simple pero sirve para ver tres de las características fundamentales de Angular:

  1. En Angular las vistas son HTML. Es decir están formadas por el propio DOM de la página. Puede parecer obvio (HTML va muy bien para definir aspecto visual) pero bueno, hay otras librerías (p. ej. Backbone) que usan código para definir las vistas.
  2. Hay un sistema de bindings para transferir datos desde el controlador (HelloController) hacia la vista (DOM). En este caso hemos usado la sintaxis más simple, al estilo <a href="http://mustache.github.io/" target="_blank" rel="noopener noreferrer">mustache</a>.
  3. El sistema de inyección de dependencias que tiene Angular (el parámetro $scope) está inyectado automáticamente por Angular. La idea es que se pueden inyectar aquellos servicios que se desee a los controladores.

Veamos el código equivalente en React:

```html
<!DOCTYPE html>
<html>
<head lang="en">
    <meta charset="UTF-8">
    <title>Hello react</title>
    <script src="bower_components/react/react.js"></script>
    <script src="bower_components/react/JSXTransformer.js"></script>
</head>
<body id="example">
    <script type="text/jsx">
        /** @jsx React.DOM */
        React.renderComponent(
          <div>Hello, eiximenis!</div>,        
          document.getElementById('example')
        );
    </script>
</html>
```

El ejemplo es muy simple, pero luce más complejo que el equivalente de Angular. Y eso es debido a la filosofía de React:

  * React se basa en el uso de pequeños componentes que cada uno de ellos se puede renderizar independientemente (algo parecido a lo que proponen librerías como Polymer, pero con la diferencia de que Polymer se basa en Web Components y extiende el DOM, mientras que React se basa en JavaScript y se olvida del DOM).
  * Las vistas **no son HTML (DOM)**, si no clases JavaScript. La idea es la misma que las vistas de Backbone, con la salvedad de que en Backbone se suele interaccionar con el DOM (usualmente a través de jQuery) y en React no.
  * React “extiende” JavaScript para permitir esta mezcla de html y JavaScript. Para ello se usa un parser específico, que está en el fichero JSXTransformer.js
  * No hay binding en React, porque no hay DOM hacia el que enlazar nada.
  * React tiene el concepto de “DOM Virtual”: no se interacciona nunca con el DOM real, si no con un “DOM Virtual” proporcionado por React. Luego es React quien se encarga de sincronizar este “DOM Virtual” con el DOM real del navegador. Eso es un “epic win” para SEO en SPA: **react puede renderizar vistas sin necesidad de un navegador**… se puede renderizar una vista React desde node p. ej. usando el mismo código en el servidor que en el cliente, y permitiendo así que los buscadores indexen nuestro sitio. Esto no es posible en Angular, ya que Angular está atado al DOM y el DOM requiere un navegador. **Ojo,** no digo que con Angular no se pueda generar aplicaciones SPA con buen soporte para SEO, solo digo que con React el mismo código sirve para renderizar en servidor y en cliente, mientras que con Angular la parte del servidor requiere código adicional. Además, teóricamente, este “DOM Virtual” permitiría a React generar otras cosas que no sean vistas en HTML… quien sabe.

Bueno… y para ser el primer post lo dejaremos aquí…