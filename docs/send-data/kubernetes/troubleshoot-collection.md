---
id: troubleshoot-collection
title: Troubleshooting Collection
sidebar_label: Troubleshooting Collection
description: Troubleshooting Collection
---

## Troubleshooting installation

### Installation fails with error `function "dig" not defined`

You'll need to use a more recent version of Helm. See [Minimum Requirements](https://github.com/SumoLogic/sumologic-kubernetes-collection/blob/main/docs/README.md#minimum-requirements).

If you are using ArgoCD or another tool that uses Helm under the hood, make sure that tool uses the required version of Helm.

### Sumo Logic fields

Sumo Logic Apps for Kubernetes and Explore require below listed fields to be added in Sumo Logic UI to your Fields table schema.

- `cluster`
- `container`
- `daemonset`
- `deployment`
- `host`
- `namespace`
- `node`
- `pod`
- `service`
- `statefulset`

This is normally done in the setup job when `sumologic.setupEnabled` is set to `true` (default behavior).

In an unlikely scenario that this fails, create them manually by visiting [Fields#Manage_fields](/docs/manage/fields/#manage-fields) in Sumo Logic UI.

This is to ensure your logs are tagged with relevant metadata.

This is a one-time setup per Sumo Logic account.

### Error: timed out waiting for the condition

If `helm upgrade --install` hangs, it usually means the pre-install setup job is failing and is in a retry loop. Due to a Helm limitation, errors from the setup job cannot be fed back to the `helm upgrade --install` command. Kubernetes schedules the job in a pod, so you can look at logs from the pod to see why the job is failing. First find the pod name in the namespace where the Helm chart was deployed. The pod name will contain `-setup` in the name.

```sh
kubectl get pods
```

:::tip
If the pod does not exist, it is possible it has been evicted. Re-run the `helm upgrade --install` to recreate it and while that command is running, use another shell to get the name of the pod.
:::

Get the logs from that pod:

```sh
kubectl logs POD_NAME -f
```

#### Error: collector with name 'sumologic' does not exist

If you get:

```sh
Error: collector with name 'sumologic' does not exist
sumologic_http_source.default_metrics_source: Importing from ID
```

...you can safely ignore it, and the installation should complete successfully. The installation process creates new [HTTP endpoints](/docs/send-data/hosted-collectors/http-source) in your Sumo Logic account, that are used to send data to Sumo. This error occurs if the endpoints had already been created by an earlier run of the installation process.

#### Secret 'sumologic::sumologic' exists, abort

If you see `Secret 'sumologic::sumologic' exists, abort.` from the logs, delete the existing secret:

```bash
kubectl delete secret sumologic -n ${NAMESPACE}
```

`helm install` should proceed after the existing secret is deleted before exhausting retries. If it did time out after exhausting retries,
rerun the `helm install` command.

### OpenTelemetry Collector Pods Stuck in CreateContainerConfigError

If the OpenTelemetry Collector Pods are in `CreateContainerConfigError` it can mean the setup job has not been completed yet. Make sure that
the `sumologic.setupEnable` parameter is set to `true`. Then wait for the setup pod to complete and the issue should resolve itself. The
setup job creates a secret and the error simply means the secret is not there yet. This usually resolves itself automatically.

If the issue does not solve resolve automatically, you will need to look at the logs for the setup pod. Kubernetes schedules the job in a pod, so you can look at logs from the pod to see why the job is failing. First find the pod name in the namespace where you installed the rendered YAML. The pod name will contain `-setup` in the name.

```sh
kubectl get pods
```

Get the logs from that pod:

```sh
kubectl logs POD_NAME -f
```

## Namespace configuration

The following `kubectl` commands assume you are in the correct namespace `sumologic`. By default, these commands will use the namespace
`default`.

To run a single command in the `sumologic` namespace, pass in the flag `-n sumologic`.

To set your namespace context more permanently, you can run

```sh
kubectl config set-context $(kubectl config current-context) --namespace=sumologic
```

## Collecting logs

If you cannot see logs in Sumo that you expect to be there, here are the things to check.

### Check log throttling

Check if [log throttling](/docs/manage/ingestion-volume/log-ingestion#log-throttling) is happening.

If it is, there will be messages like `HTTP ERROR 429 You have temporarily exceeded your Sumo Logic quota` in OpenTelemetry Collector logs.

### Check ingest budget limits

Check if an [ingest budget](/docs/manage/ingestion-volume/ingest-budgets) limit is hit.

If it is, there will be `budget.exceeded` messages from Sumo in OpenTelemetry Collector logs, similar to the following:

```console
2022-04-12 13:47:17 +0000 [warn]: #0 There was an issue sending data: id: KMZJI-FCDPN-4KHKD, code: budget.exceeded, status: 200, message: Message(s) in the request dropped due to exceeded budget.
```

### Check if collection pods are in a healthy state

Run:

```sh
kubectl get pods
```

to get a list of running pods. If any of them are not in the `Status: running` state, something is wrong. To get the logs for that pod, you can either:

Stream the logs to `stdout`:

```sh
kubectl logs POD_NAME -f
```

Or write the current logs to a file:

```sh
kubectl logs POD_NAME > pod_name.log
```

To get a snapshot of the current state of the pod, you can run

```sh
kubectl describe pods POD_NAME
```

### Prometheus Logs

To view Prometheus logs:

```sh
kubectl -n "${NAMESPACE}" logs -l app.kubernetes.io/name=prometheus --container prometheus -f
```

Where `collection` is the `helm` release name.

### OpenTelemetry Logs Collector is being CPU throttled

If OpenTelemetry Logs Collector is being throttled, you should increase CPU request to higher value, for example:

```yaml
otellogs:
  daemonset:
    resources:
      requests:
        cpu: 2
      limits:
        cpu: 5
```

If this situation affects only specific group of nodes, you can change resource configuration only for them:

```yaml
otellogs:
  additionalDaemonSets:
    ## intense will be suffix for daemonset for easier recognition
    intense:
      nodeSelector:
        ## we are using nodeSelector to select only nodes with `workingGroup` label set to `IntenseLogGeneration`
        workingGroup: IntenseLogGeneration
      resources:
        requests:
          cpu: 1
        limits:
          cpu: 10
  daemonset:
    # For main daemonset, we need to set nodeAffinity to not schedule on nodes with `workingGroup` label set to `IntenseLogGeneration`
    affinity:
      nodeAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
            - matchExpressions:
                - key: workingGroup
                  operator: NotIn
                  values:
                    - IntenseLogGeneration
```

For more information, see [Setting different resources on different nodes for logs collector](/docs/send-data/kubernetes/best-practices/#setting-different-resources-on-different-nodes-for-logs-collector).

## Collecting metrics

### Check the `/metrics` endpoint

You can `port-forward` to a pod exposing `/metrics` endpoint and verify it is exposing Prometheus metrics:

```sh
kubectl port-forward collection-sumologic-xxxxxxxxx-xxxxx 8080:24231
```

Then, in your browser, go to `http://localhost:8080/metrics`. You should see Prometheus metrics exposed.

#### Check the `/metrics` endpoint for Kubernetes services

For kubernetes services you can use the following way:

1. Create `sumologic-debug` pod.
   ```yml
   cat << EOF | kubectl apply -f -
   apiVersion: v1
   kind: Pod
   metadata:
     name: sumologic-debug
     namespace: <namespace you want to create pod in (e.g. sumologic)>
   spec:
     containers:
     - args:
       - receiver-mock
       image: sumologic/kubernetes-tools:2.9.0
       imagePullPolicy: IfNotPresent
       name: debug
     serviceAccountName: <service account name used by prometheus (e.g. collection-kube-prometheus-prometheus)>
   EOF
   ```
2. Go into the container:
   ```bash
   kubectl exec -it sumologic-debug -n <namespace> bash
   ```
3. Talk with API directly like prometheus does, for example:
   ```bash
   curl https://10.0.2.15:10250/metrics/cadvisor --insecure --cacert /var/run/secrets kubernetes.io/serviceaccount/ca.crt -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)"
   ```

### Check the Prometheus UI

First run the following command to expose the Prometheus UI:

```sh
$ kubectl -n "${NAMESPACE}" get pod -l app.kubernetes.io/name=prometheus
NAME                                                 READY   STATUS    RESTARTS   AGE
prometheus-collection-kube-prometheus-prometheus-0   2/2     Running   0          13m
$ kubectl -n "${NAMESPACE}" port-forward prometheus-collection-kube-prometheus-prometheus-0 8080:9090
Forwarding from 127.0.0.1:8080 -> 9090
Forwarding from [::1]:8080 -> 9090
```

Then, in your browser, go to `localhost:8080`. You should be in the Prometheus UI now.

From here you can start typing the expected name of a metric to see if Prometheus auto-completes the entry.

If you can't find the expected metrics, ensure that prometheus configuration is correct and up to date. In the top menu, navigate to section `Status > Configuration` or go to the `http://localhost:8080/config`. Review the configuration.

Next, you can check if Prometheus is successfully scraping the `/metrics` endpoints. In the top menu, navigate to section `Status > Targets` or go to the `http://localhost:8080/targets`. Check if any targets are down or have errors.

### Check Scrape Configs for Open Telemetry Operator

First expose the target allocator on local port 9090:

```sh
kubectl port-forward -n sumologic services/collection-sumologic-metrics-targetallocator 9090:80
```

Check if expected Service Monitor is on the list:

```sh
> curl http://localhost:9090/scrape_configs -s | jq '. | keys'
[
  "cadvisor",
  "kubelet",
  "pod-annotations",
  "serviceMonitor/sumologic/avalanche/0",
  "serviceMonitor/sumologic/collection-kube-prometheus-apiserver/0",
  "serviceMonitor/sumologic/collection-kube-prometheus-coredns/0",
  "serviceMonitor/sumologic/collection-kube-prometheus-kube-controller-manager/0",
  "serviceMonitor/sumologic/collection-kube-prometheus-kube-etcd/0",
  "serviceMonitor/sumologic/collection-kube-prometheus-kube-proxy/0",
  "serviceMonitor/sumologic/collection-kube-prometheus-kube-scheduler/0",
  "serviceMonitor/sumologic/collection-kube-prometheus-kubelet/0",
  "serviceMonitor/sumologic/collection-kube-prometheus-kubelet/1",
  "serviceMonitor/sumologic/collection-kube-state-metrics/0",
  "serviceMonitor/sumologic/collection-prometheus-node-exporter/0",
  "serviceMonitor/sumologic/collection-sumologic-metrics-collector/0",
  "serviceMonitor/sumologic/collection-sumologic-otelcol-events/0",
  "serviceMonitor/sumologic/collection-sumologic-otelcol-logs-collector/0",
  "serviceMonitor/sumologic/collection-sumologic-otelcol-logs/0",
  "serviceMonitor/sumologic/collection-sumologic-otelcol-metrics/0",
  "serviceMonitor/sumologic/collection-sumologic-otelcol-traces/0",
  "serviceMonitor/sumologic/collection-sumologic-prometheus/0",
  "serviceMonitor/sumologic/receiver-mock/0"
]
```

Now, you can list all target associated with this Service Monitor. Built URL using the following template: `http://localhost:9090/jobs/<HTML encoded service monitor name>/targets`.

Let's assume that you are looking for `serviceMonitor/sumologic/receiver-mock/0`. In that case, you need to check the following endpoint:
`http://localhost:9090/jobs/serviceMonitor%2Fsumologic%2Fcollection-sumologic-otelcol-logs%2F0/targets`

```sh
> curl -s 'http://localhost:9090/jobs/serviceMonitor%2Fsumologic%2Fcollection-sumologic-otelcol-logs%2F0/targets' | jq '.[].targets[].labels |.__meta_kubernetes_namespace + "/" + .__meta_kubernetes_pod_name'
"sumologic/collection-sumologic-otelcol-logs-2"
"sumologic/collection-sumologic-otelcol-logs-1"
"sumologic/collection-sumologic-otelcol-logs-0"
```

If all information are correct, refer to the following sections to continue investigation:

- [Check the `/metrics` endpoint](#check-the-metrics-endpoint)
- [Check the `/metrics` endpoint for Kubernetes services](#check-the-metrics-endpoint-for-kubernetes-services)

### Print metrics on stdout for OTLP source

In order to print metrics and their labels on stdout, the following configuration has to be applied:

```yaml
otellogs:
  config:
    merge:
      receivers:
        filelog/containers:
          exclude:
            - /var/log/pods/*sumologic-otelcol-metrics*/*/*.log
metadata:
  metrics:
    config:
      merge:
        exporters:
          logging:
            verbosity: detailed
        service:
          pipelines:
            metrics:
              exporters:
                - sumologic/default
                - logging
```

This configuration ensures that all metrics are printed to stdout and they are not collected by logs collector to keep your ingest low.

:::note
This configuration is prepared for OTLP source, as it doesn't use routing processor and multiple exporters in the pipeline.
:::

### Check Prometheus Remote Storage

We rely on the Prometheus [Remote Storage](https://prometheus.io/docs/prometheus/latest/storage/) integration to send metrics from
Prometheus to the metadata enrichment service.

You [check Prometheus logs](#prometheus-logs) to verify there are no errors during remote write.

You can also check `prometheus_remote_storage_.*` metrics to look for success/failure attempts.

### Missing metrics for ArgoCD installation

There is known issue with Argo CD and metrics collection. If you override `spec.source.helm.releaseName` in the `Application` or `ApplicationSet`, which are used to configure your application in Argo CD, then Kube State and Node metrics are not collected due to the following:

Service Monitor is looking for service labeled with `app.kubernetes.io/instance: <spec.source.helm.releaseName>`, but the label is actually `app.kubernetes.io/instance: <metadata.name>`.

In order to fix it, you need to ensure that the labels are matching and you can do it by adding the following to `user-values.yaml`:

```yaml
kube-prometheus-stack:
  kube-state-metrics:
    prometheus:
      monitor:
        selectorOverride:
          app.kubernetes.io/instance: <metadata.name>
          app.kubernetes.io/name: kube-state-metrics
  prometheus-node-exporter:
    prometheus:
      monitor:
        selectorOverride:
          app.kubernetes.io/name: prometheus-node-exporter
```

where `metadata.name` is the value from Argo Application manifest.

## Common Issues

### Missing metrics - cannot see cluster in Explore

If you are not seeing metrics coming in to Sumo or/and your cluster is not showing up in [Explore](/docs/observability/kubernetes/monitoring#open-explore) it is most likely due to the fact that Prometheus pod is not running.

You can verify that by using the following command:

```sh
$ kubectl get pod -n <NAMESPACE> -l app.kubernetes.io/name=prometheus
NAME                                 READY   STATUS    RESTARTS   AGE
prometheus-<NAMESPACE>-prometheus-0  2/2     Running   1          4d20h
```

In case it is not running one can check prometheus-operator logs for any related issues:

```sh
kubectl logs -n <NAMESPACE> -l app=kube-prometheus-stack-operator
```

### Pod stuck in `ContainerCreating` state

If you are seeing a pod stuck in the `ContainerCreating` state and seeing logs like this:

```sh
Warning  FailedCreatePodSandBox  29s   kubelet, ip-172-20-87-45.us-west-1.compute.internal  Failed create pod sandbox: rpc error: code = DeadlineExceeded desc = context deadline exceeded
```

you have an unhealthy node. Killing the node should resolve this issue.

### Missing `kubelet` metrics

Navigate to the `kubelet` targets using the steps above. You may see that the targets are down with 401 errors. If so, there are two known workarounds you can try.

#### 1. Enable the `authenticationTokenWebhook` flag in the cluster

The goal is to set the flag `--authentication-token-webhook=true` for `kubelet`. One way to do this is:

```sh
kops get cluster -o yaml > NAME_OF_CLUSTER-cluster.yaml
```

Then, make the following change in that file:

```yaml
spec:
  kubelet:
    anonymousAuth: false
    authenticationTokenWebhook: true # <- add this line
```

Then run

```sh
kops replace -f NAME_OF_CLUSTER-cluster.yaml
kops update cluster --yes
kops rolling-update cluster --yes
```

#### 2. Disable the `kubelet.serviceMonitor.https` flag in Kube Prometheus Stack

The goal is to set the flag `kubelet.serviceMonitor.https=false` when deploying the prometheus operator.

Add the following lines to the `kube-prometheus-stack` section of your `user-values.yaml` file:

```yaml
kube-prometheus-stack:
  ...
  kubelet:
    serviceMonitor:
      https: false
```

and upgrade the helm chart:

```sh
helm upgrade collection sumologic/sumologic --reuse-values --version=<RELEASE-VERSION> -f user-values.yaml
```

### Missing `kube-controller-manager` or `kube-scheduler` metrics

There’s an issue with backwards compatibility in the current version of the kube-prometheus-stack helm chart that requires us to override the selectors for kube-scheduler and kube-controller-manager in order to see metrics from them. If you are not seeing metrics from these two targets, you can use the following config.

```yaml
kube-prometheus-stack:
  kubeControllerManager:
    service:
      selector:
        k8s-app: kube-controller-manager
  kubeScheduler:
    service:
      selector:
        k8s-app: kube-scheduler
```

### Prometheus stuck in `Terminating` state after running `helm del collection`

Delete the pod forcefully by adding `--force --grace-period=0` to the `kubectl delete pod` command.

### Rancher

If you are running the out of the box rancher monitoring setup, you cannot run our Prometheus operator alongside it. The Rancher Prometheus Operator setup will actually kill and permanently terminate our Prometheus Operator instance and will prevent the metrics system from coming up. If you have the Rancher prometheus operator setup running, they will have to use the UI to disable it before they can install our collection process.

### Falco and Google Kubernetes Engine (GKE)

`Google Kubernetes Engine (GKE)` uses Container-Optimized OS (COS) as the default operating system for its worker node pools. COS is a security-enhanced operating system that limits access to certain parts of the underlying OS. Because of this security constraint, Falco cannot insert its kernel module to process events for system calls. However, COS provides the ability to use extended Berkeley Packet Filter (eBPF) to supply the stream of system calls to the Falco engine. eBPF is currently only supported on GKE and COS. For more information, see [Falco documentation](https://falco.org/docs/getting-started/third-party/#gke).

To install on `GKE`, use the provided override file to customize your configuration and uncomment the following lines in the `values.yaml` file referenced below:

```yaml
  #driver:
  #  kind: ebpf
```

### Falco and OpenShift

Falco does not provide modules for all kernels. When Falco module is not available for particular kernel, Falco tries to build it. Building a module requires `kernel-devel` package installed on nodes.

For OpenShift, installation of `kernel-devel` on nodes is provided through MachineConfig used by [Machine Config operator](https://github.com/openshift/machine-config-operator). When update of machine configuration is needed machine is rebooted, see the [documentation](https://github.com/openshift/machine-config-operator/blob/master/docs/MachineConfigDaemon.md#coordinating-updates). The process of changing nodes configuration may require long time during which Pods scheduled on unchanged nodes are in `Init` state.

Node configuration can be verified by following annotations:

- `machineconfiguration.openshift.io/currentConfig`
- `machineconfiguration.openshift.io/desiredConfig`
- `machineconfiguration.openshift.io/state`

After that, remove Otelcol pods and associated PVC-s.

For example, if the namespace where the collection is installed is `collection`, run the following set of commands:

```bash
NAMESPACE_NAME=collection

for POD_NAME in $(kubectl get pods --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}' | grep otelcol-logs); do
  kubectl -n ${NAMESPACE_NAME} delete pvc "buffer-${POD_NAME}" &
  kubectl -n ${NAMESPACE_NAME} delete pod ${POD_NAME}
  kubectl -n ${NAMESPACE_NAME} delete pod ${POD_NAME}
done
```

The duplicated pod deletion command is there to make sure the pod is not stuck in `Pending` state with event `persistentvolumeclaim "file-storage-sumologic-otelcol-logs-1" not found`.

### Out of memory (OOM) failures for Prometheus Pod

If you observe that Prometheus Pod needs more and more resources (out of memory failures - OOM killed Prometheus) and you are not able to
increase them then you may need to horizontally scale Prometheus. For details, refer to [Prometheus sharding](https://github.com/SumoLogic/sumologic-kubernetes-collection/blob/main/docs/prometheus.md#horizontal-scaling-sharding).

### Prometheus: server returned HTTP status 404 Not Found: 404 page not found

If you see the following error in Prometheus logs:

```text
ts=2023-01-30T16:39:27.436Z caller=dedupe.go:112 component=remote level=error remote_name=2b2fa9 url=http://sumologic-sumologic-remote-write-proxy.sumologic.svc.cluster.local:9888/prometheus.metrics msg="non-recoverable error" count=194 exemplarCount=0 err="server returned HTTP status 404 Not Found: 404 page not found"
```

you'll need to change the following configurations:

- `kube-prometheus-stack.prometheus.prometheusSpec.remoteWrite`
- `kube-prometheus-stack.prometheus.prometheusSpec.additionalRemoteWrite`

so `url` starts with `http://$(METADATA_METRICS_SVC).$(NAMESPACE).svc.cluster.local.:9888`.

See the following example:

```yaml
kube-prometheus-stack:
  prometheus:
    prometheusSpec:
      remoteWrite:
        - url: http://$(METADATA_METRICS_SVC).$(NAMESPACE).svc.cluster.local.:9888/prometheus.metrics
          ...
```

Alternatively, you can add `/prometheus.metrics` to `metadata.metrics.config.additionalEndpoints`. See the following example:

```yaml
metadata:
  metrics:
    config:
      additionalEndpoints:
        - /prometheus.metrics.kubelet
```

### OpenTelemetry: dial tcp: lookup collection-sumologic-metadata-logs.sumologic.svc.cluster.local.: device or resource busy

If you see the following error in OpenTelemetry Pods:

```yaml
2023-01-31T14:50:20.263Z        info    exporterhelper/queued_retry.go:426      Exporting failed. Will retry the request after interval.        {"kind": "exporter", "data_type": "logs", "name": "otlphttp", "error": "failed to make an HTTP request: Post \"http://collection-sumologic-metadata-logs.sumologic.svc.cluster.local.:4318/v1/logs\": dial tcp: lookup collection-sumologic-metadata-logs.sumologic.svc.cluster.local.: device or resource busy", "interval": "16.601790675s"}
```

Add the following environment variable to the affected Statefulset/Daemonset/Deployment:

```yaml
extraEnvVars:
  - name: GODEBUG
    value: netdns=go
```

For example, for OpenTelemetry Logs Collector:

```yaml
otellogs:
  daemonset:
    extraEnvVars:
      - name: GODEBUG
        value: netdns=go
```

### OpenTelemetry Operator is Restarting after collection upgrade

If the OpenTelemetry Operator is restarting after upgrade, and the following error can be found in the previous run logs:

```text
{"level":"error","ts":"2024-01-10T09:32:24Z","logger":"controller-runtime.source.EventHandler","msg":"if kind is a CRD, it should be installed before calling Start","kind":"OpAMPBridge.opentelemetry.io","error":"no matches for kind \"OpAMPBridge\" in version \"opentelemetry.io/v1alpha1\"","stacktrace":"sigs.k8s.io/controller-runtime/pkg/internal/source.(*Kind).Start.func1.1\n\t/home/runner/go/pkg/mod/sigs.k8s.io/controller-runtime@v0.16.3/pkg/internal/source/kind.go:63\nk8s.io/apimachinery/pkg/util/wait.loopConditionUntilContext.func2\n\t/home/runner/go/pkg/mod/k8s.io/apimachinery@v0.28.4/pkg/util/wait/loop.go:73\nk8s.io/apimachinery/pkg/util/wait.loopConditionUntilContext\n\t/home/runner/go/pkg/mod/k8s.io/apimachinery@v0.28.4/pkg/util/wait/loop.go:74\nk8s.io/apimachinery/pkg/util/wait.PollUntilContextCancel\n\t/home/runner/go/pkg/mod/k8s.io/apimachinery@v0.28.4/pkg/util/wait/poll.go:33\nsigs.k8s.io/controller-runtime/pkg/internal/source.(*Kind).Start.func1\n\t/home/runner/go/pkg/mod/sigs.k8s.io/controller-runtime@v0.16.3/pkg/internal/source/kind.go:56"}
```

It means that Custom Resource Definition has not been applied by Helm. It is [Helm known issue](https://helm.sh/docs/chart_best_practices/custom_resource_definitions/#some-caveats-and-explanations), and it has to be applied manually:

```shell
kubectl apply -f https://raw.githubusercontent.com/open-telemetry/opentelemetry-helm-charts/opentelemetry-operator-0.44.0/charts/opentelemetry-operator/crds/crd-opentelemetry.io_opampbridges.yaml
```
