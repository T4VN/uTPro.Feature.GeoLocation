# Offline Database

## What it is

The offline database is a local `.mmdb` file (MaxMind DB format) that maps IP addresses to country
codes. It is used as the **last resort** when no CDN header (Cloudflare, AWS, Azure) provides the
country.

No external API calls are made — the lookup is a fast, local, memory-mapped file read.

## Source

The database comes from [iplocate.io](https://www.iplocate.io/open-source), an open-source project
that provides free, daily-updated IP-to-country databases.

## Shipped with the NuGet package

When you install `uTPro.Feature.GeoLocation`, the MMDB file is deployed automatically to
`App_Data/GeoLocation/iplocate-country.mmdb` in your application root.

## Updating the database manually

IP-to-country mappings change over time as IP blocks are reassigned. You can update the database
at any time without upgrading the NuGet package.

### Steps

1. Download the latest database from
   [iplocate.io GitHub](https://github.com/iplocate/ip-address-databases/tree/main/ip-to-country)
   (file: `ip-to-country.mmdb`)

2. Rename the file to `iplocate-country.mmdb`

3. Replace the existing file at:
   ```
   {YourProject}/App_Data/GeoLocation/iplocate-country.mmdb
   ```

4. Restart the application

The MMDB reader loads the file once on the first request. After replacing the file, a restart
ensures the new data is picked up.

> **Tip:** You can automate this with a scheduled task or script that downloads the latest file
> from iplocate.io and restarts the app pool.

### NuGet upgrade does NOT overwrite your file

If you manually replaced the `.mmdb` file, upgrading the NuGet package will **not** overwrite
your custom file. NuGet only deploys contentFiles on first install. Your manual update is safe.

## Custom database path

To store the MMDB file at a different location (e.g. a shared network drive for load-balanced
deployments):

```json
{
  "uTPro": {
    "Feature": {
      "GeoLocation": {
        "DatabasePath": "App_Data/GeoLocation/iplocate-country.mmdb"
      }
    }
  }
}
```

This is the default path — relative to your project's `ContentRootPath`. You only need to set it
explicitly if you want a different location. Both relative and absolute paths are supported.

## Format compatibility

The offline provider supports two MMDB structures:

| Format | Structure | Example source |
|---|---|---|
| iplocate.io | `{ "country_code": "VN" }` | [iplocate.io](https://github.com/iplocate/ip-address-databases) |
| MaxMind GeoLite2 | `{ "country": { "iso_code": "VN" } }` | MaxMind GeoLite2-Country |

This means you can also use a GeoLite2-Country MMDB if you already have a MaxMind license.

## Privacy

- No external API calls — all lookups are local
- IP addresses are NOT persisted — used only during the request lifetime
- The MMDB file contains only IP ranges → country code mappings (no personal data)
