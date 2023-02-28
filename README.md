# Likvido github action for releasing apps

```
jobs:
  release:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Build & deploy
        uses: likvido/action-release@v1
        with:
          docker-working-directory: src
          docker-file-relative: Likvido.Project/Dockerfile
          deployment-file: src/Likvido.Project/deployment.yml
          app-name: my-app
          kubernetes-namespace: my-namespace
          acr-registry: likvido
          aks-cluster-name: staging
          aks-cluster-resource-group: kubernetes-staging
          azure-service-principal-id: ${{ secrets.AZURE_SERVICE_PRINCIPAL_ID }}
          azure-service-principal-tenant: ${{ secrets.AZURE_SERVICE_PRINCIPAL_TENANT }}
          azure-service-principal-password: ${{ secrets.AZURE_SERVICE_PRINCIPAL_PASSWORD }}
          azure-service-principal-subscription: ${{ secrets.AZURE_SERVICE_PRINCIPAL_SUBSCRIPTION }}
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
