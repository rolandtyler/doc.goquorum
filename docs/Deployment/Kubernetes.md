---
description: GoQuorum deployment Kubernetes
---

## Quorum-Kubernetes

[Quorum-Kubernetes](https://github.com/ConsenSys/quorum-Kubernetes) is a repository containing Kubernetes
manifests and Helm charts that can be customised and deployed on a local cluster or in the cloud.

!!! important

    We recommend starting with the [playground] directory and working through the example setups before
    moving to the [dev] directory, and finally to the [prod] directory.

The `dev` and `prod` directories are identical in terms of what gets deployed, but differ in that the `prod` folder uses best
practices to manage identity (Managed Identities in Azure and IAM in AWS), and vaults (KeyVault in Azure and Secrets
Manager in AWS), along with CSI drivers.

All setups use monitoring and we strongly recommend deploying the monitoring manifests or charts to
get an overview of the network, nodes, and volumes, and you can create alerts accordingly. In addition there is example
configuration for ingress and routes which you can customise to suit your requirements

### Cloud support

The repository currently supports AWS EKS and Azure AKS natively, and you can configure the provider in the
[values.yml](https://github.com/ConsenSys/quorum-kubernetes/blob/master/dev/helm/values/genesis-goquorum.yml) file
by setting `provider` to `local`, `aws`, or `azure`. You can also pass in extra configuration such as a KeyVault
name.

The repository also contains [Azure ARM templates] and [AWS eksctl templates] to deploy the required base infrastructure.

## Limitations

Kubernetes has a limitation when using multi-clusters in that its load balancers disallow TCP and UDP traffic on the same
port. This inhibits discovery working natively for each pod. Use the following solutions to mitigate this:

* Disallow discovery and use static nodes to allow only TCP traffic. This is not an issue for
    load balancers or exposing nodes publicly.
* If you need to use discovery, use something like [CNI] which is supported by all major cloud providers, and the
    cloud templates already have CNI implemented.

### CNI

With the traditional Kubelet networking mode, nodes get an IP from the virtual network subnet. Each node in turn uses NAT
to configure the pods so that they reach other pods on the virtual network. This limits where they can reach but also more
specifically what can reach them, for example an external VM which has to have custom routes does not scale well.

![Kubenet](../images/kubernetes/kubenet.jpg)

CNI on the other hand allows every pod to get a unique IP directly from the virtual subnet which removes the above restriction.
It therefore has a limit on the maximum number of pods that can be spun up, so you need to plan ahead to avoid IP exhaustion.

![CNI](../images/kubernetes/cni.jpg)

## Multi-cluster

You need to enable [CNI](#cni) to use multi-cluster, or to connect external nodes to an existing Kubernetes cluster. To
connect multiple clusters, they must each have different CIDR blocks to ensure no conflicts, and the first step
is to peer the VPCs or VNets together and update the route tables. From that point on you can use static nodes and pods
to communicate across the cluster.

The same setup also works to connect external nodes and business applications from other infrastructure, either in the
cloud or on premise.

![CNI-peering](../images/kubernetes/cni_peering.png)

[playground]: https://github.com/ConsenSys/quorum-kubernetes/tree/master/playground
[dev]: https://github.com/ConsenSys/quorum-kubernetes/tree/master/dev
[prod]: https://github.com/ConsenSys/quorum-kubernetes/tree/master/prod
[Azure ARM templates]: https://github.com/ConsenSys/quorum-kubernetes/tree/master/azure
[AWS eksctl templates]: https://github.com/ConsenSys/quorum-kubernetes/tree/master/aws
[CNI]: https://github.com/containernetworking/cni/blob/master/SPEC.md#network-configuration
