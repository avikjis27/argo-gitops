### Clean up docker images and containers
<details>
<summary>commands</summary>

```bash
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)
docker image ls -q | xargs -I {} docker image rm -f {}
```
</details>

### Setup kubernetes cluster
<details>
<summary>commands</summary>

```bash
k3d cluster create -c k3d.yaml
```
</details>

## Install argocd
Ref - https://argo-cd.readthedocs.io/en/stable/getting_started/
<details>
<summary>install commands</summary>

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
brew install argocd
```
</details>

ArgoCD API server is not exposed to outside world by default. To access the API server you have to add some load balancer service/ingress. For local setup we can do port forwarding.

```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
Access the argocd console `http://localhost:8080`

### Set admin account
- Get initial password `argocd admin initial-password -n argocd`
- Use the initial password and user `admin` to login to the web ui.
- Login from the cli `argocd login localhost:8080`
- Update admin password `argocd account update-password`

### Argocd components
Observe the replicasets installed in the argocd namespace.
```
kubectl get all -n argocd
```

- argocd-redis
- argocd-applicationset-controller
- argocd-notifications-controller
- argocd-server (api server)
- argocd-dex-server
- argocd-repo-server

## Create an application in argocd

```bash
kubectl config set-context --current --namespace=argocd
argocd app create nodejs-webapp --repo https://github.com/avikjis27/demo-apps.git --revision main --path nodejs-webapp/kubernetes --dest-namespace default --dest-server https://kubernetes.default.svc --directory-recurse 
argocd app list
argocd app get argocd/nodejs-webapp
argocd app sync argocd/nodejs-webapp
kubectl port-forward svc/nodeapp-svc -n default 8000:8080
```
Now access the application in the browser `http://localhost:8000/`

## To delete app
```
argocd app delete argocd/nodejs-webapp
```
