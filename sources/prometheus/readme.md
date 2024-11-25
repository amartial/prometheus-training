# Installation de prometheus dans le cluster kubernetes

kubectl create namespace monitoring

cd ./sources/prometheus

kubectl apply -f clusterRole.yaml
kubectl apply -f config-map.yaml
kubectl apply -f prometheus-deployment.yaml -n monitoring
kubectl apply -f prometheus-service.yaml -n monitoring

kubectl get all -n monitoring

# acceder à l'interface web prometheus
kubectl port-forward --address 0.0.0.0 svc/prometheus-service 8090:8090 -n monitoring

# pour voir le target prometheus configuré
http://localhost:8090/targets


## Installation de grafana
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

cd ./sources/grafana
helm install grafana-dashboard -f values.yaml grafana/grafana

# mot de passe du user admin de grafana
kubectl get secret -n default grafana-dashboard -o jsonpath='{.data.admin-password}' | base64 --decode ; echo

Get the grafana url to visit by running thes commands in th same shell:
export NODE_PORT=$(kubectl get -n default -o jsonpath="{.spec.ports[0].nodePort}" services grafana-dashboard)
export NODE_IP=$(kubectl get nodes -n default -o jsonpath="{.spec.ports[0].status.addresses[0].address}")
echo http://$NODE_IP:$NODE_PORT

kubectl get svc

kubectl port-forward --address 0.0.0.0 svc/grafana-service 9000:9000


# Connexion grafana prometheus - configuration datasource
url: http://prometheus-service.monitoring.svc:8090

# Création d'un nouveau dashboard
Numéro du dashboard prometheus 2.0 Overview:
https://grafana.com/grafana/dashboards/3662
3662


# Monitoring des hotes / OS: Node Exporter
# Monitoring systeme des hotes et des Noeuds
# Monitoring des hotes du cluster

- Prometheus exporter
- Il est ecrit en Go
- Il permet de récuperer des données / métriques exposées par le noyau Unix/Linux concernant
    - l'OS
    - le hardware

# Déploiement du node exporter
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install node-exporter prometheus-community/prometheus-node-exporter
helm search repo prometheus-community

kubectl get po
kubectl get svc

# Ajouter le job node-exporter dans le fichier config-map.yaml ./lab-4/

# ./lab-4/config-map.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-server-conf
  labels:
    name: prometheus-server-conf
  namespace: monitoring
data:
  prometheus.yml: |-
    global:
      scrape_interval: 20s
      evaluation_interval: 20s
    scrape_configs:
      - job_name: 'prometheus'
        static_configs:
          - targets: ['localhost:9090']
      - job_name: node-exporter
        static_configs:
          - targets: ['node-exporter-prometheus-node-exporter.default.svc:9100']

# supprimer le configmaps et le recréer
kubectl delete configmap prometheus-server-conf -n monitoring
kubectl create -f configmap.yaml

# supprimer et recréer le déploiement
kubectl delete deployment prometheus-deployment -n monitoring
kubectl apply -f prometheus-deployment.yaml -n monitoring

# Création d'un nouveau dashboard
Numéro du dashboard prometheus node-exporter
https://grafana.com/grafana/dashboards/1860
1860