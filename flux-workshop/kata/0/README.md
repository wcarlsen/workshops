# Code kata 0 - Bootstrap cluster with Flux

## Prerequisites

* [minikube](https://minikube.sigs.k8s.io/docs/start/) (or any other local Kubernetes distribution k0s, k3s, kind etc.)
* [kubectl](https://kubernetes.io/docs/tasks/tools/)
* [flux](https://fluxcd.io/docs/installation/)
* git
* Personal Github account
* [Personal access token for Github](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)

## Excercise

### 0. Spin up local Kubernetes cluster

```bash
# Start local cluster
minikube start --cpus 2 --memory 1900 --profile dev

# Verify cluster connection by listing all pods
kubectl get po -A
```

### 1. Bootstrap cluster with Flux

```bash
# Bootstrap flux in cluster (remember to replace GITHUB_OWNER variable)
flux bootstrap github --owner=${GITHUB_OWNER} --repository="fluxworkshop-monorepo" --components-extra "image-reflector-controller,image-automation-controller" --path clusters/dev --read-write-key --personal  --private=false

# Verify flux is installed
kubectl get po -n flux-system

# Check flux resources and that all is READY
flux get all
```

Last plase verify that `flux bootstrap github` command provisioned a Github repository in your account.

```bash
# Clone the repository
mkdir ~/spaghetti
git clone git@github.com:${GITHUB_OWNER}/fluxworkshop-monorepo.git ~/spaghetti/fluxworkshop-monorepo
```

**Obs: Files in the `flux-system` folder should not be altered, since flux will manage these for us.**