# test-prow
Sample repo used for testing prow

## Cookbook

Cookbook based on:
- https://github.com/kubernetes/test-infra/blob/master/prow/getting_started_deploy.md
- https://github.com/kyma-project/test-infra/blob/master/docs/prow/prow-installation-on-forks.md

### Steps
* Create cluster
```bash
gcloud container clusters create msprow --machine-type n1-standard-4 --num-nodes 2
```

* Kubeconfig
```bash
gcloud container clusters get-credentials msprow
```

* Create cluster role bindings
```bash
kubectl create clusterrolebinding cluster-admin-binding \
  --clusterrole cluster-admin --user $(gcloud config get-value account)
```

* Create the GitHub secrets
```bash
openssl rand -hex 20 > hmac-token
kubectl create secret generic hmac-token --from-file=hmac=./hmac-token
```

Generate OAuth2 token from the account's settings -> Personal access tokens -> Generate new token.
> Token must have public_repo and repo:status scopes 
```bash
kubectl create secret generic oauth-token --from-file=oauth=./token
```

* Deploy NGINX Ingress Controller
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.21.0/deploy/mandatory.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.21.0/deploy/provider/cloud-generic.yaml
```

* Apply starter YAML
```bash
kubectl apply -f https://raw.githubusercontent.com/kyma-project/test-infra/master/prow/cluster/starter.yaml
```

* Get ingress IP address
```bash
kubectl get ingress ing
```
* Add the webhook to GitHub
> bazel installed via brew: https://docs.bazel.build/versions/master/install-os-x.html#install-on-mac-os-x-homebrew

```bash
bazel run //experiment/add-hook -- \
  --hmac-path=./workspace/prow-cluster/hmac-token \
  --github-token-path=./workspace/prow-cluster/token \
  --hook-url http://35.241.139.10/hook \
  --repo mszostok/test-prow \
  --confirm=false  # Remove =false to actually add hook
```
