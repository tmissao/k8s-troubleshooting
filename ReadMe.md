kind create cluster --config ./kind/multi-node.yml

helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update


=== Nginx

helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx \
  -f ./helm/nginx-ingress-controller/values.yaml \
  --namespace nginx-ingress \
  --create-namespace \
  --force

k create ns app

k apply -n app -f ./apps/nginx
k port-forward -n app services/nginx 3000:80

=== ELK

helm repo add elastic https://helm.elastic.co
helm repo update

helm upgrade --install elastic elastic/eck-operator \
  -n  logging --create-namespace

kubectl apply -f ./helm/eck -n logging

echo "127.0.0.1 kibana.local" | sudo tee -a /etc/hosts

=== FluentBit


helm repo add fluent https://fluent.github.io/helm-charts
helm repo update

helm upgrade --install fluent-bit fluent/fluent-bit \
  -f ./helm/fluentbit/values.yaml \
  --namespace logging \
  --create-namespace \
  --force

=== Prometheus

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm upgrade --install kube-prometheus prometheus-community/kube-prometheus-stack \
  -f ./helm/prometheus/values.yaml \
  --namespace monitoring \
  --create-namespace
  --force


echo "127.0.0.1 grafana.local" | sudo tee -a /etc/hosts

k port-forward -n monitoring services/kube-prometheus-kube-prome-prometheus 9090:9090


