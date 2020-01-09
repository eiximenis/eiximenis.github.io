---
title: "Proyectos Open Source"
---

En mi perfil de [Github](https://github.com/eiximenis) puedes ver todos mis repositorios, pero aquí simplemente te cuento algunos de mis proyectos open source que desarrollé, estoy desarrollando o que tengo en mente...

## TVision2

[TVision2](https://github.com/eiximenis/Tvision2) es una libreria para desarrollar aplicaciones basadas en [TUI](https://es.wikipedia.org/wiki/Interfaz_de_texto) usando _.Net Core_. Está desarrollada de forma modular, de forma que solo "pagas por aquello que usas" y permite tanto el desarrollo de videojuegos de texto como de aplicaciones clásicas.

## Beatpulse

Con la gente de [Xabaril](https://github.com/Xabaril) colaboré en [Beatpulse](https://github.com/Xabaril/Beatpulse) una librería para agregar _liveness_ y _readiness_ checks a tus aplicaciones _.Net Core 2.1 o anteriores_. Con la salida de _.Net Core 2.2_ la librería quedó obsoleta (ya que .Net Core 2.2 ya ofrece soporte propio para _healthchecks_).

Si estás usando _.Net Core 2.2_ (o superior) y usas _healthchecks_ echa un vistazo a [AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) donde se copiaron todos los "checkers" implementados en _Beatpulse_ para usar con el nuevo sistema implementado en _.Net Core 2.2_.

## Core-Initializer

Un [pequeño proyecto](https://github.com/eiximenis/core-initializer) para diferir las tareas de inicialización en aplicaciones _.Net Core_, de forma que el _host_ se carga lo antes posible. Lo estoy rehaciendo para integrarlo con _healthchecks_ y el sistema de routing nuevo...

