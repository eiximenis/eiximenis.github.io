---
title: 'PRISM y Winforms: Mostrar vistas en nuevos formularios'
description: 'PRISM y Winforms: Mostrar vistas en nuevos formularios'
author: eiximenis

date: 2009-03-11T08:58:00+00:00
geeks_url: /?p=1441
geeks_visits:
  - 1417
geeks_ms_views:
  - 717
categories:
  - Uncategorized

---
En un post anterior ([PRISM y Winforms][1]), comentaba como usar [PRISM][2] para realizar aplicaciones Winforms.

Un comentario de Jose en esta entrada, me ha motivado a escribir el siguiente post, para mostrar como podríamos mostrar vistas en regiones que estén incrustadas no en un UserControl (típicamente un Panel) de la ventana principal, sino incrustadas en un nuevo formulario.

Para poder usar regiones en Winforms era necesario definirnos un RegionAdapter para la clase &ldquo;Control&rdquo; que era básicamente el objetivo del post anterior. En el método Adapt() teníamos el código que &ldquo;incrustaba&rdquo; la vista dentro del control. Dicho método recibía (de PRISM) la región y el target, o control donde colocar dicha región.

Antes que nada recordad tres conceptos que a veces se confunden:

  1. Region: En PRISM una región es _un conjunto_ de vistas (algunas activas, otras no activas) que se muestran en algún sitio determinado.
  2. Target: El target de una región es el lugar donde se muestran. P.ej. un TableLayout podría ser el target de una región y cada vista de la región podría mostrarse en distintas celdas.
  3. Vista: Una vista muestra una determinada información. P.ej. si tenemos una aplicación que nos muestra la cotización de varias acciones, podríamos tener varias instancias de una vista, donde cada instancia nos mostraría la cotización de una acción. En winforms generalmente es un UserControl.

En PRISM las vistas se colocan en regiones y las regiones se incrustan en los targets... recordad que una región puede tener varias vistas.

En el RegionAdapter que vimos en el post anterior, siempre recibíamos la región y el target. El target ya estaba creado porque era un control ya existente en la ventana principal.

Esto **no** nos sirve si queremos mostrar una vista en un formulario nuevo ya que ahora debemos crear el formulario cada vez que queramos mostrar la región... ¿como podemos hacerlo?

Además del método Adapt() que usamos en el post anterior, los RegionAdapter pueden redefinir otro método llamado AttachBehaviors. En este método podemos añadir la lógica que queramos para personalizar el comportamiento de la región... en este caso podremos aprovechar para crear el target.

**Un vistazo al código...**

El código que sigue a continuación es una adaptación de la clase WindowRegionAdapter de [Composite WPF Contrib][3], que he adaptado para que funcione con Windows Forms.

El método AttachBehaviors lo redefinimos de la siguiente manera:

<pre class="code"><span style="color: blue">protected override void </span>AttachBehaviors(<span style="color: #2b91af">IRegion </span>region,
    <span style="color: #2b91af">Form </span>regionTarget)
{
    <span style="color: blue">base</span>.AttachBehaviors(region, regionTarget);
    <span style="color: #2b91af">FormRegionBehavior </span>behavior =
        <span style="color: blue">new </span><span style="color: #2b91af">FormRegionBehavior
            </span>(regionTarget, region, <span style="color: #2b91af">FormBorderStyle</span>.FixedSingle);
    behavior.Attach();
}</pre>

[][4][][4]

Los parámetros que recibe son los mismos que Adapt: la región y el target (en este caso un Form) ya que derivamos de RegionAdapterBase<Form>. En este método creamos un objeto FormRegionBehavior que será el que tendrá _todo el código_ para gestionar regiones que estén dentro de un formulario. En concreto dicha clase será la responsable de:

  1. Crear un formulario por cada nueva vista añadida a la región y incrustar dicha vista en el formulario
  2. Cerrar el formulario que contiene una vista si esta se elimina de la región.
  3. Eliminar una vista de la región si se cierra el formulario que la contiene.
  4. Activar o desactivar la vista cuando su formulario es activado o desactivado.

A continuación pongo el código más relevante de dicha clase. Al final del post adjunto una aplicación de demo.

**1 y 2. Crear un formulario por cada nueva vista y eliminar el formulario de una vista eliminada**

El método Attach() de la clase FormRegionBehavior de suscribe a los dos eventos CollectionChanged de las colecciones Views y ActiveViews de la región:

<pre class="code"><span style="color: blue">internal void </span>Attach()
{
    <span style="color: #2b91af">IRegion </span>region = _regionWeakReference.Target <span style="color: blue">as </span><span style="color: #2b91af">IRegion</span>;
    <span style="color: blue">if </span>(region != <span style="color: blue">null</span>)
    {
        region.Views.CollectionChanged +=
            <span style="color: blue">new </span><span style="color: #2b91af">NotifyCollectionChangedEventHandler
                </span>(Views_CollectionChanged);
        region.ActiveViews.CollectionChanged +=
            <span style="color: blue">new </span><span style="color: #2b91af">NotifyCollectionChangedEventHandler
                </span>(ActiveViews_CollectionChanged);
    }
}</pre>

En el método Views_CollectionChanged es donde sabemos si se ha añadido una vista a la región o se ha eliminado, y así podemos crear o destruir el formulario asociado:

