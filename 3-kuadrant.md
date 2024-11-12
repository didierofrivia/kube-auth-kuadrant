# [Kubernetes Auth with Kuadrant](README.md)

## Run the steps

#### ❶ [Deploy an application as usual](1-deploy.md) ☑️
#### ❷ [Setup a gateway](2-gateway.md) ☑️
#### ❸ Enable security capabilities for the gateways

Install cert-manager:

```sh
helm repo add jetstack https://charts.jetstack.io --force-update
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.15.3 \
  --set crds.enabled=true
```

Install Kuadrant:

```sh
helm repo add kuadrant https://kuadrant.io/helm-charts/ --force-update
helm install \
 kuadrant-operator kuadrant/kuadrant-operator \
 --create-namespace \
 --namespace kuadrant-system \
 --devel
```

```sh
kubectl -n kuadrant-system apply -f - <<EOF
apiVersion: kuadrant.io/v1beta1
kind: Kuadrant
metadata:
  name: kuadrant
spec: {}
EOF
```

Enable TLS on the gateway with automatic certificate management:

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
  - name: https
    protocol: HTTPS
    port: 443
    tls:
      mode: Terminate
      certificateRefs:
      - name: ingress-gateway-cert
        kind: Secret
    hostname: "*.io"
    allowedRoutes:
      namespaces:
        from: All
---
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: kuadrant-ca
spec:
  selfSigned: {}
---
apiVersion: kuadrant.io/v1
kind: TLSPolicy
metadata:
  name: ingress-gateway-tls
  namespace: istio-system
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: Gateway
    name: ingress-gateway
  issuerRef:
    group: cert-manager.io
    kind: ClusterIssuer
    name: kuadrant-ca
EOF
```

Try to consume the News Agency API protected by Kuadrant:

```sh
curl --resolve api.news.io:443:$INGRESS_IP https://api.news.io/sports
# curl: (60) SSL certificate problem: unable to get local issuer certificate
# More details here: https://curl.se/docs/sslcerts.html
#
# curl failed to verify the legitimacy of the server and therefore could not
# establish a secure connection to it. To learn more about this situation and
# how to fix it, please visit the web page mentioned above.
```

```sh
curl --resolve api.news.io:443:$INGRESS_IP https://api.news.io/sports -i --insecure
# []
```

### Topology and request flow

![Architecture](architecture-step-3.png)

#### ❹ [Add authentication and authorization to the application](4-auth.md)
#### ❺ [Add rate limiting to the application](5-rate-limit.md)

<br/>

#### [Cleanup](cleanup.md)
