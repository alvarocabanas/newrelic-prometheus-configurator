# Get verbose logs

The `newrelic-prometheus` chart verbose logs can be enable setting the `verboseLog` or `global.verboseLog` to `true`.

Once this has been updated on the values files the helm upgrade command can be executed:
```bash
helm upgrade <RELEASE_NAME> newrelic-prometheus-configurator/newrelic-prometheus \
--namespace <NEWRELIC_NAMESPACE> \
-f values-newrelic.yaml \
[--version fixed-chart-version]
```

# Not seeing metrics for a target

There could be different reasons why a target is not being scraped.
Take into account that there should exist at least one `job` that is discovering the target in Kubernetes and matches the filter specified or the target should be listed as `static_target`.
If using the default configuration on Kubernetes check that your Pod or Service has the **annotation** `prometheus.io/scrape=true`.

## Check `up` metric status

Every target scrape will generate the `up` metric with the all target metrics. If this metrics will have value `1` if scraped was successful and `0` if not.

```SQL
FROM Metric SELECT latest(up) WHERE cluster_name= '<your-cluster-name>' AND pod = '<target-pod-name>' TIMESERIES
```

If this metric doesn't exist for the target , it could being dropped.

If the values is `0` the scrape have failed.

### Target dropped by filter rules
To check dropped targets the Prometheus API [Targets endpoint](https://prometheus.io/docs/prometheus/latest/querying/api/#targets) can be used.

Following command will list in `json` format all dropped targets, and show all discovered labels:
```bash
kubectl exec newrelic-prometheus-0 -- wget -O - 'localhost:9090/api/v1/targets?state=dropped' 2>/dev/null
```

### Target scrape fail

If the `up` metrics has value `0` means that the target is actively scraped by Prometheus but the scrape has failed. The reason can be found in the verbose logs with a  log entry similar to this:
```log
 prometheus ts=2022-09-19T15:21:05.489Z caller=scrape.go:1332 level=debug component="scrape manager" scrape_pool=kubernetes-job-pod target=http://172.17.0.5:80/metrics msg="<error>" err="<error detail>"
```

Another option is to check the Active target list from the Prometheus API [Targets endpoint](https://prometheus.io/docs/prometheus/latest/querying/api/#targets).

Following command will list in `json` format all active targets, and show all discovered labels:
```bash
kubectl exec newrelic-prometheus-0 -- wget -O - 'localhost:9090/api/v1/targets?state=active' 2>/dev/null
```

The failed target will be listed and the error will be available in the `lastError` field in a result similar to this:
```json
{
    "status": "success",
    "data": {
        "activeTargets": [
            {
                "discoveredLabels": <map of labels>,
                "labels": <map of labels>,
                "scrapePool": "kubernetes-job-pod",
                "scrapeUrl": "http://172.17.0.5:80/metrics",
                "globalUrl": "http://172.17.0.5:80/metrics",
                "lastError": <error detail>,
                "lastScrape": "2022-09-19T14:19:20.543747971Z",
                "lastScrapeDuration": 0.000372542,
                "health": "down",
                "scrapeInterval": "15s",
                "scrapeTimeout": "10s"
            },
            ...
        ]
    }
}
```



