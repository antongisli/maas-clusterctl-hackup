# maas-clusterctl-hackup

Welcome to these quick notes on how to get the maas capi provider working with the latest version. Note, the needed yaml files and source code are in this repository already, so you can clone it and skip any of the steps below regarding editing the yaml files, downloading the releases etc.  Just clone and create your clusterctl configuration file to point at the right place and you should be able to get started straight away. 

With that said, this README lists all the steps for posterity.

1. create a local directory e.g. ~/maas-capi, download the [4.1.0 source code](https://github.com/spectrocloud/cluster-api-provider-maas/releases/tag/v4.1.0-spectro) into it and extract it. create a src directory, move the extracted contents into it.
1. download the 0.4.0 infrastructure-components.yaml file into the ~maas-capi/ folder. Edit it and fix the cert-manager api versions listed in the certificates. 
    - e.g. `apiVersion: cert-manager.io/v1alpha2` → `apiVersion: cert-manager.io/v1` (at time of writing there were two certificates like this)
    - in the deployment capmaas-controller-manager you need to change the image
        `image: gcr.io/spectro-images-public/release/cluster-api-provider-maas/cluster-api-provider-maas-controller:v0.4.0` → `image: gcr.io/spectro-images-public/release/cluster-api-maas/cluster-api-provider-maas-controller:v0.2.0-spectro-4.1.0`
1. download the 0.4.0 ()[]metadata.yaml file and add an entry for 4.1.0:

        - major: 4
            minor: 1
            contract: v1beta1


1. When you’re done, your folder should look like the below (you probably dont need the cluster-template.yaml file …:man_shrugging: ):

        .
        └── infrastructure-maas
            └── v4.1.0
                ├── cluster-template.yaml
                ├── infrastructure-components.yaml
                ├── metadata.yaml
                ├── src
                └── v4.1.0-spectro.zip

1. Set the MAAS endpoint and API key env vars

        export MAAS_ENDPOINT 
        export MAAS_API_KEY="<SNIP>"

1. Install clusterctl, make sure you use a recent version.

1. create `~/.cluster-api/clusterctl.yaml`, it should look something like the below, although mtake care to specify the path that you actually used:
providers:

        # add a custom provider
            - name: "maas"
            url: "/home/ubuntu/maas-capi/infrastructure-maas/v4.1.0/infrastructure-components.yaml"
            type: "InfrastructureProvider"

    NOTE: clusterctl searches the path based off the “name” of your custom provider. since it is an infrastructure provider, it will use <infrastructure>-<name> e.g. “infrastructure-maas”. You also must obey the contract that clusterctl requires for providers, which basically means the folder structure and file names must be correct. It must be able to see the semver named directory and the infrastructure-components.yaml file.

1. now you can init the cluster. I used kind on mac (M1). I had architecture errors using multipass/kubeadm (no idea why). It worked with kind.

    `clusterctl init --infrastructure maas --v=5`

1. You should see all the CAPI deployments and pods come up without errors:

        NAMESPACE↑                           NAME                                                              PF   READY     RESTARTS STATUS     IP              NODE                   AGE      │
        │ capi-kubeadm-bootstrap-system        capi-kubeadm-bootstrap-controller-manager-6cff4fd9fd-cz8tj        ●    1/1              0 Running    10.244.0.11     kind-control-plane     26m      │
        │ capi-kubeadm-control-plane-system    capi-kubeadm-control-plane-controller-manager-5b97899497-h79cv    ●    1/1              0 Running    10.244.0.9      kind-control-plane     26m      │
        │ capi-system                          capi-controller-manager-8b6cd9fb5-qvsr6                           ●    1/1              0 Running    10.244.0.8      kind-control-plane     27m      │
        │ capmaas-system                       capmaas-controller-manager-74559bcdd8-5pc7v                       ●    2/2              0 Running    10.244.0.10     kind-control-plane     26m      │
        │ cert-manager                         cert-manager-6856c9896b-c2cnh                                     ●    1/1              0 Running    10.244.0.5      kind-control-plane     27m      │
        │ cert-manager                         cert-manager-cainjector-fc54fdc88-5clvc                           ●    1/1              0 Running    10.244.0.7      kind-control-plane     27m      │
        │ cert-manager                         cert-manager-webhook-68496779c4-gqssk                             ●    1/1              0 Running    10.244.0.6      kind-control-plane     27m      │

# References
- https://release-1-1.cluster-api.sigs.k8s.io/clusterctl/configuration#provider-repositories
- https://www.spectrocloud.com/blog/how-to-provision-bare-metal-k8s-clusters-with-cluster-api-and-canonical-maas
- https://release-1-1.cluster-api.sigs.k8s.io/clusterctl/configuration#provider-repositories
- https://cluster-api.sigs.k8s.io/clusterctl/provider-contract#creating-a-local-provider-repository
- https://github.com/spectrocloud/cluster-api-provider-maas/releases/download/v0.4.0/infrastructure-components.yaml
- https://github.com/spectrocloud/cluster-api-provider-maas/releases/download/v0.4.0/metadata.yaml
- https://cluster-api.sigs.k8s.io/user/quick-start.html

