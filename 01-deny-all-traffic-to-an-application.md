# DENY all traffic to an application

This Network Policy will drop all traffic to pods of an
application, selected using Pod Selectors.

**Use Cases:** You want to run a Pod and want to prevent any other
Pods communicating with it.

If you deploy other Network Policies that match to this application
and allow traffic, this rule will be ineffective.

### Example

Run a nginx Deployment with labels `app=web` and `env=prod`:

    kubectl run web --image=nginx --labels app=web,env=prod --expose --port 80
    
Run a temporary Pod and make a request to `web` Service:

    $ kubectl run --rm -i -t --image=alpine test-$RANDOM -- sh
    / # wget -qO- http://web
    <!DOCTYPE html>
    <html>
    <head>
    ...
    
It works, now save the following manifest to `web-deny-all.yaml` and
deploy with `kubectl apply -f web-deny-all-yaml`:

```yaml

```
