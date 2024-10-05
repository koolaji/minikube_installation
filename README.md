# Minikube Installation Guide

Minikube is a lightweight Kubernetes implementation that lets you develop and test Kubernetes applications locally.
### History of Minikube
Minikube was created to provide developers with a simple and lightweight method of running a Kubernetes cluster on their local machine. It was introduced by the Kubernetes project in 2016, offering a minimal Kubernetes environment for development and testing purposes. Before Minikube, setting up a local Kubernetes cluster was complex and required heavy resources.

### Key Milestones:
* 2016: Minikube was initially released by the Kubernetes community to allow users to run a local Kubernetes cluster easily.
* 2017-2018: Minikube evolved quickly with features like support for multiple hypervisors, networking improvements, and resource management.
* 2019: Minikube introduced features like minikube addons, providing additional tools like dashboard, metrics-server, and ingress to improve the developer experience.
* 2020: Support for container runtimes like Docker and Containerd was enhanced. Minikube could also run in virtual machines (VMs) or using Docker on the host machine.
* 2021-Present: Continuous improvements in performance and resource management, making Minikube faster and more efficient, particularly in environments with limited resources. The introduction of new drivers such as KVM2, Podman, and none (native host without a VM) made it even more versatile.
Minikube has become the go-to solution for developers looking to experiment with Kubernetes locally without needing to set up a full production cluster. It has been a fundamental tool for learning, testing, and prototyping Kubernetes-based applications.



### Architecture of Minikube
Minikube is designed to create and manage a local Kubernetes cluster, typically on a single-node setup. Below is an overview of its architecture:

1. Host Machine
Minikube is installed on the host machine, which can be a Linux, macOS, or Windows system. The host machine will run Minikube as a lightweight Kubernetes instance. Depending on the driver's selection, Minikube can run in a virtual machine (VM), container, or directly on the host.

2. Virtualization Layer
Minikube uses different drivers to manage where and how Kubernetes runs. Some popular drivers include:

* VirtualBox/HyperKit/KVM: Minikube runs in a VM and uses these hypervisors to manage the virtual environment.
* Docker Driver: Minikube uses Docker to run Kubernetes directly in containers on the host machine.
* None Driver: This driver allows Minikube to run directly on the host system without any virtualization layer.
Minikube can choose a driver based on the host’s setup and availability of resources, with Docker and none being the most popular for local development.

3. Kubernetes Components
Minikube provisions a single-node Kubernetes cluster with all the necessary Kubernetes components:

* kube-apiserver: The entry point for all administrative Kubernetes commands (like kubectl).
* kube-controller-manager: Manages the cluster state and ensures the desired state matches the actual state.
* kube-scheduler: Assigns Pods to available Nodes based on resource availability.
* kubelet: Runs on the single node and ensures that containers are running in the Pods as instructed.
* kube-proxy: Handles network routing between services and Pods inside the cluster.
These components are packaged in Minikube and run within the environment that the driver has provided (VM, Docker, or directly on the host).

4. Addons and Integrations
Minikube provides support for additional Kubernetes features and tools in the form of addons. Some popular Minikube addons include:

* Dashboard: A web-based user interface for managing and visualizing Kubernetes clusters.
* Ingress Controller: Provides an easy way to manage external access to services.
* Metrics Server: Offers CPU and memory usage metrics to help monitor and optimize resources.
5. Networking
Minikube manages its own networking layer, allowing users to access Kubernetes services from the host machine. It forwards Kubernetes services to the host, allowing for local testing without needing complicated external network setups.

For example, with the minikube service command, users can easily access running services in their browser. This simplifies development and debugging tasks.

6. Storage
Minikube supports multiple types of storage provisioners:

* HostPath: Mounts directories from the host machine into Pods as volumes.
* Local Persistent Volumes: Allows for local storage to be treated as persistent volume claims.
Minikube ensures the storage system works seamlessly with Kubernetes’ volume management system for data persistence and testing.

7. Cluster Management
Minikube provides several utilities to manage the local Kubernetes cluster:

* minikube start/stop: Start or stop the Kubernetes cluster.
* minikube status: Check the status of the cluster and its components.
* minikube dashboard: Opens the Kubernetes dashboard in a web browser.
* minikube addons enable/disable [addon]: Manages Kubernetes addons like Ingress, Dashboard, or Metrics Server.

