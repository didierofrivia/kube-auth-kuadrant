# [Kubernetes Auth with Kuadrant](README.md)

## Run the steps

#### ❶ [Deploy an application as usual](1-deploy.md) ☑️
#### ❷ [Setup a gateway](2-gateway.md) ☑️
#### ❸ [Enable security capabilities for the gateways](3-kuadrant.md) ☑️
#### ❹ [Add authentication and authorization to the application](4-auth.md) ☑️
#### ❺ [Add rate limiting to the application](5-rate-limit.md) ☑️

<br/>

#### Cleanup

Delete users, permissions, policies and rollback security features without deleting the News Agency API or infrastructure services:

```sh
kubectl delete namespace consumer

kubectl delete ratelimitpolicy news-api-rate-limit
kubectl delete authpolicy news-api-auth

kubectl delete serviceaccount alice
kubectl delete serviceaccount niko

kubectl delete rolebinding sports-readers
kubectl delete rolebinding economy-readers
kubectl delete rolebinding publishers
kubectl delete role publisher
kubectl delete role sports-reader
kubectl delete role economy-reader

kubectl delete tlspolicy ingress-gateway-tls -n istio-system
kubectl delete clusterissuer kuadrant-ca
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

Delete the News Agency API:

```sh
kubectl delete httproute news-api
kubectl delete service news-api
kubectl delete deployment news-api
```

Delete the cluster:

```sh
kind delete cluster --name kuadrant-demo
```
