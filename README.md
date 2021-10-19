# Our Umbraco TagHelpers
A community project of C# ASP.NET TagHelpers for the Open Source CMS Umbraco


## Installing
Add the following Nuget Package to your Umbraco website project `Our.Umbraco.TagHelpers` with Visual Studio, Rider or with the dotnet CLI tool as follows when inside the directory with the .CSProj file for the Umbraco website project.

```
cd MyUmbracoProject
dotnet add package Our.Umbraco.TagHelpers
```

## Setup

With the Nuget package added you need to register the collection of TagHelpers for Razor views and partials to use them.
Browse to `/Views/_ViewImports.cshtml` in your Umbraco project and add the following line at the bottom
```cshtml
@addTagHelper *, Our.Umbraco.TagHelpers
```


## `<our-dictionary>`
This is a tag helper element `<our-dictionary>` that will use the current page's request Language/Culture to use a dictionary translation from the Umbraco translation section.

* Find dictionary key
* Find translation for Current Culture/Language of the page
* If no translation found see if we have fallback attribute set fallback-lang or umb-dictionary-fallback-lang
* Attempt to find dictionary item for that ISO language fallback
* No translation found - leave default/value inside tag untouched for final fallback

```cshtml
<h3><our-dictionary key="home">My Header</our-dictionary></h3>
<h3><our-dictionary key="home" fallback-lang="da-DK">My Header</our-dictionary></h3>
```

## `our-if`
This is a tag helper attribute that can be applied to any DOM element in the razor template or partial. It will include its element and children on the page if the expression inside the `our-if` attribute evaluates to true.

### Simple Example
```cshtml
<div our-if="(DateTime.UtcNow.Minute % 2) == 0">This will only render during <strong>even</strong> minutes.</div>
<div our-if="(DateTime.UtcNow.Minute % 2) == 1">This will only render during <strong>odd</strong> minutes.</div>
```

### Example Before
```cshtml
@if (Model.ContentPickerThing != null)
{
    <a href="Model.ContentPickerThing.Url()" class="btn btn-action">
        <span>@Model.ContentPickerThing.Name</span>

        @if (Model.LinkMediaPicker != null)
        {
            <img src="@Model.LinkMediaPicker.Url()" class="img-circle" />
        }
    </a>
}
```

### Example After
```cshtml
<a our-if="Model.ContentPickerThing != null" href="@Model.ContentPickerThing?.Url()" class="btn btn-action">
    <span>@Model.ContentPickerThing.Name</span>
    <img our-if="Model.LinkMediaPicker != null" src="@Model.LinkMediaPicker?.Url()" class="img-circle" />
</a>
```

## `<our-macro>`
This tag helper element `<our-macro>` will render an Umbraco Macro Partial View and will use the current page/request for the Macro rendering & context.

If you wish, you can modify this behaviour and pass the context/content node that the Macro will render with using an optional attribute `content` on the `<our-macro>` tag and passing an `IPublishedContent` into the attribute. This allows the same Macro Partial View Macro code/snippet to work in various scenarios when the content node/context is changed.

Additionally custom Macro Parameters that can be passed through and consumed by Macro Partial Views are specified in the following way. The key/alias of the Macro Parameter must be prefixed with the following `bind:`

So to pass/set a value for the macro parameter `startNodeId` then I will need to set an attribute on the element as follows `bind:startNodeId`

```cshtml
<our-macro alias="ListChildrenFromCurrentPage" />
<our-macro alias="ListChildrenFromCurrentPage" Content="Model" />
<our-macro alias="ListChildrenFromCurrentPage" Content="Model.FirstChild()" />
<our-macro alias="ChildPagesFromStartNode" bind:startNodeId="umb://document/a878d58b392040e6ae9432533ac66ad9" />
```

## BeginUmbracoForm Replacement
This is to make it easier to create a HTML `<form>` that uses an Umbraco SurfaceController and would be an alternative of using the `@Html.BeginUmbracoForm` approach. This taghelper runs against the `<form>` element along with these attributes `our-controller`and `our-action` to help generate a hidden input field of `ufprt` containing the encoded path that this form needs to route to.

