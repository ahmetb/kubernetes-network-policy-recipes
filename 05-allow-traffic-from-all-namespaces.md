# ALLOW traffic to an application from all namespaces

This NetworkPolicy will allow traffic from all pods in all namespaces
to a particular application.

**Use Case:**
- You have a common service or a database which is used by deployments in
  different namespaces.

You do not need this policy unless there is already a NetworkPolicy [blocking traffic
to the application](01-deny-all-traffic-to-an-application.md) or a NetworkPolicy [blocking
non-whitelisted traffic to all pods in the namespace](03-deny-all-non-whitelisted-traffic-in-the-namespace.md).

![Diagram of  ALLOW traffic to an application from all namespaces policy](img/5.gif)

### Example

Start a web service on `default` namespace:

```sh
kubectl run --generator=run-pod/v1 web --image=nginx \
    --namespace default \
    --labels=app=web --expose --port 80
```

Save the following manifest to `web-allow-all-namespaces.yaml` and apply
to the cluster:

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  namespace: default
  name: web-allow-all-namespaces
spec:
  podSelector:
    matchLabels:
      app: web
  ingress:
  - from:
    - namespaceSelector: {}
```

```sh
$ kubectl apply -f web-allow-all-namespaces.yaml
networkpolicy "web-allow-all-namespaces" created"
```

Note a few things about this NetworkPolicy manifest:

- Applies the policy only to `app:web` pods in `default` namespace.
- Selects all pods in all namespaces (`namespaceSelector: {}`).
- By default, if you omit specifying a `namespaceSelector` it does not select
  any namespaces, which means it will allow traffic only from the namespace
  the NetworkPolicy is deployed to.

> Note: Dropping all selectors from the `spec.ingress.from` item has the same
> effect of matching all pods in all namespaces. e.g.:
>
>     ...
>        ingress:
>          - from:
>
> However, prefer the syntax in the full manifest clear expression of intent.


### Try it out

Create a new namespace called `secondary` and query this web service in the `default` namespace:

```sh
$ kubectl create namespace secondary

$ kubectl run --generator=run-pod/v1 test-$RANDOM --namespace=secondary --rm -i -t --image=alpine -- sh
/ # wget -qO- --timeout=2 http://web.default
<!DOCTYPE html>
<html>
<head>
```

Similarly, it also works if you query it from any pod deployed to `bar`.

### Cleanup

    kubectl delete pod web -n default
    kubectl delete service web -n default
    kubectl delete networkpolicy web-allow-all-namespaces -n default
    kubectl delete namespace secondary
