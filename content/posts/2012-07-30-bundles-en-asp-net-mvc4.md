---
title: Bundles en ASP.NET MVC4
author: eiximenis

date: 2012-07-30T17:08:11+00:00
geeks_url: /?p=1605
geeks_visits:
  - 10197
geeks_ms_views:
  - 9512
categories:
  - Uncategorized

---
¡Buenas! Este va a ser un post cortito, sobre los Bundles en ASP.NET MVC. Los bundles es el mecanismo que tiene ASP.NET MVC para incluir varios ficheros (de script o css) que están relacionados entre ellos.

Si os creáis un proyecto de ASP.NET MVC4 nuevo (sin que sea la plantilla Empy, claro) veréis el siguiente código en la página de Layout:

<div id="codeSnippetWrapper" style="overflow: auto; cursor: text; font-size: 8pt; border-top: silver 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; border-right: silver 1px solid; border-bottom: silver 1px solid; padding-bottom: 4px; direction: ltr; text-align: left; padding-top: 4px; padding-left: 4px; margin: 20px 0px 10px; border-left: silver 1px solid; line-height: 12pt; padding-right: 4px; max-height: 200px; width: 97.5%; background-color: #f4f4f4">
  <div id="codeSnippet" style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4">
    <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum1" style="color: #606060">   1:</span> <span style="color: #0000ff">&lt;!</span><span style="color: #800000">DOCTYPE</span> <span style="color: #ff0000">html</span><span style="color: #0000ff">&gt;</span></pre>
    
    <p>
      <!--CRLF-->
    </p>
    
    <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum2" style="color: #606060">   2:</span> <span style="color: #0000ff">&lt;</span><span style="color: #800000">html</span><span style="color: #0000ff">&gt;</span></pre>
    
    <p>
      <!--CRLF-->
    </p>
    
    <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum3" style="color: #606060">   3:</span> <span style="color: #0000ff">&lt;</span><span style="color: #800000">head</span><span style="color: #0000ff">&gt;</span></pre>
    
    <p>
      <!--CRLF-->
    </p>
    
    <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum4" style="color: #606060">   4:</span>     <span style="color: #0000ff">&lt;</span><span style="color: #800000">meta</span> <span style="color: #ff0000">charset</span><span style="color: #0000ff">="utf-8"</span> <span style="color: #0000ff">/&gt;</span></pre>
    
    <p>
      <!--CRLF-->
    </p>
    
    <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum5" style="color: #606060">   5:</span>     <span style="color: #0000ff">&lt;</span><span style="color: #800000">meta</span> <span style="color: #ff0000">name</span><span style="color: #0000ff">="viewport"</span> <span style="color: #ff0000">content</span><span style="color: #0000ff">="width=device-width"</span> <span style="color: #0000ff">/&gt;</span></pre>
    
    <p>
      <!--CRLF-->
    </p>
    
    <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum6" style="color: #606060">   6:</span>     <span style="color: #0000ff">&lt;</span><span style="color: #800000">title</span><span style="color: #0000ff">&gt;</span>@ViewBag.Title<span style="color: #0000ff">&lt;/</span><span style="color: #800000">title</span><span style="color: #0000ff">&gt;</span></pre>
    
    <p>
      <!--CRLF-->
    </p>
    
    <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum7" style="color: #606060">   7:</span>     @Styles.Render("~/Content/themes/base/css", "~/Content/css")</pre>
    
    <p>
      <!--CRLF-->
    </p>
    
    <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum8" style="color: #606060">   8:</span>     @Scripts.Render("~/bundles/modernizr")</pre>
    
    <p>
      <!--CRLF-->
    </p>
    
    <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum9" style="color: #606060">   9:</span> <span style="color: #0000ff">&lt;/</span><span style="color: #800000">head</span><span style="color: #0000ff">&gt;</span></pre>
    
    <p>
      <!--CRLF-->
    </p>
    
    <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum10" style="color: #606060">  10:</span> <span style="color: #0000ff">&lt;</span><span style="color: #800000">body</span><span style="color: #0000ff">&gt;</span></pre>
    
    <p>
      <!--CRLF-->
    </p>
    
    <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum11" style="color: #606060">  11:</span>     @RenderBody()</pre>
    
    <p>
      <!--CRLF-->
    </p>
    
    <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum12" style="color: #606060">  12:</span>&#160; </pre>
    
    <p>
      <!--CRLF-->
    </p>
    
    <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum13" style="color: #606060">  13:</span>     @Scripts.Render("~/bundles/jquery")</pre>
    
    <p>
      <!--CRLF-->
    </p>
    
    <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum14" style="color: #606060">  14:</span>     @RenderSection("scripts", required: false)</pre>
    
    <p>
      <!--CRLF-->
    </p>
    
    <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum15" style="color: #606060">  15:</span> <span style="color: #0000ff">&lt;/</span><span style="color: #800000">body</span><span style="color: #0000ff">&gt;</span></pre>
    
    <p>
      <!--CRLF-->
    </p>
    
    <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum16" style="color: #606060">  16:</span> <span style="color: #0000ff">&lt;/</span><span style="color: #800000">html</span><span style="color: #0000ff">&gt;</span></pre>
    
    <p>
      <!--CRLF--></div> </div> 
      
      <p>
        En lugar de incluir directamente las etiquetas <link> (para las CSS) o <script> para referenciar a archivos js externos, lo que tenemos son llamadas a Styles.Render y Scripts.Render.
      </p>
      
      <p>
        El parámetro que se pasa a esos métodos Render es el nombre del bundle a renderizar. ¿Y donde están definidos los bundles? Pues en el archivo App_Start/BundleConfig tenéis el siguiente método:
      </p>
      
      <div id="codeSnippetWrapper" style="overflow: auto; cursor: text; font-size: 8pt; border-top: silver 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; border-right: silver 1px solid; border-bottom: silver 1px solid; padding-bottom: 4px; direction: ltr; text-align: left; padding-top: 4px; padding-left: 4px; margin: 20px 0px 10px; border-left: silver 1px solid; line-height: 12pt; padding-right: 4px; max-height: 200px; width: 97.5%; background-color: #f4f4f4">
        <div id="codeSnippet" style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4">
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum1" style="color: #606060">   1:</span> public static void RegisterBundles(BundleCollection bundles)</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum2" style="color: #606060">   2:</span> {</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum3" style="color: #606060">   3:</span>     bundles.Add(new ScriptBundle("~/bundles/jquery").Include(</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum4" style="color: #606060">   4:</span>                 "~/Scripts/jquery-1.*"));</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum5" style="color: #606060">   5:</span>&#160; </pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum6" style="color: #606060">   6:</span>     bundles.Add(new ScriptBundle("~/bundles/jqueryui").Include(</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum7" style="color: #606060">   7:</span>                 "~/Scripts/jquery-ui*"));</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum8" style="color: #606060">   8:</span>&#160; </pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum9" style="color: #606060">   9:</span>     bundles.Add(new ScriptBundle("~/bundles/jqueryval").Include(</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum10" style="color: #606060">  10:</span>                 "~/Scripts/jquery.unobtrusive*",</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum11" style="color: #606060">  11:</span>                 "~/Scripts/jquery.validate*"));</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum12" style="color: #606060">  12:</span>&#160; </pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum13" style="color: #606060">  13:</span>     bundles.Add(new ScriptBundle("~/bundles/modernizr").Include(</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum14" style="color: #606060">  14:</span>                 "~/Scripts/modernizr-*"));</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum15" style="color: #606060">  15:</span>&#160; </pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum16" style="color: #606060">  16:</span>     bundles.Add(new StyleBundle("~/Content/css").Include("~/Content/site.css"));</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum17" style="color: #606060">  17:</span>&#160; </pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum18" style="color: #606060">  18:</span>     bundles.Add(new StyleBundle("~/Content/themes/base/css").Include(</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum19" style="color: #606060">  19:</span>                 "~/Content/themes/base/jquery.ui.core.css",</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum20" style="color: #606060">  20:</span>                 "~/Content/themes/base/jquery.ui.resizable.css",</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum21" style="color: #606060">  21:</span>                 "~/Content/themes/base/jquery.ui.selectable.css",</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum22" style="color: #606060">  22:</span>                 "~/Content/themes/base/jquery.ui.accordion.css",</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum23" style="color: #606060">  23:</span>                 "~/Content/themes/base/jquery.ui.autocomplete.css",</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum24" style="color: #606060">  24:</span>                 "~/Content/themes/base/jquery.ui.button.css",</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum25" style="color: #606060">  25:</span>                 "~/Content/themes/base/jquery.ui.dialog.css",</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum26" style="color: #606060">  26:</span>                 "~/Content/themes/base/jquery.ui.slider.css",</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum27" style="color: #606060">  27:</span>                 "~/Content/themes/base/jquery.ui.tabs.css",</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum28" style="color: #606060">  28:</span>                 "~/Content/themes/base/jquery.ui.datepicker.css",</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum29" style="color: #606060">  29:</span>                 "~/Content/themes/base/jquery.ui.progressbar.css",</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum30" style="color: #606060">  30:</span>                 "~/Content/themes/base/jquery.ui.theme.css"));</pre>
          
          <p>
            <!--CRLF-->
          </p>
          
          <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum31" style="color: #606060">  31:</span> }</pre>
          
          <p>
            <!--CRLF--></div> </div> 
            
            <p>
              Dicho método, que es llamado desde el Application_Start, es donde se asocia cada bundle con un conjunto de archivos (fijaos que se puede usar * para indicar un conjunto de archivos).
            </p>
            
            <p>
              Así vemos que el bundle ~/bundles/modernizr se compone de los archivos que estén en ~/scripts y cuyo nombre empiece por mondernizr-
            </p>
            
            <p>
              Esto en tiempo de ejecución no hace nada especial, la verdad. Si abrimos cualquier vista y le damos a “ver código fuente” lo que vemos es simplemente el conjunto de tags <link> o <script>:
            </p>
            
            <div id="codeSnippetWrapper" style="overflow: auto; cursor: text; font-size: 8pt; border-top: silver 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; border-right: silver 1px solid; border-bottom: silver 1px solid; padding-bottom: 4px; direction: ltr; text-align: left; padding-top: 4px; padding-left: 4px; margin: 20px 0px 10px; border-left: silver 1px solid; line-height: 12pt; padding-right: 4px; max-height: 200px; width: 97.5%; background-color: #f4f4f4">
              <div id="codeSnippet" style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4">
                <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum1" style="color: #606060">   1:</span> <span style="color: #0000ff">&lt;</span><span style="color: #800000">link</span> <span style="color: #ff0000">href</span><span style="color: #0000ff">="/Content/themes/base/jquery.ui.core.css"</span> <span style="color: #ff0000">rel</span><span style="color: #0000ff">="stylesheet"</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text/css"</span> <span style="color: #0000ff">/&gt;</span></pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum2" style="color: #606060">   2:</span> <span style="color: #0000ff">&lt;</span><span style="color: #800000">link</span> <span style="color: #ff0000">href</span><span style="color: #0000ff">="/Content/themes/base/jquery.ui.resizable.css"</span> <span style="color: #ff0000">rel</span><span style="color: #0000ff">="stylesheet"</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text/css"</span> <span style="color: #0000ff">/&gt;</span></pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum3" style="color: #606060">   3:</span> <span style="color: #0000ff">&lt;</span><span style="color: #800000">link</span> <span style="color: #ff0000">href</span><span style="color: #0000ff">="/Content/themes/base/jquery.ui.selectable.css"</span> <span style="color: #ff0000">rel</span><span style="color: #0000ff">="stylesheet"</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text/css"</span> <span style="color: #0000ff">/&gt;</span></pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum4" style="color: #606060">   4:</span> <span style="color: #0000ff">&lt;</span><span style="color: #800000">link</span> <span style="color: #ff0000">href</span><span style="color: #0000ff">="/Content/themes/base/jquery.ui.accordion.css"</span> <span style="color: #ff0000">rel</span><span style="color: #0000ff">="stylesheet"</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text/css"</span> <span style="color: #0000ff">/&gt;</span></pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum5" style="color: #606060">   5:</span> <span style="color: #0000ff">&lt;</span><span style="color: #800000">link</span> <span style="color: #ff0000">href</span><span style="color: #0000ff">="/Content/themes/base/jquery.ui.autocomplete.css"</span> <span style="color: #ff0000">rel</span><span style="color: #0000ff">="stylesheet"</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text/css"</span> <span style="color: #0000ff">/&gt;</span></pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum6" style="color: #606060">   6:</span> <span style="color: #0000ff">&lt;</span><span style="color: #800000">link</span> <span style="color: #ff0000">href</span><span style="color: #0000ff">="/Content/themes/base/jquery.ui.button.css"</span> <span style="color: #ff0000">rel</span><span style="color: #0000ff">="stylesheet"</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text/css"</span> <span style="color: #0000ff">/&gt;</span></pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum7" style="color: #606060">   7:</span> <span style="color: #0000ff">&lt;</span><span style="color: #800000">link</span> <span style="color: #ff0000">href</span><span style="color: #0000ff">="/Content/themes/base/jquery.ui.dialog.css"</span> <span style="color: #ff0000">rel</span><span style="color: #0000ff">="stylesheet"</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text/css"</span> <span style="color: #0000ff">/&gt;</span></pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum8" style="color: #606060">   8:</span> <span style="color: #0000ff">&lt;</span><span style="color: #800000">link</span> <span style="color: #ff0000">href</span><span style="color: #0000ff">="/Content/themes/base/jquery.ui.slider.css"</span> <span style="color: #ff0000">rel</span><span style="color: #0000ff">="stylesheet"</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text/css"</span> <span style="color: #0000ff">/&gt;</span></pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum9" style="color: #606060">   9:</span> <span style="color: #0000ff">&lt;</span><span style="color: #800000">link</span> <span style="color: #ff0000">href</span><span style="color: #0000ff">="/Content/themes/base/jquery.ui.tabs.css"</span> <span style="color: #ff0000">rel</span><span style="color: #0000ff">="stylesheet"</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text/css"</span> <span style="color: #0000ff">/&gt;</span></pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum10" style="color: #606060">  10:</span> <span style="color: #0000ff">&lt;</span><span style="color: #800000">link</span> <span style="color: #ff0000">href</span><span style="color: #0000ff">="/Content/themes/base/jquery.ui.datepicker.css"</span> <span style="color: #ff0000">rel</span><span style="color: #0000ff">="stylesheet"</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text/css"</span> <span style="color: #0000ff">/&gt;</span></pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum11" style="color: #606060">  11:</span> <span style="color: #0000ff">&lt;</span><span style="color: #800000">link</span> <span style="color: #ff0000">href</span><span style="color: #0000ff">="/Content/themes/base/jquery.ui.progressbar.css"</span> <span style="color: #ff0000">rel</span><span style="color: #0000ff">="stylesheet"</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text/css"</span> <span style="color: #0000ff">/&gt;</span></pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum12" style="color: #606060">  12:</span> <span style="color: #0000ff">&lt;</span><span style="color: #800000">link</span> <span style="color: #ff0000">href</span><span style="color: #0000ff">="/Content/themes/base/jquery.ui.theme.css"</span> <span style="color: #ff0000">rel</span><span style="color: #0000ff">="stylesheet"</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text/css"</span> <span style="color: #0000ff">/&gt;</span></pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum13" style="color: #606060">  13:</span> <span style="color: #0000ff">&lt;</span><span style="color: #800000">link</span> <span style="color: #ff0000">href</span><span style="color: #0000ff">="/Content/site.css"</span> <span style="color: #ff0000">rel</span><span style="color: #0000ff">="stylesheet"</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text/css"</span> <span style="color: #0000ff">/&gt;</span></pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum14" style="color: #606060">  14:</span>&#160; </pre>
                
                <p>
                  <!--CRLF-->
                </p>
                
                <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum15" style="color: #606060">  15:</span> <span style="color: #0000ff">&lt;</span><span style="color: #800000">script</span> <span style="color: #ff0000">src</span><span style="color: #0000ff">="/Scripts/modernizr-2.0.6-development-only.js"</span> <span style="color: #ff0000">type</span><span style="color: #0000ff">="text/javascript"</span><span style="color: #0000ff">&gt;&lt;/</span><span style="color: #800000">script</span><span style="color: #0000ff">&gt;</span></pre>
                
                <p>
                  <!--CRLF--></div> </div> 
                  
                  <p>
                    Pero lo bueno de usar los bundles no es tan solo poder agrupar lógicamente nuestros archivos javascript o css. Lo bueno es que estamos a una sola propiedad de distancia de poder agruparlos y minificarlos (pésima palabreja lo sé, pero soy incapaz de encontrar una traducción para <a href="http://en.wikipedia.org/wiki/Minification_(programming)" target="_blank" rel="noopener noreferrer">minification</a>).
                  </p>
                  
                  <p>
                    Así, si en el Application_Start añadimos la línea:
                  </p>
                  
                  <div id="codeSnippetWrapper" style="overflow: auto; cursor: text; font-size: 8pt; border-top: silver 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; border-right: silver 1px solid; border-bottom: silver 1px solid; padding-bottom: 4px; direction: ltr; text-align: left; padding-top: 4px; padding-left: 4px; margin: 20px 0px 10px; border-left: silver 1px solid; line-height: 12pt; padding-right: 4px; max-height: 200px; width: 97.5%; background-color: #f4f4f4">
                    <div id="codeSnippet" style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4">
                      <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum1" style="color: #606060">   1:</span> BundleTable.EnableOptimizations = <span style="color: #0000ff">true</span>;</pre>
                      
                      <p>
                        <!--CRLF--></div> </div> 
                        
                        <p>
                          Si ahora ejecutamos y miramos el código fuente veremos lo siguiente:
                        </p>
                        
                        <div id="codeSnippetWrapper" style="overflow: auto; cursor: text; font-size: 8pt; border-top: silver 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; border-right: silver 1px solid; border-bottom: silver 1px solid; padding-bottom: 4px; direction: ltr; text-align: left; padding-top: 4px; padding-left: 4px; margin: 20px 0px 10px; border-left: silver 1px solid; line-height: 12pt; padding-right: 4px; max-height: 200px; width: 97.5%; background-color: #f4f4f4">
                          <div id="codeSnippet" style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4">
                            <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum1" style="color: #606060">   1:</span> &lt;link href=<span style="color: #006080">"/Content/themes/base/css?v=UM624qf1uFt8dYtiIV9PCmYhsyeewBIwY4Ob0i8OdW81"</span> rel=<span style="color: #006080">"stylesheet"</span> type=<span style="color: #006080">"text/css"</span> /&gt; </pre>
                            
                            <p>
                              <!--CRLF-->
                            </p>
                            
                            <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum2" style="color: #606060">   2:</span> &lt;link href=<span style="color: #006080">"/Content/css?v=ZAyqul2u_47TBTYq93se5dXoujE0Bqc44t3H-kap5rg1"</span> rel=<span style="color: #006080">"stylesheet"</span> type=<span style="color: #006080">"text/css"</span> /&gt; </pre>
                            
                            <p>
                              <!--CRLF-->
                            </p>
                            
                            <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum3" style="color: #606060">   3:</span> &lt;script src=<span style="color: #006080">"/bundles/modernizr?v=EuTZa4MRY0ZqCYpBXj_MhJfFJU2QBDf0xGrV_p1fHME1"</span> type=<span style="color: #006080">"text/javascript"</span>&gt;&lt;/script&gt;</pre>
                            
                            <p>
                              <!--CRLF--></div> </div> 
                              
                              <p>
                                Podemos ver como tenemos tan solo dos tags <link> y uno <script> en lugar de todos los tags de antes. Ahora cada Bundle ha generado un solo tag. Eso, ya de por sí es bueno, ya que se reduce el número de peticiones http (una de las cosas que más tardan a la hora de cargar una página web es ir abriendo y cerrando conexiones http).
                              </p>
                              
                              <p>
                                Pero no solo eso, sino que además ahora el contenido está “minificado”, es decir se han eliminado todos los espacios y saltos de línea redundantes y se renombran todas las variables y nombres de función posibles para que tengan una longitud menor, de forma que el tamaño final sea menor, lo que a su vez redunda en un mejor rendimiento.
                              </p>
                              
                              <p>
                                Y todo ello tan solo habilitando la <em>EnableOptimizations</em>.
                              </p>
                              
                              <p>
                                <strong>Un par de curiosidades…</strong>
                              </p>
                              
                              <p>
                                Hoy en día muchas librerías js incluyen dos .js: uno sin normal y el otro minificado. Usualmente (por una de esas convenciones que terminan siendo estándares de facto) el archivo minificado tiene el sufijo “min”. Pongamos a jQuery como ejemplo. Si te lo descargas verás dos archivos como p.ej.
                              </p>
                              
                              <ol>
                                <li>
                                  jquery-1.6.2.js
                                </li>
                                <li>
                                  jquery-1.6.2.min.js
                                </li>
                              </ol>
                              
                              <p>
                                El primero es el archivo .js normal (para usar en debug p.ej.) y el otro es el archivo minificado. Funcionalmente ambos son equivalentes pero el primero ocupa unas 230K y el segundo unas 91.
                              </p>
                              
                              <p>
                                Pues bien, imaginad que tenemos un bundle para jQuery, definido de la siguiente forma:
                              </p>
                              
                              <div id="codeSnippetWrapper" style="overflow: auto; cursor: text; font-size: 8pt; border-top: silver 1px solid; font-family: &#39;Courier New&#39;, courier, monospace; border-right: silver 1px solid; border-bottom: silver 1px solid; padding-bottom: 4px; direction: ltr; text-align: left; padding-top: 4px; padding-left: 4px; margin: 20px 0px 10px; border-left: silver 1px solid; line-height: 12pt; padding-right: 4px; max-height: 200px; width: 97.5%; background-color: #f4f4f4">
                                <div id="codeSnippet" style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4">
                                  <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: white"><span id="lnum1" style="color: #606060">   1:</span> bundles.Add(<span style="color: #0000ff">new</span> ScriptBundle(<span style="color: #006080">"~/bundles/jquery"</span>)</pre>
                                  
                                  <p>
                                    <!--CRLF-->
                                  </p>
                                  
                                  <pre style="border-top-style: none; overflow: visible; font-size: 8pt; border-left-style: none; font-family: &#39;Courier New&#39;, courier, monospace; border-bottom-style: none; color: black; padding-bottom: 0px; direction: ltr; text-align: left; padding-top: 0px; border-right-style: none; padding-left: 0px; margin: 0em; line-height: 12pt; padding-right: 0px; width: 100%; background-color: #f4f4f4"><span id="lnum2" style="color: #606060">   2:</span> .Include(<span style="color: #006080">"~/Scripts/jquery-1.6.2*"</span>));</pre>
                                  
                                  <p>
                                    <!--CRLF--></div> </div> 
                                    
                                    <p>
                                      Fijaos que es un bundle que incluiría cualquier archivo js cuyo nombre empezase por jquery-1.6.2. Y en nuestro directorio ~/Scripts tenemos dos archivos que empiezan por este nombre, jquery-1.6.2.js y jquery-1.6.2.min.js. Pues bien el bundle es lo suficientemente inteligente como:
                                    </p>
                                    
                                    <ul>
                                      <li>
                                        Generar tan solo el tag <script> para el archivo jquery-1.6.2.js en el caso de que las optimizaciones estén deshabilitadas (y no generar el tag para el archivo jquery-1.6.2.min.js).
                                      </li>
                                      <li>
                                        Retornar el contenido del archivo jquery-1.6.2.min.js en el caso de que las optimizaciones estén habilitadas. Aunque ASP.NET igualmente minifica el propio archivo (aunque ya esté minificado). Es decir coje el archivo jquery-1.6.2.min.js, lo minifica y devuelve el contenido. Debo reconocer que me ha sorprendido que minifique de nuevo el contenido, pero es es lo que hace.
                                      </li>
                                    </ul>
                                    
                                    <p>
                                      De hecho en el propio Solution Explorer de VS2012 ya se aprecia que VS “sabe” que ambos archivos están relacionados:
                                    </p>
                                    
                                    <p>
                                      <img title="image" style="border-top: 0px; border-right: 0px; border-bottom: 0px; border-left: 0px; display: inline" border="0" alt="image" src="http://geeks.ms/cfs-file.ashx/__key/CommunityServer.Blogs.Components.WeblogFiles/etomas/image_5F00_620C7F9B.png" width="222" height="87" />
                                    </p>
                                    
                                    <p>
                                      Espero que este post os haya resultado de interés! 😉
                                    </p>
                                    
                                    <p>
                                      Saludos!
                                    </p>