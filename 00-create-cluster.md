## Create a cluster

Most of the Kubernetes installation methods out there do not get you a cluster
with [Network
Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
feature. You manually need to install and configure a Network Policy provider
such as Weave Net or Calico.

**[Google Kubernetes Engine (GKE)][gke]** easily lets you get a Kubernetes
cluster with Network Policies feature. You do not need to install a network
policy provider yourself, as GKE configures Calico as the networking provider
for you. (This feature is generally available as of GKE v1.10.)

To create a GKE cluster named `np` with Network Policy feature enabled, run:

    gcloud beta container clusters create np \
        --enable-network-policy \
        --zone us-central1-b

This will create a 3-node Kubernetes cluster on Kubernetes Engine and turn on
the Network Policy feature.

Once you complete this tutorial, you can delete the cluster by running:

    gcloud container clusters delete -q --zone us-central1-b np


[gke]: https://cloud.google.com/kubernetes-engine/
