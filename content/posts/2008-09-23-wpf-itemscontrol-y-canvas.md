---
title: '[WPF] ItemsControl y Canvas'

author: eiximenis

date: 2008-09-23T17:30:00+00:00
geeks_url: /?p=1421
geeks_visits:
  - 1940
geeks_ms_views:
  - 1250
categories:
  - wpf

---
Comentemos la jugada: Tenemos un ItemsControl (o un derivado de él como una ListBox) y queremos posicionar sus elementos dentro de un Canvas.

<!--more-->

La definición del ItemsControl puede ser algo parecido a:

```xml
<ItemsControl Margin=”5″ Name=”battleField”>
    <ItemsControl.Template>
        <ControlTemplate TargetType=”ItemsControl”>
            <Border BorderBrush=”Aqua” BorderThickness=”1″ CornerRadius=”15″>
                <ItemsPresenter />
            </Border>
        </ControlTemplate>
    </ItemsControl.Template>
    <ItemsControl.ItemsPanel>
        <ItemsPanelTemplate>                      
            <Canvas Width=”500″ Height=”300″ Background=”Yellow”></Canvas>
        </ItemsPanelTemplate>
    </ItemsControl.ItemsPanel>
</ItemsControl>
```

Hasta aquí nada especial:

  * Definimos el ControlTemplate para definir que forma tendrá todo el control en sí. En este caso un borde redondeado de un color azul tirando a cyan...
  * Definimos el ItemsPanel, que nos permite especificar que panel usará el control para posicionar sus elementos. En este caso queremos posicionar los elementos en coordenadas X e Y absolutas, por lo que ponemos un Canvas de un amarillo chillón que echa para atrás.

Ahora especificamos el `ItemTemplate`, que indica como renderizar cada uno de los elementos que contiene el control:

```xml
<ItemsControl.ItemTemplate >
    <DataTemplate>
        <Button Width=”60″ Height=”25″ Content=”{Binding Path=UnitName}”
               Canvas.Left=”{Binding Path=PosX}” Canvas.Top=”{Binding Path=PosY}”>
            <Button.RenderTransform>
                <SkewTransform AngleX=”-45″ AngleY=”0″ />
            </Button.RenderTransform>
        </Button>
    </DataTemplate>
</ItemsControl.ItemTemplate>
```

En este caso especificamos que cada elemento que se añada, se renderizará con un botón, al cual se le aplicará una transformacion (una SkewTransform en este caso). El contenido del botón será el valor de la propiedad &#8220;UnitName&#8221; y, aquí viene lo bueno, posicionaremos el botón donde indiquen los valores de las propiedades PosX y PosY.

En el code-behind tenemos el código para rellenar nuestro ItemsControl. Simplemente le añadimos objetos de cualquier clase que tenga las propiedades necesarias (una llamada UnitName, otra llamada PosX y otra llamada PosY).

```cs
for (int row = 1; row <= 9; row++)
{
    for (int col = 1; col <= 9; col++)
    {
        CasellaInfo ci = new CasellaInfo();
        ci.UnitName = string.Format(“{0}-{1}”, row, col);
        ci.PosX = col * 40;
        ci.PosY = row * 20;
        battleField.Items.Add(ci);
    }
}
battleField.Items.Refresh();
```

Siendo la clase `CasellaInfo`; algo parecido a:

```cs
class CasellaInfo
{
    public string UnitName { get; set; }
    public int PosX { get; set; }
    public int PosY { get; set; }
}
```

Ok! Todo listo! Ejecutamos y...

Aparecen los 81 botones... uno encima del otro, en las coordenadas 0,0 del Canvas (bueno, supongo que aparecen los 81 porque claro, yo sólo veo el último :P). Es decir, **los valores de Canvas.Top y Canvas.Left son ignorados**.

¿Entonces? Bueno, el truco está en que NO debemos aplicar los valores de `Canvas.Top` y `Canvas.Left` al `ItemTemplate`. En un `ItemsControl`, cada uno de los elementos (items) se coloca dentro de un control que se genera por cada elemento. Por defecto en un ItemsControl este control es del tipo `ContentPresenter`. El `ItemsControl` proporciona una propiedad, llamada `ItemContainerStyle` que permite especificar un estilo a aplicar a cada uno de esos contenedores:

```xml
<ItemsControl.ItemContainerStyle>
    <Style TargetType=”{x:Type ContentPresenter}”>                       
        <Setter Property=”Canvas.Left” Value=”{Binding Path=PosX}”/>
        <Setter Property=”Canvas.Top” Value=”{Binding Path=PosY}”/>
    </Style>
</ItemsControl.ItemContainerStyle>
```

Ahora sí! El `ItemsControl`, por cada elemento que tenga, creará un `ContentPresenter`, añadirá dentro de él todos los controles que hayamos especificado en el ItemTemplate (en nuestro caso simplemente un botón) y añadirá este `ContentPresenter` al `ItemsPanel` (que en nuestro caso era el Canvas). Además aplicará el estilo especificado en `ItemContainerStyle` a cada uno de los `ContentPresente`r creados, lo que los posicionará en las coordenadas que nosotros queremos.

Una vez visto esto está claro que son los `ContentPresenter` generados lo que tenemos que posicionar en el canvas (y no los `ItemTemplate`) pero hasta que uno no se da cuenta.... buf!!!

Saludos! 😉