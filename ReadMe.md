# Local Kubernetes Environment

## 📘 Overview
---

This project provides a complete local Kubernetes environment using Kind (Kubernetes-in-Docker) with ready-to-use monitoring, logging, and ingress infrastructure.
It is designed for developers and DevOps engineers who want to test, debug, and validate observability stacks locally before deploying to production.

## ✅ Advantages
---

 - Full observability: integrated logging (ELK) and monitoring (Prometheus + Grafana) stacks.

- Production-like ingress routing: NGINX Ingress Controller exposes local services through friendly hostnames (grafana.local, kibana.local).

- Fast feedback loop: no need for a remote cluster—everything runs locally in Docker.

- Reproducible: all components deployed with Helm and version-controlled YAML values.

- Extensible: easily add new charts, CRDs, or test your own workloads.

## ⚙️ Setup
---

In order to run this project the following resources are required

- [Install Docker](https://docs.docker.com/engine/install/ubuntu/)
- [Install Docker-Compose](https://docs.docker.com/compose/install/)
- [Install KinD](https://kind.sigs.k8s.io/docs/user/quick-start#installing-from-release-binaries)
- [Install Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

## 🧩 Project Structure
---
The components of this project are organized in the following way

```
├── apps                       # Application Folders to deploy validations
├── helm                       # Helm related files, such as values.yaml
├── kind                       # KinD configuration
```

## 🚀 Usage

In order to deploy the Local K8s Cluster follow these steps

1. `Create the Kind Cluster`
```bash
kind create cluster --config ./kind/multi-node.yml
```

2. `Deploy Nginx Ingress Controller`

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx \
  -f ./helm/nginx-ingress-controller/values.yaml \
  --namespace nginx-ingress \
  --create-namespace \
  --force
```

> This deployments maps your localhost port `80` and `443` to K8s cluster, so it is possible to use the `/etc/hosts` file in your machine to "fake" the FQDN URLs

In order to validate the Ingress configuration deploy the demo app through the command
```bash
kubectl create ns app
kubectl apply -n app -f ./apps/nginx

# Creates a local DNS resolution in your machine for the address "app.local"
echo "127.0.0.1 app.local" | sudo tee -a /etc/hosts

```

After that, accessing the address [app.local](http://app.local) will lead to a custom welcome page

3. `Deploy Prometheus Stack - Metrics` - Optional

    Deploys the Prometheus stack on Local K8s cluster, allowing to access the Grafana interface through the address [grafana.local](http://grafana.local)

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm upgrade --install kube-prometheus prometheus-community/kube-prometheus-stack \
  -f ./helm/prometheus/values.yaml \
  --namespace monitoring \
  --create-namespace
  --force

echo "127.0.0.1 grafana.local" | sudo tee -a /etc/hosts

```

If required you can also create a `port-forward` to access the prometheus instance directly. (Useful to validate ServiceMonitor configuration)

```bash
kubectl port-forward -n monitoring services/kube-prometheus-kube-prome-prometheus 9090:9090
```

After, check the address [localhost:9090](http://localhost:9090).

4. `Deploy ELK Stack - Logging` - Optional

    Deploys the ELK stack on Local K8s cluster, allowing to access the Kibana interface through the address [kibana.local](http://kibana.local)

    > ELK Stack is resource intensive so it may freeze your machine
```
helm repo add elastic https://helm.elastic.co
helm repo update

helm upgrade --install elastic elastic/eck-operator \
  -n  logging --create-namespace

kubectl apply -f ./helm/eck -n logging

echo "127.0.0.1 kibana.local" | sudo tee -a /etc/hosts
```

5. `Deploy FluentBit - Logging Shipping` - Optional
    
    Deploys Fluentbit daemon, configuring it by default to ship logs to ELK, so it requires the ELK deployment or the customization of its pipeline located [here](./helm/fluentbit/values.yaml).
    
```
helm repo add fluent https://fluent.github.io/helm-charts
helm repo update

helm upgrade --install fluent-bit fluent/fluent-bit \
  -f ./helm/fluentbit/values.yaml \
  --namespace logging \
  --create-namespace \
  --force
```

## 🧹 Cleanup

To remove everything and reclaim resources:
```
kind delete cluster
```

## 📚 References
---

- [KinD](https://kind.sigs.k8s.io/)
- [Nginx Ingress Gateway](https://kubernetes.github.io/ingress-nginx/)
- [Prometheus Stack](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack)
- [ELK Stack](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s)
- [FluentBit](https://docs.fluentbit.io/manual/installation/downloads/kubernetes)