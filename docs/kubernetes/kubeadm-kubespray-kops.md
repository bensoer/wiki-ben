# KOPS vs Kubeadm vs Kubespray

All three are are setting up Kubernetes clusters, but each is intended for different use cases

| Service | Usage | Links |
| ------- | ----- | ----- |
| `kops` | For setting up K8 clusters in cloud environments. Integrating with AWS, GCP, etc | |
| `kubeadm` | Meant to be a minimum production ready installation of K8s. It contains no tools for handling infrastructure it is running on | |
| `kubespray` | A production ready deployment meant to be generic and work on all kinds of environments from bare-metal to cloud. Includes handling and creation of infrastructure for its deployment. Uses Ansible to automate provisioning of infrastructure and components | |