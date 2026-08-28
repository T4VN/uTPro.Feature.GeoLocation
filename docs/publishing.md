# Publishing

## NuGet API Key setup (for GitHub Action auto-publish)

The `update-db.yml` workflow pushes a new NuGet package version when the offline database changes.
It requires a `NUGET_API_KEY` secret in the GitHub repository.

### Step 1: Create a NuGet API key

1. Sign in at [nuget.org](https://www.nuget.org/account/apikeys)
2. Click **Create**
3. Configure:
   - **Key Name**: `uTPro.Feature.GeoLocation Auto Publish` (or any name)
   - **Expiration**: 365 days (maximum)
   - **Scopes**: Push → Push new packages and package versions
   - **Glob Pattern**: `uTPro.Feature.GeoLocation`
4. Click **Create** and copy the key immediately (it's shown only once)

### Step 2: Add the secret to your GitHub repository

1. Go to your repository → **Settings** → **Secrets and variables** → **Actions**
2. Click **New repository secret**
3. Name: `NUGET_API_KEY`
4. Value: paste the key from step 1
5. Click **Add secret**

### Step 3: Verify

The workflow runs on schedule every 2 days. You can also trigger it manually:
- Go to **Actions** → **Update GeoLocation Database** → **Run workflow**

### Key renewal

NuGet API keys expire after the configured duration (max 365 days). Set a calendar reminder to
renew the key before expiration. When renewing:
1. Create a new key on nuget.org
2. Update the `NUGET_API_KEY` secret in GitHub repository settings
3. Delete the old key on nuget.org

## Manual publish

To publish manually without the GitHub Action:

```powershell
# 1. Pack
pwsh ./pack.ps1

# 2. Push to NuGet
dotnet nuget push "Build\uTPro.Feature.GeoLocation.*.nupkg" `
  --api-key YOUR_KEY `
  --source "https://api.nuget.org/v3/index.json" `
  --skip-duplicate
```

## Umbraco Marketplace

After the first NuGet publish, register the package on the Umbraco Marketplace:

1. Go to [marketplace.umbraco.com](https://marketplace.umbraco.com)
2. Sign in with your Umbraco account
3. Click **Submit a package**
4. Enter the NuGet package ID: `uTPro.Feature.GeoLocation`
5. The marketplace reads metadata from `umbraco-marketplace.json` in the package

The screenshots referenced in `umbraco-marketplace.json` must be accessible at the public
repository URLs (e.g. `https://raw.githubusercontent.com/T4VN/uTPro.Feature.GeoLocation/main/Image/Screenshots/...`).
