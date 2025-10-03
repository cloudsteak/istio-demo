# Istio demo

## Create kind cluster

```bash
kind create cluster --name istio-demo --config istio-cluster.yaml
```

## Install Istio

```bash
brew install istioctl
```

```bash
istioctl install --set profile=demo -y
```

## Verify the installation

```bash
kubectl get pods -n istio-system
```

## Create test namespace

```bash
kubectl create namespace istio-test
kubectl label namespace istio-test istio-injection=enabled --overwrite
kubectl get ns istio-test --show-labels
```

## Deploy sample application

```bash
kubectl apply -f 01-httpbin.yaml
kubectl get pods -n istio-test -o wide
kubectl get svc -n istio-test
```

## Create VirtualService and Gateway

```bash
kubectl apply -f 02-gw-vs.yaml
kubectl get gateway,virtualservice -n istio-test
```

## Modify /etc/hosts

```bash
echo "127.0.0.1 istio.demo.local" | sudo tee -a /etc/hosts
cat /etc/hosts | grep istio.demo.local
```

## Port forward Istio Ingress Gateway (on local machine)

```bash
kubectl -n istio-system port-forward svc/istio-ingressgateway 8080:80 &
```

*Note: You can stop the port-forwarding with `pkill -f "kubectl -n istio-system port-forward svc/istio-ingressgateway 8080:80"`*

## Test the application

```bash
curl -I http://istio.demo.local:8080
```

## Functional testing

```bash
curl -s http://istio.demo.local:8080/get | head -n 15
```


## Check service ingress

```bash
kubectl -n istio-system get svc istio-ingressgateway -o wide
```


## Centralized error handling

```bash
kubectl apply -f ingress-error-redirect.yaml
```

*Note: You can delete the error handling with `kubectl delete -f ingress-error-redirect.yaml` or `kubectl -n istio-system delete envoyfilter ingress-error-redirect`*


## Simulate error

```bash
kubectl -n istio-test scale deploy httpbin --replicas=0
```

## Test the error handling
```bash
curl -I http://istio.demo.local:8080

curl -s -i http://istio.demo.local:8080/get | sed -n '1p;/^location:/Ip'
```



