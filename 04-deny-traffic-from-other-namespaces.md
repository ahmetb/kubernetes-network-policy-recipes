## DENY all traffic from other namespaces

You can configure a NetworkPolicy to drop all the traffic
from other namespaces while allowing all the traffic coming
from the namespace the pod is living on.

**Use Cases**
- You do not want deployments in `test` namespace to accidentally
  send traffic to other services or databases in `prod` namespace.
- You host applications from different customers in separate Kubernetes
  namespaces and you would like to block traffic coming from outside a
  namespace.

## Example

Create a new namespace called `secondary` and start a web service:

    kubectl create namespace secondary
    
    kubectl run web --namespace secondary --image=nginx \
        --labels=app=web --expose --port 80

Query this web service from the `default` namespace:

```sh
$ kubectl run test-$RANDOM --namespace=default --rm -i -t --image=alpine -- sh
/ # wget -qO- --timeout=2 http://web.secondary
<!DOCTYPE html>
<html>
(works)
```

Save the following manifest to `web-deny-other-namespaces.yaml` and apply
to the cluster:

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  namespace: secondary
  name: web-deny-other-namespaces
spec:
  podSelector:
    matchLabels:
      app: web
  ingress:
  - from:
    - podSelector: {}
```

```
$ kubectl apply web-deny-other-namespaces.yaml
networkpolicy "web-deny-other-namespaces" created"
```

Note a few things about this manifest:

- `namespace: secondary` deploys it to the `secondary` namespace.
- This manifest allows all connections to Pods with `app=web` label
  from the `secondary` namespace, and blocks the traffic from all
  other namespaces.
  
Query this web service from the `default` namespace again:

```sh
$ kubectl run test-$RANDOM --namespace=default --rm -i -t --image=alpine -- sh
/ # wget -qO- --timeout=2 http://web.secondary
wget: download timed out
(blocked)
```

Any pod in `secondary` namespace should work fine:

```sh
$ kubectl run test-$RANDOM --namespace=secondary --rm -i -t --image=alpine -- sh
/ # wget -qO- --timeout=2 http://web.secondary
<!DOCTYPE html>
<html>
(works)
```

### Cleanup

    kubectl delete deployment web -n secondary
    kubectl delete service web -n secondary
    kubectl delete networkpolicy --all -n secondary
    kubectl delete namespace secondary
