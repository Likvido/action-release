# Likvido github action for releasing apps

This action builds Docker images and deploys them via GitOps to Kubernetes clusters.

## Features

- ✅ Builds and pushes Docker images to Azure Container Registry
- ✅ Automatically updates GitOps repository with new image tags
- ✅ Supports custom deployment file paths for complex deployments
- ✅ Optional registry-based build cache for faster builds

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
          use-registry-cache: 'true'  # Optional: Enable ACR build cache
```

## Inputs

See `action.yml` for all available inputs.

### Registry Build Cache (Optional)

To enable faster builds using Azure Container Registry as a build cache, set `use-registry-cache: 'true'`:

```yaml
- name: Build & deploy
  uses: likvido/action-release@v3
  with:
    # ... other inputs ...
    use-registry-cache: 'true'
```

The cache is scoped per application (using `app-name`) and is shared between PR builds and release builds for optimal performance.

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
