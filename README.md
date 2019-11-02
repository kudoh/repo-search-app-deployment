# Sample app deployment resources

## Pre-requirement

Create namspace resource for app.

```bash
kubectl create ns dev
```

Install minimum Redis using Helm.

```bash
kubectl create ns redis
helm upgrade redis --install stable/redis --namespace redis \
  --set master.persistence.enabled=false \
  --set cluster.enabled=false \
  --set password=frieza-redis-pass
```

Create Secret resources.

```bash
GITHUB_USER=<your-github-username>
GITHUB_PASSWORD=<your-github-password>
kubectl create secret -n dev generic github-secret --from-literal user=${GITHUB_USER} --from-literal password=${GITHUB_PASSWORD}
kubectl create secret -n dev generic redis-secret --from-literal password=frieza-redis-pass
```

## Using Flux

```bash
brew install fluxctl
kubectl create ns flux

helm repo add fluxcd https://charts.fluxcd.io && helm repo update

GITHUB_USER=<your-github-name>
helm upgrade flux --install fluxcd/flux --wait \
  --set git.url=git@github.com:${GITHUB_USER}/repo-search-app-deployment.git \
  --set git.branch=master \
  --set git.path=overlays/flux \
  --set registry.pollInterval=1m \
  --set git.pollInterval=1m \
  --set manifestGeneration=true \
  --namespace flux

fluxctl identity --k8s-fwd-ns flux
https://github.com/kudoh/repo-search-app-deployment/settings/keys/new


fluxctl sync --k8s-fwd-ns flux
fluxctl list-workloads --k8s-fwd-ns flux -n dev
fluxctl list-images --k8s-fwd-ns flux -n dev
```

## Argo CD

```bash
```
