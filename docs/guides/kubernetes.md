Kubernetes is a powerfull platform for deploying Docker and container based applications

==Setup A Kubernetes Cluster on Hyper-V==

=== Configure Virtual Machines===

==== Hyper-V Configuration ====
From the Hyper-V Manager you will be creating 3 new Virtual Machines. 1 Virtual Machine will be the master machine, and 2 others will be minion/node machines. This setup is a basic minimum setup for
configuring a kubernetes cluster

# From the UI select New > Virtual Machine
# Click Next if prompted the "Before You Begin" section
# Set the virtual machine name. Note that each virtual machine MUST have a unique name. A recommended naming scheme is kmaster1 for the master node and kminion1 and kminion2 for the minions. If you choose to expand your cluster later, you can simply add each vm in their appropriate roles and increment their number value
# Select "Generation 2"
# Assign a minimum of 2 GB of RAM. Recommended is 4GB. Anything less than 2GB your kubernetes cluster will struggle just to run on its own. Also check the "Dynamic Memory" option as this will release memory to the system if the virtual machine is not requiring it. This is especially useful if your hosting system has limited resources
# Configure your networking by selecting the a public switch from the drop-down. Your virtual machines will need an internet connection in order to update and install aswell as Kubernetes in order to contact all of its nodes or use features such as Helm or Kubeapps. Using a public switch is the easiest way to avoid connectivity issues
# Create a virtual hard disk. The default size is sufficient, 200GB is ideal
# Install the operating system from an .iso. For this tutorial we use Debian 9.6. You can download a Debian iso from here: https://www.debian.org/distrib/netinst. These are net install images which will require and internet connection to complete the operating system installation
# Select Finish, and then select the VM from the list and open Settings
# Select "Security" from the Hardware section and uncheck "Enable Secure Boot". This feature does not work well with Linux on Hyper-V
# Select "Processor" from the Hardware section and set the "Number of virtual processors" at minimum to 2. The more CPUs you can provide the better. It is also not an issue to overlap or assign all CPUs on your host provider to each of the kubernetes virtual machines.
# Select "Advanced Features" under the "Network Adapter" section. You may need to expand "Network Adapter" to see the "Advanced Features". Selecting "Advanced Features" set the "MAC Address" to "Static". Kubernetes uses the MAC address and IP to uniquely identify each of its nodes. Hyper-V though will change the MAC address after reboot. You can use https://www.miniwebtool.com/mac-address-generator/ to generate MAC addresses for you to fill in
# Select "OK" at the bottom to close and submit your settings

You have now configured 1 of the 3 Virtual Machines for your kubernetes cluster. Repeast these steps for the other nodes in the cluster. Remember to give unique name and MAC addresses to each virtual machine created

==== Router Configuration ====
Kubernetes requires each node to have a unique hostname, MAC and IP address in order to uniquely identify and contact each member of the cluster. By assigning unique names in Hyper-V and static MAC addresses we have already resolves 2 of these 3 requirements. The last requirement is to assign static IP addresses to each of the virtual machines. This is done by configuring static routing or DHCP reservation in your router where Hyper-V shares the network with. We configured each virtual machine to use the public switch - which means each virtual machine will pretend to be a device on the Hyper-V host machines' network. This means the router will be in charge of assigning IP addresses.

Access your network router and configure either static routing which will enforce assignment of static IPs to every device in the network or DHCP reservation. DHCP reservation is forcing certain IPs to be assigned to certain machines based on their MAC address. This setup is likely how your own router is configured. Refer to your routers manual or help on how to configure these settings. Having assigned the static MAC addresses and machine names in Hyper-V you can setup the static routing or DHCP reservations before even starting the VMs. This will ensure everything is configured properly before any of the installation process begins.

==== Debian 9 Configuration ====
On bootup of the Virtual Machines, the Debian 9 installer will launch. Start the graphical install. Configure the system to have a minimal install. Configure HDD partitioning to put everything on the same partition. Make sure the installation has no GUI as this will consume resources that will not be taken advantage of. This is especially important if your virtual machine is running with limited resources

==== Check IP and MAC ====
Once you have setup static routing or DHCP reservation and completed the Debian 9 install, check that your IP and MAC have been assigned correctly. Check with the following command
 <nowiki>
