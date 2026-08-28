# Getting Started

## 1. Install

```bash
dotnet add package uTPro.Feature.GeoLocation
```

No `Program.cs` changes needed — the package self-wires via Umbraco composers.

## 2. First boot

On startup, the middleware activates automatically. It detects the visitor's country on every
request and stores the result in `HttpContext.Items`.

If no CDN headers are available, the offline MMDB database is used as a fallback. The default
database ships with the NuGet package at `App_Data/GeoLocation/iplocate-country.mmdb`.

## 3. Access the result

### From a SurfaceController or RenderController

```csharp
using uTPro.Feature.GeoLocation.Extensions;

public class MySurfaceController : SurfaceController
{
    public IActionResult Index()
    {
        var geo = HttpContext.GetGeoLocation();
        
        if (geo?.IsDetected == true)
        {
            ViewBag.Country = geo.CountryCode;   // "VN"
            ViewBag.Culture = geo.Culture?.Name;  // "vi-VN"
            ViewBag.Provider = geo.Provider;      // "Cloudflare"
        }

        return CurrentUmbracoPage();
    }
}
```

### From a ViewComponent

```csharp
using uTPro.Feature.GeoLocation.Extensions;

public class LanguageSwitcherViewComponent : ViewComponent
{
    public IViewComponentResult Invoke()
    {
        var countryCode = HttpContext.GetGeoCountryCode(); // "US"
        // Use countryCode to highlight the detected language in a switcher UI.
        return View(countryCode);
    }
}
```

### Inject IGeoLocationService directly

```csharp
using uTPro.Feature.GeoLocation.Services;

public class MyApiController : ControllerBase
{
    private readonly IGeoLocationService _geo;

    public MyApiController(IGeoLocationService geo) => _geo = geo;

    [HttpGet("/api/my/locale")]
    public IActionResult GetLocale()
    {
        var code = _geo.GetCountryCode(HttpContext);       // Raw: "JP"
        var culture = _geo.MapToCulture(code ?? "US");     // CultureInfo("ja-JP")
        return Ok(new { code, culture = culture?.Name });
    }
}
```

## 4. Configure (optional)

Add to `appsettings.json`:

```json
{
  "uTPro": {
    "Feature": {
      "GeoLocation": {
        "AutoSetCulture": true,
        "CultureMap": {
          "VN": "vi-VN",
          "US": "en-US"
        }
      }
    }
  }
}
```

Set `AutoSetCulture: true` to have the middleware automatically set the thread culture for the
entire Umbraco request pipeline. This affects `SurfaceController`, `ViewComponent`, `ApiController`,
and Razor views.

## 5. Detection chain

Providers run in this order (first non-null wins):

| Priority | Provider | Source |
|---|---|---|
| 10 | Cloudflare | `CF-IPCountry` header |
| 20 | AWS CloudFront | `CloudFront-Viewer-Country` header |
| 30 | Azure Front Door | `X-Azure-ClientIP-Country` or `X-Azure-Geo-Country` |
| 40 | Forwarded | Resolves real client IP from proxy headers |
| 100 | Offline | Looks up IP in local `.mmdb` database |
