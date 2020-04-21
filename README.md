# kong-openfaas <!-- omit in toc -->

Kong Ingress Controller + OpenFaas integration sample

- [What you need](#what-you-need)
- [What to do (k3d)](#what-to-do-k3d)
  - [Create a local cluster with k3d](#create-a-local-cluster-with-k3d)
  - [Deploy OpenFaas using arkade](#deploy-openfaas-using-arkade)
  - [Deploy Kong Ingress Controller](#deploy-kong-ingress-controller)
  - [Open OpenFaas Gateway](#open-openfaas-gateway)
  - [Using faas-cli](#using-faas-cli)

## What you need

- [k3d](https://github.com/rancher/k3d)
- [Helm 3](https://helm.sh/docs/intro/install/) (MUST be Helm 3 at least)
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [arkade](https://github.com/alexellis/arkade)

## What to do (k3d)

### Create a local cluster with k3d

Use `k3d` to create a local development cluster:

```sh
k3d create -n kong-local \
  --publish 8080:32080 \
  --workers="1" \
  --server-arg "--no-deploy=traefik" \
  --server-arg "--no-deploy=servicelb"
```

The arguments abova disable Trefik (a default on `k3d`), since we want Kong as Ingress Controller instead.
After cluster is created you can obtain its KUBECONFIG:

```sh
export KUBECONFIG="$(k3d get-kubeconfig --name='kong-local')"
kubectl cluster-info
```

### Deploy OpenFaas using arkade

```sh
arkade install openfaas --load-balancer="false" --basic-auth-password "password" --operator="true"
```

### Deploy Kong Ingress Controller

We need Kong official helm chart and our custom ingress:

```sh
helm repo add kong https://charts.konghq.com
helm repo update
helm upgrade -i kong kong/kong --version 1.5.0 \
  --set ingressController.installCRDs=false \
  --set ingressController.enabled=true \
  --set proxy.type=NodePort \
  --set proxy.http.enabled=true \
  --set proxy.http.nodePort=32080
kubectl apply -f ingress.yml
```

### Open OpenFaas Gateway

Create an entry in `/etc/hosts`:

```
127.0.0.1 	gateway.localdomain
```

Open "http://gateway.localdomain:8080/ui/" in you browser and you are good to go.

Try installing and invoking the **figlet** function from the store.

### Using faas-cli

```sh
export OPENFAAS_URL=http://gateway.localdomain:8080
faas-cli login --password password -u admin
echo -n OpenFaaS | faas-cli invoke figlet
```
