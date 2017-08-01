# Kubernetes Network Policies &mdash; Definitive User Guide

This repository contains various use cases of Kubernetes
[Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
and sample YAML files to leverage in your setup. If you ever wondered
how to drop/restrict traffic to applications running on Kubernetes, read on.

I recommend creating an empty cluster on [Google Container Engine](https://cloud.google.com/container-engine)
to try out the following examples. Applying Network Policies on your existing cluster can disrupt
the networking. Also, not all Kubernetes providers support Network Policies.

### Before you begin
- [Create a cluster](00-create-cluster.md)

### Basics

- [DENY all traffic to an application](01-deny-all-traffic-to-an-application.md)
- [LIMIT traffic to an application](02-limit-traffic-to-an-application.md)

### Namespaces

- [DENY all non-whitelisted traffic in the current namespace](03-deny-all-non-whitelisted-traffic-in-the-namespace.md)
- [DENY all traffic from other namespaces + LIMIT access to the current namespace](04-deny-traffic-from-other-namespaces.md)
- [ALLOW all traffic from all namespaces](05-allow-traffic-from-all-namespaces.md)
- [ALLOW all traffic from a namespace](06-allow-traffic-from-a-namespace.md)
- [ALLOW traffic from some pods in another namespace](07-allow-traffic-from-some-pods-in-another-namespace.md)
- [LIMIT traffic to an application the current namespace](08-limit-traffic-to-an-application-to-current-namespace.md)

### Advanced

- [ALLOW traffic only to certain port numbers of an application](08-allow-traffic-only-to-a-port-number.md)
- [ALLOW traffic from apps using multiple selectors](09-allowing-traffic-with-multiple-selectors.md)

----- 

##### Author

Ahmet Alp Balkan ([@ahmetb](https://twitter.com/ahmetb)).

Copyright 2017, Google Inc. Licensed under Apache 2.0. Disclaimer: This is not an official Google product.
