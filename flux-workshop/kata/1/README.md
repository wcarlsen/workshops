# Code kata 1 - Deploy helm chart via Flux

## Excercise

### 0. Go to you Flux repository
```bash
# Go to your GitOps repository
cd ~/spaghetti/fluxworkshop-monorepo

# Copy infrastructure directories into your repository
cp -r infrastructure ~/spaghetti/fluxworkshop-monorepo
```

### 1. Copy `infrastructure/` folder into your Flux repository

Read and understand the content of the files.

```bash
infrastructure
├── base
│   └── external-secrets
│       ├── clustersecretstore.yaml
│       ├── crds.yaml
│       ├── kustomization.yaml
│       ├── namespaces.yaml
│       ├── releases.yaml
│       └── sources.yaml
└── clusters
    └── dev
        ├── kustomization.yaml
        └── patches.yaml
```

Copy and commit the content in `infrastructure/`.

```bash
# Copy infrastructure directories into your repository
cp -r infrastructure ~/spaghetti/fluxworkshop-monorepo

# Commit changes
git add infrastructure
git commit -m "Added infrastructure with external-secrets"
git push
```

Reconcile the git source and see if Flux will pull and apply the changes.

```bash
# Reconcile using Flux cli 
flux reconcile source git flux-system

# Get all Flux resources
flux get all
```

### 2. Go look for the resources deployed

We know that flux should have created a namespace called `external-secrets`.

```bash
# List namespaces (hint: it isn't there)
kubectl get ns
```

### 3. Why isn't our manifests being deployed?

It is important to distinguish between regular kustomize/kustomization resources (kustomize.config.k8s.io/v1beta1) and Flux kustomize/kustomization (kustomize.toolkit.fluxcd.io/v1beta2). We currently only have regular kustomize, so let's add the last piece of the puzzle.

```bash
# Copy infrastructure.yaml into fluxworkshop-monorepo
cp infrastructure.yaml ~/spaghetti/fluxworkshop-monorepo/clusters/dev/

# Commit changes
git add clusters/dev/infrastructure.yaml
git commit -m "Added Flux kustomize"
git push

# Reconcile using Flux cli 
flux reconcile source git flux-system

# Get all Flux resources
flux get all

# List namespaces (hint: it should be there)
kubectl get ns
```

### 4. Play time

Try adding another helm chart or alter the existing one. Remember to add new files to kustomization.yaml, so that changes will be picked up.