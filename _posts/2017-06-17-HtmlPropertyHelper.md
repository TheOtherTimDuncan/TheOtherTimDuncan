---
layout: post
title: "An Alternative To HtmlHelper Methods For Model Properties"
date: 2017-06-17
disqus_identifier: 6
---
tl;dr: Use HtmlPropertyHelper to avoid anonymous objects and gain more control of the html generated for model properties.

[Source code](https://github.com/TheOtherTimDuncan/HtmlPropertyHelperDemo)

The HtmlHelper extension methods such as [LabelFor](https://msdn.microsoft.com/en-us/library/system.web.mvc.html.labelextensions.labelfor(v=vs.118).aspx) or [TextBoxFor](https://msdn.microsoft.com/en-us/library/system.web.mvc.html.inputextensions.textboxfor(v=vs.118).aspx) as shown in the example below have their uses, but I find their use of anonymous objects difficult to read and use. And the nested anonymous objects needed when using [EditorFor](https://cpratt.co/html-editorfor-and-htmlattributes/) is even worse. You also lose the ability to get intellisense for your CSS classes.

```
 <div class="form-group">
    @Html.LabelFor(m => m.Email, new { @class = "col-md-2 control-label" })
    <div class="col-md-10">
        @Html.TextBoxFor(m => m.Email, new { @class = "form-control" })
    </div>
</div>
```
These methods are definitely valuable as a strongly-typed way to get the correct ID and name on your html elements. But I came up with what I think is a better way that still gives me the benefits of the MVC helpers but with html that is easier to read. The above using my helper class would look like this:

```html
<div class="form-group">
    @using (var property = Html.BeginProperty(x => x.Email))
    {
        <label for="@property.ID" class="col-md-2 control-label" @property.ValidationAttributes>Email</label>
        <div class="col-md-10"> 
            <input type="email" class="form-control" id="@property.ID" name="@property.Name" value="@property.Value" @property.ValidationAttributes>
            @property.ValidationMessage
        </div>
    }
</div>
```

You may also notice that I've added the unobtrusive validation attributes to the label. What this allows is targeting the ```[data-val-required]``` attribute with CSS to auto-add something like a red asterisk for required fields. The only time this has caused me a problem is when I have validation for one field to match another (such as for confirming a password). The jQuery unobtrusive validation will show an error in the console like "```Cannot read property substr of undefined```" in this scenario. Since I usually don't need the validation attributes on the label for this, I just leave them out. See the [Register view](https://github.com/TheOtherTimDuncan/HtmlPropertyHelperDemo/blob/master/HtmlPropertyHelperDemo/Views/Account/Register.cshtml) from the source code for a more complete example.

The code for HtmlPropertyHelper is shown below. It starts with the ModelPropertyAttribute class which contains the properties needed for the html. Note that is also implements ```IDisposable``` which allows the ```using``` syntax. I double-checked with the [MVC source code](http://aspnetwebstack.codeplex.com/) to ensure that my code was doing the same things the helper methods do. This helper was relatively simple to create. 

```csharp
public class ModelPropertyAttribute : IDisposable
{
    public string ID { get; set; }
    public string Name { get; set; }
    public MvcHtmlString Value { get; set; }
    public string PropertyName { get; set; }
    public MvcHtmlString ValidationAttributes { get; set; }
    public MvcHtmlString ValidationMessage { get; set; }

    public void Dispose() { }
}

public static class HtmlPropertyHelper
{
    public static ModelPropertyAttribute BeginProperty<TModel, TProperty>(this HtmlHelper<TModel> htmlHelper, Expression<Func<TModel, TProperty>> expression)
    {
        ModelMetadata metadata = ModelMetadata.FromLambdaExpression(expression, htmlHelper.ViewData);
        string expressionText = ExpressionHelper.GetExpressionText(expression);

        ModelPropertyAttribute propertyAttribute = new ModelPropertyAttribute()
        {
            ID = htmlHelper.ViewData.TemplateInfo.GetFullHtmlFieldId(expressionText),
            Name = htmlHelper.ViewData.TemplateInfo.GetFullHtmlFieldName(expressionText),
            PropertyName = expressionText,
            Value = htmlHelper.Value(expressionText),
            ValidationMessage = htmlHelper.ValidationMessage(expressionText)
        };

        StringBuilder sb = new StringBuilder();

        var attributes = htmlHelper.GetUnobtrusiveValidationAttributes(expressionText, metadata);
        foreach (var keyValue in attributes)
        {
            sb
                .Append(keyValue.Key)
                .Append("=\"")
                .Append(htmlHelper.Encode(keyValue.Value))
                .Append('"');
        }

        propertyAttribute.ValidationAttributes = MvcHtmlString.Create(sb.ToString());

        return propertyAttribute;
    }
}
```

The next helper got more challenging...and interesting. Phil Haack has a good post on [how to model bind a list in MVC](http://haacked.com/archive/2008/10/23/model-binding-to-a-list.aspx/). For example, the code below (copied from the linked post) shows the html needed to model bind to a list:

```csharp
public class Book 
{
    public string Title { get; set; }
    public string Author { get; set; }
    public DateTime DatePublished { get; set; }
}

//Action method on HomeController
public ActionResult UpdateProducts(ICollection<Book> books) 
{
    return View(books);
}
```
```html
<form method="post" action="/Home/UpdateProducts">

    <input type="text" name="[0].Title" value="Curious George" />
    <input type="text" name="[0].Author" value="H.A. Rey" />
    <input type="text" name="[0].DatePublished" value="2/23/1973" />
    
    <input type="text" name="[1].Title" value="Code Complete" />
    <input type="text" name="[1].Author" value="Steve McConnell" />
    <input type="text" name="[1].DatePublished" value="6/9/2004" />
    
    <input type="text" name="[2].Title" value="The Two Towers" />
    <input type="text" name="[2].Author" value="JRR Tolkien" />
    <input type="text" name="[2].DatePublished" value="6/1/2005" />
    
    <input type="submit" />
</form>
```
The key was another helpful post from Phil about [Razor delegates](http://haacked.com/archive/2011/02/27/templated-razor-delegates.aspx/). The drawback to using delegates is the requirement to wrap the code in an html element, but the majority of the time you'll want to anyway. The full source code can be seen [here](https://github.com/TheOtherTimDuncan/HtmlPropertyHelperDemo/blob/master/HtmlPropertyHelperDemo/HtmlCollectionHelper.cs), and I added a [view](https://github.com/TheOtherTimDuncan/HtmlPropertyHelperDemo/blob/master/HtmlPropertyHelperDemo/Views/Home/Books.cshtml) using the model above as an example. In the example, I deliberately cheated and did not redirect to GET to avoid having to create the route values for the redirect. Just as with the form values, the route values would need converted into the array syntax in order to pass them back to the GET action. TempData would have also worked. I'll leave either approach as an exercise for the reader.