# [#Gardener](https://gardener.cloud) 's logging stack is moving to [#Loki](https://grafana.com/oss/loki)

## About [Gardener](https://gardener.cloud/)
Gardener delivers fully managed Kubernetes clusters as a service and provides a fully validated extensibility framework that can be adjusted to any programmatic cloud or infrastructure provider.

Gardener's main principle is to **leverage Kubernetes concepts for all of its tasks**.

![](images/gardener-dashboard.gif)

In essence, Gardener is an [extension API server](https://kubernetes.io/docs/tasks/access-kubernetes-api/setup-extension-api-server) that comes along with a bundle of custom controllers. It introduces new API objects in an existing Kubernetes cluster (which is called **garden** cluster) in order to use them for the management of end-user Kubernetes clusters (called **shoot** clusters). These shoot clusters are described via [declarative cluster specifications](https://github.com/gardener/gardener/blob/master/example/90-shoot.yaml) which are observed by the controllers and their control planes are hosted on dedicated **seed** clusters. They will bring up the clusters, reconcile their state, perform automated updates and make sure they are always up and running.

![](images/gardener-architecture.png)

## Motivation
Today [Gardener](https://gardener.cloud/) already manages thousands of Kubernetes clusters around the globe for all of SAP’s development, quality as well internal and external production requirements. It also serves other [Community members](https://gardener.cloud/adopter/) ([PingCap](https://gardener.cloud/blog/2020-05/00/), [FI-TS](https://metal-stack.io/blog/01-why-metal-stack/), and more) as their multi-cloud orchestration tool of choice. [#Gardener](https://gardener.cloud) clusters come with a monitoring and logging stack for the control planes of these clusters. While the monitoring stack is straight-forward, the logging stack was more of a challenge as it had to come with these requirements:
1) Resource efficient (cpu, memory)
2) Secure and reliable
3) Multi-tenant (per cluster)
4) Dynamic vertical scaling

### Accessing the logs
[Gardener](https://gardener.cloud/) provides automatically created and configured logging stack for each cluster. The end user has access the logs from controlplane components like `kube-apiserver`, `kube-controller-manager` and `kube-scheduller` via automatically created Grafana dashboards.

## What we had until now
![](images/current-logging-architecture.png)
![](images/grafana-logs.png)

#### Limitations
1) Two middle layer components ([fluentd](https://www.fluentd.org/) and [elastic-search](https://www.elastic.co/elasticsearch/)) which make it difficult to trace an issue.
2) Elastic-search is a heavyweight Java application  for which performing dynamic vertical scaling is really hard.
3) Using [Grafana](https://grafana.com/) as monitoring UI and [Kibana](https://www.elastic.co/kibana) as logging UI is not optimal from usability point of view.

## Gardener's new logging stack architecture
![](images/new-logging-architecture.png)

#### Improvements:
1) [fluentd](https://www.fluentd.org/) is replaced by [Fluent-bit](https://fluentbit.io/) Golang plugin which dynamically serves the log messages to the right `Loki` instance.
2) It is really easy to scale `Loki` vertically
3) One harmonized monitoring and logging UI (`Grafana`), so that the metrics and the logs are in one place.
4) Requiring a lot less resources (cpu, memory) compared to the previous (EFFK) stack.
