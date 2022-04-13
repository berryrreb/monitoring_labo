# Monitoring Workshop

Monitoring repo for DOU/TechM University class.

## Prerequisites

+ Docker desktop · [Download](https://hub.docker.com/editions/community/docker-ce-desktop-mac)
    * Minikube (Only if not installed with Docker Desktop) · [Installation](https://minikube.sigs.k8s.io/docs/start/)
+ Visual Studio Code (Recommended) · [Download](https://code.visualstudio.com/download)
+ Helm · [Install](https://helm.sh/docs/intro/install/)

## Start K8s minikube environment

Delete old and start new minikube.

```bash
minikube delete && minikube start --cpus='6' --memory=6g
kubectl cluster-info
kubectl get pods -A
```

## Prometheus + Grafana Operator

Clone Prometheus + Grafana Operator repo.

```bash
git clone https://github.com/prometheus-operator/kube-prometheus.git
cd kube-prometheus
```

Create namespace and Custom Resource Definitions.

```bash
kubectl create -f manifests/setup
kubectl get ns
```

Create Prometheus + Grafana Operator Stack.

```bash
kubectl create -f manifests/
kubectl get pods -n monitoring -w
kubectl get svc -n monitoring
```

## Expose and accessing services

Expose Prometheus.

```bash
kubectl --namespace monitoring port-forward svc/prometheus-k8s 9090
```

Access Prometheus going to [localhost:9090](localhost:9090).

Query examples for prometheus:

```bash
sum((rate(container_cpu_usage_seconds_total{container!="POD",container!=""}[30m]) - on (namespace,pod,container) group_left avg by (namespace,pod,container)(kube_pod_container_resource_requests{resource="cpu"})) * -1 >0) # How many CPU cores are underutilized

sum((container_memory_usage_bytes{container!="POD",container!=""} - on (namespace,pod,container) avg by (namespace,pod,container)(kube_pod_container_resource_requests{resource="memory"})) * -1 >0 ) / (1024*1024*1024) # How much requested memory is underutilized
```

Close 'port-forward' for prometheus.

```bash
control + c
```

Expose Grafana.

```bash
kubectl --namespace monitoring port-forward svc/grafana 3000
```

Access Grafana going to [localhost:3000](localhost:3000).

Default Logins are:

```yml
Username: admin
Password: admin
```

Navigate to see metrics and dashboards

## Clean Environment

Delete Prometheus + Grafana Operator Stack.

```bash
kubectl delete --ignore-not-found=true -f manifests/ -f manifests/setup
```

Delete Minikube

```bash
minikube delete
```

## EFK Stak

Elasticsearch + Fluentd + Kibana Stack to centralize logs.

Create Namespace.

```bash
kubectl create -f efk/kube-log.yaml
```

Create Elasticsearch StatefulSet

```bash
kubectl create -f efk/elasticsearch_statefulset.yaml
kubectl rollout status sts/es-cluster --namespace=kube-logging
```

Create Service.

```bash
kubectl create -f efk/elasticsearch_svc.yaml
kubectl get services --namespace=kube-logging
```

Confirm elasticsearch is up and running.

```bash
kubectl port-forward es-cluster-0 9200:9200 --namespace=kube-logging
```

In another terminal, test the elactisearch service.

```bash
curl http://localhost:9200/_cluster/state?pretty
```

> Stop port-forwarding for elasticsearch.

Create Kibana Deployment and Service.

```bash
kubectl create -f efk/kibana.yaml
```

List pods

```bash
kubectl get pods -n kube-logging
```

Expose Kibana Service with port-forwarding

```bash
kubectl port-forward service/kibana 5601:5601 --namespace=kube-logging
```

Create fluentd Daemonset

```bash
kubectl create -f efk/fluentd.yaml
```

Access Kibana over [localhost:5601](localhost:5601).

On ***Discover*** tab:

Index pattern: 'logstash*'

Click: 'Next step'

Time Filter field name: @timestamp

Click: 'Create index pattern'

Click '***Discover***' tab again.

You can see the complete logs of your Kubernetes cluster.

## Test your container logging

Start counter pod

```bash
kubectl create -f efk/counter.yaml
```

Follow logs from counter pod on terminal

```bash
kubectl logs -n default counter -f
```

### See logs in kibana

Over the ***Discover*** tab:

On the search field: 'kubernetes.pod_name:counter'

You can see the counter pod logs in Kibana.

## Work for home

Follow this tutorial to deploy ELK Stack and Metricbeat with HELM.

[Deploying the ELK Stack on Kubernetes with Helm
](https://logz.io/blog/deploying-the-elk-stack-on-kubernetes-with-helm/)