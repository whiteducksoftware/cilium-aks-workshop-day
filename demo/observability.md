# Tetragon: Awareness and Observability

## Prerequisites

1. Create Managed Grafana and managed Prometheus and link it to your AKS Cluster
1. Assign Grafana managed identity `Monitoring Reader` on Sub level
1. Enable ContainerLogV2 & monitoring of kube-system namespace on your AKS cluster = `kubectl apply -f ../src/container-azm-ms-agentconfig.yaml`
1. Enable scraping of Tetragon metrics via Prometheus config = `kubectl apply -f ../src/ama-metrics-prometheus-config.yaml`
1. Verify that the Tetragon target is up = `kubectl port-forward pods/ama-metrics-5bb5b47b97-vtw2p 9090` & http://localhost:9090/targets

## View process executions

```Kusto
ContainerLogV2
| where PodNamespace == "kube-system"
| where PodName startswith "tetragon"
| where ContainerName startswith "export-stdout"
| where LogMessage has "process_exec"
| evaluate bag_unpack(LogMessage)
| evaluate bag_unpack(process_exec)
| evaluate bag_unpack(process)
| evaluate bag_unpack(pod)
| project TimeGenerated, name, namespace, arguments, binary
```

## View shell executions

```Kusto
ContainerLogV2
| where PodNamespace == "kube-system"
| where PodName startswith "tetragon"
| where ContainerName startswith "export-stdout"
| where LogMessage has "process_exec"
| evaluate bag_unpack(LogMessage)
| evaluate bag_unpack(process_exec)
| evaluate bag_unpack(process)
| evaluate bag_unpack(pod)
| where binary == '/bin/sh' or binary == '/bin/bash'
| project TimeGenerated, name, namespace, binary
```

## View Syscalls from Tracing policies

```Kusto
ContainerLogV2
| where PodNamespace == "kube-system"
| where PodName startswith "tetragon"
| where ContainerName startswith "export-stdout"
| where LogMessage has "process_kprobe"
| evaluate bag_unpack(LogMessage)
| evaluate bag_unpack(process_kprobe)
| evaluate bag_unpack(process)
| evaluate bag_unpack(pod)
| project TimeGenerated, name, namespace, action, function_name, arguments, binary
```

## Alert on SIGKILL

```Kusto
ContainerLogV2
| where PodNamespace == "kube-system"
| where PodName startswith "tetragon"
| where ContainerName startswith "export-stdout"
| project TimeGenerated, PodName, LogMessage
| where LogMessage has "process_kprobe"
| evaluate bag_unpack(LogMessage)
| evaluate bag_unpack(process_kprobe)
| evaluate bag_unpack(process)
| evaluate bag_unpack(pod)
| where action == "KPROBE_ACTION_SIGKILL"
| where namespace == "log4shell"
| summarize count() by binary, TimeGenerated
| project TimeGenerated, Count=count_
```

## Useful links

- [Tetragon documentation](https://tetragon.cilium.io/docs/overview/)
- [Tracing Policy examples](https://github.com/cilium/tetragon/tree/main/examples/tracingpolicy)
- [Tetragon announcement](https://isovalent.com/blog/post/2022-05-16-tetragon/)