### Why Use Minikube?
* Lightweight: Minikube provides a lightweight environment to run Kubernetes without the overhead of managing a full-blown production cluster.
* Fast Setup: It’s designed to be easy to install and quick to start, making it ideal for rapid development and testing.
* Portable: Minikube works on multiple platforms, such as Linux, macOS, and Windows, and supports a variety of drivers.
* Complete Kubernetes Experience: Despite its simplicity, Minikube offers a full Kubernetes environment, meaning developers can test applications as they would in a production cluster.


## Prerequisites

Before you install Minikube, make sure your system meets the following requirements:

- **OS**: Linux(Debian)
- **Tools**:
  - Docker 

## install docker 
# Add Docker's official GPG key:
```bash
echo "deb https://deb.debian.org/debian bookworm main contribe" >  /etc/apt/sources.list
apt update
apt install sudo vim curl
sudo apt-get -y install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```
# Add the repository to Apt sources:
```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin  
```

### Docker socks proxy 
if you want to add docker proxy please use these commands 
```bash
mkdir -p /etc/systemd/system/docker.service.d
echo -e "[Service] \nEnvironment=\"HTTP_PROXY=socks5://192.168.31.76:9999\"\nEnvironment=\"HTTPS_PROXY=socks5://192.168.31.76:9999\"" >  /etc/systemd/system/docker.service.d/proxy.conf
systemctl daemon-reload
systemctl restart docker
usermod -aG docker $USER # add user that you want to install minikube, minikube could not install with root user 
```
Check if all things is ok 
```bash
sudo docker run hello-world
```
### offline installation
if you want to install offline the Minikube you cloud download the images and import to docker daemon that you want to install Minikube 
```bash
#!/bin/bash

# List of Docker images
images=(
  "gcr.io/k8s-minikube/kicbase@sha256:aeed0e1d4642"
  "registry.k8s.io/kube-scheduler:v1.31.0"
  "registry.k8s.io/kube-apiserver:v1.31.0"
  "registry.k8s.io/kube-controller-manager:v1.31.0"
  "registry.k8s.io/kube-proxy:v1.31.0"
  "registry.k8s.io/etcd:3.5.15-0"
  "registry.k8s.io/pause:3.10"
  "registry.k8s.io/coredns/coredns:v1.11.1"
  "hello-world:latest"
  "gcr.io/k8s-minikube/storage-provisioner:v5"
)

# Function to pull Docker images
pull_images() {
  for image in "${images[@]}"; do
    echo "Pulling $image..."
    docker pull "$image"
  done
  echo "All images pulled successfully!"
}

# Function to save Docker images to a tar file
save_images() {
  tar_file="docker_images_backup.tar"
  echo "Saving images to $tar_file..."
  docker save -o "$tar_file" "${images[@]}"
  echo "Images saved to $tar_file successfully!"
}

# Function to load Docker images from a tar file
load_images() {
  tar_file="docker_images_backup.tar"
  if [ -f "$tar_file" ]; then
    echo "Loading images from $tar_file..."
    docker load -i "$tar_file"
    echo "Images loaded successfully!"
  else
    echo "Error: $tar_file not found. Please ensure the file exists."
    exit 1
  fi
}

# Main menu
echo "Choose an option:"
echo "1) Pull Docker images"
echo "2) Save Docker images"
echo "3) Load Docker images"
read -p "Enter your choice (1/2/3): " choice

case $choice in
  1)
    pull_images
    ;;
  2)
    save_images
    ;;
  3)
    load_images
    ;;
  *)
    echo "Invalid choice! Exiting."
    exit 1
    ;;
esac
```

## Installing Minikube

Follow these steps to install Minikube:

### 1. Download Minikube

Download the latest version of Minikube for your platform.
```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
sudo dpkg -i minikube_latest_amd64.deb
```
#### Linux:

If you do not want to use a proxy please do not export commands 
```bash
export HTTP_PROXY=socks5://192.168.31.76:9999
export HTTPS_PROXY=socks5://192.168.31.76:9999
minikube start --driver=docker
```
