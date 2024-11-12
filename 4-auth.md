# [Kubernetes Auth with Kuadrant](README.md)

## Run the steps

#### ❶ [Deploy an application as usual](1-deploy.md) ☑️
#### ❷ [Setup a gateway](2-gateway.md) ☑️
#### ❸ [Enable security capabilities for the gateways](3-kuadrant.md) ☑️
#### ❹ Add authentication and authorization to the application

Create an AuthPolicy:

```sh
kubectl apply -f -<<EOF
apiVersion: kuadrant.io/v1
kind: AuthPolicy
metadata:
  name: news-api-auth
spec:
  targetRef:
    group: gateway.networking.k8s.io
    kind: HTTPRoute
    name: news-api
  rules:
    authentication:
      k8s-tokens:
        kubernetesTokenReview:
          audiences:
          - https://kubernetes.default.svc.cluster.local
    authorization:
      k8s-rbac:
        kubernetesSubjectAccessReview:
          user:
            selector: auth.identity.user.username
          resourceAttributes:
            group:
              value: news-api.io
            verb:
              selector: request.method.@case:lower
            resource:
              selector: request.path.@extract:{"sep":"/","pos":1}
            namespace:
              value: default
EOF
```

Try the News Agency API with auth enabled:

```sh
curl -k --resolve api.news.io:443:$INGRESS_IP https://api.news.io/sports -i
# HTTP/2 401
# www-authenticate: Bearer realm="k8s-tokens"
# x-ext-auth-reason: credential not found
```

Create a ServiceAccount to represent a user called Alice inside the cluster:

```sh
kubectl apply -f -<<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: alice
EOF
```

Consume the News Agency API with a valid access token:

```sh
curl -k --resolve api.news.io:443:$INGRESS_IP -H "Authorization: Bearer $(kubectl create token alice)" https://api.news.io/sports -i
# HTTP/2 403
```

Define roles:

```sh
kubectl apply -f -<<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: publisher
rules:
- apiGroups: ["news-api.io"]
  resources:
  - sports
  - economy
  verbs:
  - post
  - get
  - delete
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: sports-reader
rules:
- apiGroups: ["news-api.io"]
  resources: ["sports"]
  verbs: ["get"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: economy-reader
rules:
- apiGroups: ["news-api.io"]
  resources: ["economy"]
  verbs: ["get"]
EOF
```

Add permissions for Alice:

```sh
kubectl apply -f -<<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: sports-readers
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: sports-reader
subjects:
- kind: ServiceAccount
  name: alice
EOF
```

Consume the application again now with permissions:

```sh
curl -k --resolve api.news.io:443:$INGRESS_IP -H "Authorization: Bearer $(kubectl create token alice)" https://api.news.io/sports -i
# []
```

Try a forbidden endpoint:

```sh
curl -k --resolve api.news.io:443:$INGRESS_IP -H "Authorization: Bearer $(kubectl create token alice)" https://api.news.io/economy -i
# HTTP/2 403
```

Create another user, called Niko, with permission to publish news articles:

```sh
kubectl apply -f -<<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: niko
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: publishers
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: publisher
subjects:
- kind: ServiceAccount
  name: niko
EOF
```

Create a news article as Niko:

```sh
curl -k --resolve api.news.io:443:$INGRESS_IP -H "Authorization: Bearer $(kubectl create token niko)" \
  -X POST -d '{"title": "Italy’s Luna Rossa showed two skippers are better than one on the America’s Cup yachts", "body": "BARCELONA, Spain (AP) — Before the last America’s Cup, Italy’s sailing team had what helmsman Francesco Bruni called a “crazy” idea: run the boat with two skippers, each taking turns steering as the foiling yacht crisscrossed the race course.\nWhile the other crews lost valuable time as their sole skipper scampered back and forth with each tack or jibe, Bruni and Jimmy Spithill stayed put, each manning their own helm on their side of the boat.\nIt turned out to be a stroke of genius. Source: https://apnews.com/article/americas-cup-sailing-italy-luna-rossa-spithill-d99078e7047bda9d9416c92e333a345d"}' \
  https://api.news.io/sports
# {"id": …}
```

Read news articles as Niko:

```sh
curl -k --resolve api.news.io:443:$INGRESS_IP -H "Authorization: Bearer $(kubectl create token niko)" https://api.news.io/sports -i
# [{"id": …}]
```

```sh
curl -k --resolve api.news.io:443:$INGRESS_IP -H "Authorization: Bearer $(kubectl create token niko)" https://api.news.io/economy -i
# []
```

#### ❺ [Add rate limiting to the application](5-rate-limit.md)

<br/>

#### [Cleanup](cleanup.md)
