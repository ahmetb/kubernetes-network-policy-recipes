## DENY egress traffic from an application

**Use Cases:**

- You want to prevent an application from establishing any connections to
  outside of the Pod.
- Useful for restricting outbound traffic of single-instance databases and
  datastores.

> **NOTE:** If you are using Google Kubernetes Engine (GKE), make sure you have
> at least `1.8.4-gke.0` master and nodes version to be able to use egress
> policies.

## Example

Run a web application with `app=web` label:

    kubectl run --generator=run-pod/v1 web --image=nginx --port 80 --expose \
        --labels app=web

Save the following to `foo-deny-egress.yaml` and apply to the cluster:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: foo-deny-egress
spec:
  podSelector:
    matchLabels:
      app: foo
  policyTypes:
  - Egress
  egress: []
```

Remarks about this manifest file:
- `podSelector` matches to `app=foo` pods
- `policyTypes: ["egress"]` indicates that this policy enforces policies for the
  egress (outbound) traffic.
- `egress: []` empty rule set does not whitelist any traffic, therefore all
  egress (outbound) traffic is blocked.
  - You can drop this field altogether and have the same effect.

```sh
kubectl apply -f foo-deny-egress.yaml
networkpolicy "foo-deny-egress" created
```

### Try it out

Run a pod with label `app=foo`, and try to connect to the `web` service:

```sh
$ kubectl run --generator=run-pod/v1 --rm --restart=Never --image=alpine -i -t -l app=foo test -- ash

/ # wget -qO- --timeout 1 http://web:80/
wget: bad address 'web:80'

/ # wget -qO- --timeout 1 http://www.example.com/
wget: bad address 'www.example.com'
```

What's failing is not establishing connection to the `web` hostname: The pod is
**failing to resolve the the address**, because the network policy is not
allowing it to establish connections to the `kube-dns` Pods.

### Allowing DNS traffic

So we slightly modify the YAML file to allow all outbound traffic on DNS ports
(`53/udp` and `53/tcp`):

```sh
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: foo-deny-egress
spec:
  podSelector:
    matchLabels:
      app: foo
  policyTypes:
  - Egress
  egress:
  # allow DNS resolution
  - ports:
    - port: 53
      protocol: UDP
    - port: 53
      protocol: TCP
```

Now when we try again, we actually see the IP addresses are resolved, but
the traffic is blocked:

```sh
/ # wget --timeout 1 -O- http://web
Connecting to web (10.59.245.232:80)
wget: download timed out

/ # wget --timeout 1 -O- http://www.example.com
Connecting to www.example.com (93.184.216.34:80)
wget: download timed out

/ # ping google.com
PING google.com (74.125.129.101): 56 data bytes
(...)
(ping does not work, hit ctrl+C to terminate)

/ # exit
```

Beware that the egress rule above allows Pod to connect not only `kube-dns`,
but any host that serves traffic over port `53`.

## Cleanup

```
kubectl delete pod,service web
kubectl delete networkpolicy foo-deny-egress
```
