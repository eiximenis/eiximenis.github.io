---
title: Adiós geeks! Nuevo blog :)
author: eiximenis
description: Estreno este nuevo blog. Te cuento las razónes y el por qué dejo geeks.ms. También te cuento como configurar github pages con dominio propio.
date: 2020-01-10
categories:
  - general
---

Buf... durante casi 12 años [geeks.ms](https://geeks.ms/etomas) ha sido mi casa. Pero todo el mundo se termina emancipando tarde o temprano y había llegado ya el momento de tener un nuevo hogar...

## ¿Por qué dejo geeks.ms?

Hay varias razones que desgranaré a contiunación, pero la principal: **el tener tus datos, tu blog en propiedad**. Mirad, os cuento... hace un porrón de años tuve un blog de desarrollo en una comunidad llamada _Clearscreen_. Allí iba yo publicando mis cosillas hasta que un buen día, de golpe y porrazo el servidor desapareció.

No, no culpo a nadie que administrase _clearscreen_: en aquellos tiempos bastante era que alguien te ofreciera un espacio gratuito para tener tu blog (eran épocas en que [blogger](https://blogger.com) apenas existía). En aquel momento no le dí mucha importancia, simplemente estuve buscando algún sitio donde escribir, hasta que algún tiempo después vi _geeks.ms_. Me armé de valor, le mandé un mail a [Rodrigo](https://twitter.com/r_corral) me presenté y le pedí que me diera un blog... Sorprendentemente lo hizo.

En aquel momento _geeks.ms_ estaba bien y además estaba allí la flor y la nata del desarrollo hispano. Vamos, creo que salvo [algún que otro blog excepcional](https://www.variablenotfound.com/) todo el resto estaba allí. Sí, sin duda no había mejor sitio para hospedar el blog.

Pero el tiempo fue pasando y varias cosas sucedieron: por un lado _geeks.ms_ estaba basado en _Community Server_ que se discontinuó y por lo tanto quedó obsoleto. Al principio no era grave, total, todo funcionaba. Pero con el paso del tiempo se le empezaron a ver las costuras y cuando internet pasó a ser _mobile first_, _Community Server_ quedó fuera de juego.

También hay que destacar que, con el tiempo, casi todo el mundo se fue de _geeks.ms_. La mayoría de gente abrió otros blogs y o bien abandonaron el de _geeks_ o bien lo convirtieron en un "mirror secundario" de su blog principal. Con el tiempo la gran comunidad de _geeks_ se fue esfumando y cada vez más era un desierto.

Aguanté bastante tiempo pero al final me cansé, y decidí buscar una alternativa. Y encontré [Jekyll](https://jekyllrb.com/). Debo reconocer que **me enamoré**: Básicamente seleccionabas un tema, creabas markdowns y publicabas tu blog. Era una maravilla. Así que me puse manos a la obra y creé un blog que hospedé en github pages (que tiene integración nativa con Jekyll). [Me despedí de Geeks](https://geeks.ms/etomas/2016/01/25/nuevo-blog-eiximenis-github-io/). El enamoramiento duró poco tiempo: En Windows, Jekyll era pesado, y por otro lado si usabas la integración de Github pages, algunos módulos no se podían usar y algunas funcionalidades que te funcionaban en local no lo hacían online. Pero no os engañaré, hubiese seguido con ese nuevo blog **si no hubiese sucedido un hecho importante**: _Geeks.ms_ se actualizó a Wordpress. Y se migraron los datos.

Con la migración de _Community Server_ a _Wordpress_ algunas cosas se perdieron: especialmente todas las descargas que estaban en Community Server no se migraron a _Wordpress_ por lo que muchos de los enlaces con código de los primeros posts, se esfumaron. No me "preocupó" demasiado (son posts muy viejos, obsoletos probablemente) pero bueno... otra vez el riesgo de no tener todos los datos. Pero ahora teníamos un Wordpress, moderno, _responsive_ y todo eso.

Ahí me pudo la pereza: Tenía todo mi blog en Wordpress, empezar de cero me daba mucha pereza. Así que volví a _geeks.ms. Pero la espinilla de tener mi blog en ficheros `.md` locales ya estaba clavada...

Por otro, cada vez me daba "más miedo" (no es la palabra exacta) no ser propietario de mis datos: todos mis posts de los últimos años estaban ahí, en un _wordpress_ que no era propiedad mía y aunque no temía por su desaparición, veía que se estaba convirtiendo en un erial, que la gran comunidad que había detrás se había esfumado y que, honestamente, no tenía mucho sentido permanecer allí. Sumad eso a que, por alguna razón que desconozco, el filtro de comentarios no va bien (y todos van a spam) y algunos detalles más. Hace algún tiempo lo tuve claro: si quería potenciar el blog (un objetivo de este 2020) debía irme de _geeks_ a un sitio con más libertad y sobre todo, ser propietario de mis datos.

## La migración a Hugo

Al final opté por usar [Hugo](https://gohugo.io/). Hugo, es como Jekyll pero mucho más rápido y mucho más sencillo de usar (al menos en Windows donde configurar _Ruby_ es un dolor). Con el tiempo ha ido ganando en popularidad y ahora ya hay [muchos temas a elegir](https://themes.gohugo.io/). Eso era importante para mi, que soy un negado en el tema de diseño.

La duda era ¿como pasaba todo el contenido viejo de _Geeks.ms_ a formato _Markdown_ para incluírlo en el nuevo blog? Para ello al final usé [Wordpress to Hugo exporter](https://github.com/SchumacherFM/wordpress-to-hugo-exporter) que básicamente me generó todos los _markdowns_ de mis posts. La verdad es que hizo lo que pudo, pero el resultado es aceptable. Así al menos ahora tengo todos mis posts antiguos en el nuevo blog. Hay que reconocer que la gran mayoría quizá no aporten valor a día de hoy, hablando de cosas obsoletas, pero **son la historia de mi blog** y simplemente, no quería empezar otro blog, quería mover el existente de sitio.

Para ejecutar este exportador necesitaba modificar unos PHPs del código de _Wordpress_ de _Geeks.ms_ y eso no era una opción. Así que hice un paso intermedio, asumiendo que con eso podía perder algunos datos.

Primero exporté desde geeks.ms todos mis posts a formato xml. Eso exporta solo el contenido, no las imágenes. Luego, me instalé un _Wordpress_ en local e importé el xml. El resultado: un _Wordpress_ local con los mismos posts que el de _geeks.ms_. Dado que este era mi _wordpress_ local, ahí si que podía modificar los PHPs necesarios. Lo hice y corrí el exportador.

Para tener my _wordpress_ local, usé (como igual te puedes imaginar xD) _Docker_:

```yaml
version: "3"

services:
  mariadb:
    image: mariadb
    stop_grace_period: 30s
    environment:
      MYSQL_ROOT_PASSWORD: Passw0rd!
      MYSQL_DATABASE: wp
      MYSQL_USER: wp
      MYSQL_PASSWORD: Passw0rd!

  wordpress:
    image: wordpress
    environment:
      DB_HOST: mariadb
      DB_USER: wp
      DB_PASSWORD: Passw0rd!
      DB_NAME: wp
    ports:
    - 8080:80
```

El exportador me generó todos los markdowns y luego solo se trató de buscar un tema. Al final encontré [Zzo](https://themes.gohugo.io/hugo-theme-zzo/) que oye, se ve bien, tiene todo lo que necesito y **soportaba los infames markdowns que tenía**. Lo de infames no es culpa del exportador obviamente. Es culpa de que los HTMLs que había en el _wordpress_ eran infumables por culpa del coloreado de sintaxis. Vamos, que tengo código como este en el markdown:

```html
<divre class="code"></divre><span style="color: blue"><</span><span style="color: #a31515">Window </span><span style="color: red">x</span><span style="color: blue">:</span><span style="color: red">Class</span><span style="color: blue">=&#8221;DockReader.Window1&#8243; </span><span style="color: red">xmlns</span><span style="color: blue">=&#8221;http://schemas.microsoft.com/winfx/2006/xaml/presentation&#8221; </span><span style="color: red">xmlns</span><span style="color: blue">:</span><span style="color: red">x</span><span style="color: blue">=&#8221;http://schemas.microsoft.com/winfx/2006/xaml&#8221; </span><span style="color: red">Title</span><span style="color: blue">=&#8221;Window1&#8243; </span><span style="color: red">Height</span><span style="color: blue">=&#8221;300&#8243; </span><span style="color: red">Width</span><span style="color: blue">=&#8221;300&#8243; </span><span style="color: red">xmlns</span><span style="color: blue">:</span><span style="color: red">ad</span><span style="color: blue">=&#8221;clr-namespace:AvalonDock;assembly=AvalonDock&#8221;> <</span><span style="color: #a31515">Grid</span><span style="color: blue">> <</span><span style="color: #a31515">ad</span><span style="color: blue">:</span><span style="color: #a31515">DockingManager </span><span style="color: red">Name</span><span style="color: blue">=&#8221;dockManager&#8221;/> </</span><span style="color: #a31515">Grid</span><span style="color: blue">> </</span><span style="color: #a31515">Window</span><span style="color: blue">> </span>

<pre></pre>
```

Eso es un código coloreado de cuando _geeks.ms_ iba con el community server y copiaba y pegaba código como HTML mediante una extensión de Visual Studio. Muchos de los temas no procesaban bien este código (no les culpo). No sé si es por la forma en como convertían de _markdown_ al HTML final, o lo que sea, pero vamos... tampoco me preocupa mucho. Con el tiempo no descarto ir quitando esas aberraciones y pasarlo a formato _markdown_, pero no es algo que me quite el sueño. 

## Hosting en Github pages

El siguiente paso fue elegir donde hospedar el blog. A ver, Hugo al igual que Jekyll es un "generador estático" eso significa que a partir de tus _markdowns_ más unos ficheros de configuración y un tema, generan un directorio que contiene solo ficheros estáticos (htmls, css, js). No hay APIs, no hay bases de datos. Para publicar el blog, debes publicar esos ficheros estáticos generados, no necesitas para nada los _markdowns_. Eso implica que _Hugo_ es independiente de donde hospedes tu blog.

Al final opté por _Github pages_. Hay otras opciones como [Netlify](https://www.netlify.com/), pero _Github pages_ tiene todo lo que necesito:

1. Es totalmente gratuita
2. Soporta dominios personalizados y https
3. Y se puede integrar con Hugo usando [Github Actions](https://github.com/features/actions)

Así la estrategia es la siguiente. Tengo un [repositorio](https://github.com/eiximenis/eiximenis.github.io) que contiene mi blog y que está publicado mediante _github pages_. Este repo tiene dos ramas:

1. `master`: Es la que se publica mediante _github pages_ y contiene el código generado por Hugo
2. `hugo`: Hay el código de mi blog (los _markdowns_, la configuración y el tema).

Luego hay un _workflow_ que cuando se hace un _push_ a `hugo` utiliza Hugo para generar los ficheros estáticos y los publica en la rama `master`:

```yaml
name: github pages
on:
  push:
    branches:
      - hugo
jobs:
  build-deploy:
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@v1
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2.3.2
        with:
          hugo-version: "latest"
          extended: true
      - name: Build
        run: hugo --gc --minify --cleanDestinationDir
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v2.3.2
        env:
          ACTIONS_DEPLOY_KEY: ${{ secrets.ACTIONS_DEPLOY_KEY }}
          PUBLISH_BRANCH: master
          PUBLISH_DIR: ./public
```

Este es el código de mi _workflow_. Debo dar las gracias a [Carlos](https://twitter.com/cmendibl3) por hacerme ver esa estrategia. Básicamente copié el _workflow_ que tiene él en su blog. El gran mérito es para [Shohei Ueda](https://github.com/peaceiris) que tiene publicadas las _actions_ necesarias:

* [Acciones para usar Hugo](https://github.com/peaceiris/actions-hugo)
* [Acciones para publicar en github pages](https://github.com/peaceiris/actions-gh-pages)

De este modo cuando hago un _push_ a la rama `hugo`, se ejecuta el _workflow_ que genera mi blog otra vez y lo republica a la rama _master_ donde es servido por _github pages_.

Por supuesto, dado que mi blog es realmente un repositorio _git_ puedo editar lo que quiera en local y publicarlo cuando tenga conexión y, la verdad, usar un editor como Visual Studio Code o similar para editar el markdown es una maravilla. Mucho más rápido que cualquier WYSIWYG que puedas necesitar. Además, que mientras editas puedes tener Hugo en marcha que te sirve el blog en `localhost:1313` por lo que puedes ver tus cambios en tiempo real: guardas, Hugo regenera y se actualiza la página automáticamente.

## ¿Y qué ocurre con los comentarios?

No tenemos base de datos, así que **no hay sistema de comentarios**. Por suerte se puede integrar fácilmente _Hugo_ con [Disqus](https://disqus.com/). Así que los comentarios están en _Disqus_. Es la única parte del blog que "no me pertenece" (en el sentido de que no la tengo en local).

## Dominio propio

Bueno, ya que me mudaba pensaba hacerlo a lo grande. Así que me pillé un dominio (`eiximenis.dev`) y lo configuré para usar con _github pages_. En mi caso quería usar el dominio tal cual (sin prefijo), lo que se conoce como "dominio apex". La recomendación de _Github_ es que si lo haces, crees también el subdominio `www` y lo configures. Si lo haces "bien" _github pages_ es capaz de crear una redirección HTTP entre ambos dominios. Os voy a contar como lo he configurado yo, porque no es tan trivial como parece.

Hay que hacer varias cosas. La primera es indicarle a _github pages_ que el blog está servido en el dominio que quieres. Eso se puede hacer desde la configuración del blog **pero no lo hagas desde allí**. En el fondo, si actualizas este valor, lo que se hace es generar un _commit_ con un fichero llamado `CNAME`. Eso es lo que espera realmente _github pages_: el fichero `CNAME`.

No crees tu el _commit_ manualmente en la rama `master` (yo lo hice y la pifié). La razón es que cuando republiques tu post, todo el contenido del directorio de la rama se borra y se sustituye por el nuevo generado por _Hugo_, por lo que tu fichero `CNAME` se pederá también. En su lugar **crea el fichero `CNAME` en el directorio `/static` de la rama `hugo`**. Cuando _Hugo_ genera el blog, el contenido de `/static` se copia tal cual en la raíz, por lo que al publicar vía _Hugo_, el fichero `CNAME` te aparecerá allí. El contenido de dicho fichero es simplemente el dominio a usar:

```
www.eiximenis.dev
```

**Sí, con el subdominio www**. ¡Esa es la clave! Para _github_ ya lo tienes todo configurado. Ahora debes ir a tu _registrar_ de DNS y configurar el dominio:

* El dominio apex (`eiximenis.dev` en mi caso) debe tener cuatro registros `A` que apunten a las 4 IPs de _github pages_:
  * `185.199.111.153`
  * `185.199.110.153`
  * `185.199.109.153`
  * `185.199.108.153`
* El subdominio www (`www.eiximenis.dev` en mi caso) debe tener un registro `CNAME` que apunte a `<tu-usuario>.github.io`

Puedes verificarlo usando el comando `dig`:

![Salida del comando "dig www.eiximenis.dev" donde se ven los registros A y el CNAME](/images/posts/2020-01-10-dig.png)

Dale tiempo a que se propaguen los DNS y listos: puedes acceder mediante tu dominio APEX o el dominio `www`, ya que _github pages_ crea una redirección desde el dominio APEX al dominio `www`:

![Salida de curl a eiximenis.dev donde se ve el 301](/images/posts/2020-01-10-curl.png)

¡Y listos! Ya tienes tu blog publicado, un último paso que puedes hacer es añadir tu blog a [Cloudfare](https://www.cloudflare.com/) y así disfrutar de todo lo que ofrecen (¡que no es poco!).

Honestamente que **solo por el coste del dominio** tengas un blog editable en local, hospedado, con soporte para TLS y todos los servicios de protección DNS que ofrece _Cloudfare_ es algo que me lo dicen hace unos años... y no me lo creo :)

Saludos y... nos seguimos viendo por ahí.