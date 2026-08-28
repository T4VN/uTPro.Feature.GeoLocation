# Configuration

All configuration lives under the `uTPro:Feature:GeoLocation` section in `appsettings.json`.
Every key is optional — the package works with zero configuration.

## Full example

```json
{
  "uTPro": {
    "Feature": {
      "GeoLocation": {
        "AutoSetCulture": false,
        "DatabasePath": "App_Data/GeoLocation/iplocate-country.mmdb",
        "UseCookie": true,
        "CookieName": "utpro_geo_country",
        "CookieMaxAgeDays": 30,
        "FallbackCulture": "en",
        "Providers": [],
        "CultureMap": {
          "VN": "vi-VN",
          "US": "en-US",
          "JP": "ja-JP",
          "KR": "ko-KR",
          "CN": "zh-CN",
          "TW": "zh-TW",
          "FR": "fr-FR",
          "DE": "de-DE",
          "TH": "th-TH"
        }
      }
    }
  }
}
```

## Key reference

| Key | Type | Default | Description |
|---|---|---|---|
| `AutoSetCulture` | `bool` | `false` | Set `Thread.CurrentThread.CurrentCulture` and `CurrentUICulture` on every request. Compatible with Umbraco SurfaceController, ViewComponent, and API controllers. |
| `DatabasePath` | `string` | `App_Data/GeoLocation/iplocate-country.mmdb` | Path to the offline MMDB file. Relative paths are resolved from `ContentRootPath`. |
| `UseCookie` | `bool` | `true` | Store the detected country in a cookie so repeat visits skip detection. |
| `CookieName` | `string` | `utpro_geo_country` | Name of the persistence cookie. |
| `CookieMaxAgeDays` | `int` | `30` | Cookie lifetime in days. |
| `FallbackCulture` | `string` | `en` | Fallback when no country is detected. |
| `Providers` | `string[]` | `[]` (all) | Ordered list of provider names. Empty array means all providers run in default priority order. Valid names: `Cloudflare`, `AwsCloudFront`, `AzureFrontDoor`, `Forwarded`, `Offline`. |
| `CultureMap` | `Dictionary<string,string>` | `{}` | Country code → culture name overrides. Keys are ISO 3166-1 alpha-2 (uppercase). Values are .NET culture names. |

## Culture mapping behaviour

1. If the country code exists in `CultureMap`, that exact culture is used.
2. Otherwise, the system uses .NET `RegionInfo` + `CultureInfo.GetCultures()` to find the standard
   specific culture for that country (e.g. `DE` → `de-DE`, `BR` → `pt-BR`).
3. If all else fails, the `FallbackCulture` value is used.

## Provider filtering

To use only Cloudflare + Offline (skip AWS/Azure):

```json
"Providers": ["Cloudflare", "Forwarded", "Offline"]
```

The `Forwarded` provider is important because it resolves the real client IP for the `Offline`
provider. Always include it before `Offline` unless you're sure
`HttpContext.Connection.RemoteIpAddress` is already correct.
