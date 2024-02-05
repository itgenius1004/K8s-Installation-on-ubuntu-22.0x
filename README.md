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

1. Upgrade your Ubunut servers
sudo apt update
sudo apt -y full-upgrade
[ -f /var/run/reboot-required ] && sudo reboot -f

2. 
  
