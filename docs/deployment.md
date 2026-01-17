# Deployment Guide

## Overview

GameTime .NET Quiz uses a two-project Cloudflare Pages setup:

| Project | Repository | Content |
|---------|------------|---------|
| `gametime-dotnet-quiz` | Code repo | Blazor WASM app |
| `gametime-dotnet-quiz-data` | Data repo | Question JSON files |

Both projects auto-deploy on push to `main` branch.

## Cloudflare Pages Setup

### Prerequisites

- Cloudflare account (free tier)
- GitHub account with repository access
- Repository pushed to GitHub

### Step 1: Connect Code Repository

1. Go to [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. Select **Pages** from sidebar
3. Click **Create a project**
4. Select **Connect to Git**
5. Authorize Cloudflare to access GitHub
6. Select `gametime-dotnet-quiz` repository

### Step 2: Configure Build Settings

| Setting | Value |
|---------|-------|
| Project name | `gametime-dotnet-quiz` |
| Production branch | `main` |
| Framework preset | None |
| Build command | `chmod +x build.sh && ./build.sh` |
| Build output directory | `publish` |
| Root directory | `/` |

### Step 3: Environment Variables

| Variable | Value |
|----------|-------|
| `DOTNET_VERSION` | `10.0.100` |

### Step 4: Save and Deploy

Click **Save and Deploy**. First build may take 3-5 minutes.

## Build Script (build.sh)

The `build.sh` script is used by Cloudflare Pages:

```bash
#!/bin/bash
set -e

# Install .NET SDK
curl -sSL https://dot.net/v1/dotnet-install.sh | bash /dev/stdin --version $DOTNET_VERSION
export PATH="$HOME/.dotnet:$PATH"

# Restore and publish
dotnet restore
dotnet publish src/QuizApp -c Release -o publish
```

## Custom Domain (Optional)

1. Go to project settings in Cloudflare Pages
2. Select **Custom domains**
3. Add your domain (e.g., `quiz.example.com`)
4. Follow DNS configuration instructions

## Caching Headers

The `src/QuizApp/wwwroot/_headers` file configures caching:

```
# Blazor framework files (immutable, content-hashed)
/_framework/*
  Cache-Control: public, max-age=31536000, immutable

# App entry point (always fresh)
/index.html
  Cache-Control: no-cache

# CSS (daily refresh)
/css/*
  Cache-Control: public, max-age=86400
```

## Data Repository Deployment

### Step 1: Create Data Cloudflare Pages Project

Same process as code repo, but with different settings:

| Setting | Value |
|---------|-------|
| Project name | `gametime-dotnet-quiz-data` |
| Production branch | `main` |
| Build command | *(leave empty)* |
| Build output directory | `publish` |
| Root directory | `/` |

No build step needed—Cloudflare just serves the `publish/` folder.

### Step 2: Data Caching Headers

`publish/_headers` in data repo:

```
# CORS for cross-origin requests
/*
  Access-Control-Allow-Origin: *
  Access-Control-Allow-Methods: GET, OPTIONS
  Access-Control-Allow-Headers: Content-Type

# Manifest (always check for updates)
/manifest.json
  Cache-Control: no-cache, must-revalidate

# Question chunks (immutable, content-addressed)
/chunks/*
  Cache-Control: public, max-age=31536000, immutable
```

## Deployment URLs

After setup, your apps will be available at:

| Project | URL |
|---------|-----|
| Quiz App | `https://gametime-dotnet-quiz.pages.dev` |
| Data API | `https://gametime-dotnet-quiz-data.pages.dev` |

## Configuring Data Endpoint

Update `src/QuizApp/Program.cs` with data URL:

```csharp
builder.Services.AddHttpClient("DataApi", client =>
{
    client.BaseAddress = new Uri("https://gametime-dotnet-quiz-data.pages.dev/");
});
```

## Deployment Verification

### Check App Deployment

1. Open `https://gametime-dotnet-quiz.pages.dev`
2. Verify app loads without errors
3. Check DevTools Console for issues

### Check Data Deployment

```powershell
# Fetch manifest
curl https://gametime-dotnet-quiz-data.pages.dev/manifest.json

# Verify CORS headers
curl -I https://gametime-dotnet-quiz-data.pages.dev/manifest.json
```

### Verify Caching

Use browser DevTools Network tab:

| Request | Expected Cache Header |
|---------|----------------------|
| `manifest.json` | `Cache-Control: no-cache` |
| `chunks/*.json` | `Cache-Control: max-age=31536000, immutable` |
| `_framework/*.dll` | `Cache-Control: max-age=31536000, immutable` |

## Rollback

### Via Cloudflare Dashboard

1. Go to Pages project
2. Select **Deployments**
3. Find previous deployment
4. Click **...** menu
5. Select **Rollback to this deployment**

### Via Git Revert

```powershell
git revert HEAD
git push origin main
# Auto-deploys reverted version
```

## Monitoring

### Cloudflare Analytics

- **Pages > Project > Analytics**
- View requests, bandwidth, errors

### Build Logs

- **Pages > Project > Deployments**
- Click deployment to see build logs

## Troubleshooting

### Build Fails

1. Check build logs in Cloudflare dashboard
2. Verify `build.sh` is executable
3. Ensure `DOTNET_VERSION` matches `global.json`

### 404 Errors

1. Verify `publish` directory contains output
2. Check `Build output directory` setting
3. Ensure SPA fallback works (Blazor handles routing)

### CORS Errors

1. Verify `_headers` file in data repo `publish/` folder
2. Check `Access-Control-Allow-Origin: *` header
3. Test with `curl -I` to see actual headers

### Old Content Serving

1. Clear Cloudflare cache: **Pages > Project > Settings > Cache > Purge**
2. Clear browser cache
3. Unregister service worker if present
