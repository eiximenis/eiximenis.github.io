---
title: "Validando con FluentValidation en Blazor"
author: eiximenis
description: "Estos días estoy trasteando con Blazor y me preguntaba como se podría integrar FluentValidation en Blazor, de forma que los controles se validen en base a las validaciones definidas en FluentValidation"
date: 2020-10-21
draft: false
categories:
  - blazor
tags:
---

Blazor tiene un soporte para formularios bastante bien montado. Por un lado, existe el componente "padre", que representa al propio formulario. Este componente padre (solemos utilizar [EditForm](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.components.forms.editform?view=aspnetcore-5.0&WT.mc_id=AZ-MVP-4039791)) es el encargado de crear "el contexto de edición" y proveerlo a todos los componentes hijos que están dentro de él. Este contexto consta básicamente de un modelo (el objeto que se está editando) y el estado de la validación de sus propiedades.
 
Dentro de un `EditForm` usamos los componentes de formulario (tales como [`InputText`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.components.forms.inputtext?view=aspnetcore-5.0&WT.mc_id=AZ-MVP-4039791)). Esos controles consultan en dicho contexto de edición si el valor de la propiedad es válido o inválido y se renderizan en consecuencia. Pero esos componentes no modifican nunca el estado de una propiedad. Esto se reserva a otros componentes (que también deben ser hijos del `EditForm`), que son los validadores. Así pues la responsabilidad está claramente separada:

* El formulario padre (`EditForm`) provee del contexto de edición
* Los componentes de edición usan dicho contexto para consultar si el valor de la propiedad a la que están enlazados es correcto o no (y renderizarse en consecuencia)
* Los validadores validan los valores de las propiedades, modificando el contexto de edición

Blazor viene de serie con un componente de validación que es el [`DataAnnotationsValidator`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.components.forms.dataannotationsvalidator?view=aspnetcore-5.0&WT.mc_id=AZ-MVP-4039791) que examina el modelo asociado al contexto y busca atributos de _Data Annotations_. Así si tenemos un modelo tal que:

```csharp
public class UserData 
{
  [Required]
  public string Name {get; set;}
}
```

Podemos crear un formulario para editarlo **y validarlo** de forma sencilla:

```html
<EditForm Model="@User">
  <DataAnnotationsValidator />
  <InputText @bind-Value="Name" >
</EditForm>
```

Ahora bien, si en lugar de _Data Annotations_ prefieres usar algun otro mecanismo de validación, ¿como lo puedes implementar? Pues la solución pasa por crearte un componente validador. [En la documentación está muy bien explicado](https://docs.microsoft.com/en-us/aspnet/core/blazor/forms-validation?view=aspnetcore-5.0&WT.mc_id=AZ-MVP-4039791#validator-components) aunque el ejemplo es un poco rebuscado. En este post **os voy a mostrar como crear un validador que valide en base al código de [FluentValidation](https://fluentvalidation.net/) que tengáis**.

## Creando el validador de FluentValidation

Primero, el código entero y luego lo comentamos por partes :)

```csharp
  public class FluentValidationValidator : ComponentBase
  {
      [CascadingParameter] EditContext CurrentEditContext { get; set; }
      protected override void OnInitialized()
      {
          if (CurrentEditContext == null)
          {
              throw new InvalidOperationException("You must use FluentValidationValidator inside an EditForm or any other EditContext provider");
          }
          CurrentEditContext.AddFluentValidation();
      }
  }
```

