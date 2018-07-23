# ALLOW all traffic to an application

**Use Case:** After applying a
[deny-all](01-deny-all-traffic-to-an-application.md) policy which blocks all
non-whitelisted traffic to the application, now you to allow access to an
application from all pods in the current namespace.

Applying this policy makes any other policies restricting the traffic to the pod
void, and allow all traffic to it from its namespace and other namespaces.

## Example

Start a `web` application:

    kubectl run web --image=nginx \
        --labels=app=web --expose --port 80

Save the following manifest to `web-allow-all.yaml`:

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: web-allow-all
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: web
  ingress:
  - {}
```

A few remarks about this manifest:

- `namespace: default` deploy this policy to the `default` namespace.
- `podSelector` applies the ingress rule to pods with `app: label`
- Only one ingress rule is specified, and **it is empty**.
  - Empty ingress rule (`{}`) allows traffic from all pods in the current
    namespace, as well as other namespaces. It corresponds to:

        - from:
            podSelector: {}
            namespaceSelector: {}

Now apply it to the cluster:

```sh
$ kubectl apply -f web-allow-all.yaml
networkpolicy "web-allow-all" created"
```

Also apply the [`web-deny-all`
policy](01-deny-all-traffic-to-an-application.md). This way you can validate
that applying `web-allow-all` will make the `web-deny-all` void.

### Try it out

    $ kubectl run test-$RANDOM --rm -i -t --image=alpine -- sh
    / # wget -qO- --timeout=2 http://web
    <!DOCTYPE html>
    <html><head>
    ...

Traffic is allowed.

### Cleanup

```sh
kubectl delete deployment,service web
kubectl delete networkpolicy web-allow-all web-deny-all
```
