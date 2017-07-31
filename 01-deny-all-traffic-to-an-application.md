# DENY all traffic to an application

This NetworkPolicy will drop all traffic to pods of an
application, selected using Pod Selectors.

**Use Cases:** You want to run a Pod and want to prevent any other
Pods communicating with it.

If you deploy other Network Policies that match to this application
and allow traffic, this rule will be ineffective.

### Example

Run a nginx Pod with labels `app=web` and `env=prod` and expose it at port 80:

    kubectl run web --image=nginx --labels app=web,env=prod --expose --port 80
    
Run a temporary Pod and make a request to `web` Service:

    $ kubectl run --rm -i -t --image=alpine test-$RANDOM -- sh
    / # wget -qO- http://web
    <!DOCTYPE html>
    <html>
    <head>
    ...
    
It works, now save the following manifest to `web-deny-all.yaml`,
then apply to the cluster:

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: web-deny-all
spec:
  podSelector:
    matchLabels:
      app: web
      env: prod
```

```
$ kubectl apply -f web-deny-all.yaml
networkpolicy "access-nginx" created
```

Run a test container again, and try to query web:

    $ kubectl run --rm -i -t --image=alpine test-$RANDOM -- sh
    / # wget -qO- --timeout=2 http://web
    wget: download timed out

Traffic dropped!

-----

In the manifest above, we target Pods with `app=web,env=prod` labels
to police the newtork. This manifest file is missing the `spec.ingress` field.
Therefore it is not allowing any traffic into the Pod.

If you create another NetworkPolicy that gives some Pods access to this application
directly or indirectly, this NetworkPolicy will be ineffective. If there is at least
one NetworkPolicy with a rule allowing the traffic, any policies blocking the traffic
are ineffective.
