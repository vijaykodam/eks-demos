## Install Helm wget https://get.helm.sh/helm-v3.17.0-linux-amd64.tar.gz

```
tar -zxvf helm-v3.17.0-linux-amd64.tar.gz
sudo mv linux-amd64/helm /usr/local/bin/helm
helm version
```

## Prerequisites for prometheus. Must be installed before Grafana

```
kubectl create namespace prometheus
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm install prometheus prometheus-community/prometheus \
    --namespace prometheus \
    --set alertmanager.persistentVolume.storageClass="gp2" \
    --set server.persistentVolume.storageClass="gp2"

```

## Set up Grafana Helm repository

```
helm repo add grafana https://grafana.github.io/helm-charts
helm repo list

#To download latest grafana helm chart
helm repo update
```

# Deploy grafana helm chart

```
cat << EoF > grafana-values.yaml
persistence:
  type: pvc
  enabled: true
  storageClassName: gp2
  size: 1Gi
datasources:
  datasources.yaml:
    apiVersion: 1
    datasources:
    - name: Prometheus
      type: prometheus
      url: http://prometheus-server.prometheus.svc.cluster.local
      access: proxy
      isDefault: true
EoF

kubectl create namespace grafana

helm install grafana grafana/grafana \
    --namespace grafana \
    --values grafana-values.yaml

```

After grafana is up and running run below commands to get the admin password and port-forward to open Grafana dashboard locally on your computer.
 
```
kubectl get secret --namespace monitoring my-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

export POD_NAME=$(kubectl get pods --namespace monitoring -l "app.kubernetes.io/name=grafana,app.kubernetes.io/instance=my-grafana" -o jsonpath="{.items[0].metadata.name}")
kubectl --namespace monitoring port-forward $POD_NAME 3000
```


Open the Grafana dashboard locally at localhost:3000

## Cluster Monitoring Dashboard

For creating a dashboard to monitor the cluster:

- Click '+' button on left panel and select ‘Import’.
- Enter 3119 dashboard id under Grafana.com Dashboard.
- Click ‘Load’.
- Select ‘Prometheus’ as the endpoint under prometheus data sources drop down.
- Click ‘Import’.

This will show monitoring dashboard for all cluster nodes

