# ALLOW traffic from some pods in another namespace

Since Kubernetes v1.11, it is possible to combine `podSelector` and `namespaceSelector`
with an `AND` (intersection) operation.

:warning: This feature is available on Kubernetes v1.11 or after.  Most networking
plugins do not yet support this feature. Make sure to test this policy after you
deploy it to make sure it is working correctly.

## Example

Start a `web` application:

    kubectl run --generator=run-pod/v1 web --image=nginx \
        --labels=app=web --expose --port 80

Create a `other` namespace and label it:

    kubectl create namespace other
    kubectl label namespace/other team=operations

The following manifest restricts traffic to only pods with label `type=monitoring` in namespaces labelled `team=operations`. Save it to `web-allow-all-ns-monitoring.yaml` and apply to the cluster:

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: web-allow-all-ns-monitoring
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: web
  ingress:
    - from:
      - namespaceSelector:     # chooses all pods in namespaces labelled with team=operations
          matchLabels:
            team: operations  
        podSelector:           # chooses pods with type=monitoring
          matchLabels:
            type: monitoring
```

```sh
$ kubectl apply -f web-allow-all-ns-monitoring.yaml
networkpolicy.networking.k8s.io/web-allow-all-ns-monitoring created
```

## Try it out

Query this web server from `default` namespace, *without* labelling the application `type=monitoring`, observe it is **blocked**:

```sh
$ kubectl run --generator=run-pod/v1 test-$RANDOM --rm -i -t --image=alpine -- sh
If you don't see a command prompt, try pressing enter.
/ # wget -qO- --timeout=2 http://web.default
wget: download timed out

(traffic blocked)
```

Query this web server from `default` namespace, labelling the application `type=monitoring`, observe it is **blocked**:

```sh
kubectl run --generator=run-pod/v1 test-$RANDOM --labels type=monitoring --rm -i -t --image=alpine -- sh
If you don't see a command prompt, try pressing enter.
/ # wget -qO- --timeout=2 http://web.default
wget: download timed out

(traffic blocked)
```

Query this web server from `other` namespace, *without* labelling the application `type=monitoring`, observe it is **blocked**:

```sh
$ kubectl run --generator=run-pod/v1 test-$RANDOM --namespace=other --rm -i -t --image=alpine -- sh
If you don't see a command prompt, try pressing enter.
/ # wget -qO- --timeout=2 http://web.default
wget: download timed out

(traffic blocked)
```

Query this web server from `other` namespace, labelling the application `type=monitoring`, observe it is **allowed**:

```sh
kubectl run --generator=run-pod/v1 test-$RANDOM --namespace=other --labels type=monitoring --rm -i -t --image=alpine -- sh
If you don't see a command prompt, try pressing enter.
/ # wget -qO- --timeout=2 http://web.default
<!DOCTYPE html>
<html>
<head>
...
(traffic allowed)
```

## Cleanup

    kubectl delete networkpolicy web-allow-all-ns-monitoring
    kubectl delete namespace other
    kubectl delete pod web
    kubectl delete service web
