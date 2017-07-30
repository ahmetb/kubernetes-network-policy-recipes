# Kubernetes Network Policies &mdash; Definitive User Guide

This repository contains various use cases of Kubernetes
[Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
and sample YAML files to leverage in your setup.

## Tasks

I recommend creating an empty cluster on [Google Container Engine](https://cloud.google.com/container-engine)
to try out the following examples. Applying Network Policies on your existing cluster can disrupt
the networking.

- Create a cluster
- DENY all traffic to an application
- LIMIT traffic to an application
- DENY all non-whitelisted traffic in a namespace
- ALLOW traffic from other applications
- DENY traffic from other namespaces
- ALLOW traffic from other namespaces