sudo ip addr</nowiki>
Check the listed interfaces that "eth0" has been assigned the IP configured in your router and the MAC configured in Hyper-V


=== Configure Software On Virtual Machines ===
You have now setup 3 Virtual Machines in Hyper-V. One is labelled as the master and two others labelled as mionion/nodes. Next we need to apply the bare bones software packages
and system configurations needed for these virtual machines to work in a cluster. Apply the following steps to all 3 Virtual Machines:

==== Update/Upgrade And Install SSH ====
Installing SSH on each node is ideal for remote management of the nodes, especially since you will be flipping through them alot during the remainder of this tutorial. Opening more
ports, especially on a production build, is a risk but comes with added ease of management. SSH can also be easily disabled and removed after completion of the install

Before any install with new machines, its also good to make sure your nodes are up to date. Execute the following commands to check and install updates for your Debian 9 nodes
 <nowiki>
sudo apt-get update
sudo apt-get upgrade -y</nowiki>

After updating, install ssh server
 <nowiki>
sudo apt-get install openssh-server</nowiki>
In the latest version of Debian (Debian 9.6 as of this writing), ssh is already included. Executing this command may prompt that it is already installed

Next, open <code>sshd_config</code> in nano and add or edit the following setting:
 <nowiki>
PermitRootLogin yes</nowiki>
This setting will allow the root user to login over ssh to the system. For best security practices, it is recommended you disable this setting later. Setting this up is
purely for convenience. Restart SSH to apply your setting changes
 <nowiki>
sudo systemctl sshd restart</nowiki>

You can now access your nodes over SSH!. If you are on linux you can simply connect using ssh in terminal, or for Windows install Putty. This is primarily a convenience feature for all future steps of the installation process

==== Install Docker ====
Install Docker on Debian 9 with the following commands:
 <nowiki>
sudo apt update
sudo apt install apt-transport-https ca-certificates curl gnupg2 software-properties-common
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
sudo apt update
apt-cache policy docker-ce
sudo apt install docker-ce</nowiki>

==== Install Kubernetes ====
Install Kubernetes for Debian 9 with the following commands:
 <nowiki>
apt-get update && apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl</nowiki>

==== Disable Swap ====
Before starting any nodes, make sure to disable swap. Execute the following command to find the location of your swap file first:
 <nowiki>
cat /proc/swaps</nowiki>
Additionally execute:
 <nowiki>
blkid</nowiki>
To find the id of the swap file block

Then disable swap on the system by entering:
 <nowiki>
sudo swapoff -a</nowiki>
This will temporarily disable it and anything using it. If you enter the command <code>cat /proc/swaps</code> again there should
be nothing listed

Next, open the file /etc/fstab and comment out any lines referring to the swap path mentioned. Also comment out any entries that
match the same block id as the swap. It is possible there will be none.

After commenting it out, delete the swap partition using parted. Execute the following commands
 <nowiki>
sudo apt-get install parted
parted</nowiki>

> PLEASE READ NEXT PART CAREFULY AS IT MAY BRICK THE SYSTEM <

This starts up parted. Type <code>print</code> and then enter. This will list all the partitions on the system, look for the one with the same
name listed when checking <code>cat /proc/swaps</code>. Note the number listed in the one column for that partition. Type <code>rm <number></code> and then
enter. Confirm the action as you may be prompted by parted that the kernel will not know of the change. Ignore any warnings. Type <code>print</code> again to confirm
successful removal of the parititon. Type <code>quit</code> to exit parted

At this point, reboot your server. You will notice the server takes a notable longer amount of time to boot. It is possible it will also fail to boot if
you rebooted via a restart. A full shutdown and then startup may be required. Again boot up will take longer, this is because swap has been disabled and
you have essentially disabled system local caching. If booting fails, you have likely deleted the wrong partition or commented out the wrong data
in <code>/etc/fstab</code>

==== Configure IPTables ====
If reboot is successful, next edit <code>/etc/sysctl.conf</code> and add to the end of the file the following:
 <nowiki>
...
#Kubernetes
net.bridge.bridge-nf-call-iptables = 1
...</nowiki>
This allows bridged networks to have their traffic passed through iptables

==== Start Docker Service ====
Lastly, start the docker service:
 <nowiki>
sudo systemctl enable docker.service</nowiki>

