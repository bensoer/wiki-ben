# Container Runtimes & Their Tools

Container runtimes are the engines that run your containers. Different container runtimes are designed for different purposes and can have varying compliance with the Open Container Initiative and the Container Runtime Interface.

The Open Container Initiative (OCI) and Container Runtime Interface (CRI) specifications are two standards for dictating container image packaging and kubernetes compatibility. OCI typically refers to the standard of formatting and packaging of container images, but in order for a container runtime to use images that follow the OCI standard, the runtime must be OCI compatible. If a runtime is to be used by Kubernetes, it must be CRI compliant.

A summary of some common container runtimes and their compliance is listed in the following table:


| Container Runtime | Open Container Initiative (OCI) Compatible | Container Runtime Interface (CRI) Compatible | Runtime Is Used In |
| ----------------- | ------------------------------------ | -------------------------------------- | ------------------------------ |
| ContainerD        | YES                                  | YES                                    | Docker, Most Cloud K8s         |
| Docker            | YES                                  | NO                                     | Docker                         |
| CRI-O             | YES                                  | YES                                    | Minikube, OpenShift, Oracle    |
| rkt (deprecated)  | YES                                  | YES                                    |                                |
| podman            | YES                                  | NO                                     | RHEL based systems, OpenShift  |


## Open Container Initiative (OCI)
OCI is a standardisation of bundling of container images. It was created to standardise images creation and execution. OCI packages are made up of two components:

| Component | Description |
| --------- | ----------- |
| imagespec | contains information on how the image should be created |
| runtimespec | contains information on how to run the image |

!!! note "TL;DR"
    * If the runtime is OCI compliant, this means it follows cross compatible standard when it builds container images. 
    * If the runtime is OCI compatible, this means it can execute OCI compliant images
    
    Pretty much all images these days are OCI compliant and thus most runtime engines, are at minimum OCI compatible.


## Container Runtime Interface (CRI)
CRI is another standardisation specifically from K8s defining how K8s will communicate with runtime systems, allowing development of alternative runtimes that can work with K8s. Kubernetes used to _only_ work with Docker as its container runtime. This was eventually abstracted and thus the Container Runtime Interface was created.

With the creation of the CRI, tooling was also made to test out and debug CRI implementations. The `crictl` is a CLI tool for interacting with CRI compatible services using the container runtime defined interface

!!! note "TL;DR"
    If something is CRI compliant, this means it is compatible to be run by Kubernetes as a runtime engine

## Containe Runtime Toolings
Each of these runtimes do not _have_ to run in kubernetes to run. Many of them can be used independently. In order to interact with them outside of kubernetes, these runtime engines come with a handful of CLI tools for working with them. CRI compatibility offers an extra bonus of also allow cross-compatibility where CLI tools can work on multiple container runtimes. 

A list of runtimes, compatible CLI tools and details about each tool is listed below:

| CLI Tool | Description | Runtimes Compatible With |
| -------- | ----------- | ------------------------ |
| `docker` CLI | For interacting with Docker | Docker |
| `ctr` | Built in debug CLI for ContainerD | ContainerD |
| `nerdctl` | Docker-like general purpose CLI for ContainerD | ContainerD |
| `crictl` | CRI Compatible CLI for debugging and testing CRI compliant container runtimes | ContainerD, CRI-D, rkt | 

## A Big Map
Below is a diagram of a bunch of the toolings all listed above and how they relate and interact with eachother


``` mermaid
graph TD
    K[docker CLI]
    A[Docker];
    B[ContainerD];
    E[CRI-O];
    F[nerdctl];
    G[ctr];
    H[crictl];
    I[rkt];
    J[podman];
    
    K --> | Interacts With | A;
    A <--> | Is Bundled With | B;
    G --> | CLI To Debug | B;
    F --> | General Purpose CLI For | B;
    H --> | CLI for CRI compatible services | E;
    H --> | CLI for CRI compatible services | I;
    H --> | CLI for CRI compatible services | B;

```
