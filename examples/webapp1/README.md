# NodeJS WebApp

- Node 18 LTS
- Image on ACR

### Local "Build and run"

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

### Push image to ACR

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

### Create WebApp in K3s

#### Check deployment file: [examples/webapp1/node-webapp.yaml](https://raw.githubusercontent.com/cloudsteak/k3s-raspberry-pi5/main/examples/webapp1/node-webapp.yaml)

Important parameters:

- Namespace: node-webapp
- replicas: 2 (it could be 1 ot more)
- Image: finallcloudacr.azurecr.io/node-webapp:latest
- host: node-webapp.finall.cloud (it must be your FQDN for your Raspberry Pi 5. Ip address is not acceptable)

#### Create WebApp

```bash
kubectl apply -f examples/webapp1/node-webapp.yaml
```
