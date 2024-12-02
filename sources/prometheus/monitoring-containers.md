## Monitoring des Conteneurs: cAdvisor

cAdvisor (Container Advisor) est un outil développé par Google pour analyser et exposer les métriques de ressources et de performance des conteneurs en cours d'exécution.
Il fonctionne comme un démon qui collecte, agrège, traite et exporte des informations sur les conteneurs

## Principales fonctionnalités de cAdvisor:
- Collecte de données en temps réel : cAdvisor collecte des données sur l'utilisation des ressources (CPU, mémoire, disque, réseau) des conteneurs en cours d'exécution.

- Visualisation des métriques : Il fournit une interface web pour visualiser les métriques collectées.

- Intégration avec Prometheus : cAdvisor expose des métriques Prometheus, ce qui permet une intégration facile avec des outils de surveillance comme Prometheus.

- Support de Docker : Il est nativement compatible avec Docker et peut également fonctionner avec d'autres types de conteneurs

# Dashboard Monitoring Architectural Flow
Docker containers --> cAdvisor --> Prometheus --> Grafana Dashboard


# Visualisation des métriques docker

- Modifier le configmap prometheus afin d'y intégrer le endpoint de kubernetes-cadvisor ( regarder la doc)
- Vous devez supprimer et recréer le configmap aisin que le deployment de prometheus pour appliquer les modifications
- Verifier sur l'interface de prometheus que la target kubernetes-cadvisor est bien présente et up
- Vous pouvez maintenant visualiser les métriques docker sur l'interface de prometheus
- Vérifiez que le dashboard nouvellement importé affiche des données


# ./lab-7/config-map.yaml

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
      - job_name: 'kube-state-metrics'
        static_configs:
          - targets: ['kube-state-metrics.kube-system.svc:8080']
      - job_name: 'kubernetes-cadvisor'
        scheme: https
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        kubernetes_sd_configs:
        - role: node
        relabel_configs:
        - action: labelmap
          regex: __meta_kubernetes_node_label_(.+)
        - target_label: __address__
          replacement: kubernetes.default.svc:443
        - source_labels: [__meta_kubernetes_node_name]
          regex: (.+)
          target_label: __metrics_path__
          replacement: /api/v1/nodes/${1}/proxy/metrics/cadvisor


# Supprimer et redeployer le configmap et le deployment
kubectl delete configmaps prometheus-server-conf -n=monitoring
kubectl create -f config-map.yaml

kubectl delete deployment prometheus-deployment -n monitoring
kubectl apply -f prometheus-deployment.yaml -n monitoring

# relancer le port forwarding du service grafana
kubectl port-forward --address 0.0.0.0 svc/grafana-dashboard -n monitoring 9000:9000

# relancer le port forwarding du service prometheus
kubectl port-forward --address 0.0.0.0 svc/prometheus-dashboard -n monitoring 8090:8090

# Création d'un dashboard Kubernetes cAdvisor
Numéro du dashboard
https://grafana.com/grafana/dashboards/12206
12206