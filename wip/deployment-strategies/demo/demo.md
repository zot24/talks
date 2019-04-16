Questions?
===

- Does anyone know what Docker and a container is?
- Does anyone know what Kubernetes is for?
- Does anyone know what a Kubernetes manifest file is for?
- Does anyone know what kubectl is for?
- Does anyone know what a service mesh is for?

---

Setup
===

#### clone container solutions k8s-deployment-strategies repo

```ssh
git clone git@github.com:ContainerSolutions/k8s-deployment-strategies.git strategies
cd k8s-deployment-strategies
git checkout 733386675d911aabcf8516fa59d320cc5559623a
```

#### run minikube

```ssh
minikube start \
    --kubernetes-version v1.13.5 \
    --memory=8192 \
    --cpus 2 \
    --bootstrapper=kubeadm \
    --extra-config=kubelet.authentication-token-webhook=true \
    --extra-config=kubelet.authorization-mode=Webhook \
    --extra-config=scheduler.address=0.0.0.0 \
    --extra-config=controller-manager.address=0.0.0.0
```

#### install helm

```ssh
helm init
```

#### install prometheus

```ssh
helm install \
    --namespace=monitoring \
    --name=prometheus \
    stable/prometheus
```

#### install grafana configmap for dashboards and datasources

```ssh
kubectl apply -f manifests/
```

#### install grafana

```ssh
helm install \
    --namespace=monitoring \
    --name=grafana \
    --set=adminUser=admin \
    --set=adminPassword=admin \
    --set=service.type=NodePort \
    --set=sidecar.dashboards.enabled=true \
    --set=sidecar.datasources.enabled=true \
    stable/grafana
```

#### access grafana dashboard

```ssh
minikube service list
```

---

Recreate
===

will terminate all running instances and recreate them with the new deployed version

- easier to implement
- method focus on development purposes
- downtime depends on shutdown + boot duration of the application

```yaml
spec:
  replicas: 3
  strategy:
    type: Recreate
```

## Demo

```ssh
# open a new terminal and watch the deployments in action
watch kubectl get po,rs

# deploy version 1
kubectl apply -f strategies/recreate/app-v1.yaml

# check deployment
curl $(minikube service my-app --url)

# open a new terminal, hammer it and watch the next step progress
service=$(minikube service my-app --url)
while sleep 0.1; do curl -sS  "$service"; done | lolcat

# deploy version 2 | WHAT DO YOU THINK WILL HAPPEN NOW!?
kubectl apply -f strategies/recreate/app-v2.yaml

# remove it all!
kubectl delete all -l app=my-app
```

# Grafana dashboard

> Version 1 is terminated then version 2 is rolled out

![](img/grafana/recreate.png)

---


Ramped (slow roll out)
===

slowly rolling out a version of an application by replacing instances one after the other until all the instances are rolled out

- 0 downtime
- slowly release
- rollout/rollback can take time
- no control over traffic

![](img/strategies/ramped.png)

```yaml
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # how many instances to add in addition of the current amount
      maxUnavailable: 0  # number of unavailable instances during the rolling update
```

## Demo

```ssh
# open a new terminal and watch the deployments in action
watch kubectl get po,rs

# deploy version 1
kubectl apply -f strategies/ramped/app-v1.yaml

# check deployment
curl $(minikube service my-app --url)

# open a new terminal, hammer it and watch the next step progress
service=$(minikube service my-app --url)
while sleep 0.1; do curl -sS  "$service"; done | lolcat
          
# deploy version 2 | WHAT DO YOU THINK WILL HAPPEN NOW!?
kubectl apply -f strategies/ramped/app-v2.yaml

# remove it all!
kubectl delete all -l app=my-app
```

## Grafana dashboard

> Version 2 is slowly rolled out and replacing version 1. Also known as rolling-update or incremental


![](img/grafana/ramped.png)

---

Blue / Green (w/traefik)
===

version 2 it's deploy in fully along side version 1, once we are happy with the deployment we switch the traffic and release the new version

- costly
- 0 downtime 
- easy/fast rollbacks


![](img/strategies/blue-green.png)

## Demo