Once this is all complete, reboot your system again

==== Create Checkpoints in Hyper-V ====

Upon this all being successfully applied to the master and all minion nodes. Its ideal at this point to create checkpoints in hyper-v. The boxes are ready now to start running as a kubernetes cluster. This is a good point to revert back to if there are any issues during setup. Creating a checkpoint can be done within hyper-v

=== Configure Master and Minion Nodes ===

==== Master Node ====

===== Initialization =====
Execute the following command on the master node to startup kubernetes:
 <nowiki>
sudo kubeadm init</nowiki>
Upon that being successful you will be prompted with "Your Kubernetes master has initialized successfully". Below this is some important instructions.

The first section contains the following commands:
 <nowiki>
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config</nowiki>
Run these commands on the master node. This is to properly setup the config information generated by kubeadm. Do this now

The second section contains a kubeadm command on how to have your nodes join the master node. It looks something like this
 <nowiki>
kubeadm join 192.168.1.200:6443 --token **** --discovery-token-ca-cert-hash **********</nowiki>

===== Install Weave Net =====
Before running this second section, kubeadm has not installed all the required components. There is still missing a networking addon. There are multiple
options available but for this tutorial we will use Weave Net as it requires the least amount of configuration. You can view all other networking addons here: https://kubernetes.io/docs/tasks/administer-cluster/network-policy-provider/weave-network-policy/. 

Execute the following command on the master node to install Weave Net:
 <nowiki>
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"</nowiki>

==== Minion Node ====
===== Join Master Node =====
Once Weave Net has successfully installed you can then run the kubeadm join command on all of the nodes. The nodes will then join the master node. Again, the join command will have been supplied in the terminal output when initializing the master node. It will look something like this:
 <nowiki>
kubeadm join 192.168.1.200:6443 --token **** --discovery-token-ca-cert-hash **********</nowiki>

==== Validate And Check Connection Health/Status ====
Once all nodes have successfully joined. Execute the following command on the master node to check their health status:
 <nowiki>
kubectl cluster-info
kubectl get nodes</nowiki>
cluster-info will show general endpoint information of the cluster. get nodes will then list the connection status of all nodes. If the nodes are not
all listed as status "Ready" give it a couple minutes and try again. Otherwise enter <code>kubectl describe nodes</code> to get a more verbose log
output

Upon all of the nodes displaying the "Ready" status, your Kubernetes cluster is now built. From the master node using kubectl you can now execute commands
to interact with your cluster.

==Configure Local Kubectl To Access Outside Of Cluster==
With the above setup complete, you are able to interact and use your cluster from within the master node. This may not be desirable, especially if you do not
want to SSH into the machine, or keep SSH enabled on the cluster virtual machines at all. To resolve this issue, you can install and configure a kubectl
on your local machine to access your cluster. There are better ways to setup this configuration using RBAC, but this a quick and easy method for home and
private solutions.

===Install Kubectl===
Install Kubectl on your local machine. There are many ways to do this depending on your system. See https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl for details. For debain 9 this can be simply done in terminal by executing the following commands:
 <nowiki>
sudo apt-get update && sudo apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl </nowiki>

===Configure Kubectl===
This solution simply copied the configuration setup in your master node to your local machine.

On your local machine create the kubectl config directory:
 <nowiki>
sudo mkdir $HOME/.kube</nowiki>

Then copy over the config file located in the same directory as created above on the master node. You are essentially copying from the file on the master node located at <code>$HOME/.kube/config</code> to your locale machine located also at <code>$HOME/.kube/config</code>

Next, set permissions for the config file so that your local kubectl client can access the config
 <nowiki>
sudo chown $(id -u):$(id -g) $HOME/.kube/config</nowiki>

Your local kubectl is now configured to connect with your kubernetes cluster. You can test your connection works by calling
 <nowiki>
kubectl cluster-info
kubectl get nodes</nowiki>
Giving you cluster info and node status'

== Install Kubernetes Dashboard ==
The Kubernetes Dashboard is a useful minimal dashboard that lets you browse the current state of your Kubernetes cluster. It is extremely useful for debugging and early configuration.

=== Add RBAC Roles For Dashboard ===
Before installing dashboard, you need to have configured the appropriate Roles and User permissions for the Dashboard to operate within. Create the following YAML files to configure RBAC:

dashboard-adminuser.yaml
<syntaxhighlight lang="YAML" line="line">
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
</syntaxhighlight>

dashboard-rbac.yaml
<syntaxhighlight lang="YAML" line="line">
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
</syntaxhighlight>

Apply the above configuration templates using kubectl with the following commands:
 <nowiki>
kubectl apply -f ./dashboard-adminuser.yaml
kubectl apply -f ./dashboard-rbac.yaml</nowiki>

=== Install Kubernetes Dashboard ===
Install Kubernetes Dashboard with kubectl:
 <nowiki>
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml</nowiki>

=== Access Kubernetes Dashboard ===
The Kubernetes Dashboard is installed as non public facing, and thus requires the use of the kubectl proxy in order to access it. This requires kubectl to be configured and installed on your local system to and accessing your cluster. Access the dashboard by running the following commands in your local kubectl:
 <nowiki>
kubectl proxy</nowiki>
Note the terminal will hang after executing the command. With the command hanging, all applications from your kubernetes cluster are mapped to localhost. If you terminate the proxy, the mapping will be removed. With the call hanging, you can then access the dashboard at the following url on localhost:
 <nowiki>
http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/</nowiki>

When loading the page you will be prompted to enter a token. This token has been configured with the installation and the RBAC configuration. You can get the value of this token by calling the following command
 <nowiki>
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')</nowiki>

Copy the secret key into the field and you will be forwarded to the dashboard page. If the proxy is closed or token expiry occurs, you will need to reenter this secret key.

You have now configured Kubernetes Dashboard in your kubernetes cluster

== Install Helm==
Helm is a kubernetes package manager that allows easy install of popular kubernetes deployable applications. Helm is made of 2 components, a client - which is installed on your local machine, and a server - which is installed inside the kubernetes cluster. Using the kubectl client configured on your local machine, Helm can automatically detect and install itself into the cluster and on your local machine on its own.

Note as a prerequisite to installing Helm, you must have installed kubectl on your local system, allowing you to remotely access and control your kubernetes cluster from your local computer

=== Add RBAC Role For Helm ===
Helm can be installed and configured without RBAC but it is a preferred best practice for kubernetes installations and securing of helm specifically.

Create the following yaml file to create the user account adn RBAC role

rbac-config.yaml
<syntaxhighlight lang="YAML" line="line">
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
</syntaxhighlight>

Apply the above configuration using kubectl with the following command:
 <nowiki>
kubectl apply -f ./rbac-config.yaml</nowiki>

=== Install Helm ===
On your local machine run the following commands:

 <nowiki>
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get > get_helm.sh
$ chmod 700 get_helm.sh
$ ./get_helm.sh</nowiki>

This will download and install the helm client on your machine. To install the server portion of helm into your kubernetes cluster. Execute the following on your local machine aswell
 <nowiki>
helm init --service-account tiller</nowiki>
Note 'tiller' is the name of the user account we created for RBAC earlier. Note that the install will still succeed if you do not include the <code>--service-account</code> parameter, but the installation will be less secure.

After the command completes successfully helm will have been installed on your cluster. Helm works by using its local installation and the local kubectl configuration to find the location of your cluster and to map the corresponding piece. Uniquely, with this configuration, you can now also easily install and configure helm in your kubernetes cluster by calling helm on your local machine, just like as you can do with kubectl.

== Install Kubeapps ==
Source: https://github.com/kubeapps/kubeapps/blob/master/docs/user/getting-started.md

Kubeapps is a UI page for browsing, installing and upgrading helm packages. It essentially works as a UI Helm package manager. Note that because of this, installing Helm is a prerequisite to installing Kubeapps

=== Install Kubeapps ===
With Helm already installed, you can install kubeapps by running the following commands with helm:
 <nowiki>
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install --name kubeapps --namespace kubeapps bitnami/kubeapps</nowiki>

Note that this installation will take awhile, as several kubernetes pods have to be deployed, including a mongodatabase. To monitor the startup of kubeapps run the following
 <nowiki>
kubectl get pods -w --namespace kubeapps</nowiki>

This will give you a live status of the kubeapps nodes as they are spawned and started up. If you see errors spawning, it is recommended to load the Kubernetes Dashboard to view startup errors. The kubernetes dashboard will also allow you to view pod status' and logs of each pod.

