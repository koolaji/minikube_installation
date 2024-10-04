# Minikube Installation Guide

Minikube is a lightweight Kubernetes implementation that lets you develop and test Kubernetes applications locally.

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
