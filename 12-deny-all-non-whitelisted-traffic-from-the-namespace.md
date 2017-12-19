# DENY all non-whitelisted traffic from a namespace

💡 **Use Case:** This is a fundamental policy, blocking all outgoing (egress)
traffic from a namespace by default (including DNS resolution). After deploying
this, you can deploy Network Policies that allow the specific outgoing traffic.

Consider applying this manifest to any namespace you deploy workloads to
(except `kube-system`).

💡 **Best Practice:**  This policy will give you a default "deny all"
functionality. This way, you can clearly identify which components have
dependency on which components and deploy Network Policies which can be
translated to dependency graphs between components.

## Manifest

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: default-deny-all-egress
  namespace: default
spec:
  policyTypes:
  - Egress
  podSelector: {}
  egress: []
```


Note a few things about this manifest:

- `namespace: default` deploy this policy to the `default` namespace.
- `podSelector:` is empty, this means it will match all the pods. Therefore,
  the policy will be enforced to ALL pods in the `default` namespace.
- List of `egress` rules is an empty array: This causes all traffic (including
  DNS resolution) to be dropped if it’s originating from Pods in `default`.

Save this manifest to `default-deny-all-egress.yaml` and apply:

```sh
$ kubectl apply -f default-deny-all-egress.yaml
networkpolicy "default-deny-all-egress" created
```

### Cleanup

    kubectl delete networkpolicy default-deny-all-egress
