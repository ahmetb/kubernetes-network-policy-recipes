# ALLOW traffic from apps using multiple selectors

NetworkPolicy lets you define multiple pod selectors to allow traffic from.

**Use Case**
- Create a combined NetworkPolicy that has the list of microservices that
  are allowed to connect to an application.

### Example

Run a Redis database on your cluster:

    kubectl run --generator=run-pod/v1 db --image=redis:4 --port 6379 --expose \
        --labels app=bookstore,role=db

Suppose you would like to share this Redis database between multiple
microservices:

| service    | labels |
|------------|--------|
| `search`   | `app=bookstore`<br/>`role=search` |
| `api`      | `app=bookstore`<br/>`role=api` |
| `catalog`  | `app=inventory`<br/>`role=web` |

The following NetworkPolicy will allow traffic from only these microservices.
Save it to `redis-allow-services.yaml` and apply to the cluster:

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: redis-allow-services
spec:
  podSelector:
    matchLabels:
      app: bookstore
      role: db
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: bookstore
          role: search
    - podSelector:
        matchLabels:
          app: bookstore
          role: api
    - podSelector:
        matchLabels:
          app: inventory
          role: web
```

```sh
$ kubectl apply -f redis-allow-services.yaml
networkpolicy "redis-allow-services" created
```

Note that:

- Rules specified in `spec.ingress.from` are `OR`'ed.
- This means the pods selected by the selectors are combined
  are whitelisted altogether.

### Try it out

Run a pod that looks like the "catalog" microservice:

```sh
$ kubectl run --generator=run-pod/v1 test-$RANDOM --labels=app=inventory,role=web --rm -i -t --image=alpine -- sh

/ # nc -v -w 2 db 6379
db (10.59.242.200:6379) open

(works)
```

Pods with labels not matching these microservices will not be able to connect:

```sh
$ kubectl run --generator=run-pod/v1 test-$RANDOM --labels=app=other --rm -i -t --image=alpine -- sh

/ # nc -v -w 2 db 6379
nc: db (10.59.252.83:6379): Operation timed out

(traffic blocked)
```

### Cleanup

    kubectl delete pod db
    kubectl delete service db
    kubectl delete networkpolicy redis-allow-services
