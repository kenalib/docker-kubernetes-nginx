
# Docker Container Kubernetes について


## トピック

* Container とは
  * container による package 化のメリット
  * コンテナによる仮想化と仮想マシンとの違い
* Docker とは
  * Dockerfile
  * Docker Hub
  * Docker compose
* Kubernetes とは
  * kubctl
  * Pod (host に相当) と Deployment (Pod の管理単位)
  * Service (外部向け IP アドレス)
  * minikube での利用
  * Container Service 上での利用


## Nginx を使った Hello World Hands-on

### 初期設定

* install home brew https://brew.sh/
* install VirtualBox https://www.virtualbox.org/wiki/Downloads

```
brew install bash-completion
brew install kubectl
brew cask reinstall minikube
minikube completion bash > /usr/local/etc/bash_completion.d/minikube
```

### クラスター作成

```
minikube start
minikube status
minikube dashboard
```

または、Container Service 上でクラスター作成

### run nginx by local Docker

```
docker build -t my-nginx .
docker run --name my-nginx -d -p 8080:80 my-nginx
open http://localhost:8080
```

### run nginx on cluster

```
export DOCKER_ID_USER="username"
docker login
docker tag my-nginx $DOCKER_ID_USER/my-nginx
docker push $DOCKER_ID_USER/my-nginx
kubectl run my-nginx --image=$DOCKER_ID_USER/my-nginx --port=80
kubectl get deployment
kubectl expose deployment my-nginx --type=NodePort
kubectl get service                       # Port 番号に注目
open $(minikube service my-nginx --url)
```

* Container Service の場合 `--type=LoadBalancer`
* `kubectl get service` で `EXTERNAL-IP` が表示される

### Scale

```
kubectl scale deployment my-nginx --replicas=2
```

```
git clone https://github.com/kubernetes/heapster.git
cd heapster
kubectl create -f deploy/kube-config/influxdb/
kubectl create -f deploy/kube-config/rbac/heapster-rbac.yaml
kubectl get services --namespace=kube-system
kubectl top node
kubectl top pod
```

### Container の中に入る方法

```
kubectl get pod
kubectl exec -it my-nginx-98946476b-hb66c -- /bin/bash
```


## 片付け

```
docker ps -a
docker stop my-nginx
docker rm my-nginx
```

```
kubectl delete service my-nginx
kubectl delete deployment my-nginx
minikube stop
```


## おまけ

```
eval $(minikube docker-env)
eval $(minikube docker-env -u)
kubectl edit pod
```


## 参考文献

Docker（コンテナ型仮想化）と Kubernetes についての簡単な紹介
https://ubiteku.oinker.me/2017/02/21/docker-and-kubernetes-intro/

Running Kubernetes Locally via Minikube
https://kubernetes.io/docs/getting-started-guides/minikube/
