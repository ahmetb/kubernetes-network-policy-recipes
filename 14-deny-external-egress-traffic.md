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
  # For DNS
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: kube-system
      podSelector:
        matchLabels:
          k8s-app: kube-dns
    ports:
      - port: 53
        protocol: UDP
      - port: 53
        protocol: TCP
  # For application with label app=web
  - to: 
    - podSelector:
        matchLabels:
          app: web
```

Few remarks about this policy:

* This policy applies to pods with `app=foo` and in Egress (outbound) direction.
* Similar to [DENY egress traffic from an
  application](11-deny-egress-traffic-from-an-application.md) example, this policy
  allows all outbound traffic on ports 53/udp and 53/tcp to the kube-dns pods for DNS resolution.
* `to:` specifies a `namespaceSelector` which matches `kubernetes.io/metadata.name: kube-system` and a `podSelector` which matches `k8s-app: kube-dns`. This will select only the kube-dns pods in the kube-system namespace, so the outbound traffic to the kube-dns pods in the kube-system namespace will be allowed.
* Allow to access the application with label app=web
* And since others are not listed except above rule, traffic to the IP addresses outside the cluster
  are denied.

Now apply it to the cluster:

```sh
kubectl apply -f foo-deny-external-egress.yaml
networkpolicy "foo-deny-egress" created
```

## Try it out

Run a web application named `web`:

    kubectl run web --image=nginx --labels="app=web" --expose --port=80

Run a pod with label `app=foo`. The policy will be enforced on this pod:

```sh
$ kubectl run --rm --restart=Never --image=alpine -i -t --labels="app=foo" test -- ash

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
kubectl delete pod,service web
kubectl delete networkpolicy foo-deny-external-egress
```

## Fun

The meme below* can be used to explain how your cluster looks like with a policy like this.

![image](https://user-images.githubusercontent.com/14810215/126358758-5b14dcd3-79df-4f85-a248-ee6b36e2e90e.png)

*: https://twitter.com/memenetes/status/1417227948206211082