An error that can occur is the mongodb pod will fail to start. The error is related to IPv6 as kubeapps assumes your kubernetes cluster supports it. If this is not the case, you will find your deployment failing. The solution is discussed further here: https://github.com/kubeapps/kubeapps/issues/479. From the Kubernetes Dashboard, find the deployment of the mongo database, edit the configuration and find the
settings <code>MONGODB_ENABLE_IPV6</code>. By default this value is set to "yes", change this value to "no" and save. Kubernetes will automatically redeploy the mongodb node and after a few more minutes the deployment will heal. You can monitor this progress within the Kubernetes Dashboard or by calling
 <nowiki>
kubectl get pods -w --namespace kubeapps</nowiki>

=== Configure Kubeapps ===
The Kubeapps page will require an API token in order to access your kubernetes cluster. Create an account and api token with the following commands
 <nowiki>
kubectl create serviceaccount kubeapps-operator
kubectl create clusterrolebinding kubeapps-operator --clusterrole=cluster-admin --serviceaccount=default:kubeapps-operator</nowiki>

Then retrieve the token for usage when accessing the dashboard with the following command:
 <nowiki>
kubectl get secret $(kubectl get serviceaccount kubeapps-operator -o jsonpath='{.secrets[].name}') -o jsonpath='{.data.token}' | base64 --decode</nowiki>

=== Access Kubeapps ===
You can securely access the kubeapps dashboard by executing the following command:
 <nowiki>
export POD_NAME=$(kubectl get pods -n kubeapps -l "app=kubeapps,release=kubeapps" -o jsonpath="{.items[0].metadata.name}")
kubectl port-forward -n kubeapps $POD_NAME 8080:8080</nowiki>
The above command will hang. You can access the kubeapps dashboard in your browser at http://127.0.0.1:8080. When the page loads you will be prompted to enter the kubernetes API token. Fetch the token as described earlier in "Configure Kubeapps" and enter it. Select Login and you will be brought to the Kubeapps home page

you have now successfully installed and logged into Kubeapps! From here you can install additional tools and applications from within the GUI

== Install MetalLB ==
Resources: https://metallb.universe.tf/

MetalLB is a loadbalancing software for barebone installations of kubernetes (This installation). The problem with kubernetes is that its loadbalancer service implementation is designed to work with cloud providers such as Azure or AWS and thus contains no functionality for barebone installations. kubernetes will allow you to create loadbalancer services, but it will never assign them a public IP, making it impossible to make your cluster application publicly accessible

=== Installation ===
MetalLB can be easily installed with Helm:
 <nowiki>
helm install --name metallb stable/metallb </nowiki>

=== Configuration ===
MetalLB requires a ConfigMap to be setup before it will start working. The following is a minimum setup for MetalLB:
 <nowiki>
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 192.168.1.240-192.168.1.250 </nowiki>
This implementation simply lets you specify the public ip range that is available on all of your kubernetes nodes. The above for example will make all ips from 240 to 250 available.

When creating your services now simply use the <code>loadBalancerIP</code> parameter in the spec section of your LoadBalancer config yaml. Specify one of the IPs available in the range
and create your service as usual. MetalLB comes with also error logging which will show up by typing <code>kubectl describe svc "servicename"</code>. It will print output to the logging output
normaly presented in this command.

Make sure to run the ConfigMap configuration above and then redeploy your LoadBalancers using the loadBalancerIP parameter and MetalLB will ensure they are assigned the specified IP. If you have
LoadBalancers already deployed, you will have to redeploy them for MetalLB to notice.

Note that MetalLB also does not need to be in the same namespace as the LoadBalancer in order to work



==Notes==

==Sources==
http://vijayshinva.github.io/kubernetes/2018/07/28/setting-up-a-kubernetes-cluster-on-a-windows-laptop-using-hyper-v.html <br>
https://www.miniwebtool.com/mac-address-generator/ <br>
https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-debian-9 <br>
https://kubernetes.io/docs/setup/independent/install-kubeadm/ <br>
https://serverfault.com/questions/684771/best-way-to-disable-swap-in-linux <br>
https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/storage_administration_guide/s2-disk-storage-parted-remove-part
