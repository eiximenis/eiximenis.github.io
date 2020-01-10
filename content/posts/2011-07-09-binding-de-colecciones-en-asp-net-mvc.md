---
title: Binding de colecciones en ASP.NET MVC

author: eiximenis

date: 2011-07-09T12:32:13+00:00
geeks_url: /?p=1571
geeks_visits:
  - 6942
geeks_ms_views:
  - 4655
categories:
  - Uncategorized

---
Buenas! Hoy voy a comentar un temilla que me comentó un colega el otro día y que puede dar _algunos_ quebraderos de cabeza: el binding de colecciones.

<!--more-->

Supongamos el siguiente viewmodel:

<div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> QuestionModel<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">int</span> IdQuestion {get; set;}<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Text { get; set; }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">int</span> IdAnswer { get; set; }   <span style="color: #008000">// Id de la respuesta seleccionada</span><br />    <span style="color: #0000ff">public</span> IEnumerable&lt;Answer&gt; Answers { get; set; }<br />}<br /><br /><span style="color: #0000ff">public</span> <span style="color: #0000ff">class</span> Answer<br />{<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">int</span> IdAnswer{ get; set; }<br />    <span style="color: #0000ff">public</span> <span style="color: #0000ff">string</span> Text { get; set; }<br />}<br /></pre>
  
  <p>
    </div> 
    
    <p>
      Básicamente una pregunta contiene un Id, un texto y una lista de respuestas. Bien, supongamos que tenemos un método en el controlador que nos cree una lista de 10 preguntas, cada una de las cuales con 3 posibles respuestas, y la mande a una vista:
    </p>
    
    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">public</span> ActionResult Index()<br />{<br />    var questionModelList = <span style="color: #0000ff">new</span> List&lt;QuestionModel&gt;();<br />    <span style="color: #0000ff">for</span> (var i = 1; i &lt;= 10; i++)<br />    {<br />        questionModelList.Add(<span style="color: #0000ff">new</span> QuestionModel()<br />                                  {<br />                                      IdQuestion = i,<br />                                      Text = <span style="color: #006080">"Texto pregunta "</span> + i,<br />                                      IdAnswer = 0,<br />                                      Answers = <span style="color: #0000ff">new</span> List&lt;Answer&gt;() {<br />                                                        <span style="color: #0000ff">new</span> Answer() {Text = <span style="color: #006080">"Respuesta A de "</span> + i, IdAnswer = 1},<br />                                                        <span style="color: #0000ff">new</span> Answer() {Text = <span style="color: #006080">"Respuesta B de "</span> + i, IdAnswer = 2},<br />                                                        <span style="color: #0000ff">new</span> Answer() {Text = <span style="color: #006080">"Respuesta C de "</span> + i, IdAnswer = 3} }<br />                                  });<br />    }<br />    <span style="color: #0000ff">return</span> View(questionModelList);<br />}<br /></pre>
      
      <p>
        </div> 
        
        <p>
          Al igual que a la vista le mandamos una <em>List<QuestionModel></em>, esperamos que eso sea lo que la vista nos devuelva en el controlador:
        </p>
        
        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">[HttpPost]<br /><span style="color: #0000ff">public</span> ActionResult Index(List&lt;QuestionModel&gt; questionModelList)<br />{<br />    <span style="color: #008000">// codigo...</span><br />}<br /></pre>
          
          <p>
            </div> 
            
            <p>
              Bueno, ahora veamos una <em>posible</em> implementación de la vista Index:
            </p>
            
            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">@model List<span style="color: #0000ff">&lt;</span><span style="color: #800000">Ejemplo.ConRadioButtonNormal.Models.QuestionModel</span><span style="color: #0000ff">&gt;</span><br />@{<br />    ViewBag.Title = "Index";<br />}<br />@using (Html.BeginForm())<br />{<br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br />        @foreach (var question in Model)<br />        {<br />            <span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br />            @Html.LabelFor(x=<span style="color: #0000ff">&gt;</span>question.Text, question.Text)<br />            @foreach (var answer in question.Answers)<br />            {<br />                @Html.RadioButtonFor(m =<span style="color: #0000ff">&gt;</span> answer.Text, answer.IdAnswer)                <br />            }<br />            <span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br />        }<br />    <span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">p</span><span style="color: #0000ff">&gt;&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="submit"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="Submit"</span> <span style="color: #0000ff">/&gt;&lt;/</span><span style="color: #800000">p</span><span style="color: #0000ff">&gt;</span><br />}</pre>
              
              <p>
                </div> 
                
                <p>
                  Buenooo… parece una implementación correcta no? Iteramos sobre todas las preguntas y por cada pregunta generamos una label y tantas radio buttons como respuestas tenga la pregunta… El problema es que… no es correcta! 🙂
                </p>
                
                <p>
                  Veamos el código HTML que nos genera:
                </p>
                
                <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                  <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">label</span> <span style="color: #ff0000">for</span><span style="color: #0000ff">="question_Text"</span><span style="color: #0000ff">&gt;</span>Texto pregunta 1<span style="color: #0000ff">&lt;/</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="answer_Text"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="answer.Text"</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="radio"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="1"</span> <span style="color: #0000ff">/&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="answer_Text"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="answer.Text"</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="radio"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="2"</span> <span style="color: #0000ff">/&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="answer_Text"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="answer.Text"</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="radio"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="3"</span> <span style="color: #0000ff">/&gt;</span>            <br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">label</span> <span style="color: #ff0000">for</span><span style="color: #0000ff">="question_Text"</span><span style="color: #0000ff">&gt;</span>Texto pregunta 2<span style="color: #0000ff">&lt;/</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="answer_Text"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="answer.Text"</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="radio"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="1"</span> <span style="color: #0000ff">/&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="answer_Text"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="answer.Text"</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="radio"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="2"</span> <span style="color: #0000ff">/&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">id</span><span style="color: #0000ff">="answer_Text"</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="answer.Text"</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="radio"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="3"</span> <span style="color: #0000ff">/&gt;</span>            <br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #008000">&lt;!-- Y así sucesivamente... :p --&gt;</span></pre>
                  
                  <p>
                    </div> 
                    
                    <p>
                      Fijaos que tanto el id como el name de todas las radiobutton son iguales! Y eso? Eso es porque name e id se sacan de la lambda expresion que se pasa como parametro al helper <em>Html.RadioButtonFor<></em>. Y esa lambda <strong>es siempre la misma</strong>. Bueno… al menos el error es fácil de detectar porque una vez marqueis una radio button, al marcar <em>cualquier otra de cualquier otra pregunta</em> la primera se seleccionará. Recordad que HTML <em>agrupa</em> las radiobuttons según el atributo “name”, y si todas tienen el mismo, todas las radiobutton forman un solo grupo, y por lo tanto sólo una puede estar seleccionada.
                    </p>
                    
                    <blockquote>
                      <p>
                        <strong>Nota: </strong>Recordad la clave: la expresión lambda que se pasa a Html.RadioButtonFor<> es la que se usa para generar el valor del atributo <em>name</em>.
                      </p>
                    </blockquote>
                    
                    <p>
                      La siguiente pregunta que debemos hacernos es… <em>cual debe ser el valor correcto de los atributos </em>name<em> de cada radiobutton</em>? Para eso que ver como el DefaultModelBinder enlaza los datos cuando hay colecciones, y por suerte la regla es muy sencilla. Si estamos enlazando datos de una colección el DefaultModelBinder espera que el valor del atributo <em>name</em> tenga el formato: <em>parametro[idx]</em>. Es decir, analicemos la estructura de clases que el controlador espera.
                    </p>
                    
                    <p>
                      El controlador espera una List<QuestionModel> llamada questionModelList. De ese modo:
                    </p>
                    
                    <ul>
                      <li>
                        questionModelList[0].Text se enlazaría con la propiedad Text del primer elemento de la lista.
                      </li>
                      <li>
                        questionModelList[1].Text se enlazaría con la propiedad Text del primer elemento de la lista.
                      </li>
                      <li>
                        questionModelList[0].Answers[0].Id se enlazaria con el valor de la propiedad Id de la primera respuesta del primer elemento de la lista.
                      </li>
                      <li>
                        questionModelList[0].Answers[1].Id se enlazaria con el valor de la propiedad Id de la segunda respuesta del primer elemento de la lista.
                      </li>
                    </ul>
                    
                    <p>
                      ¿Lo veis? El DefaultModelBinder espera que el valor del atributo <em>name</em> sea “lo mismo que usaríamos en C#”.
                    </p>
                    
                    <blockquote>
                      <p>
                        <strong>Nota:</strong> Si te “hace daño a los ojos” lo de questionModelList al principio del atributo <em>name</em> (recuerdo que esto es el nombre del parámetro), que sepas que si el controlador sólo recibe una colección (como en nuestro caso) puedes eliminar el nombre del parámetro. En nuestro caso podriamos usar [0].Text como valor del atributo <em>name</em> p.ej.
                      </p>
                    </blockquote>
                    
                    <p>
                      Así, p.ej. podríamos modificar la vista para que quede parecida a:
                    </p>
                    
                    <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                      <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">@model List<span style="color: #0000ff">&lt;</span><span style="color: #800000">Ejemplo.ConRadioButtonNormal.Models.QuestionModel</span><span style="color: #0000ff">&gt;</span><br />@{<br />    ViewBag.Title = "Index";<br />}<br />@using (Html.BeginForm())<br />{<br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br />    @for (int idx = 0; idx <span style="color: #0000ff">&lt;</span> Model.Count; idx++)<br />    {<br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span>@Model[idx].Text<span style="color: #0000ff">&lt;/</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span><br />        @for (int idxAnswer = 0; idxAnswer <span style="color: #0000ff">&lt;</span> Model[idx].Answers.Count(); idxAnswer++)<br />        {<br />            var lstAnswers = Model[idx].Answers.ToList();<br />            <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="radio"</span> <br />                <span style="color: #ff0000">name</span><span style="color: #0000ff">="@string.Format("</span><span style="color: #ff0000">questionModelList</span>[{<span style="color: #ff0000"></span>}].<span style="color: #ff0000">Answers</span>[<span style="color: #ff0000"></span>].<span style="color: #ff0000">IdAnswer</span><span style="color: #0000ff">", idx)"</span><br />                <span style="color: #ff0000">value</span><span style="color: #0000ff">="@lstAnswers[idxAnswer].IdAnswer"</span> <span style="color: #0000ff">/&gt;</span><br />        }<br />        <span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br />    }<br />    <span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">p</span><span style="color: #0000ff">&gt;&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="submit"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="Submit"</span> <span style="color: #0000ff">/&gt;&lt;/</span><span style="color: #800000">p</span><span style="color: #0000ff">&gt;</span><br />}</pre>
                      
                      <p>
                        </div> 
                        
                        <p>
                          Eso genera un HTML como el siguiente:
                        </p>
                        
                        <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                          <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet"><span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span>Texto pregunta 1<span style="color: #0000ff">&lt;/</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="radio"</span> <br />        <span style="color: #ff0000">name</span><span style="color: #0000ff">="questionModelList[0].Answers[0].IdAnswer"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="1"</span> <span style="color: #0000ff">/&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="radio"</span> <br />        <span style="color: #ff0000">name</span><span style="color: #0000ff">="questionModelList[0].Answers[0].IdAnswer"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="2"</span> <span style="color: #0000ff">/&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="radio"</span> <br />        <span style="color: #ff0000">name</span><span style="color: #0000ff">="questionModelList[0].Answers[0].IdAnswer"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="3"</span> <span style="color: #0000ff">/&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br /><span style="color: #0000ff">&lt;</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span>Texto pregunta 2<span style="color: #0000ff">&lt;/</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="radio"</span> <br />        <span style="color: #ff0000">name</span><span style="color: #0000ff">="questionModelList[1].Answers[0].IdAnswer"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="1"</span> <span style="color: #0000ff">/&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="radio"</span> <br />        <span style="color: #ff0000">name</span><span style="color: #0000ff">="questionModelList[1].Answers[0].IdAnswer"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="2"</span> <span style="color: #0000ff">/&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="radio"</span> <br />        <span style="color: #ff0000">name</span><span style="color: #0000ff">="questionModelList[1].Answers[0].IdAnswer"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="3"</span> <span style="color: #0000ff">/&gt;</span><br /><span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span></pre>
                          
                          <p>
                            </div> 
                            
                            <p>
                              Ahora los campos siguen la convención que espera el <em>DefaultModelBinder</em>. Con lo que los datos ahora si que serán recibidos por el controlador.
                            </p>
                            
                            <p>
                              <strong>Bueno… ¿pero esto que sentido tiene?</strong>
                            </p>
                            
                            <p>
                              Si os fijáis veréis que las radiobuttons de cada pregunta tienen todas el mismo name. Eso es correcto ya que sólo una respuesta puede seleccionarse por cada pregunta. Así todas las radiobuttons de la primera pregunta tienen el valor de <em>questionModelList[0].Answers[0].IdAnswer</em> y así sucesivamente.
                            </p>
                            
                            <p>
                              Así pues, cual es la información que viaja del cliente al servidor? Aunque tengamos tres radiobuttons por pregunta, todas tienen el mismo name y sólo una viajará. Es decir, dadas las 10 radiobuttons viajarán 10 valores del cliente al servidor:
                            </p>
                            
                            <ul>
                              <li>
                                <em>questionModelList[0].Answers[0].IdAnswer</em>
                              </li>
                              <li>
                                <em>questionModelList[1].Answers[0].IdAnswer</em>
                              </li>
                              <li>
                                …
                              </li>
                              <li>
                                <em>questionModelList[9].Answers[0].IdAnswer</em>
                              </li>
                            </ul>
                            
                            <p>
                              Es decir, para el servidor tendremos una lista de 10 preguntas cada una de las cuales tendrá UNA sola respuesta (la seleccionada por el usuario). Y eso es exactamente lo que se bindea en el controlador:
                            </p>
                            
                            <p>
                              <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_6E4CA3A9.png"><img style="background-image: none; border-right-width: 0px; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_12255537.png" width="244" height="116" /></a>
                            </p>
                            
                            <p>
                              Fijaos como p.ej. el campo Text de la pregunta es “null”. Normal, la vista no lo está enviando al controlador.
                            </p>
                            
                            <p>
                              Parémonos un momento en este punto. Desde el controlador hemos mandado a la vista <strong>toda</strong> una List<QuestionModel> llena. Es decir la vista si que recibe el texto de las preguntas y las respuestas… Por que cuando estamos de vuelta en el controlador hemos perdido esta información?
                            </p>
                            
                            <p>
                              Si te estás preguntando esto: bienvenido al mundo stateless de la web. Efectivamente el controlador envía toda una List<QuestionModel> a la vista y la vista usa esta información para generar un HTML, que es mandado al cliente. En este punto el trabajo del servidor <em>ha terminado</em>. El controlador muere y será creado de nuevo cuando se reciba otra petición del browser. Esta otra petición llega cuando pulsamos el botón de submit, pero el controlador <em>sólo recibe lo que la vista envía</em>. Y la vista sólo envia lo que está en el formulario: los IDs de las respuestas seleccionadas. La vista <strong>no</strong> envia los textos ni de la pregunta, ni de la respuesta. Por eso no los tenemos de vuelta. Recordad: la web es stateless. Si venís de Webforms, webforms os ocultaba esto y os hacía creer que la web es statefull. Pero MVC no oculta nada de nada: lo que enseña es, simplemente, lo que hay.
                            </p>
                            
                            <p>
                              Bueno… dicho esto, ahora preguntaos, si tiene sentido que este método del controlador reciba una lista de <em>QuestionModel</em>. El tema es que si esta vista tiene que enviar las respuestas eso debería ser lo que debería esperar el controlador. Es decir, la vista <em>recibe</em> un <em>List<QuestionModel></em> pero <em>devuelve</em> un array con los IDs de las preguntas seleccionadas… Total… la vista no va a <em>devolver</em> el texto de las preguntas y las respuestas al controlador, no? Si lo que la vista devuelve son los IDs de las respuestas seleccionadas esto es lo que debería recibir el controlador:
                            </p>
                            
                            <p>
                              <a href="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_5540AD97.png"><img style="background-image: none; border-right-width: 0px; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; border-top-width: 0px; border-bottom-width: 0px; border-left-width: 0px; padding-top: 0px" title="image" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_thumb_5F00_1A7904C1.png" width="244" height="97" /></a>
                            </p>
                            
                            <p>
                              Fijaos simplemente que el controlador recibe una colección con los IDs de las respuestas seleccionados. Así idRespuestas[0] el ID de la respuesta seleccionada de la primera pregunta y así sucesivamente. La información del texto de las preguntas (en caso de ser necesaria) la obtendría el propio controlador (leyéndola de la BBDD, de la cache, de donde fuera).
                            </p>
                            
                            <p>
                              Y como nos quedaría la vista? Pues como ahora no mandamos una estructura tan <em>compleja</em> al controlador, generar el nombre de los atributos <em>name</em> es mucho más fácil:
                            </p>
                            
                            <div style="border-bottom: silver 1px solid; text-align: left; border-left: silver 1px solid; padding-bottom: 4px; line-height: 12pt; background-color: #f4f4f4; margin: 20px 0px 10px; padding-left: 4px; width: 97.5%; padding-right: 4px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; max-height: 200px; font-size: 8pt; overflow: auto; border-top: silver 1px solid; cursor: text; border-right: silver 1px solid; padding-top: 4px" id="codeSnippetWrapper">
                              <pre style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: &#39;Courier New&#39;, courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px" id="codeSnippet">@model List<span style="color: #0000ff">&lt;</span><span style="color: #800000">Ejemplo.ConRadioButtonNormal.Models.QuestionModel</span><span style="color: #0000ff">&gt;</span><br />@{<br />    ViewBag.Title = "Index";<br />}<br />@using (Html.BeginForm())<br />{<br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br />    @for (int idx = 0; idx <span style="color: #0000ff">&lt;</span> Model.Count; idx++)<br />    {<br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br />        <span style="color: #0000ff">&lt;</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span>@Model[idx].Text<span style="color: #0000ff">&lt;/</span><span style="color: #800000">label</span><span style="color: #0000ff">&gt;</span><br />        @for (int idxAnswer = 0; idxAnswer <span style="color: #0000ff">&lt;</span> Model[idx].Answers.Count(); idxAnswer++)<br />        {<br />            var lstAnswers = Model[idx].Answers.ToList();<br />            <span style="color: #0000ff">&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="radio"</span> <br />                <span style="color: #ff0000">name</span><span style="color: #0000ff">="@string.Format("</span>[{<span style="color: #ff0000"></span>}]<span style="color: #0000ff">", idx)"</span><br />                <span style="color: #ff0000">value</span><span style="color: #0000ff">="@lstAnswers[idxAnswer].IdAnswer"</span> <span style="color: #0000ff">/&gt;</span><br />        }<br />        <span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br />    }<br />    <span style="color: #0000ff">&lt;/</span><span style="color: #800000">div</span><span style="color: #0000ff">&gt;</span><br />    <span style="color: #0000ff">&lt;</span><span style="color: #800000">p</span><span style="color: #0000ff">&gt;&lt;</span><span style="color: #800000">input</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="submit"</span> <span style="color: #ff0000">value</span><span style="color: #0000ff">="Submit"</span> <span style="color: #0000ff">/&gt;&lt;/</span><span style="color: #800000">p</span><span style="color: #0000ff">&gt;</span><br />}</pre>
                              
                              <p>
                                </div> 
                                
                                <p>
                                  Vale, fijaos en dos cosillas:
                                </p>
                                
                                <ol>
                                  <li>
                                    La vista <strong>sigue recibiendo una List<QuestionModel></strong>. Normal, porque esto es lo que el controlador le sigue mandando. Pero eso <strong>NO</strong> implica que la vista deba mandar de vuelta esto mismo!
                                  </li>
                                  <li>
                                    El valor de los atributos “name” de las distintas radiobutton, es simplemente [0],[1],… y así sucesivamente (recordad como hemos comentado antes que si el controlador recibe una sola colección no es necesario poner el nombre del parámetro en el atributo <em>name</em>).
                                  </li>
                                </ol>
                                
                                <p>
                                  <strong>Una nota final…</strong>
                                </p>
                                
                                <p>
                                  Bueno, hemos visto como funciona el binding de colecciones en ASP.NET MVC, pero os quiero comentar sólo una cosilla más 🙂
                                </p>
                                
                                <p>
                                  Que ocurriria si le enviasemos al controlador los siguientes campos (sus valores son irrelevantes)?
                                </p>
                                
                                <ol>
                                  <li>
                                    [0], [1], [3], [4]
                                  </li>
                                  <li>
                                    [1], [2], [3], [4<sub></sub>]
                                  </li>
                                </ol>
                                
                                <p>
                                  En el primer caso vemos que falta [2]. Entonces el DefaultModelBinder se para en este punto. Es decir el controlador recibirá una colección con dos elementos (los valores de [0] y [1]. Los valores de [3] y [4] se han perdido).
                                </p>
                                
                                <p>
                                  En el segundo caso vemos que falta el primer elemento [0]. Entonces el DefaultModelBinder ni enlaza el parámetro. Es decir el controlador no recibirá una colección vacía, no… recibirá null en el parámetro.
                                </p>
                                
                                <p>
                                  En un próximo post veremos como podemos evitar esto 😉
                                </p>
                                
                                <p>
                                  Un saludo!
                                </p>
                                
                                <p>
                                  ds
                                </p>