<pre class="code"><span style="color: blue">private void </span>Views_CollectionChanged(<span style="color: blue">object </span>sender,<br /><span style="color: #2b91af">NotifyCollectionChangedEventArgs </span>e)
{
    <span style="color: #2b91af">Form </span>owner = _ownerWeakReference.Target <span style="color: blue">as </span><span style="color: #2b91af">Form</span>;
    <span style="color: blue">if </span>(owner == <span style="color: blue">null</span>)
    {
        Detach();
        <span style="color: blue">return</span>;
    }
    <span style="color: blue">if </span>(e.Action == <span style="color: #2b91af">NotifyCollectionChangedAction</span>.Add)
    {
        <span style="color: blue">foreach </span>(<span style="color: blue">object </span>view <span style="color: blue">in </span>e.NewItems)
        {
            <span style="color: green">// Creamos un formulario por cada vista y lo mostramos...
</span>        }
    }
    <span style="color: blue">else if </span>(e.Action == <span style="color: #2b91af">NotifyCollectionChangedAction</span>.Remove)
    {
        <span style="color: blue">foreach </span>(<span style="color: blue">object </span>view <span style="color: blue">in </span>e.OldItems)
        {
            <span style="color: green">// Buscamos el formulario que contiene cada vista
            // para cerrarlo y "disposarlo" 🙂
</span>        }
    }
}</pre>

[][4]

**3. Eliminar la vista de la región si se cierra el formulario**

Para ello, cuando creamos un formulario, nos suscribimos al evento Closed para poder eliminar la vista asociada a la región:

<pre class="code"><span style="color: blue">private void </span>Form_Closed(<span style="color: blue">object </span>sender, <span style="color: #2b91af">EventArgs </span>e)
{
    <span style="color: #2b91af">Form </span>frm = sender <span style="color: blue">as </span><span style="color: #2b91af">Form</span>;
    <span style="color: #2b91af">IRegion </span>region = _regionWeakReference.Target
        <span style="color: blue">as </span><span style="color: #2b91af">IRegion</span>;
    <span style="color: blue">if </span>(frm != <span style="color: blue">null </span>&& frm.Controls.Count &gt; 0
        && region != <span style="color: blue">null</span>)
        <span style="color: blue">if </span>(region.Views.Contains(frm.Controls[0]))
            region.Remove(frm.Controls[0]);
}</pre>

[][4][][4]

**4. Activar o desactivar la vista cuando su formulario es activado o desactivado**

Para ello, cuando creamos un formulario nos suscribimos a los eventos Activated y Deactivate, para desactivar o activar la vista correspondiente:

<pre class="code"><span style="color: blue">private void </span>Form_Activated(<span style="color: blue">object </span>sender, <span style="color: #2b91af">EventArgs </span>e)
{
    <span style="color: #2b91af">IRegion </span>region = _regionWeakReference.Target
        <span style="color: blue">as </span><span style="color: #2b91af">IRegion</span>;
    <span style="color: #2b91af">Form </span>frm = sender <span style="color: blue">as </span><span style="color: #2b91af">Form</span>;
    <span style="color: blue">if </span>(frm != <span style="color: blue">null </span>&& frm.Controls.Count &gt; 0 &&
        !region.ActiveViews.Contains(frm.Controls[0]))
        region.Activate(frm.Controls[0]);
}</pre>

[][4]

**5. Algunas consideraciones finales**

Para PRISM toda región tiene un solo contenedor (o target) que es aquel que se usa cuando se llama a AttachNewRegion. Aunque este RegionAdapter va creando distintos formularios, para PRISM la región debe estar vinculada a un único target... que debe existir cuando llamemos a&nbsp; AttachNewRegion y cuyo tipo debe ser Form puesto que nuestro RegionAdapter trabaja con targets de tipo Form.

Esto nos deja en una situación curiosa, puesto que necesitamos tener un Form creado para poder crear la región. Aunque luego este formulario **no** contendr&agrave; ninguna de las vistas de la región. Consideraremos a este formulario el formulario _padre_, y cuando el formulario padre _muera_ el RegionAdapter dejar&agrave; de tratar a la región (digamos que la región habrá muerto).

Otra cosa que también debemos solucionar es dado una vista, encontrar que formulario la contiene. El RegionAdapter no se guarda una lista de los formularios creados, en su lugar solo tiene una WeakReference al formulario _padre_ (el usado para llamar a AttachNewRegion y que nunca contendrá vistas de esta región) y usa el formulario padre para encontrar a los formularios que contiene la vista. Esto es posible porque al crear un formulario que contiene una vista se indica que su _padre_ es el formulario padre:

<pre class="code"><span style="color: #2b91af">Form </span>owner = _ownerWeakReference.Target <span style="color: blue">as </span><span style="color: #2b91af">Form</span>;<br /><span style="color: green">// Creamos un formulario por cada vista y lo mostramos...</span><br /><br />frm.Owner = owner;</pre>

[][4]

De este modo se puede usar la propiedad OwnedForms del formulario _padre_ para iterar sobre todos los formularios creados y ver cual contiene la vista en cuestión.

Finalmente, el hecho de tener que pasar un formulario ya existente a AttachNewRegion, no es excesivamente problemático: lo más fácil es usar el formulario principal del programa!

Adjunto el programa de demostración. El RegionAdapter DialogRegionAdapter es el que muestra vistas en nuevos formularios, mientras que el RegionAdapter ControlRegionAdapter las muestra en paneles (de hecho en cualquier control).

[Descargar el código de demostración][5].

Espero que os sea útil! 🙂

 [1]: /blogs/etomas/archive/2009/03/06/141026.aspx
 [2]: http://www.codeplex.com/CompositeWPF
 [3]: http://compositewpfcontrib.codeplex.com/
 [4]: http://11011.net/software/vspaste
 [5]: /cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas.11032009/PrismWFApp.zip