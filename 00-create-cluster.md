## Create a cluster

Most of the Kubernetes installation methods out there do not get
you a cluster with [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
feature. You manually need to install and configure a Network
Policy provider such as Weave Net or Calico.

**[Google Container Engine (GKE)][gke]** easily lets you get a Kubernetes cluster
with Network Policies feature. You do not need to install a network
policy provider yourself, as GKE configures Calico as the networking provider for you.
(This feature is currently in beta as of Kubernetes v1.8.)

To create a GKE cluster named `np` with networking policies enabled, run:

    gcloud beta container clusters create np \
        --enable-network-policy \
        --zone us-central1-b
        
This will create a 3-node Kubernetes 1.7.0 cluster  named `np`, with alpha features turned on
to test out rest of this tutorial. 

Once you complete this tutorial, you can delete the cluster by running:

    gcloud container clusters delete -q --zone us-central1-b np


[gke]: https://cloud.google.com/container-engine/
