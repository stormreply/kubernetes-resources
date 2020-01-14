# ALLOW all traffic from a namespace

Run a web server in the `default` namespace:

    kubectl run web --image=nginx \
        --labels=app=web --expose --port 80


```
kubectl create namespace dev
kubectl label namespace/dev purpose=testing
```

```
kubectl create namespace prod
kubectl label namespace/prod purpose=production
```

The following manifest restricts traffic to only pods in namespaces
that has label `purpose=production`. Save it to `web-allow-prod.yaml`
and apply to the cluster:

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: web-allow-prod
spec:
  podSelector:
    matchLabels:
      app: web
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          purpose: production
```

```
$ kubectl apply -f web-allow-prod.yaml
```

The web server from `dev` namespace is blocked:

```
$ kubectl run test-$RANDOM --namespace=dev --rm -i -t --image=alpine -- sh

# wget -qO- --timeout=2 http://web.default
wget: download timed out

(traffic blocked)
```

Query it from `prod` namespace, observe it is allowed:

```sh
$ kubectl run test-$RANDOM --namespace=prod --rm -i -t --image=alpine -- sh
 # wget -qO- --timeout=2 http://web.default
<!DOCTYPE html>
<html>
<head>
...
(traffic allowed)
```

    kubectl delete networkpolicy web-allow-prod
    kubectl delete deployment web
    kubectl delete service web
    kubectl delete namespace {prod,dev}
