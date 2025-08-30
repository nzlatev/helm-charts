# Hekm Charts url-metrics-checker

This includes:
•	What it does
•	How to install
•	Configuration
•	Example usage
•	Prometheus integration

# url-metrics-checker

A lightweight Prometheus exporter that monitors external URLs (e.g., APIs, websites) and exposes their availability and response time as metrics.

Useful for black-box monitoring of services **outside your Kubernetes cluster**.

---

## Metrics Exported

| Metric                              | Description                                      |
|-------------------------------------|--------------------------------------------------|
| `sample_external_url_up`            | 1 if URL is up (HTTP 200), 0 otherwise           |
| `sample_external_url_response_ms`   | Response time in milliseconds                    |

Example output at `/metrics`:

sample_external_url_up{url="https://httpstat.us/200"} 1
sample_external_url_response_ms{url="https://httpstat.us/200"} 108.4

## URLs Monitored (Default)

- `https://httpstat.us/200`
- `https://httpstat.us/503`

You can customize the list by modifying the Python source (or extend the Helm chart to support config maps or env vars).

Installation via Helm

### Prerequisites

- Kubernetes cluster
- [Helm 3](https://helm.sh/docs/intro/install/)
- Optional: [Prometheus Operator](https://github.com/prometheus-operator/prometheus-operator)

### Add Helm Repo

It hosted on GitHub Pages:

```bash
helm repo add url-metrics-checker https://nzlatev.github.io/helm-charts
helm repo update

Install Chart

Create namespace under K8s cluster
kubectl -n checker

Run helm as the following:

helm install url-metrics-checker url-metrics-checker/url-metrics-checker -n checker

Or install directly from a local folder:

wget https://nzlatev.github.io/helm-charts/url-metrics-checker-0.1.0.tgz
tar xvfz url-metrics-checker-0.1.0.tgz
helm install url-metrics-checker ./url-metrics-checker -n checker

Configuration
You can override default values with --set or a custom values.yaml.
Example:
replicaCount: 1

image:
  repository: yourusername/url-metrics-checker
  tag: 0.1.0

service:
  type: ClusterIP
  port: 8000

prometheus:
  enabled: false   #Optional set to "true" if you're using the Prometheus Operator
  interval: 30s

Prometheus Integration - Optional

If you're using the Prometheus Operator, and prometheus.enabled: true in values:
A ServiceMonitor will be created automatically to let Prometheus scrape the /metrics endpoint.
Make sure the label release: prometheus matches your Prometheus install.

Security Notes
•	URLs are hardcoded by default — if you need secret handling, consider integrating with ConfigMaps or sealed secrets.
•	HTTPS requests are made without cert pinning — add cert validation if monitoring sensitive endpoints.

Docker Image

Docker image used by this chart:
nzlatev/url-metrics-checker:v1.0

Docker Hub › url-metrics-checker
https://hub.docker.com/repository/docker/nzlatev/url-metrics-checker/general

Development

To build the Docker image:
docker build -t nzlatev /url-metrics-checker .
docker push nzlatev /url-metrics-checker

To test the Helm chart locally:
helm install test ./url-metrics-checker

# Test the Setup

Confirm the pod is running:

kubectl get pods -n checker

or via Helm

$ helm list -n checker
NAME                 NAMESPACE   REVISION  UPDATED                                     STATUS       CHART                           APP VERSION
url-metrics-checker  checker      1        2025-08-29 19:04:57.757691389 +0300 EEST    deployed     url-metrics-checker-0.1.0       1.0


Port-forward to access and test metrics locally:

kubectl port-forward service/url-metrics-service 8000:8000

Then visit: http://localhost:8000/metrics

You should see the Prometheus-compatible metrics.

---
# TYPE sample_external_url_up gauge
sample_external_url_up{url="https://httpstat.us/200"} 0.0
sample_external_url_up{url="https://httpbin.org/status/503"} 0.0
# HELP sample_external_url_response_ms Response time of the external URL in milliseconds
# TYPE sample_external_url_response_ms gauge
sample_external_url_response_ms{url="https://httpstat.us/200"} 0.0
sample_external_url_response_ms{url="https://httpbin.org/status/503"} 4418.2257652282715



License
MIT © 2025 Nikolay Zlatev

Questions or Contributions?
•	File an issue or PR on GitHub
•	Contributions and improvements are welcome!


