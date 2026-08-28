# Offline Database

## Source

The offline IP-to-country database comes from [iplocate.io](https://github.com/iplocate/ip-address-databases),
an open-source project that provides free, daily-updated IP geolocation databases in MMDB
(MaxMind DB) format.

## Included database

The NuGet package ships with a snapshot of the database at `App_Data/GeoLocation/iplocate-country.mmdb`.
This file is deployed automatically when you install the package.

## Updating the database

### Option 1: Manual replacement

1. Download the latest `iplocate-country.mmdb` from the
   [iplocate.io releases](https://github.com/iplocate/ip-address-databases/tree/main/ip-to-country).
2. Replace the file at `App_Data/GeoLocation/iplocate-country.mmdb` in your deployed application.
3. Restart the application (the MMDB reader is loaded once on first use).

### Option 2: NuGet package upgrade

A GitHub Action on the `uTPro.Feature.GeoLocation.Src` repository:
- Runs on a schedule (every 1–2 days)
- Downloads the latest iplocate.io database
- Compares checksums — if changed, bumps the patch version and packs a new NuGet
- Pushes to NuGet.org automatically

Upgrading the package (`dotnet update package uTPro.Feature.GeoLocation`) gives you the latest DB.

## Custom database path

If you prefer to store the MMDB file elsewhere:

```json
{
  "uTPro": {
    "Feature": {
      "GeoLocation": {
        "DatabasePath": "D:\\GeoData\\custom-country.mmdb"
      }
    }
  }
}
```

Absolute and relative paths are both supported. Relative paths resolve from `ContentRootPath`.

## Database format compatibility

The offline provider supports two MMDB structures:

1. **iplocate.io format**: `{ "country_code": "VN" }`
2. **MaxMind GeoLite2 format**: `{ "country": { "iso_code": "VN" } }`

This means you can also use a GeoLite2-Country MMDB if you already have a MaxMind license.
