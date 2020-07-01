# [#Gardener](https://gardener.cloud/) 's logging stack is moving to [#Loki](https://grafana.com/oss/loki/)

## About [Gardener](https://gardener.cloud/)
Gardener delivers fully-managed Kubernetes clusters as a service and provides a fully validated extensibility framework that can be adjusted to any programmatic cloud or infrastructure provider.

Gardener's main principle is to **leverage Kubernetes concepts for all of its tasks**.

In essence, Gardener is an [extension API server](https://kubernetes.io/docs/tasks/access-kubernetes-api/setup-extension-api-server/) that comes along with a bundle of custom controllers. It introduces new API objects in an existing Kubernetes cluster (which is called **garden** cluster) in order to use them for the management of end-user Kubernetes clusters (which are called **shoot** clusters). These shoot clusters are described via [declarative cluster specifications](https://github.com/gardener/gardener/blob/master/example/90-shoot.yaml) which are observed by the controllers and their controlplanes are hosted on dedicated **seed** clusters. They will bring up the clusters, reconcile their state, perform automated updates and make sure they are always up and running.

![](images/gardener-architecture.png)

## Motivation behind the integration of Loki
As of today [#Gardener](https://gardener.cloud/) manages `1461` Kubernetes clusters for one of the SAP Landscapes and this number is expected to explode in the future. [#Gardener](https://gardener.cloud/) should provides a Logging stack for the `controlplanes` for all these clusters. For this purpose we had to design a logging stack which meets the following requirements:
1) Resources efficiency (CPU, Memory)
2) Secure and Reliable
3) Dynamic vertical scaling


## What we had until now
![](images/old-logging-architecture.png)

#### Limitations:
1) Two middle layer components ([fluentd](https://www.fluentd.org/) and [Elasticsearch](https://www.elastic.co/elasticsearch/)) which makes it difficult to trace an issue.
2) Elasticsearch is a heavyweight Java application where the dynamic vertical scaling is really hard.
3) Using [Grafana](https://grafana.com/) as monitoring UI and [Kibana](https://www.elastic.co/kibana) as logging UI is not optimal from usability point of view. 

## Gardener's new logging stack architecture
![](images/new-logging-architecture.png)

#### Improvements:
1) Getting rid of the [fluentd](https://www.fluentd.org/). It is replaced by [Fluent-bit](https://fluentbit.io/) Golang plugin which dynamically serves the log messages to the right `Loki` instance.
2) It is really easy to scale `Loki` vertically
3) One monitoring UI (`Grafana`), so that the logs and the metrics are in one place.
4) Requiring much less resources (cpu, memory) compared to the previous stack.
