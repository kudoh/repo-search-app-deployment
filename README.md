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

export GHUSER="kudoh"
fluxctl install \
--git-user=${GHUSER} \
--git-email=${GHUSER}@users.noreply.github.com \
--git-url=git@github.com:${GHUSER}/flux-get-started \
--git-path=namespaces,workloads \
--namespace=flux | kubectl delete -f -

kubectl rollout status deployment/flux -n flux
fluxctl identity --k8s-fwd-ns flux
```

## Argo CD

```bash
```
