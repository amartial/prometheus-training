# Monitoring du Cluster (1/2):
# Kubernetes API health and metrics

- Kubernetes API Server
- Controller Manager
- Scheduler
- etcd

# Monitoring du Cluster (2/2): kube-state-metrics

- Le kube-state-metrics est un service qui permet d'évaluer:
    - Le nombre de pods démarrés / arretés / terminés
    - Le nombre de fois qu'un pod a été redémarré

-  Analyse le temps de réponse des services kubernetes afin de:
    - Déterminer les endpoints les plus adressés par les utilisateurs
    - Déterminer les endpoint http le plus lent
    - Déterminer les requetes qui ont renvoyées un code d'erreur


# Mise en place des métrics k8s API:
- Modifier le configmap prometheus afin d'y intégrer le endpoint de l'API k8s
- Vous devez supprimer et recréer le configmap ainsi que le deployment de prometheus pour appliquer les modifications
- Vérifier sur l'interface de prometheus que la target apiserver est bien présernte et up
- Pour terminer, importer le dashboard permettant de visuatliser les metriques des API k8s
- Vérifiez que le dashboard nouvellemnt importé affiche des données

# ./lab-5/config-map.yaml
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
      - job_name: 'kubernetes-apiservers'
        kubernetes_sd_configs:
        - role: endpoints
        scheme: https
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        relabel_configs:
        - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
          action: keep
          regex: default;kubernetes;https

# Supprimer et redeployer le configmap et le deployment
kubectl delete configmaps prometheus-server-conf -n=monitoring
kubectl create -f config-map.yaml

kubectl delete deployment prometheus-deployment -n monitoring
kubectl apply -f prometheus-deployment.yaml -n monitoring

# relancer le port forwarding du service grafana
kubectl port-forward --address 0.0.0.0 svc/grafana-dashboard -n monitoring 9000:9000

# relancer le port forwarding du service prometheus
kubectl port-forward --address 0.0.0.0 svc/prometheus-dashboard -n monitoring 8090:8090

# Création d'un dashboard Kubernetes apiserver
Numéro du dashboard
https://grafana.com/grafana/dashboards/12006
12006


# Mise en place des metriques k8s: State-metrics
- Déployez kube-state-metrics à l'aide de la documentation officielle en déployant l'ensemble des manifests présents
- Modifier le configmap prometheus afin d'y intégrer le endpoint de kube-state-metrics précédement déployé (n'hésitez pas à regarder la Doc)
- Vous devez supprimer et recréer le configmap ainsi que le deployment de prometheus pour  appliquer les modifications.
- Vérifiez sur l'interface de Prometheus que la target kube-state est bien présente et up
- Toujours sur l'interface de Prometheus vous pouvez vous assurer que la métrique kube_deployment_status_replicas renvoie un resultat
- Pour terminer, importer le dashboard permettant de visualiser les métriques de kube-state
- Vérifez que le dashboard nouvellement import affiche des données.

# Déploiement du k8s state-metrics
git clone https://github.com/kubernetes/kube-state-metrics.git
cd kube-state-metrics
cd examples/standard
kubectl apply -f .
kubectl apply -k .

# voir le service deployé
kubectl get svc -n kube-system

kubectl delete configmaps prometheus-server-conf -n=monitoring
kubectl create -f config-map.yaml

kubectl delete deployment prometheus-deployment -n monitoring
kubectl apply -f prometheus-deployment.yaml -n monitoring

# relancer le port forwarding du service grafana
kubectl port-forward --address 0.0.0.0 svc/grafana-dashboard -n monitoring 9000:9000

# relancer le port forwarding du service prometheus
kubectl port-forward --address 0.0.0.0 svc/prometheus-dashboard -n monitoring 8090:8090


# Création d'un dashboard kube-state-metrics
Numéro du dashboard
https://grafana.com/grafana/dashboards/13332
13332