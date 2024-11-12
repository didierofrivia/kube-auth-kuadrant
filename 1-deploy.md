# [Kubernetes Auth with Kuadrant](README.md)

## Run the steps

#### ❶ Deploy an application as usual

Create a cluster:

```sh
kind create cluster --name kuadrant-demo
```

Deploy the **News Agency API** application (`news-api`):

```sh
kubectl apply -f -<<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: news-api
  labels:
    app: news-api
spec:
  selector:
    matchLabels:
      app: news-api
  template:
    metadata:
      labels:
        app: news-api
    spec:
      containers:
      - name: news-api
        image: quay.io/kuadrant/authorino-examples:news-api
        imagePullPolicy: Always
        env:
        - name: PORT
          value: "3000"
        tty: true
        ports:
        - containerPort: 3000
  replicas: 1
---
apiVersion: v1
kind: Service
metadata:
  name: news-api
  labels:
    app: news-api
spec:
  selector:
    app: news-api
  ports:
  - name: http
    port: 3000
    protocol: TCP
EOF

kubectl wait deployment/news-api --for=condition=Available --timeout=5m
```

Expose and send requests to the application:

```sh
kubectl port-forward service/news-api 3000:3000 2>&1 >/dev/null &
```

```sh
curl http://localhost:3000/sports -i
# []
```

### Topology and request flow

![Architecture](architecture-step-1.png)

#### ❷ [Setup a gateway](2-gateway.md)
#### ❸ [Enable security capabilities for the gateways](3-kuadrant.md)
#### ❹ [Add authentication and authorization to the application](4-auth.md)
#### ❺ [Add rate limiting to the application](5-rate-limit.md)

<br/>

#### [Cleanup](cleanup.md)
