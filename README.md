# uTPro GeoLocation for Umbraco

IP-based visitor country detection for Umbraco multi-language sites. Resolves visitor country via
CDN headers (Cloudflare, AWS CloudFront, Azure Front Door) with an **offline MMDB fallback** —
then maps the country code to a .NET `CultureInfo` so your site can switch language automatically.

Works with **Umbraco 16, 17 and 18** (multi-targeted `net9.0` / `net10.0`).

[![NuGet](https://img.shields.io/nuget/v/uTPro.Feature.GeoLocation.svg)](https://www.nuget.org/packages/uTPro.Feature.GeoLocation)
[![NuGet Downloads](https://img.shields.io/nuget/dt/uTPro.Feature.GeoLocation.svg)](https://www.nuget.org/packages/uTPro.Feature.GeoLocation)
[![Umbraco Marketplace](https://img.shields.io/badge/Umbraco-Marketplace-blue)](https://marketplace.umbraco.com/package/utpro.feature.geolocation)
[![Umbraco 16+](https://img.shields.io/badge/Umbraco-16%2B-3544B1)](https://umbraco.com)
[![License: Free (proprietary)](https://img.shields.io/badge/License-Free%20(proprietary)-green.svg)](LICENSE.txt)

---

## Features

- **Multi-source detection chain** (first hit wins):
  1. **Cloudflare** — `CF-IPCountry` header
  2. **Akamai** — `X-Akamai-Edgescape` header (parses `country_code=XX`)
  3. **Fastly** — `X-Geo-Country` or `Fastly-Client-Country` header
  4. **Vercel** — `X-Vercel-IP-Country` header
  5. **Netlify** — `X-Country` header
  6. **AWS CloudFront** — `CloudFront-Viewer-Country` header
  7. **Azure Front Door** — `X-Azure-ClientIP-Country` / `X-Azure-Geo-Country` header
  8. **Forwarded IP resolution** — handles `X-Forwarded-For`, `X-Real-IP`, `CF-Connecting-IP`, `True-Client-IP`
  9. **Offline MMDB** — local iplocate.io database (no external API calls)
- **Two output modes**:
  - `GetCountryCode()` — raw ISO 3166-1 alpha-2 code (e.g. "VN", "US")
  - `Detect()` → full `GeoLocationResult` with mapped `CultureInfo`
- **Culture mapping**:
  - Custom map via `appsettings.json` (country → specific culture)
  - Automatic fallback using .NET `RegionInfo` (ISO standard derivation)
- **Cookie persistence** — skip detection on subsequent requests
- **Optional auto-culture** — middleware sets `CurrentCulture` / `CurrentUICulture` for the entire
  Umbraco pipeline (SurfaceControllers, ViewComponents, Views, API controllers)
- **Pluggable** — add custom `IGeoLocationProvider` implementations via DI
- **Zero-config install** — self-wires via Umbraco composers, no `Program.cs` changes
- **Offline DB update** — replace the `.mmdb` file manually or upgrade the NuGet package

---

## Quick Start

```bash
dotnet add package uTPro.Feature.GeoLocation
```

That's it. The middleware runs automatically. Access the result from any controller or view:

```csharp
// In a SurfaceController, RenderController, or ViewComponent:
var geo = HttpContext.GetGeoLocation();
// geo.CountryCode  → "VN"
// geo.Culture      → CultureInfo("vi-VN")
// geo.Provider     → "Cloudflare"
```

```csharp
// Or inject IGeoLocationService for raw detection:
var countryCode = _geoService.GetCountryCode(HttpContext);  // "US"
var culture = _geoService.MapToCulture("US");               // CultureInfo("en-US")
```

| Umbraco | .NET | Target |
|---|---|---|
| 16 | .NET 9 | `net9.0` |
| 17 & 18 | .NET 10 | `net10.0` |

---

## Configuration

All settings are optional — the package works out of the box. Configure under
`uTPro:Feature:GeoLocation` in `appsettings.json`:

```json
{
  "uTPro": {
    "Feature": {
      "GeoLocation": {
        "EnableProviders": {
          "Cloudflare": true
        },
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

| Key | Default | Description |
|---|---|---|
| `EnableProviders` | all `false` | Toggle CDN providers: `Cloudflare`, `Akamai`, `Fastly`, `Vercel`, `Netlify`, `AwsCloudFront`, `AzureFrontDoor`. Only enable what you use. |
| `CustomCountryHeader` | `""` | Custom header with 2-letter country code (highest priority, only active when set). |
| `CustomIpHeader` | `""` | Custom header with client IP (only active when set). |
| `AutoSetCulture` | `false` | When `true`, middleware sets `Thread.CurrentThread.CurrentCulture` for the Umbraco pipeline. |
| `DatabasePath` | `App_Data/GeoLocation/iplocate-country.mmdb` | Path to the offline MMDB file (relative to ContentRoot or absolute). |
| `UseCookie` | `true` | Persist detected country in a cookie to skip detection on subsequent requests. |
| `CookieName` | `utpro_geo_country` | Cookie name. |
| `CookieMaxAgeDays` | `30` | Cookie lifetime in days. |
| `FallbackCulture` | `en` | Culture used when no country is detected. |
| `CultureMap` | `{}` | Custom country→culture overrides. Keys: ISO 3166-1 alpha-2. Values: .NET culture names. |

> **Note:** By default only Forwarded (IP resolution) and Offline (MMDB) providers are active.
> CDN providers must be explicitly enabled via `EnableProviders`.

---

## Culture Mapping Strategy

The package provides two levels of country-to-culture resolution:

1. **Custom map** (`CultureMap` in config): highest priority. You define exact country→culture pairs.
2. **ISO standard** (automatic): uses .NET `RegionInfo` to derive the default culture for a country.
   For example, `"DE"` → `de-DE`, `"BR"` → `pt-BR`.

This means you only need to configure `CultureMap` for exceptions or preferences — the ISO standard
handles the rest.

---

## Updating the Offline Database

The offline `.mmdb` ships with the NuGet package. You can update it **manually at any time**:

1. Download the latest `ip-to-country.mmdb` from
   [iplocate.io](https://github.com/iplocate/ip-address-databases/tree/main/ip-to-country)
2. Rename to `iplocate-country.mmdb`
3. Replace `App_Data/GeoLocation/iplocate-country.mmdb` in your deployed site
4. Restart the application

NuGet package upgrades will **not overwrite** your manually updated file — your custom DB is safe.

---

## Extensibility

### Custom provider
Implement `IGeoLocationProvider` and register it in your own composer:

```csharp
public sealed class MyCustomGeoProvider : IGeoLocationProvider
{
    public string Name => "MyCustom";
    public int Priority => 5; // Runs before Cloudflare (10)

    public string? Detect(HttpContext httpContext)
    {
        // Your custom detection logic.
        return httpContext.Request.Headers["X-My-Country"].ToString();
    }
}

// In your composer:
builder.Services.AddSingleton<IGeoLocationProvider, MyCustomGeoProvider>();
```

---

## Documentation

Full documentation is available at the [uTPro Docs site](https://docs.utpro.dev/uTPro.Feature.GeoLocation/):

| Guide | Description |
|---|---|
| [Getting Started](https://docs.utpro.dev/uTPro.Feature.GeoLocation/getting-started/) | Install, first boot, access the result |
| [Configuration](https://docs.utpro.dev/uTPro.Feature.GeoLocation/configuration/) | All `appsettings` keys and their effects |
| [Culture Mapping](https://docs.utpro.dev/uTPro.Feature.GeoLocation/culture-mapping/) | Custom map vs ISO standard derivation |
| [Offline Database](https://docs.utpro.dev/uTPro.Feature.GeoLocation/offline-database/) | MMDB source, update strategies |
| [Extensibility](https://docs.utpro.dev/uTPro.Feature.GeoLocation/extensibility/) | Custom providers, advanced usage |

---

## Screenshots

![Detection flow](/Image/Screenshots/detection-flow.png)

---

## Security

- No external API calls — all detection uses request headers or the local MMDB file.
- The cookie is functional (not tracking) and marked `Secure`, `SameSite=Lax`, `IsEssential=true`.
- IP addresses are not persisted — they're used only during the request lifetime.

---

## License & Author

By [T4VN](https://github.com/T4VN). Free to use — including in commercial projects — under a
proprietary [End User License Agreement](LICENSE.txt). Issues welcome on the
[GitHub repository](https://github.com/T4VN/uTPro.Feature.GeoLocation).
