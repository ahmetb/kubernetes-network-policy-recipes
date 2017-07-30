# Kubernetes Network Policies &mdash; Definitive User Guide

This repository contains various use cases of Kubernetes
[Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
and sample YAML files to leverage in your setup. If you ever wondered
how to drop/restrict traffic to applications running on Kubernetes, read on.

I recommend creating an empty cluster on [Google Container Engine](https://cloud.google.com/container-engine)
to try out the following examples. Applying Network Policies on your existing cluster can disrupt
the networking. Also, not all Kubernetes providers support Network Policies.

### Before you begin
- Create a cluster

### Basics

- DENY all traffic to an application
- LIMIT traffic to an application

### Namespaces

- DENY all non-whitelisted traffic in the current namespace
- DENY all traffic from other namespaces
- ALLOW all traffic from other namespaces
- ALLOW traffic from applications in other namespaces
- ALLOW all traffic to an application in a namespace denying all non-whitelisted traffic