```ssh
# open a new terminal and watch the deployments in action
watch kubectl get po,rs

# install traefik loadbalancer/proxy
helm install \
    --name=traefik \
    --set rbac.enabled=true \
    --set dashboard.enabled=true \
    --set dashboard.serviceType=NodePort \
    --set dashboard.domain=traefik.dashboard.com \
    stable/traefik

# deploy version 1
kubectl apply \
    -f strategies/blue-green/multiple-services/app-a-v1.yaml \
    -f strategies/blue-green/multiple-services/app-b-v1.yaml \
    -f strategies/blue-green/multiple-services/ingress-v1.yaml

# open a two new terminals, hammer both versions and watch the next step progress
ingress=$(minikube service traefik --url | head -n1)
while sleep 0.1; do curl -sS  "$ingress" -H "Host: a.domain.com" ; done | lolcat
while sleep 0.1; do curl -sS  "$ingress" -H "Host: b.domain.com" ; done | lolcat

# deploy version 2 | WHAT DO YOU THINK WILL HAPPEN NOW!?
kubectl apply \
    -f strategies/blue-green/multiple-services/app-a-v2.yaml \
    -f strategies/blue-green/multiple-services/app-b-v2.yaml

# release version 2 | WHAT DO YOU THINK WILL HAPPEN NOW!?
kubectl apply -f strategies/blue-green/multiple-services/ingress-v2.yaml

# let's rollback coz I want to! :)
kubectl apply -f strategies/blue-green/multiple-services/ingress-v1.yaml

# remove it all!
kubectl delete \
    -f strategies/blue-green/multiple-services/app-a-v2.yaml \
    -f strategies/blue-green/multiple-services/app-b-v2.yaml

kubectl delete \
    -f strategies/blue-green/multiple-services/app-a-v1.yaml \
    -f strategies/blue-green/multiple-services/app-b-v1.yaml \
    -f strategies/blue-green/multiple-services/ingress-v1.yaml
    
helm del traefik && helm del --purge traefik
```

## Grafana dashboard

> Version 2 is released alongside version 1, then the traffic is switched to version 2. Also known as red/black deployment

![](img/grafana/blue-green.png)

---

Canary (w/ingress-nginx)
===

gradually shifting production traffic from version 1 to version 2. Usually the traffic is split based on weight. For example, 90 percent of the requests go to version 1, 10 percent go to version 2

- 0 downtime
- easy/fast rollbacks
- real traffic testing
- check your app progresivelly
- bit more complex to implement
- useful when app lack test, it's no reliable or not stable

![](img/strategies/canary.png)

## Demo

```ssh
# open a new terminal and watch the deployments in action
watch kubectl get po,rs

# deploy ingress-nginx
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/mandatory.yaml

# expose the ingress-nginx
kubectl expose deployment \
    -n ingress-nginx nginx-ingress-controller \
    --port 80 \
    --type NodePort \
    --name ingress-nginx
    
# check the ingress-nginx deployment
kubectl get all -n ingress-nginx
curl $(m service ingress-nginx --url -n ingress-nginx)

# deploy version 1
kubectl apply \
    -f strategies/canary/nginx-ingress/app-v1.yaml \
    -f strategies/canary/nginx-ingress/ingress-v1.yaml
    
# deploy version 2
kubectl apply -f strategies/canary/nginx-ingress/app-v2.yaml

# open a new terminal, hammer it and watch the next step progress
nginx_service=$(minikube service ingress-nginx -n ingress-nginx --url)
while sleep 0.1; do curl -sS "$nginx_service" -H "Host: my-app.com"; done | lolcat

# create a canary ingress in order to split traffic: 90% to v1, 10% to v2
kubectl apply -f strategies/canary/nginx-ingress/ingress-v2-canary.yaml

# play with different values and re-apply config, e.g. 25% to v1, 75% to v2
kubectl apply -f strategies/canary/nginx-ingress/ingress-v2-canary.yaml

# When you are happy, delete the canary ingress
kubectl delete -f strategies/canary/nginx-ingress/ingress-v2-canary.yaml

# Then finish the rollout, set 100% traffic to version 2
kubectl apply -f strategies/canary/nginx-ingress/ingress-v2.yaml

# remove it all!
kubectl delete all,ingress -l app=my-app
kubectl delete namespace ingress-nginx
```

## Grafana dashboard

> Version B is released to a subset of users, then proceed to a full rollout

![](img/grafana/canary.png)

---

A/B testing (w/istio)
===

routing a subset of users to a new functionality under specific conditions, widely used for taking marketing decisions, to test conversion of a given feature and only roll-out the version that converts the most

- 0 downtime
- making business decisions
- conditions that can be used to distribute traffic amongst the versions
	- Weight
	- Cookie value
	- Query parameters
	- Geolocalisation
	- Technology support: browser version, screen size, operating system, etc.
	- Language

![](img/strategies/a-b-testing.png)

## Demo

