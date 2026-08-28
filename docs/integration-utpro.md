# Integration with uTPro Main Project

## How GeoLocation integrates with the existing localization middleware

The uTPro main project uses `RequestLocalizationOptionMiddleware` (in `uTPro.Foundation.Middleware`)
to handle multi-language URL routing and culture persistence. The GeoLocation package integrates
seamlessly without modifying that middleware's contract.

### Detection flow (with GeoLocation installed)

```
Request arrives
  ↓
[1] GeoLocation middleware runs (prePipeline via UmbracoPipelineFilter)
    → Detects country from CDN headers / IP → stores result in HttpContext.Items
  ↓
[2] RequestLocalizationOptionMiddleware runs
    → If URL has language prefix (e.g. /vi/page) → use that culture ✓
    → If root URL ("/"):
        → Try .uTPro.Culture cookie → if found, use it ✓
        → Try GeoLocation result from HttpContext.Items → if detected, use it ✓ (NEW)
        → Fallback to site's default culture
    → Set Thread.CurrentThread.CurrentCulture
    → Store cookie, redirect if needed
```

### What changes in the uTPro project

1. **`nuget.config`** — added local source for the GeoLocation Build folder
2. **`uTPro.Feature.csproj`** — added `PackageReference` to `uTPro.Feature.GeoLocation`
3. **`RequestLocalizationOptionMiddleware.cs`** — added `GetGeoLocationCulture()` helper that reads
   the GeoLocation result from `HttpContext.Items` (duck-typed via `dynamic` to avoid a hard
   reference from Foundation → Feature layer)

### Layer compliance

The integration respects uTPro's Helix layers:
- **Foundation.Middleware** does NOT reference the Feature package
- It reads via `HttpContext.Items["uTPro.GeoLocation.Result"]` using dynamic dispatch
- If the GeoLocation package is not installed, the Items key is absent and the fallback works unchanged

### Configuration

Add to `appsettings.json` of the web project:

```json
{
  "uTPro": {
    "Feature": {
      "GeoLocation": {
        "AutoSetCulture": false,
        "CultureMap": {
          "VN": "vi-VN",
          "US": "en-US",
          "JP": "ja-JP"
        }
      }
    }
  }
}
```

> **Note:** `AutoSetCulture` should remain `false` in the uTPro project because
> `RequestLocalizationOptionMiddleware` handles culture-setting itself. GeoLocation only needs
> to detect the country and provide a mapped culture — the existing middleware does the rest.
