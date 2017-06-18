---
layout: post
title: Documentar tu proyecto ASP.Net y mostrarlo como un formulario mas
---

Ahora mostraré como puedes hacer que tu proyecto WebForms ASP.Net genere cada vez su propia documentación en base a los comentarios del código fuente y además se muestre como un formulario.

### Para empezar
Para ello utilizaremos dos recursos:

    * [VsXMd](https://github.com/lijunle) de [lijunle](https://github.com/lijunle)
    * [CommonMark.Net](https://github.com/Knagis/CommonMark.NET) de [Knagis](https://github.com/Knagis)

Estos dos proyectos los podemos obtener a traves de su respectivo repositorio en GitHub o a traves de Nuget.

Para nuestro propósito, vamos a trabajar con el Codigo Fuente de VsXMd y con el paquete Nuget de CommonMark.Net

### Generando la documentación en XML

Como bein sabemos, VisualStudio tiene la posibilidad de generar la documentación de nuestro proyecto en un archivo XML

La documentación se genera en base a los comentarios que se realicen sobre los métodos, propiedades y atributos de las clases de nuestro proyecto.

Si no conoces cómo generar esta documentación, te sugiero que te pases por [aqui](https://msdn.microsoft.com/es-es/library/x4sa0ak0(v=vs.100).aspx)

### Generando la documentación en Markdown

Lo primero que tenemos que hacer es abrir de manera separada el proyecto **VsXMd**.
Una vez abierto, podremos compilarlo y generar sus respectivos ejecutables, tal como muestra en la figura:

![_config.yml]({{ site.baseurl }}/images/2017-06-17-Documentar-proyecto-asp-net-01.png)

Necesitamos tener el ejecutable (y todas sus dependencias) por una razon:vamos a utilizar este ejecutable para generar un Script PostCompilacion en nuestro proyecto que queremos documentar

Además, la razon por la que trabajé con el codigo fuente (y no con el paquete Nuget), fue por que con el codigo fuente tengo la libertad de poder elegir lo que quiero que aparezca en mi documentacion. Asi por ejemplo, no me interesa poner en la documentación los nombres y las descripciones de todos los controles.

Para hacer este cambio, modifiqué el metodo: **private static IEnumerable<IUnit> ToUnits(XElement docElement)** en la clase **Converter.cs** del proyecto **Vsxmd** para que quede asi:

~~~csharp
private static IEnumerable<IUnit> ToUnits(XElement docElement)
{
    // assembly unit
    var assemblyUnit = new AssemblyUnit(docElement.Element("assembly"));

    // member units
    var memberUnits = docElement
        .Element("members")
        .Elements("member")
        .Select(element => new MemberUnit(element))
        .Where(member => member.Kind != MemberKind.NotSupported && member.Kind != MemberKind.Constants)
        .GroupBy(unit => unit.TypeName)
        .Select(MemberUnit.ComplementType)
        .SelectMany(group => group)
        .OrderBy(member => member, MemberUnit.Comparer);

    // table of contents
    var tableOfContents = new TableOfContents(memberUnits);

    return new IUnit[] { tableOfContents }
        .Concat(new[] { assemblyUnit })
        .Concat(memberUnits);
}
~~~

Si se fijan en la clausula **Where** (y la comparan con la que originalmente esta en el proyecto de lijunle), podrán ver que estoy excluyendo los elementos de tipo **Constants**.

Una vez realizado este cambio, compilo nuevamente el proyecto para obtener el nuevo archivo ejecutable

### Creando el Script PostCompilación en nuestro proyecto