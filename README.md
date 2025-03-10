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

### Documentation

Start here: https://k3s.io

### Installation

- Deploy K3s without traefik:

```bash

sudo -i
# Disable traefik
export INSTALL_K3S_EXEC="server --disable=traefik --write-kubeconfig-mode=664"

# Create k3s cluster (for remote access use --tls-san {your server fqdn})
curl -sfL https://get.k3s.io | sh -s - server --tls-san {your server fqdn}
```

- Check installation:

```bash
systemctl status k3s.service
```

- Exit root mode

```bash
exit
```

- Check installation:

```bash
sudo k3s kubectl get node
```

```bash
k3s kubectl get node
```

Config file: `sudo nano /etc/systemd/system/k3s.service`

### Connect to cluster

- Install kubectl

```bash
snap install kubectl --classic
```

- Check connection:

```bash
kubectl get node
```

### Kubeconfig file

Location on the server: `/etc/rancher/k3s/k3s.yaml`

- Copy the file to your userr's home directory:

```bash
cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
```

- Copy the file to your local machine:

```bash
scp user@server:/etc/rancher/k3s/k3s.yaml ~/k3s.yaml
```

- Add it to your existing kubeconfig file:

```bash
cat k3s.yaml >> ~/.kube/config
```

- Modify ip address in the kubeconfig file:

```bash
# Linux
sed -i 's/127.0.0.1/{your server ip}/g' ~/.kube/config

# MacOS
sed -i '' 's/127.0.0.1/{your server ip}/g' ~/.kube/config

```

- Modify server name from default to your server name:

```bash
# Linux
sed -i 's/default/{your server name}/g' ~/.kube/config

# MacOS
sed -i '' 's/default/{your server name}/g' ~/.kube/config
```

### Uninstall K3s

```bash
sudo /usr/local/bin/k3s-uninstall.sh
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

- Restart K3s

```bash
sudo systemctl restart k3s
```

- Delete K3s

```bash
sudo /usr/local/bin/k3s-uninstall.sh
```

## <a name="nginx"></a>Ingress NGINX installation and configuration

Because NGINX is pretty common in this world, I use it for this scenario. Related document: https://kubernetes.github.io/ingress-nginx/deploy/

> Notes:

> _I don't use Helm here._

> _Check the available releases for controller: https://github.com/kubernetes/ingress-nginx/releases_

- Install the controller

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/cloud/deploy.yaml
```

- Check result

```bash
kubectl get svc -n ingress-nginx
```

Note: You need to see something similar (EXTERNAL-IP is the IP of your Raspberry Pi 5)

```bash
NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)
ingress-nginx-controller             LoadBalancer   10.xxx.xxx.xxx   xxx.xxx.xxx.xxx   80:31449/TCP,443:32554/TCP
```

## <a name="example1"></a>Example webapp installation with ingress

Full documentation: [examples/webapp1](examples/webapp1/README.md)
