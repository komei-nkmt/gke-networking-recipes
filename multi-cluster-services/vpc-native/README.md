# Multi-cluster Services with Shared VPC and Peered VPC clusters


[Multi-cluster Services](https://cloud.google.com/kubernetes-engine/docs/concepts/multi-cluster-services) work only for clusters that are [VPC-native instead of routes-based](https://cloud.google.com/kubernetes-engine/docs/how-to/alias-ips). VPC-native clusters use Google Cloud's VPC networks to allocate IPs and route traffic between pods. The VPC network used by a given cluster can stand on its own, be networked to anoher VPC ("Peered VPC"), or can be a VPC hosted in another project ("Shared VPC"). In this recipe, we will show you how to set up MCS in these other configurations of VPC-native clusters.

### Use-cases

- Using MCS in an organization that shares VPC networks from a parent project that can have different access permissions than the projec the clusters are in ("Shared VPC")
- Using MCS between clusters that are in different VPC networks ("Peered VPC")


### Relevant documentation

- [VPC-native clusters](https://cloud.google.com/kubernetes-engine/docs/concepts/alias-ips)
- [VPC Network Peering overview](https://cloud.google.com/vpc/docs/vpc-peering)
- [Using VPC Network Peering](https://cloud.google.com/vpc/docs/using-vpc-peering)
- [Shared VPC overview](https://cloud.google.com/vpc/docs/shared-vpc)
- [Multi-cluster Services Concepts](https://cloud.google.com/kubernetes-engine/docs/concepts/multi-cluster-services)
- [Setting Up Multi-cluster Services](https://cloud.google.com/kubernetes-engine/docs/how-to/multi-cluster-services)
- [OSS Multi-cluster Services API](https://github.com/kubernetes/enhancements/tree/master/keps/sig-multicluster/1645-multi-cluster-services-api)

#### Versions

- GKE clusters on GCP
- 1.17 and later versions of GKE supported
- Tested and validated wih X.XX.XX-gke.XXXX on Apr XXth 2021

### Networking Manifests

### Try it out: Shared VPC

> This recipe uses the same application manifests as the "Multi-cluster Services basic" recipe

1. Download this repo and navigate to the folder with the manifests for the basic Multi-cluster Services recipe:


    ```sh
    $ git clone https://github.com/GoogleCloudPlatform/gke-networking-recipes.git
    Cloning into 'gke-networking-recipes'...

    $ cd gke-networking-recipes/multi-cluster-services/multi-cluster-services-basic
    ```

2. Deploy the two clusters `gke-1` and `gke-2`, and register them to your Hub, as specified in [cluster setup](../../cluster-setup.md) for this recipe. **It is important to use the cluster setup directions for Peered VPC.**

3. Now log into `gke-1` and deploy the `app.yaml` manifest. (This assumes you set up your kubectl contexts as described in [cluster setup](../../cluster-setup.md).)

    ```bash
    $ kubectl --context=gke-1 apply -f app.yaml
    namespace/multi-cluster-demo unchanged
    deployment.apps/whereami created
    service/whereami created

    # Shows that pod is running and happy
    $ kubectl --context=gke-1 get deploy -n multi-cluster-demo
    NAME              READY   UP-TO-DATE   AVAILABLE   AGE
    whereami          1/1     1            1           44m
    ```


5. Now create ServiceExport in `export.yaml` to export service to other cluster.

    ```bash
    $ kubectl --context=gke-1 apply -f export.yaml
    serviceexport.net.gke.io/whereami created
    ```

6. It can take up to 5 minutes to propagate endpoints when initially exporting Service from a cluster. Create the same Namespace in `gke-2` to indicate you want to import service, and inspect ServiceImport and Endpoints.

    ```bash
    $ kubectl --context=gke-2 create ns multi-cluster-demo
    namespace/multi-cluster-demo created
    
    # Shows that service is imported and ClusterSetIP is assigned.
    $ kubectl --context=gke-2 get serviceimport -n multi-cluster-demo
    NAME       TYPE           IP              AGE
    whereami   ClusterSetIP   [10.124.4.24]   4m50s
    
    # Shows that endpoints are propagated.
    $ kubectl --context=gke-2 get endpoints -n multi-cluster-demo
    NAME                 ENDPOINTS        AGE
    gke-mcs-7pqvt62non   10.16.4.7:8080   6m1s
    ```

7. Let's inspect our Peered VPC settings more closely so we can see that these multi cluster Services are being shared over a Peered network.