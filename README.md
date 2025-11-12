# Collect Metric from exporters using the managed service for prometheus 
In this project, I will manage the Prometheus service to collect metrics on nodes in GKE.

## How i do run this project?

1. Deploy GKE Cluster and Get credentials for cluster
```bash
gcloud beta container clusters create gmp-cluster --num-nodes=1 --zone Zone --enable-managed-prometheus
```
```bash
gcloud container clusters get-credentials gmp-cluster --zone=Zone
```

2. Set up a namespace 
```bash
kubectl create ns cluster-1
```

3. Deploy example app from google cloud 
```bash
kubectl -n cluster-1 apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/prometheus-engine/v0.2.3/examples/example-app.yaml
```

4. Configure Pod Monitoring Resource, Target scraping and metric ingestion are configured using kubernetes custom resource.The managed service use `PodMonitoring` custom resource (CRs). To apply this resource run following command
```bash
kubectl -n cluster-1 apply -f https://raw.githubusercontent.com/MochamadAbdulRouf/gke-managed-prometheus-exporter-metrics/refs/heads/main/pod-monitoring.yaml
```

5. Download the prometheus binary and run following command
```bash
git clone https://github.com/GoogleCloudPlatform/prometheus && cd prometheus
```
```bash
git checkout v2.28.1-gmp.4
```
```bash
wget https://storage.googleapis.com/kochasoft/gsp1026/prometheus
```
```bash
chmod a+x prometheus
```

6. Run the Prometheus Binary, Run this command
- Save your project to a variable 
```bash
export PROJECT_ID=$(gcloud config get-value project)
```
- Save your zone to a variable
```bash
export ZONE=Zone
```
- Run the prometheus binary on cloud shell
```bash
./prometheus --config.file=documentation/examples/prometheus.yaml --export.label.project-id=$PROJECT_ID --export.label.location=$ZONE
```

7. Download and run the node exporter, Run following command here
```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz
```
```bash
 tar xvfz node_exporter-1.3.1.linux-amd64.tar.gz
```
```bash
cd node_exporter-1.3.1.linux-amd64
```
```bash
 ./node_exporter
```

8. Create a configuration file to retrieve metrics from the exporter node.
```bash
vi config.yaml
```
- Create `config.yaml` file with the following spec:
```bash
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: node
    static_configs:
      - targets: ['localhost:9100']
```
- Or you can use my config file
```bash
curl -L -O https://raw.githubusercontent.com/MochamadAbdulRouf/gke-managed-prometheus-exporter-metrics/refs/heads/main/config.yaml
```

9. Upload the `config.yaml` file, Run following command
```bash
export PROJECT=$(gcloud config get-value project)
```
```bash
gsutil mb -p $PROJECT gs://$PROJECT
```
```bash
gsutil cp config.yaml gs://$PROJECT
```
```bash
gsutil -m acl set -R -a public-read gs://$PROJECT
```

10. Run again promotheus pointing to new configuration file, Run this command
```bash
./prometheus --config.file=config.yaml --export.label.project-id=$PROJECT --export.label.location=$ZONE
```

11. In cloud shell, Click web preview icon, Set the port to 9090.Write Query `node_cpu_seconds_total` in the PromQL query editor and select graphical editor.

note: Try selecting other metrics that appear to view the data exported.

