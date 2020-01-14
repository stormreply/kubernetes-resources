# ALLOW traffic from apps using multiple selectors

Run a Redis database on your cluster:

    kubectl run db --image=redis:4 --port 6379 --expose \
        --labels app=bookstore,role=db

The following NetworkPolicy will allow traffic from only these microservices.
Save it to `redis-allow-services.yaml` and apply to the cluster:

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: redis-allow-services
spec:
  podSelector:
    matchLabels:
      app: bookstore
      role: db
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: bookstore
          role: search
    - podSelector:
        matchLabels:
          app: bookstore
          role: api
    - podSelector:
        matchLabels:
          app: inventory
          role: web
```

```sh
$ kubectl apply -f redis-allow-services.yaml
```

```sh
$ kubectl run test-$RANDOM --labels=app=inventory,role=web --rm -i -t --image=alpine -- sh

/ # nc -v -w 2 db 6379
db (10.59.242.200:6379) open

(works)
```

Pods with labels not matching these microservices will not be able to connect:

```sh
$ kubectl run test-$RANDOM --labels=app=other --rm -i -t --image=alpine -- sh

/ # nc -v -w 2 db 6379
nc: db (10.59.252.83:6379): Operation timed out

(traffic blocked)
```

    kubectl delete deployment db
    kubectl delete service db
    kubectl delete networkpolicy redis-allow-services
