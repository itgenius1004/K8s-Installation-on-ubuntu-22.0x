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

2. Install kubelet, kubeadm and kubectl

        sudo apt install curl apt-transport-https -y
        curl -fsSL  https://packages.cloud.google.com/apt/doc/apt-key.gpg|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/k8s.gpg
        echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

        sudo apt update
        sudo apt install wget curl vim git kubelet kubeadm kubectl -y
        sudo apt-mark hold kubelet kubeadm kubectl

        kubectl version --client
        kubeadm version
   
3. Disable Swap Space

        sudo swapoff -a
        free -h
        sudo sed -i.bak -r 's/(.+ swap .+)/#\1/' /etc/fstab
        sudo vim /etc/fstab
        sudo mount -a
        free -h
        # Enable kernel modules
        sudo modprobe overlay
        sudo modprobe br_netfilter
        
        # Add some settings to sysctl
        sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
        net.bridge.bridge-nf-call-ip6tables = 1
        net.bridge.bridge-nf-call-iptables = 1
        net.ipv4.ip_forward = 1
        EOF
        
        # Reload sysctl
        sudo sysctl --system

4. Install Container runtime(all nodes) - you have to choose one runtime at a time
   * Docker
   * CRI-O
   * Containerd

   a. Docker runtime
     
          # Add repo and Install packages
          sudo apt update
          sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
          #### curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
          curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/docker.gpg
          sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
          sudo apt update
          sudo apt install -y containerd.io docker-ce docker-ce-cli
          
          # Create required directories
          sudo mkdir -p /etc/systemd/system/docker.service.d
          
          # Create daemon json config file
          sudo tee /etc/docker/daemon.json <<EOF
          {
            "exec-opts": ["native.cgroupdriver=systemd"],
            "log-driver": "json-file",
            "log-opts": {
              "max-size": "100m"
            },
            "storage-driver": "overlay2"
          }
          EOF
          
          # Start and enable Services
          sudo systemctl daemon-reload 
          sudo systemctl restart docker
          sudo systemctl enable docker
          
          # Configure persistent loading of modules
          sudo tee /etc/modules-load.d/k8s.conf <<EOF
          overlay
          br_netfilter
          EOF
          
          # Ensure you load modules
          sudo modprobe overlay
          sudo modprobe br_netfilter
          
          # Set up required sysctl params
          sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
          net.bridge.bridge-nf-call-ip6tables = 1
          net.bridge.bridge-nf-call-iptables = 1
          net.ipv4.ip_forward = 1
          EOF
     
   b. Installing CRI-O runtime

          # Configure persistent loading of modules
          sudo tee /etc/modules-load.d/k8s.conf <<EOF
          overlay
          br_netfilter
          EOF
          
          # Ensure you load modules
          sudo modprobe overlay
          sudo modprobe br_netfilter
          
          # Set up required sysctl params
          sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
          net.bridge.bridge-nf-call-ip6tables = 1
          net.bridge.bridge-nf-call-iptables = 1
          net.ipv4.ip_forward = 1
          EOF
          
          # Reload sysctl
          sudo sysctl --system
          
          # Add Cri-o repo
          OS="xUbuntu_22.04"
          VERSION=1.28
          echo "deb https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/ /" > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list
          echo "deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable:/cri-o:/$VERSION/$OS/ /" > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable:cri-o:$VERSION.list
          curl -L https://download.opensuse.org/repositories/devel:kubic:libcontainers:stable:cri-o:$VERSION/$OS/Release.key | apt-key add -
          curl -L https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$OS/Release.key | apt-key add -
          
          # Install CRI-O
          sudo apt update
          sudo apt install cri-o cri-o-runc
          
          # Start and enable Service
          sudo systemctl daemon-reload
          sudo systemctl restart crio
          sudo systemctl enable crio
          systemctl status crio

     c. Installing Containerd
   
          # Configure persistent loading of modules
          sudo tee /etc/modules-load.d/k8s.conf <<EOF
          overlay
          br_netfilter
          EOF
          
          # Load at runtime
          sudo modprobe overlay
          sudo modprobe br_netfilter
          
          # Ensure sysctl params are set
          sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
          net.bridge.bridge-nf-call-ip6tables = 1
          net.bridge.bridge-nf-call-iptables = 1
          net.ipv4.ip_forward = 1
          EOF
          
          # Reload configs
          sudo sysctl --system
          
          # Install required packages
          sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
          
          # Add Docker repo
          curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/docker-archive-keyring.gpg
          sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
          
          # Install containerd
          sudo apt update
          sudo apt install -y containerd.io
          
          # Configure containerd and start service
          sudo mkdir -p /etc/containerd
          sudo containerd config default|sudo tee /etc/containerd/config.toml
          
          # restart containerd
          sudo systemctl restart containerd
          sudo systemctl enable containerd
          systemctl status containerd
   

6. 
