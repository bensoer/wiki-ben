# Tool Suite
A personal list of favourite tools to use and consider when working in the kubernetes ecosystem


## Essentials
Install these on your laptop for basically every day use, development, testing, debugging, etc

| Name | Description | Links |
| --------- | ----------- | ----- |
| `kubectl` | K8 API Tool | |
| `helm` | Manifest packaging system | |
| `minikube` | Local cluster environment service | |

## CLI Addons
May be part of the essentials, but not necessary. Would not recommend if wanting to study for the CKA or CKD as you won't have these and they will make you lazy

| Name | Description | Links |
| ---- | ----------- | ----- |
| `kubectx` | Single command context switcher | [https://github.com/ahmetb/kubectx](https://github.com/ahmetb/kubectx) |
| `kubens` | Single command namespace switcher (bundled with kubectx) | [https://github.com/ahmetb/kubectx](https://github.com/ahmetb/kubectx) |

## Cluster Management
Tools for making cluster wide changes

| Name | Description | Links |
| ---- | ----------- | ----- |
| `kube-ops-view` | Terminal view of cluster activity | |
| `kubespray` | Setup Production ready clusters | [https://github.com/kubernetes-sigs/kubespray/tree/master](https://github.com/kubernetes-sigs/kubespray/tree/master) |
| `kubeadm` | Cluster Administration. CRUD nodes, etc. Does not create clusters | | 


## Security
Security scanners and tools

| Name | Description | Links |
| ---- | ----------- | ----- |
| 

## CI/CD Tools


## Local Development K8 Tools
A number of tools for building clusters on your local machine. These are alternatives to `minikube`

| Name | Description | Links |
| ---- | ----------- | ----- |
| `kind` | Builds k8 clusters all within docker. More designed for testing k8 then actually developing ontop of | |
| `micro-k8s` | Made specifically to work with Canonical / Snap / Ubuntu. Working elsewhere is not available. Working with Snap, except for simple use cases between user <-> service, sucks in general | 
| `k3s` | A mostly production ready k8 but slimmed down to run on fewer resources. TrueNAS uses this under the hood for self hosting service management. Have been told though working with it can create more verbose typing as all the tools are wrapped within the `k3s` CLI binary (ex: to use `kubectl` you can't configure and point `kubectl` to your `k3s` cluster, you _have_ to go `k3s kubectl`) | |