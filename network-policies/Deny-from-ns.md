## DENY all traffic from other namespaces

Create a new namespace called `secondary` and start a web service:

```sh
kubectl create namespace priv

kubectl run web --namespace priv --image=nginx \
    --labels=app=web --expose --port 80
```

Save the following manifest to `deny-from-other-namespaces.yaml` and apply
to the cluster:

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  namespace: priv
  name: deny-from-other-namespaces
spec:
  podSelector:
    matchLabels:
  ingress:
  - from:
    - podSelector: {}
```

```sh
$ kubectl apply -f deny-from-other-namespaces.yaml
```

## Test it

Query this web service from the `default` namespace:

```sh
$ kubectl run test-$RANDOM --namespace=default --rm -i -t --image=alpine -- sh
/ # wget -qO- --timeout=2 http://web.priv
wget: download timed out
```

It blocks the traffic from `default` namespace!

Any pod in `priv` namespace should work fine:

```sh
$ kubectl run test-$RANDOM --namespace=priv --rm -i -t --image=alpine -- sh
/ # wget -qO- --timeout=2 http://web.priv
<!DOCTYPE html>
<html>
```

### Cleanup

    kubectl delete deployment web -n secondary
    kubectl delete service web -n secondary
    kubectl delete networkpolicy deny-from-other-namespaces -n secondary
    kubectl delete namespace secondary
