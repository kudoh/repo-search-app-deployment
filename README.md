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
brew tap argoproj/tap
brew install argoproj/tap/argocd

kubectl create ns argocd
helm repo add argo https://argoproj.github.io/argo-helm && helm repo update
helm upgrade argo-cd --install argo/argo-cd --wait \
  --set server.serviceType=LoadBalancer
  --namespace=argocd

# Web UI  
kubectl get svc -n argocd argocd-server \
  -o custom-columns=IP:status.loadBalancer.ingress[0].ip,PORT:spec.ports[0].port

# admin password
ARGO_IP=$(kubectl get svc -n argocd argocd-server -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
ARGO_PASS=$(kubectl get pods -n argocd -l app.kubernetes.io/name=argo-cd-server -o jsonpath='{.items[].metadata.name}')
argocd login --username admin --password $ARGO_PASS $ARGO_IP
CONTEXT=local-k8s-tester
argocd cluster add $CONTEXT

# create application
GITHUB_USER=<your-github-name>
argocd proj create mamezou --src https://github.com/${GITHUB_USER}/repo-search-app-deployment.git \
  --dest "https://kubernetes.default.svc,dev"
argocd app create repo-search-app \
  --project mamezou \
  --repo https://github.com/${GITHUB_USER}/repo-search-app-deployment.git \
  --path overlays/argocd \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace dev \
  --sync-policy automated

```
