# Extensibility

## Custom providers

You can add your own geolocation provider by implementing `IGeoLocationProvider` and registering
it in your own Umbraco composer.

### Example: Custom header provider

```csharp
using Microsoft.AspNetCore.Http;
using uTPro.Feature.GeoLocation.Services;

public sealed class MyEdgeGeoProvider : IGeoLocationProvider
{
    public string Name => "MyEdge";
    public int Priority => 5; // Runs before Cloudflare (10)

    public string? Detect(HttpContext httpContext)
    {
        if (!httpContext.Request.Headers.TryGetValue("X-My-Edge-Country", out var values))
            return null;

        var code = values.ToString().Trim().ToUpperInvariant();
        return code.Length == 2 ? code : null;
    }
}
```

### Register in your composer

```csharp
using Umbraco.Cms.Core.Composing;
using Umbraco.Cms.Core.DependencyInjection;

public sealed class MyGeoComposer : IComposer
{
    public void Compose(IUmbracoBuilder builder)
    {
        builder.Services.AddSingleton<IGeoLocationProvider, MyEdgeGeoProvider>();
    }
}
```

Your custom provider will be picked up automatically and sorted by its `Priority` value.

## Custom culture mapping

Beyond the `CultureMap` in configuration, you can override the mapping entirely by replacing
the `IGeoLocationService` registration:

```csharp
builder.Services.AddSingleton<IGeoLocationService, MyCustomGeoLocationService>();
```

## Accessing the result in middleware

The result is stored in `HttpContext.Items["uTPro.GeoLocation.Result"]` after the
GeoLocation middleware runs. Any middleware that runs later can access it:

```csharp
if (context.Items["uTPro.GeoLocation.Result"] is GeoLocationResult geo && geo.IsDetected)
{
    // Use geo.CountryCode, geo.Culture, etc.
}
```
