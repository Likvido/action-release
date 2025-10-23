# Likvido github action for releasing apps

This action builds Docker images and deploys them via GitOps to Kubernetes clusters.

## Features

- ✅ Builds and pushes Docker images to Azure Container Registry
- ✅ Automatically updates GitOps repository with new image tags
- ✅ **Smart BuildKit detection**: Uses persistent BuildKit cache on self-hosted runners for faster builds
- ✅ **Cross-runner compatibility**: Works seamlessly on GitHub-hosted, Warp, and self-hosted runners
- ✅ Supports custom deployment file paths for complex deployments

## Performance Optimization

The action automatically detects if a persistent BuildKit service is available (on self-hosted runners) and uses it for caching:

| Runner Type | BuildKit Mode | Cache Behavior | Build Time |
|-------------|---------------|----------------|------------|
| Self-hosted (ARC) | Persistent BuildKit | ✅ Cached (NuGet, layers, base images) | ~2m 30s |
| GitHub-hosted | Ephemeral | ❌ No cache | ~4m 30s |
| Warp | Ephemeral | ❌ No cache | ~4m 30s |

On self-hosted runners with persistent BuildKit, you can expect:
- **45% faster builds** after the first build
- **Cached NuGet packages** (saves ~70s)
- **Cached base images** (saves ~20s)
- **Reused BuildKit instance** (saves ~25s)

## Usage

```
jobs:
  release:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Build & deploy
        uses: likvido/action-release@v3
        with:
          docker-working-directory: src
          docker-file-relative: Likvido.Project/Dockerfile
          app-name: my-app
          environment: staging
          kubernetes-namespace: my-namespace
          acr-registry: likvido
          azure-service-principal-id: ${{ secrets.AZURE_SERVICE_PRINCIPAL_ID }}
          azure-service-principal-password: ${{ secrets.AZURE_SERVICE_PRINCIPAL_PASSWORD }}
          gitops-repo-url: 'https://github.com/Likvido/Likvido.Kubernetes'
          github-app-id: ${{ secrets.LIKVIDO_DEPLOYMENT_PUSHER_APP_ID }}
          github-app-private-key-base64: ${{ secrets.LIKVIDO_DEPLOYMENT_PUSHER_PRIVATE_KEY_BASE64 }}
          github-app-installation-id: ${{ secrets.LIKVIDO_DEPLOYMENT_PUSHER_INSTALLATION_ID }}
          gitops-deployment-file: 'my/deployment/file.yaml'
```

## Maximizing Build Performance with Cache Mounts

To get the full benefit of persistent BuildKit on self-hosted runners, add cache mounts to your Dockerfiles:

### Before (no caching):
```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY . .
RUN dotnet restore "MyApp.csproj"
RUN dotnet build "MyApp.csproj" -c Release
RUN dotnet publish "MyApp.csproj" -c Release -o /app/publish
```

### After (with cache mounts):
```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY . .
RUN --mount=type=cache,target=/root/.nuget/packages \
    dotnet restore "MyApp.csproj"
RUN --mount=type=cache,target=/root/.nuget/packages \
    dotnet build "MyApp.csproj" -c Release
RUN --mount=type=cache,target=/root/.nuget/packages \
    dotnet publish "MyApp.csproj" -c Release -o /app/publish
```

**Important**: Cache mounts require BuildKit. This action automatically uses BuildKit when available, so no additional configuration is needed.

On runners **without** persistent BuildKit (GitHub-hosted, Warp):
- Cache mounts still work within a single build (saves time between restore/build/publish)
- Cache is lost between builds
- Still provides ~20s savings per build

On runners **with** persistent BuildKit (self-hosted ARC):
- Cache persists across all builds
- Full ~70s savings for NuGet packages
- Maximum performance benefit

## How BuildKit Detection Works

The action uses this logic to determine which BuildKit to use:

```
1. Check if buildkitd.github-arc-runners.svc.cluster.local:1234 is reachable
2. If YES → Use persistent BuildKit (self-hosted runner)
3. If NO  → Use ephemeral BuildKit (GitHub-hosted/Warp runner)
```

This means:
- ✅ **No configuration needed** - works automatically
- ✅ **No breaking changes** - existing workflows continue to work
- ✅ **Optimal performance** - uses best available BuildKit for each runner type

## Inputs

See `action.yml` for all available inputs.

# Releasing new version

Either create the new release + new version tag directly in the Github UI, or create it like this:

1. Commit and push your changes
2. Create a new tag: `git tag -a -m "Description of this release" <version>`
3. Push the tag: `git push --follow-tags`

## Example

```
git tag -a -m "First version" v1
git push --follow-tags
```
