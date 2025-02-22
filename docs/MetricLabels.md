# Metric Labels

There are a few sources of Metrics labels (AKA metadata or attributes). Below there is a table that maps these sources and when the labels are added:

All targets:
| Label | Description | Example
|:--|:--|:--|
| `prometheus_server` | Name of the Prometheus instance that scraped the metric. When installed in Kubernetes the name of the Pod is assigned. | `prometheus_server: newrelic-prometheus-0`
| `instance` | Host and Port of the scraped target. | `instance: 172.17.0.5:8083` |
| `job` | Name of the scrape job that discovered the target.| `job: kubernetes-job-pod` |
| `instrumentation.name` | New Relic instrumentation name. | `instrumentation.name: remote-write` |
| `instrumentation.provider` | New Relic instrumentation provider name. | `instrumentation.provider: prometheus` |
| `instrumentation.source` | Will match the name in `prometheus_server`. | `instrumentation.source: newrelic-prometheus-0` |
| `instrumentation.version` | New Relic remote-write endpoint API version. | `instrumentation.version: 0.0.2` |
| `newrelic.source` | New Relic instrumentation source name. | `newrelic.source: prometheusAPI` |

Kubernetes targets:
| Kind | Label | Description | Example
|:--|:--|:--|:--|
| Pod/Endpoints | `cluster_name` | Name of the cluster defined on installation. | `cluster_name: my-cluster` |
| Pod/Endpoints | `namespace` | Namespace of target. | `namespace: default` |
| Pod/Endpoints | `pod` | Pod name. | `pod: my-pod` |
| Pod/Endpoints | `node` | Node name where the Pod/Endpoints is running. | `node: minikube` |
| Endpoints | `service` | Service name of the Endpoints. | `service: my-service` |
| Endpoints | `<service labels>` | Kubernetes labels of the Endpoints Service. | `k8s_io_service: MyService` |
| Pod | `<pod labels>` | Kubernetes labels of the Pod. | `k8s_io_app: MyApp` |

Note: To comply with [Prometheus DataModel](https://prometheus.io/docs/concepts/data_model/#metric-names-and-labels) Kubernetes labels names are sanitized after scraped to replace any non allowed character to `_`. For example the Kubernetes label `k8s.io/app` is added as `k8s_io_app`. Labels values remain untouched. 

# Generated metrics
The following metrics are generated per each scraped target:

| Metric name | Description |
|:--|:--|:--|
| `up` | The value will be `0` if the scrape has failed otherwise will be `1`. | 
| `scrape_duration_seconds` | Duration of the scrape. |
| `scrape_samples_scraped` | The number of samples the target exposed. |
| `scrape_samples_post_metric_relabeling` | The number of samples remaining after metric relabeling was applied. |
| `scrape_series_added` | The approximate number of new series in this scrape. |

These metrics will contain all metric labels related to the target mentioned [above](#metric-labels).
