<div align="center">
    This library offers a simple way to handle GDPR compliant cookie consent concerns in your Blazor WASM/Serverside app.
</div>

<br />

<div align="center" style="padding-top: 3rem; padding-bottom: 5rem;">
<img src=".github/resources/1.png">
<img src=".github/resources/2.png">
</div>

<br />

## How to install
```ps1
~~Install-Package BytexDigital.Blazor.Components.CookieConsent~~
```
```ps1
This version must be built and used as a reference at this time.
```

<br />

### Requirements
.NET >= 6.0

<br />

### Configure in your project
Add the `CookieConsentHandler` your App.razor, wrapping around the `Router` component, like so:

```html
<BytexDigital.Blazor.Components.CookieConsent.CookieConsentHandler>
    <Router AppAssembly="@typeof(App).Assembly">
        <Found Context="routeData">
            <RouteView RouteData="@routeData" DefaultLayout="@typeof(EmptyLayout)" />
            <FocusOnNavigate RouteData="@routeData" Selector="h1" />
        </Found>
        <NotFound>
            <PageTitle>Not found</PageTitle>
            <LayoutView Layout="@typeof(EmptyLayout)">
                <p role="alert">Sorry, there's nothing at this address.</p>
            </LayoutView>
        </NotFound>
    </Router> 
</BytexDigital.Blazor.Components.CookieConsent.CookieConsentHandler>
```

<br />

Add the following css include to your index.html/_Host.cshtml file.
```html
<link rel="stylesheet" href="_content/BytexDigital.Blazor.Components.CookieConsent/styles.min.css" />
```

<br />

**Installing the font**  
By default, the components use the following order of fonts

```css
Inter var, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, "Noto Sans", sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol", "Noto Color Emoji"
```

`Inter` is the font used in the screenshots. If you wish the components to use this font, import the inter font by additionally adding the following CSS link:

```html
<link rel="stylesheet" href="https://rsms.me/inter/inter.css" />
```

<br />

Add the required services in your Program.cs/Startup.cs and configure cookie categories present in your application. The library implicitly adds a `necessary` (value of constant `CookieCategory.NecessaryCategoryIdentifier`) category. For example:

```cs
builder.Services.AddCookieConsent(o =>
{
    o.Revision = 1;
    o.ConsentModalPosition = ConsentModalPosition.BottomRight;
    o.ConsentModalLayout = ConsentModalLayout.Cloud;
    o.ConsentSecondaryActionOpensSettings = false;

    o.Categories.Add(new CookieCategory
    {
        TitleText = new()
        {
            ["en"] = "Google Services",
            ["de"] = "Google Dienste"
        },
        DescriptionText = new()
        {
            ["en"] = "Allows the integration and usage of Google services.",
            ["de"] = "Erlaubt die Verwendung von Google Diensten."
        },
        Identifier = "google",
        IsPreselected = true,

        Services = new()
        {
            new CookieCategoryService
            {
                Identifier = "google-maps",
                PolicyUrl = "https://policies.google.com/privacy",
                TitleText = new()
                {
                    ["en"] = "Google Maps",
                    ["de"] = "Google Maps"
                },
                ShowPolicyText = new()
                {
                    ["en"] = "Display policies",
                    ["de"] = "Richtlinien anzeigen"
                }
            },
            new CookieCategoryService
            {
                Identifier = "google-analytics",
                PolicyUrl = "https://policies.google.com/privacy",
                TitleText = new()
                {
                    ["en"] = "Google Analytics",
                    ["de"] = "Google Analytics"
                },
                ShowPolicyText = new()
                {
                    ["en"] = "Display policies",
                    ["de"] = "Richtlinien anzeigen"
                }
            }
        }
    });
});
```

<br />

## Localization
For now, localization is done entirely inside the configuration of the services as seen in the example above. The library ships with default texts in English and German.

The library uses the current `CurrentCulture` by default. Blazor's `.AddLocalization(..)` will automatically set the current culture. We aim at adding proper support for `IStringLocalizer` aswell, so that all localization can be done inside resource files instead.

## Customizing the font
To overwrite which font to use, set the following CSS rule:

```css
.cc-font-overwrite {
    font-family: "your-font";
}
```

<br />

## Available ways to hide/show content based on cookie preferences

### Javascript tags
If you wish to use services like Google Analytics, you can integrate them with this library the following way. This will make it so the script tags do not get run unless allowed to do so by the user.
1. Change the script tags type attribute from `type="text/javascript"` to `type"text/plain"`.
2. Add the attribute `data-consent-category="IDENTIFIER"`.
3. Replace `IDENTIFIER` with the Identifier given to a configured category. In the example given earlier, this could be `google`.

The result should look like so:
```html
<script type="text/plain" data-consent-category="google">
    (function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
    (i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
    m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
    })(window,document,'script','https://www.google-analytics.com/analytics.js','ga');

    ga('create', 'UA-XXXXX-Y', 'auto');
    ga('send', 'pageview');
</script>
```

This Google Analytics script will only load if given permission.

<br />

### `CookieConsentCheck` component
You can use the prebuilt component to show content only if given permission in a category. This can be useful for displaying iframes, for example Google Maps or YouTube videos:

```html
<CookieConsentCheck RequiredCategory="google">
    <Allowed>
        <iframe loading="lazy" allowfullscreen src="https://www.google.com/maps/embed/v1/place?q=place_id:ChIJAVkDPzdOqEcRcDteW0YgIQQ&key=..."></iframe>
    </Allowed>
</CookieConsentCheck>
```

You can customize what this component will render when the given `RequiredCategory` is not allowed by defining a `NotAllowed` tag. By default, the component will render this:

<br />

<div align="center">
<img src=".github/resources/3.png">
</div>

<br />

Defining something custom to render can be done the following way. It's a good idea to set the `Context` parameter on the `CookieConsentCheck` component so you can easily access it's properties inside your custom `NotAllowed` block (for example the `Component.Category` property to access the display name of the required category).

```html
<CookieConsentCheck RequiredCategory="google" Context="Component">
    <Allowed>
        <h1>It works!</h1>
    </Allowed>
    <NotAllowed>
        <button @onclick="async () => await Component.AcceptRequiredAsync()">Show it!</button>
    </NotAllowed>
</CookieConsentCheck>
```

<br/>

## Manually open the preferences modal

Call the following method to show the preferences menu. This could be done from an element inside your footer for example.

```csharp
CookieConsentService.ShowSettingsModalAsync();
```

# Changelog

## 1.0.10

<details>
  <summary>Click to expand!</summary>
  
   <br /> 

  - Applied fix from PR #8
</details>

## 1.0.9

<details>
  <summary>Click to expand!</summary>
  
   <br /> 

  - Implemented CSS reset to isolate the components of this library from any other CSS influence
</details>

## 1.0.6

<details>
  <summary>Click to expand!</summary>
  
   <br /> 

  - Improved support for overwriting of font used
</details>

