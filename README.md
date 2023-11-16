# Home Kubernetes on Raspberry Pi 5

## Background

- In this configuration I use **only one** Raspberry Pi 5.
- I use K3s with disabled traefik

## Technical information:

- Board: Raspberry Pi 5
- Memory: 8 Gb
- Storage: 64 GB (SANDISK EXTREME microSDXC 64 GB 170/80 MB/s UHS-I U3)
- OS: Ubuntu 23.10 for Raspberry Pi 5 (https://ubuntu.com/download/raspberry-pi)

## Table of content

- [Basic installation and configuration](/#installation)
- [Private image repository configuration (Azure ACR)](/#cr)
- [Ingress NGINX installation and configuration](/#nginx)
- [Example webapp installation with ingress](/#example1)

## <a name="installation"></a>Basic installation and configuration

### Installation

Start here: https://k3s.io

- Deploy K3s without traefik:

```bash
# Disable traefik
export INSTALL_K3S_EXEC="server --no-deploy traefik"

# Create k3s cluster
curl -sfL https://get.k3s.io | sh -s -
```

- Check installation:

```bash
sudo k3s kubectl get node
```

### Connect to cluster

- Install kubectl

```bash
sudo snap install kubectl --classic
```

- Use K3s config for kubectl to connect the cluster

```bash
# Create .kube directory
mkdir ~/.kube

# Copy k3s.yaml to my user directory
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config

# Modify ownership on file. If you have config there, it will overwrite it!
sudo chown $USER:$USER ~/.kube/config
```

- Check connection:

```bash
kubectl get node
```

## <a name="cr"></a>Private image repository configuration (Azure ACR)

## <a name="nginx"></a>Ingress NGINX installation and configuration

## <a name="example1"></a>Example webapp installation with ingress
