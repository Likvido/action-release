# Likvido github action for releasing apps

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