Este es el componente, que lo único que hace es recojer el contexto de edición que provee el `EditForm`. Para ello declara una propiedad de tipo `EditContext` decorada con `[CascadingParameter]`, ya que `EditForm` provee del contexto de edición usando una [cascading value](https://docs.microsoft.com/en-us/aspnet/core/blazor/components/cascading-values-and-parameters?view=aspnetcore-5.0&WT.mc_id=AZ-MVP-4039791).

Hay trampa claro, ese código depende del método `AddFluentValidation` que es un método de extensión sobre `EditContext`, y que es el que realmente hace el trabajo.

```csharp
    static class EditContextExtensions
    {
        public static EditContext AddFluentValidation(this EditContext editContext)
        {
            if (editContext == null)
            {
                throw new ArgumentNullException(nameof(editContext));
            }

            var messages = new ValidationMessageStore(editContext);

            editContext.OnValidationRequested +=
                (sender, eventArgs) => ValidateModel((EditContext)sender, messages);

            editContext.OnFieldChanged +=
                (sender, eventArgs) => ValidateField(editContext, messages, eventArgs.FieldIdentifier);

            return editContext;
        }

        private static void ValidateModel(EditContext editContext, ValidationMessageStore messages)
        {
            var validator = GetValidatorForModel(editContext.Model);
            var context = CreateValidationContextForModel(editContext.Model);
            var validationResults = validator.Validate(context);
            messages.Clear();
            foreach (var validationResult in validationResults.Errors)
            {
                messages.Add(editContext.Field(validationResult.PropertyName), validationResult.ErrorMessage);
            }
            editContext.NotifyValidationStateChanged();
        }

        private static void ValidateField(EditContext editContext, ValidationMessageStore messages, in FieldIdentifier fieldIdentifier)
        {

            var properties = new[] { fieldIdentifier.FieldName };

            var context = CreateValidationContextForProperties(fieldIdentifier.Model, properties);
            var validator = GetValidatorForModel(fieldIdentifier.Model);
            var validationResults = validator.Validate(context);

            messages.Clear(fieldIdentifier);
            messages.Add(fieldIdentifier, validationResults.Errors.Select(error => error.ErrorMessage));
            editContext.NotifyValidationStateChanged();
        }

        private static IValidationContext CreateValidationContextForProperties(object model, string[] properties)
        {
            var valType = typeof(ValidationContext<>).MakeGenericType(model.GetType());
            return (IValidationContext)Activator.CreateInstance(valType, new[] { model, new PropertyChain(), new MemberNameValidatorSelector(properties) })!;
        }

        private static IValidationContext CreateValidationContextForModel(object model)
        {
            var valType = typeof(ValidationContext<>).MakeGenericType(model.GetType());
            return (IValidationContext)Activator.CreateInstance(valType, new[] { model });
        }

        private static IValidator GetValidatorForModel(object model)
        {
            var abstractValidatorType = typeof(AbstractValidator<>).MakeGenericType(model.GetType());
            var modelValidatorType = Assembly.GetExecutingAssembly().GetTypes().FirstOrDefault(t => t.IsSubclassOf(abstractValidatorType));
            var modelValidatorInstance = (IValidator)Activator.CreateInstance(modelValidatorType)!;
            return modelValidatorInstance;
        }
    }
```

Ahora sí, podemos ir método a método.

1. El método `AddFluentValidation` es el método inicial que lo que hace es **suscribirse a los eventos que lanza el `EditContext`**. Hay dos eventos a los que debemos suscribirnos: `OnFieldChanged` que se lanza cuando un componente de edición pide validar la propiedad a la que está enlazado y `OnValidationRequested` que se lanza cuando se debe validar todo el formulario. Así en el primero debemos validar solo una propiedad y en el segundo pues todo el modelo. Luego este método **también crea la `ValidationMessageStore` que es, básicamente, un diccionario de propiedades y errores de validación asociados**.
2. El método `ValidateModel` es el que se ejecuta en respuesta al evento `OnValidationRequested` **y a partir de ese punto ya todo es código de FluentValidation**: Se obtiene el validador asociado al modelo (`GetValidatorForModel`) y se crea el contexto de validación de FluentValidation (`CreateValidationContextForModel`), luego se valida el modelo, **se añaden los errores a la `ValidationMessageStore`** y finalmente **se notifica al `EditContext` que ha habido cambios en el estado de la validación**.
3. El método `ValidateField` es parecido, simplemente se cambia como se crea el contexto de validación de FluentValidation (ahora es un contexto para validar solo una propiedad no todo el modelo). Para ello se usa el parámetro `FieldIdentifier` que (básicamente) contiene el nombre de la propiedad que debemos validar.

Luego los siguientes métodos son métodos usados por esos dos, y son un poco "complejos" porque se debe usar reflection. La API de FluentValidation no está pensada para "obtener un validador de un tipo que solo conocemos en runtime", pero vamos... tampoco son gran cosa. Por ejemplo, para validar nuestro tipo `UserData` se debe crear un `ValidationContext<UserData>`. Eso está perfecto si conoces en tiempo de compilación el tipo, pero no es nuestro caso, de ahí la necesidad de usar reflection. Lo mismo ocurre con el método `GetValidatorForModel`, que debe crear una instancia de cualquier clase que herede de `AbstractValidator<UserData>`.

## Usando nuestro validador

Bueno, eso no tiene ningún secreto. Ahora, en lugar de decorar con `[Required]` la propiedad `Name` de `UserData` tendríamos que crear un validador de FluentValidation:

```csharp
public class UserDataValidator : AbstractValidator<UserData> {
  public UserDataValidator() {
    RuleFor(u => u.Name).NotEmpty();
  }
}
```

Y en nuestro `EditForm` en lugar de usar el `DataAnnotationsValidator`, pues usamos nuestro nuevo validador:

```html
<EditForm Model="@User">
  <FluentValidationValidator />
  <InputText @bind-Value="Name" >
</EditForm>
```

¡Y listos! Ya hemos integrado FluentValidation en nuestro proyecto de Blazor.

## Combinando los validadores

Un formulario **no debe por que estar validado por un único validador**. Este modelo de validación en el que los validadores son componentes adicionales que comparten el `EditContext` provisto por el `EditForm` permite que en un mismo formulario haya dos, tres o los validadores que sea:

```html
<EditForm Model="@User">
  <DataAnnotationsValidator />
  <FluentValidationValidator />
  <InputText @bind-Value="Name" >
</EditForm>
```

Ahora se aplicarán tanto las validaciones de _Data Annotations_ como las de FluentValidation. Un validador no puede interferir con las validaciones que aplique otro validador (para ello deberían compartir el `ValidationMessageStore` y en este caso cada uno está creando el suyo propio). Así si `UserData` tuviera tanto el `[Required]` aplicado y también existiese el validador de FluentValidation, si dejas el nombre en blanco recibirias dos errores en la propiedad `Name`. Si p. ej. usaras un `<ValidationSummary>` para mostrar el resúmen de errores, te aparecerían dos mensjaes de error asociados al nombre del usuario.

Y... ¡nada más!