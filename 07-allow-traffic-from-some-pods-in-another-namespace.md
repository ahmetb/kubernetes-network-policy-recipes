# ALLOW traffic from some pods in another namespace

![#f03c15](https://placehold.it/15/f03c15/000000?text=+) **This policy is not possible today.** ![#f03c15](https://placehold.it/15/f03c15/000000?text=+) 

If you need to allow Pods of a particular application to connect
to your application, you need to combine `podSelector` and `namespaceSelector`
with an `AND` (intersection) operation. However, `spec.ingress.from[]` field of the
NetworkPolicy object merges the result of  `podSelector` and `namespaceSelector`
with an `OR` (union), making this impossible.

For example, the following syntax is currently not possible as a single
rule must have either a `namespaceSelector` or a `podSelector`, not both:

```yaml
ingress:
  from:
  - namespaceSelector: {}  # chooses all pods in all namespaces
    podSelector:           # chooses pods with type=monitoring
      matchLabels:
        type: monitoring
```

See discussion at: https://groups.google.com/forum/#!topic/kubernetes-sig-network/prmH5Tq69dE
