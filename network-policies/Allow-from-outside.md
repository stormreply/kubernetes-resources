# ALLOW traffic from external clients

This Network Policy enables external clients from the public Internet directly
or via a Load Balancer to access to the pod.

```
kubectl run web --image=nginx \
    --labels=app=web --port 80

kubectl expose deployment/web --type=LoadBalancer
```

Wait until an EXTERNAL-IP appears on `kubectl get service` output. Visit the
`http://[EXTERNAL-IP]` on your browser and verify it is accessible.


The following manifest allows traffic from all sources (both internal from the
cluster and external). Save it to `web-allow-external.yaml` and apply to the
cluster:

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: web-allow-external
spec:
  podSelector:
    matchLabels:
      app: web
  ingress:
  - from: []
```

```
$ kubectl apply -f web-allow-external.yaml
```

Visit the `http://[EXTERNAL-IP]` on your browser again and verify it still
works.

To restrict external access only to port 80, you can deploy an ingress rule
such as:

```yaml
  ingress:
  - ports:
    - port: 80
    from: []
```

    kubectl delete deployment web
    kubectl delete service web
    kubectl delete networkpolicy web-allow-external