```ssh
# open a new terminal and watch the deployments in action
watch kubectl get all,virtualservice,gateway

# install istio
curl -L https://git.io/getLatestIstio | ISTIO_VERSION=1.0.7 sh -
curl -L https://git.io/getLatestIstio | ISTIO_VERSION=1.0.0 sh -
helm install istio-*/install/kubernetes/helm/istio --name istio --namespace istio-system

# enable istio automatic sidecar injection
kubectl label namespace default istio-injection=enabled

# deploy version 1 and version 2 and identify the proxy/sidecar describing it
kubectl apply -f strategies/ab-testing/app-v1.yaml -f strategies/ab-testing/app-v2.yaml

# expose both services via the Istio Gateway and create a VirtualService to match
requests to the my-app-v1 service:
kubectl apply -f strategies/ab-testing/gateway.yaml -f strategies/ab-testing/virtualservice.yaml

# check deployment works as expected
curl $(minikube service istio-ingressgateway -n istio-system --url | head -n1) -H 'Host: my-app.local'

# check traffic per header
istio_ingressgateway=$(minikube service istio-ingressgateway -n istio-system --url | head -n1)
while sleep 0.1; do curl -sS "$istio_ingressgateway" -H "Host: my-app.local" -H "X-API-Version: v1.0.0"; done | lolcat
while sleep 0.1; do curl -sS "$istio_ingressgateway" -H "Host: my-app.local" -H "X-API-Version: v2.0.0"; done | lolcat

# test traffic base on specific user (header) | WHAT DO YOU THINK WILL HAPPEN NOW!?
kubectl apply -f strategies/ab-testing/virtualservice-match.yaml

# remove it all!
kubectl delete all,ingress,gateway,virtualservice -l app=my-app
helm delete --purge istio
kubectl -n istio-system delete job --all
kubectl delete -f istio-*/install/kubernetes/helm/istio/templates/crds.yaml -n istio-system
kubectl delete namespace istio-system
rm -rf istio-*

?

This is the same as the canary example but using Istio

# test deployment
# istio_ingressgateway=$(minikube service istio-ingressgateway -n istio-system --url | head -n1)
# while sleep 0.1; do curl -sS "$istio_ingressgateway" -H "Host: my-app.local"; done | lolcat

# test shiftt traffic base on percentage (canary)
# kubectl apply -f strategies/ab-testing/virtualservice-weight.yaml
```

## Grafana dashboard

> Version B is released to a subset of users under specific condition

![](img/grafana/ab-testing.png)

---

Shadow (w/istio)
===

A shadow deployment consists of releasing version 2 alongside version 1, fork version 1’s incoming requests and send them to version 2 as well without impacting production traffic. 

- test production load on a new feature
- rollout happens when we hit stability and performance
- complex to implement, e.g. egress traffic on an ecommerce solution

## Demo

```ssh
# open a new terminal and watch the deployments in action
watch kubectl get all,virtualservice,gateway

# install istio
curl -L https://git.io/getLatestIstio | sh -
helm install istio-*/install/kubernetes/helm/istio --name istio --namespace istio-system

# enable istio automatic sidecar injection
kubectl label namespace default istio-injection=enabled

# deploy version 1 and version 2 and identify the proxy/sidecar describing it
kubectl apply -f strategies/shadow/app-v1.yaml -f strategies/shadow/app-v2.yaml

# expose both services via the Istio Gateway and create a VirtualService to match
requests to the my-app-v1 service:
kubectl apply -f strategies/shadow/gateway.yaml -f strategies/shadow/virtualservice.yaml

# check deployment works as expected
curl $(minikube service istio-ingressgateway -n istio-system --url | head -n1) -H 'Host: my-app.local'

# check traffic only V1 should be hit at this point
istio_ingressgateway=$(minikube service istio-ingressgateway -n istio-system --url | head -n1)
while sleep 0.1; do curl -sS "$istio_ingressgateway" -H "Host: my-app.local"; done | lolcat

# enable traffic mirroying / nothing change on the console command
kubectl apply -f strategies/shadow/virtualservice-mirror.yaml

# remove it all!
kubectl delete all,ingress,gateway,virtualservice -l app=my-app
helm delete --purge istio
kubectl -n istio-system delete job --all
kubectl delete -f istio-*/install/kubernetes/helm/istio/templates/crds.yaml -n istio-system
kubectl delete namespace istio-system
rm -rf istio-*
```

## Grafana dashboard

> Version 2 receives real-world traffic alongside version 1 and doesn’t impact the response

![](img/grafana/shadow.png)

---

Cleaning up
===

```ssh
# remove grafana
helm delete --purge grafana

# remove prometheus
helm delete --purge prometheus

# remove monitoring namespace
kubectl delete namespace monitoring
```

---


Extra commands
===

```ssh
# deploy grafana as w/guest user enable by default
helm install \
    --namespace=monitoring \
    --name=grafana \
    --set=service.type=NodePort \
    --set=sidecar.dashboards.enabled=true \
    --set=sidecar.datasources.enabled=true \
    --set 'grafana\.ini'.'auth\.anonymous'.enabled=true \
    stable/grafana
```
