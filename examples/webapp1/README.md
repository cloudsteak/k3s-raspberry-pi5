# NodeJS WebApp

- Node 18 LTS
- Image on ACR

## Local "Build and run"

- Go to work dir

```bash
cd examples/webapp1
```

- Define ACR name

```bash
acrName=finallcloudacr.azurecr.io
```

- Image build

```bash
docker build --tag $acrName/node-webapp .
```

- Check result

```bash
docker images
```

Note: If you build the image on Apple Silicon processor, use the following flag: `--platform linux/amd64` or `--platform linux/arm/v7`

- Local run:

```bash
docker run -d -p 8080:3000 --name nodedemo $acrName/node-webapp:latest
```

Check: http://localhost:8080

## Push image to ACR

- Login to ACR

```bash
az acr login --name $acrName
```

- Create real version tag

```bash
docker tag $acrName/node-webapp:latest $acrName/node-webapp:1.0.0
```

- Upload images to ACR

```bash
docker push $acrName/node-webapp:latest
docker push $acrName/node-webapp:1.0.0
```

## Create WebApp in K3s

### Check deployment file: [node-webapp.yaml](https://raw.githubusercontent.com/cloudsteak/k3s-raspberry-pi5/main/examples/webapp1/node-webapp.yaml)

Important parameters:

- Namespace: node-webapp
- replicas: 2 (it could be 1 ot more)
- Image: finallcloudacr.azurecr.io/node-webapp:latest
- host: node-webapp.finall.cloud (it must be your FQDN for your Raspberry Pi 5. Ip address is not acceptable)

### Create WebApp

```bash
kubectl apply -f node-webapp.yaml
```

### Check result

- Get all resources from related namespace

```bash
kubectl get all -n node-webapp
```

- Get ingress info

```bash
kubectl get ingress -n node-webapp
```

Note: Ingress must have same address to Raspberry Pi 5.

## Scale

```bash
# More Pods
kubectl -n node-webapp scale --replicas=5 deployment node-webapp-deployment
```

```bash
# Less pods
kubectl -n node-webapp scale --replicas=1 deployment node-webapp-deployment
```

## Upgrade app if new version is available

```bash
# Delete all old pods
kubectl get pods -n node-webapp -o name  | grep node-webapp-deployment | xargs kubectl delete -n node-webapp --wait=false
```

## Delete resource

- Delete namespace

```bash
kubectl delete ns node-webapp --wait=false
```
