# k3s-raspberry-pi5

Home Kubernetes on Raspberry Pi 5

<u>Background</u>

- In this configuration I use **only one** Raspberry Pi 5.
- I use K3s with disabled traefik

<u>Technical information:</u>

- Board: Raspberry Pi 5
- Memory: 8 Gb
- Storage: 64 GB (SANDISK EXTREME microSDXC 64 GB 170/80 MB/s UHS-I U3)
- OS: Ubuntu 23.10 for Raspberry Pi 5 (https://ubuntu.com/download/raspberry-pi)


<u>Table of content</u>

- [Basic installation and configuration](/#installation)
- [Private image repository configuration (Azure ACR)](/#cr)
- [Ingress NGINX installation and configuration](/#nginx)
- [Example webapp installation with ingress](/#example1)