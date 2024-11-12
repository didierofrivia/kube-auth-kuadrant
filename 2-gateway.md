# [Kubernetes Auth with Kuadrant](README.md)

## Run the steps

#### ❶ [Deploy an application as usual](1-deploy.md) ☑️
#### ❷ Setup a gateway

Install Kubernetes Gateway API:

```sh
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.1.0/standard-install.yaml
```

Install MetalLB:

```sh
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.7/config/manifests/metallb-native.yaml
kubectl wait -n metallb-system deployment/controller --for=condition=Available --timeout=300s
kubectl wait -n metallb-system pod --selector=app=metallb --for=condition=ready --timeout=60s
curl -s https://raw.githubusercontent.com/Kuadrant/kuadrant-operator/refs/tags/v0.10.0/utils/docker-network-ipaddresspool.sh | bash -s -- kind yq 1 | kubectl apply -n metallb-system -f -
```


Install the Istio Gateway Controller:

```sh
helm install sail-operator \
		--create-namespace \
		--namespace istio-system \
		--wait \
		--timeout=300s \
		https://github.com/istio-ecosystem/sail-operator/releases/download/0.1.0/sail-operator-0.1.0.tgz

kubectl apply -f -<<EOF
apiVersion: sailoperator.io/v1alpha1
kind: Istio
metadata:
  name: default
spec:
  # Supported values for sail-operator v0.1.0 are [v1.22.4,v1.23.0]
  version: v1.23.0
  namespace: istio-system
  # Disable autoscaling to reduce dev resources
  values:
    pilot:
      autoscaleEnabled: false
EOF
```


Deploy an ingress gateway:

```sh
kubectl apply -f -<<EOF
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: ingress-gateway
  namespace: istio-system
spec:
  gatewayClassName: istio
  listeners:
  - name: http
    protocol: HTTP
    port: 80
    hostname: "*.io"
    allowedRoutes:
      namespaces:
        from: All
EOF
```

Connect the News Agency API through the gateway:

```sh
kubectl apply -f -<<EOF
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: news-api
spec:
  parentRefs:
  - name: ingress-gateway
    namespace: istio-system
  hostnames:
  - api.news.io
  rules:
  - matches:
    - method: POST
    - method: GET
    - method: DELETE
    backendRefs:
    - name: news-api
      port: 3000
EOF
```

Consume the News Agency API through the gateway:

```sh
export INGRESS_IP=$(kubectl get gateway/ingress-gateway -n istio-system -o jsonpath='{.status.addresses[0].value}')
```

```sh
curl --resolve api.news.io:80:$INGRESS_IP http://api.news.io/sports -i
# []
```

### Topology and request flow

![Architecture](architecture-step-2.png)

#### ❸ [Enable security capabilities for the gateways](3-kuadrant.md)
#### ❹ [Add authentication and authorization to the application](4-auth.md)
#### ❺ [Add rate limiting to the application](5-rate-limit.md)

<br/>

#### [Cleanup](cleanup.md)
