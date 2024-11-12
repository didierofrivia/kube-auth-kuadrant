# [Kubernetes Auth with Kuadrant](README.md)

## Run the steps

#### ❶ [Deploy an application as usual](1-deploy.md) ☑️
#### ❷ [Setup a gateway](2-gateway.md) ☑️
#### ❸ [Enable security capabilities for the gateways](3-kuadrant.md) ☑️
#### ❹ [Add authentication and authorization to the application](4-auth.md) ☑️
#### ❺ Add rate limiting to the application

Create a RateLimitPolicy:

```sh
kubectl apply -f - <<EOF
apiVersion: kuadrant.io/v1
kind: RateLimitPolicy
metadata:
  name: news-api-rate-limit
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: HTTPRoute
    name: news-api
  limits:
    global:
      rates:
      - limit: 2
        window: 10s
      counters:
      - expression: request.path
EOF
```

Try to consume the News Agency API with limits enforced:

```sh
for i in {1..10}; do curl -k --resolve api.news.io:443:$INGRESS_IP -H "Authorization: Bearer $(kubectl create token niko)" https://api.news.io/sports && sleep 1; done
```

```sh
for i in {1..10}; do curl -k --resolve api.news.io:443:$INGRESS_IP -H "Authorization: Bearer $(kubectl create token niko)" https://api.news.io/economy && sleep 1; done
```

### Topology and request flow

![Architecture](architecture-step-5.png)

<br/>

#### [Cleanup](cleanup.md)
