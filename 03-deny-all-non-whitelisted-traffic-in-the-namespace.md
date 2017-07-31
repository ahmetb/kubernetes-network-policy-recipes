# DENY all non-whitelisted traffic in the current namespace

This is a fundamental policy, blocking all cross-pod networking other
than the ones whitelisted via the other Network Policies you deploy.

**Use Case:** Consider applying this manifest to any namespace you deploy
workloads to (anything but `kube-system`). This will give you a base "deny all"
policy so you can clearly identify components that need to talk to each other
and deploy Ntework Policies for.


## Manifest

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: default-deny-all
  namespace: default
spec:
  podSelector:
    matchLabels:
```

Note a few things about this manifest:

- `namespace: default` deploy this policy to the `default` namespace.
- `matchLabels:` is empty, this means it will match all the pods. Therefore
  the policy will be enforced to ALL pods in this namspace.
- There are no `ingress` rules, effectively causing traffic to be dropped to
  the selected (all) pods.
  
Save this manifest to `default-deny-all.yaml` and apply:

```sh
$ kubectl apply -f default-deny-all.yaml
networkpolicy "default-deny-all" created
```

### Cleanup

    kubectl delete networkpolicy default-deny-all
