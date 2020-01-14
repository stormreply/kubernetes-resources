# DENY all traffic to an application

Run a nginx Pod with labels `app=web`  and expose it at port 80:

    kubectl run web --image=nginx --labels app=web --expose --port 80

Run a temporary Pod and make a request to `web` Service:

    $ kubectl run --rm -i -t --image=alpine test-$RANDOM -- sh
    / # wget -qO- http://web
    <!DOCTYPE html>
    <html>
    <head>
    ...

It works, now save the following manifest to `web-deny-all.yaml`,
then apply to the cluster:

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: web-deny-all
spec:
  podSelector:
    matchLabels:
      app: web
  ingress: []
```

```sh
$ kubectl apply -f web-deny-all.yaml
```

## Try it out

Run a test container again, and try to query web:

    $ kubectl run --rm -i -t --image=alpine test-$RANDOM -- sh
    / # wget -qO- --timeout=2 http://web
    wget: download timed out

Traffic dropped!

```sh
kubectl delete deploy web
kubectl delete service web
kubectl delete networkpolicy web-deny-all
```
