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


## Validation / Linting
Tools for validating / verifying your Kubernetes manifests, helm charts, etc

| Name | Description | Links |
| ---- | ----------- | ----- |
| `kubeval` | Validate your kubernetes config files. **Note:** No longer maintained - `kubeconform` seems to be the replacement  | <ul><li>[https://github.com/instrumenta/kubeval](https://github.com/instrumenta/kubeval)</li><li>[https://github.com/yannh/kubeconform](https://github.com/yannh/kubeconform)</li></ul> |
| `tflint` | Linting and validation for your Terraform. Includes plugins for cloud providers so it can validate things such as instance types and cloud specific configurations | [https://github.com/terraform-linters/tflint](https://github.com/terraform-linters/tflint) |

## Security
Security scanners and tools

| Name | Description | Links |
| ---- | ----------- | ----- |
| `trivy` | Works as a CLI tool but also can be installed as an operator in the cluster. Has a handful of scanning tools | |
| `kube-bench` | CLI scanning tool to validate your cluster against CIS standard. Is integrated into `trivy` as well. | [https://github.com/aquasecurity/kube-bench](https://github.com/aquasecurity/kube-bench) |


## CI/CD Tools

| Name | Description | Links |
| ---- | ----------- | ----- |
| ArgoCD | Suite of CD tools including - ArgoCD, ArgoRollouts, ArgoWorkflows and ArgoEvents | |
| Flux | Is a direct competitor with ArgoCD, however its not as actively maintained and the user base is more in favor of ArgoCD | |
| Keel | A very simplified ArgoCD-like service. Uses annotations on `Deployments` to configure and track registry container changes and apply updates | |
| Jenkins | The defacto CI tool. Java, large and clunky, but does do everything. Works well when linked with pipelines via Jenkinsfiles, and use ECS for spawning worker nodes | [https://keel.sh/](https://keel.sh/) |
| Drone |  Similar to Concourse, but appears even simpler. Has quite a bit of integration with Digital Ocean, Gitea, Github, Bitbucket etc. Looks very small and written all in go | [https://www.drone.io/](https://www.drone.io/) |
| Concourse | A self-hosted like CircleCI or Github Actions tool. Simple YAML based with container environments for building and deploying services. Doesn't have much best practices or abilities to handle running at scale (common libs) | |



## Local Development K8 Tools
A number of tools for building clusters on your local machine. These are alternatives to `minikube`

| Name | Description | Links |
| ---- | ----------- | ----- |
| `kind` | Builds k8 clusters all within docker. More designed for testing k8 then actually developing ontop of | |
| `micro-k8s` | Made specifically to work with Canonical / Snap / Ubuntu. Working elsewhere is not available. Working with Snap, except for simple use cases between user <-> service, sucks in general | 
| `k3s` | A mostly production ready k8 but slimmed down to run on fewer resources. TrueNAS uses this under the hood for self hosting service management. Have been told though working with it can create more verbose typing as all the tools are wrapped within the `k3s` CLI binary (ex: to use `kubectl` you can't configure and point `kubectl` to your `k3s` cluster, you _have_ to go `k3s kubectl`) | |