https://our.umbraco.com/Documentation/Fundamentals/Code/Creating-Forms/

### Before
```cshtml
@using (Html.BeginUmbracoForm("ContactForm", "Submit", FormMethod.Post, new { @id ="customerForm", @class = "needs-validation", @novalidate = "novalidate" }))
{
    @Html.ValidationSummary()

    <div class="input-group">
        <p>Name:</p>
        @Html.TextBoxFor(m => m.Name)
        @Html.ValidationMessageFor(m => m.Name)
    </div>
    <div>
        <p>Email:</p>
        @Html.TextBoxFor(m => m.Email)
        @Html.ValidationMessageFor(m => m.Email)
    </div>
    <div>
        <p>Message:</p>
        @Html.TextAreaFor(m => m.Message)
        @Html.ValidationMessageFor(m => m.Message)
    </div>
    <br/>
    <input type="submit" name="Submit" value="Submit" />
}
```

### After
```cshtml
<form our-controller="ContactForm" our-action="Submit" method="post" id="customerForm" class="fneeds-validation" novalidate>
    <div asp-validation-summary="All"></div>

    <div class="input-group">
        <p>Name:</p>
        <input asp-for="Name" />
        <span asp-validation-for="Name"></span>
    </div>
    <div>
        <p>Email:</p>
        <input asp-for="Email" />
        <span asp-validation-for="Email"></span>
    </div>
    <div>
        <p>Message:</p>
        <textarea asp-for="Message"></textarea>
        <span asp-validation-for="Message"></span>
    </div>
    <br/>
    <input type="submit" name="Submit" value="Submit" />
</form>
```

## `<our-lang-switcher>`
This tag helper element `<our-lang-switcher>` will create a simple unordered list of all languages and domains, in order to create a simple language switcher.
As this produces alot of HTML markup that is opionated with certain class names and elements, you may wish to change and control the markup it produces. 

With this tag helper the child DOM elements inside the `<our-lang-switcher>` element is used as a Mustache templating language to control the markup.

### Example
```cshtml
<our-lang-switcher>
    <div class="lang-switcher">
        {{#Languages}}
            <div class="lang-switcher__item">
                <a href="{{Url}}" lang="{{Culture}}" hreflang="{{Culture}}" class="lang-switcher__link {{#IsCurrentLang}}selected{{/IsCurrentLang}}">{{Name}}</a>
            </div>
        {{/Languages}}
    </div>
</our-lang-switcher>

<div class="lang-switcher">
    <div class="lang-switcher__item">
         <a href="https://localhost:44331/en" lang="en-US" hreflang="en-US" class="lang-switcher__link selected">English</a>
     </div>
    <div class="lang-switcher__item">
         <a href="https://localhost:44331/dk" lang="da-DK" hreflang="da-DK" class="lang-switcher__link ">dansk</a>
     </div>
</div>
```

If you do not specify a template and use `<our-lang-switcher />` it will use the following Mustache template
```mustache
<ul class='lang-switcher'>
    {{#Languages}}
        <li>
            <a href='{{Url}}' lang='{{Culture}}' hreflang='{{Culture}}' class='{{#IsCurrentLang}}selected{{/IsCurrentLang}}'>
                {{Name}}
            </a>
        </li>
    {{/Languages}}
</ul>
```


## `<our-svg>`
This tag helper element `<our-svg>` will read the file contents of an SVG file and output it as an inline SVG in the DOM.
It can be used in one of two ways, either by specifying the `src` attribute to a physcial static file served from wwwRoot or by specifying the `media-item` attribute to use a picked IPublishedContent Media Item.

```cshtml
<our-svg src="/assets/icon.svg" />
<our-svg media-item="@Model.Logo" />
```

## Attribution
The logo for Our.Umbraco.TagHelpers is made by <a href="https://www.freepik.com" title="Freepik">Freepik</a> from <a href="https://www.flaticon.com/" title="Flaticon">www.flaticon.com</a>
