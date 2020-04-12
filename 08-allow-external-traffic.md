# ALLOW traffic from external clients

This Network Policy enables external clients from the public Internet directly
or via a Load Balancer to access to the pod.

**Use Cases:**
- You need to expose the pods to the public Internet in a namespace [denying all
  non-whitelisted
  traffic](03-deny-all-non-whitelisted-traffic-in-the-namespace.md)

![Diagram of ALLOW traffic from external clients policy](img/8.gif)

### Example

Run a web server and expose it to the internet with a Load Balancer:

```sh
kubectl run --generator=run-pod/v1 web --image=nginx \
    --labels=app=web --port 80

kubectl expose pod/web --type=LoadBalancer
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

```sh
$ kubectl apply -f web-allow-external.yaml
networkpolicy "web-allow-external" created
```

Visit the `http://[EXTERNAL-IP]` on your browser again and verify it still
works.

### Remarks

This manifest specifies one ingress rule for the `app=web` pods. Since it does
not specify a particular `podSelector` or `namespaceSelector`, it allows traffic
from all resources, including external.

To restrict external access only to port 80, you can deploy an ingress rule
such as:

```yaml
  ingress:
  - ports:
    - port: 80
    from: []
```

### Cleanup

    kubectl delete pod web
    kubectl delete service web
    kubectl delete networkpolicy web-allow-external
