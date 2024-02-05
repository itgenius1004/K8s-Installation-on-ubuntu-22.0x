# K8s-Installation-on-ubuntu-22.0x
* Services provided by Kubernetes
1. Self-healing
2. Horizontal scaling
3. Compute scheduling
4. Service discovery/load balancing
5. Automated rollouts & rollbacks
6. Secret & configuration management
7. Volume management

* Master Nodes : This handles the control API calls for the pods, replications controllers, services, nodes and other components of a Kubernetes cluster.
* Node : Provides the run-time environments for the containers. A set of container pods can span multiple nodes.

* Minimum hardware requirements recommendation
  - Memory : 2 GB or more of RAM per machine
  - CPUs: At least 2 CPUs on the control plane machine
  - Internet connectivity for pulling containers required
  - Full network connectivity between machines in the cluster

* Installing Kubernetes Cluster on Ubuntu 22.0x using kubeadm
  - How to disable cloud-init in Ubunut
      How to disable cloud-init in Ubuntu
      Prevent start
        Create an empty file to prevent the service from starting
        sudo touch /etc/cloud/cloud-init.disabled
      Uninstall
      Disable all services (uncheck everything except "None"):
      
        sudo dpkg-reconfigure cloud-init
      Uninstall the package and delete the folders
      
        sudo dpkg-reconfigure cloud-init
        sudo apt-get purge cloud-init
        sudo rm -rf /etc/cloud/ && sudo rm -rf /var/lib/cloud/
      Restart the computer

        sudo reboot
    
1. Upgrade your Ubunut servers
 
        sudo apt update
        sudo apt -y full-upgrade && sudo reboot -f

3. Install kubelet, kubeadm and kubectl

        sudo apt install curl apt-transport-https -y
        curl -fsSL  https://packages.cloud.google.com/apt/doc/apt-key.gpg|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/k8s.gpg
        echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

        sudo apt update
        sudo apt install wget curl vim git kubelet kubeadm kubectl -y
        sudo apt-mark hold kubelet kubeadm kubectl

        kubectl version --client
        kubeadm version
