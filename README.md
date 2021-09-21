# Datadog-K8s-Quickstart-Guide

## Datadog Deployment

- Create a text file with the structure below. You can get your API Key in [here](https://app.datadoghq.com/account/settings#api).  The 32 long token will be used to secure communication between the node Agent and the Datadog Cluster Agent. You can generate one online [here](http://www.unit-conversion.info/texttools/random-string-generator/).

```text
api-key=<API KEY>
token=<32 characters long token>
```

- Create a `datadog` namespace.

```shell
kubectl create ns datadog
```

- Create the secret using the file created ealier.

```shell
kubectl create secret generic datadog-keys --from-env-file=secrets.txt -n datadog
```

- Download the [default values file](https://github.com/DataDog/helm-charts/blob/main/charts/datadog/values.yaml) or use [the one attached in this repo for AKS](values.yaml)for your Kubernetes flavour. Update values like `datadog.clusterName` and `DD_ENV`. If creating a secret with a different name in the previous step, update `datadog.apiKeyExistingSecret`.

- I recommend taking a look to [this repo](https://github.com/yafernandes/k8s-cluster/tree/master/kubernetes/helm) with different values.yaml for different k8s distribution.

- Deploy Datadog using the updated values file.

```shell
helm install datadog datadog/datadog -n datadog -f <values file>
```

If you want to validate the resources being deployed first, you can still use helm to just output it.

```shell
helm template datadog datadog/datadog -n datadog -f <values file>
```

### Installing Datadog Helm Charts

```shell
helm repo add datadog https://helm.datadoghq.com
helm repo update
```

[Documentation](https://github.com/DataDog/helm-charts/tree/master/charts/datadog)

## Installing Guestbook

- To test Datadog I recommend to use [this Google Guestbook App](https://cloud.google.com/kubernetes-engine/docs/tutorials/guestbook) with a frontend based on php and backend in redis

- Create a `guestbook` namespace.

```shell
kubectl create ns guestbook
```
- For APM we need to install php client tracer. I have already do it by adding the php tracer in the [dockerfile](Dockerfile)


- Download and Deploy [Frontend](frontend) and [Backend](redis-master)

```shell
kubectl apply -f frontend --namespace=guestbook
kubectl apply -f redis-master/ --namespace=guestbook
```
- Collect frontend IP and connect to the portal
```shell
kubectl get svc --namespace=guestbook
NAME           TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)        AGE
frontend       LoadBalancer   10.0.111.83    52.158.120.95   80:32364/TCP   90s
redis-master   ClusterIP      10.0.133.134   <none>          6379/TCP       77s
```
![alt text](https://a.cl.ly/12uvbEAB)

- As you can see, message is not printed. So Guestbook is not working!

## Using Datadog

### Visibility

- Check [Kubernetes Out of the box dashboard](https://app.datadoghq.com/screen/integration/86/kubernetes---overview?from_ts=1632251058272&to_ts=1632254658272&live=true)

![]((images/ootbdashboard.json))

- Check [Container Map](https://app.datadoghq.com/infrastructure/map?host=d33f0500ce64af0e4bd95726e4f9b74b32667424&fillby=avg%3Aprocess.stat.container.cpu.user_pct&sizeby=avg%3Anometric&groupby=kube_deployment&filter=kube_namespace%3Aguestbook&nameby=name&nometrichosts=false&tvMode=false&nogrouphosts=true&palette=hostmap_blues&paletteflip=false&node_type=container)

![alt text](images/containermap.json)


- [Orchestrator Explorer](https://app.datadoghq.com/orchestration/overview/pod?sort=metadata.name%3Aasc&tags=kube_namespace%3Aguestbook&start=1632253978831&end=1632254878831&paused=false)


- [Network Map](https://app.datadoghq.com/network/map?destG=tag_dest_kube_deployment&destQ=&node_type=pod_name&srcG=tag_source_kube_namespace&srcQ=&unresolved=false&from_ts=1632243273824&to_ts=1632244173824&live=false)

![alt text](images/networkmap.json)

### Troubleshooting

- APM Trace error


### Proactivity

- Synthetic test

- Recommended Monitor


### Resource Optimization Dashboard

- Download [Kubernetes Optimization Dashboard](KubernetesOptimization--2021-09-15T10_32_25.json
)

- Import it

![alt text](images/dashboard.json)

