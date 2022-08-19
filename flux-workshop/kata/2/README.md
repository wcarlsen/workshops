# Code kata 2 - Flux and multi-tenancy

## Prerequisites

* [kustomize](https://kubectl.docs.kubernetes.io/installation/kustomize/)

## Excercise

### 0. Setup new tenant repository

Create a new public Github repository called `fluxworkshop-tenant` and clone it.

```bash
# Clone the repository
git clone git@github.com:${GITHUB_OWNER}/fluxworkshop-tenant.git ~/spaghetti/fluxworkshop-tenant
```

### 1. Copy folders into new repository

Copy and commit the content in `base/`, `dev/` and `prod/`.

```bash
# Copy tenant directories into your repository
cp -r {base,dev,prod} ~/spaghetti/fluxworkshop-tenant/

# Commit changes
git add base dev prod
git commit -m "Added tenant manifests"
git push
```

### 2. Prepare `fluxworkshop-monorepo` for tenant

```bash
# Create tenant directory
mkdir -p tenants/{prod,dev} tenants/base/dev-team

# Let Flux generate RBAC needed
flux create tenant dev-team --with-namespace=api \
    --export > tenants/base/dev-team/rbac.yaml

# Let flux create source, flux kustomization and regular kustomization resource
flux create source git dev-team \
    --namespace=api \
    --url=https://github.com/${GITHUB_OWNER}/fluxworkshop-tenant \
    --branch=main \
    --export > ./tenants/base/dev-team/sync.yaml

flux create kustomization dev-team \
    --namespace=api \
    --service-account=dev-team \
    --source=GitRepository/dev-team \
    --path="./" \
    --export >> ./tenants/base/dev-team/sync.yaml

cd tenants/base/dev-team/ && kustomize create --autodetect

cat << EOF | tee ./tenants/dev/dev-team-patch.yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1beta2
kind: Kustomization
metadata:
  name: dev-team
  namespace: apps
spec:
  path: ./staging
EOF

cat << EOF | tee ./tenants/dev/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - ../base/dev-team
patchesStrategicMerge:
  - dev-team-patch.yaml
EOF
```

### 3. Figure out what is missing to make it work?