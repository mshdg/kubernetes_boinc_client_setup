# BOINC Client Setup on Kubernetes #

![GitHub release (latest by date)](https://img.shields.io/github/v/release/jaysgrant/kubernetes_boinc_client_setup)

This project provides baseline manifests to setup the BOINC computing client on Kubernetes. This evolved from my experimetntation of wanting to run it at home, and contribute to the Rosetta@Home project.

The yaml manifests provided are a starting point, which I have been running on my MicroK8S development machine in my home lab.

There are two installation options here, one based on the standard BOINC docker client, without GPU support, and a second that includes Intel legacy GPU support. Legacy in this case is with a Haswell based integrated GPU.

Additional GPU support options are available, and documented on the [BOINC project page](https://github.com/BOINC/boinc-client-docker).

For Intel GPU support, I am utilizing the [Intel Kubernetes GPU Plugin](https://github.com/intel/intel-device-plugins-for-kubernetes/).

My Kubernetes instance uses [MetalLB](https://github.com/metallb/metallb) for loadbalancing. As such, the service entries in each manifest contain an annotation for this. The service can be edited as needed to use NodePort, or ClusterIP setups as well.

## Getting Started ##

### Requirements ###

The requirements for this project are simple. All you need is a functional Kubernetes setup.

### Non GPU Setup ###

The non GPU setup is the easiest to run. Open **boinc_k8s/boinc_client_non_gpu.yaml**, and edit service entry as needed. 
1. If you are using the MetalLB L2 configuration, add the name of your pool.
    * If you prefer Nodeport, or ClusterIP setups, edit the service as appropriate.
    * Example of the service in its defaults is here:
    ```yaml
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: rosettaathome
      namespace: boinc
      annotations:
        metallb.universe.tf/address-pool: #Replace this comment with your MetalLB Pool Name
    spec:
      selector:
        app: rosettaathome
      ports:
        - port: 80
          name: http
          protocol: TCP
        - port: 443
          name: https
          protocol: TCP
        - port: 31416
          name: manager
          protocol: TCP
      type: LoadBalancer
    ```
1. Once the needed edits are made, from your terminal run:
    ```sh
    kubectl apply -f boinc_k8s/boinc_client_non_gpu.yaml
    ```
1. Thsi will create a the service, PVC, and deployment with a single running pod. The pod can be connected to via BOINC manager for further configuration.

### Intel GPU Setup ###

The setup of the BOINC client to utilize an integrated Intel GPU is very similar to the non GPU steps. The difference being that in order for me to utilize the integrated GPU in my Haswell based CPU, I had to install the Intel GPU Kubernetes Plugin first.

The plugin runs as a daemonset on each node of your cluster. I have provided a very basic manifest to install the plugin, which functions with the integrated GPU in my Haswell based i3-4130.

1. Start by ensuring the i915 driver is installed on your node(s).
1. Install the GPU plugin daemonset by running
    ```sh
    kubectl apply -f /intel_gpu_plugin/intel_gpu_plugin.yaml
    ```
1. Once the plugin is installed, and running, you can verify the GPU is available via the following command.
    ```sh
    kubectl get nodes -o=jsonpath="{range .items[*]}{.metadata.name}{'\n'}{' i915: '}{.status.allocatable.gpu\.intel\.com/i915}{'\n'}"
    ```
1. You should see similar output to the following, showing the GPU available for use.
    ```sh
    dc-kube-01
    i915: 1
    ```
1. Once the GPU plugin installation is completed, open /boinc_k8s/boinc_client_intel_gpu.yaml, and edit the service entry as noted in the Non GPU Setup portion of this document.
1. After your edits are complete, apply the GPU manifest.
    ```sh
    kubectl apply -f boinc_k8s/boinc_client_intel_gpu.yaml
    ```
1. Verify all pods are running, and then you can use BOINC manager to configure the project.