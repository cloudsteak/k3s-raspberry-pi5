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
- [Private image repository configuration](/#cr)
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

## <a name="cr"></a>Private image repository configuration

### Azure Container Registry (ACR)

#### Create ACR from

- Install Azure-Cli

```bash
sudo curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

- Check Azure-Cli

```bash
az -v
```

- Login to Azure

```bash
az login
# Then follow the instructions
```

- Check the right subscription

```bash
az account list -o table
```

- Create ACR

```bash
# Random name for resource group
resourceGroupName=$(cat /dev/urandom | tr -cd 'a-f0-9' | head -c 8)
# Region
resourceGroupRegion="swedencentral"
# Random name for acr
acrName=$(cat /dev/urandom | tr -cd 'a-f0-9' | head -c 12)

az group create --name $resourceGroupName --location $resourceGroupRegion

az acr create --resource-group $resourceGroupName --name $acrName --sku Basic

az acr update -n $acrName  --admin-enabled true
```

- Get connection information for ACR:
  1. Open Azure Portal: http://portal.azure.com
  2. Find the newly created ACR: https://portal.azure.com/#view/HubsExtension/BrowseResource/resourceType/Microsoft.ContainerRegistry%2Fregistries
  3. open it and find the `Access Keys` menu item
  4. Copy the following information:
     - Login server
     - Username
     - password

#### Connect to ACR from K3s

- Edit K3s registries.yaml

```bash
sudo nano /etc/rancher/k3s/registries.yaml
```

- Add the ACR related configuration to end of it (https://docs.k3s.io/installation/private-registry#without-tls)

```yaml
mirrors:
  "{ Your ACR Login server value }":
    endpoint:
      - "https://{ Your ACR Login server value }"
configs:
  "{ Your ACR Login server value }":
    auth:
      username: xxxxxxxxxx # this is the { ACR Username }
      password: xxxxxxxxxx # this is the { ACR password }
```

- Restart K3d

```bash
sudo systemctl restart k3s
```


## <a name="nginx"></a>Ingress NGINX installation and configuration

## <a name="example1"></a>Example webapp installation with ingress
