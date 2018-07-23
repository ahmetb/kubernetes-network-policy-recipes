# DENY external egress traffic

_(a.k.a LIMIT traffic to pods in the cluster)_

**Use Cases:**

- You want to prevent certain type of applications from establishing connections
  to the external networks.

> **NOTE:** If you are using Google Kubernetes Engine (GKE), make sure you have
> at least `1.8.4-gke.0` master and nodes version to be able to use egress
> policies.

## Example

Save this policy to `foo-deny-external-egress.yaml`:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: foo-deny-external-egress
spec:
  podSelector:
    matchLabels:
      app: foo
  policyTypes:
  - Egress
  egress:
  - ports:
    - port: 53
      protocol: UDP
    - port: 53
      protocol: TCP
  - to:
    - namespaceSelector: {}
```

Few remarks about this policy:

* This policy applies to pods with `app=foo` and in Egress (outbound) direction.
* Similar to [DENY egress traffic from an
  application](11-deny-egress-traffic-from-an-application.md) example, this policy
  allows all outbound traffic on ports 53/udp and 53/tcp for DNS resolution.
* `to:` specifies an empty `namespaceSelector`. This will select **all pods in
  all namespaces**, so the outbound traffic to pods in the cluster will be
  allowed.
  * And since they are not listed, traffic to the IP addresses outside the cluster
    are denied.

Now apply it to the cluster:

```sh
kubectl apply -f foo-deny-egress.yaml
networkpolicy "foo-deny-egress" created
```

## Try it out

Run a web application named `web`:

    kubectl run web --image=nginx --port 80 --expose \
        --labels app=web

Run a pod with label `app=foo`. The policy will be enforced on this pod:

```sh
$ kubectl run --rm --restart=Never --image=alpine -i -t -l app=foo test -- ash

/ # wget -O- --timeout 1 http://web:80
Connecting to web (10.59.245.232:80)
<!DOCTYPE html>
<html>
...
(connection is allowed)
```

The pod with `app=foo` label is able to connect to `web` Service.

Now try with an external address:

```sh
/ # wget -O- --timeout 1 http://www.example.com
Connecting to www.example.com (93.184.216.34:80)
wget: download timed out
(connection is blocked)

/ # exit
```

The pod is able to resolve the IP address of `www.example.com`, however it
cannot establish a connection. Effectively, external traffic is blocked.

## Cleanup

```sh
kubectl delete deployment,service web
kubectl delete networkpolicy foo-deny-external-egress
```
