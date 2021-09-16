# Datadog-K8s-Quickstart-Guide

## QuickStart

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

- Download the [values file](helm) for your Kubernetes flavour. Update values like `datadog.clusterName` and `DD_ENV`. If creating a secret with a different name in the previous step, update `datadog.apiKeyExistingSecret`.

- I recommend taking a look to [this repo](https://github.com/yafernandes/k8s-cluster/tree/master/kubernetes/helm) with different values for different k8s distribution.

- Deploy Datadog using the updated values file.

```shell
helm install datadog datadog/datadog -n datadog -f <values file>
```

If you want to validate the resources being deployed first, you can still use helm to just output it.

```shell
helm template datadog datadog/datadog -n datadog -f <values file>
```

## Installing Datadog Helm Charts

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

- Deploy Frontend and Backend

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

- Check [Events]()

- Check Container Map

### Notes

#### AKS

AKS clusters using Kubernetes version 1.19 node pools and greater use *containerd* as its container runtime. -- [here](https://docs.microsoft.com/en-us/azure/aks/cluster-configuration#container-runtime-configuration)

#### Kubernetes 1.20

Getting error message `Warning: Cannot retrieve leader election record` for *kube-controller-manager* and *kube-scheduler*. Set `leader_election: false` for those integrations. 

## Proxy

The values in the `no_proxy` variable are:

- k8s.aws.pipsquack.ca - Hosts domain
- cluster.local - Kubernetes dns zone - `kubectl describe configmap coredns -n kube-system`
- 10.0.0.0/16 - Cluster address space
- 172.16.0.0/12 - Kubernets service address space - `/etc/kubernetes/manifests/kube-controller-manager.yaml`
- 192.168.0.0/16 - Kubernetes pods address space - `/etc/kubernetes/manifests/kube-controller-manager.yaml`
