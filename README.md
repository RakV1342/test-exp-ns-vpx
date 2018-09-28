Exporter to Monitor Ingress VPX/MPX Devices
===

Description:
---

This guide describes how to setup an [Exporter](https://github.com/Rakshith1342/netscaler-stat-exporter) to monitor Ingress VPX/MPX devices which are linked to a Kubernetes cluster. More details about the Exporter can be obtained [here](https://github.com/Rakshith1342/netscaler-stat-exporter).

There are two ways to configure the monitoring system:
1. Exporter inside k8s cluster as a pod detected by Prometheus Operator
2. Exporter outside k8s cluster as a container detected by a Prometheus container


Usage:
---
<details>
<summary>1. Exporter Inside K8s Cluster</summary>
<br>
   
This method assumes [Prometheus Operator](https://github.com/coreos/prometheus-operator) has been configured (using the [kube-prometheus manifest files](https://github.com/coreos/prometheus-operator/tree/master/contrib/kube-prometheus/manifests)) in Kubernetes for monitoring. 

Running ```kubectl create -f prometheus-operator/contrib/kube-prometheus/manifests/``` will setup Prometheus Operator using the kube-prometheus manifest files. 

Once Prometheus Operator has been setup, an image for the exporter will need to be built and loaded to docker on all the nodes. The image can be built using ```docker build -f Dockerfile -t ns-exporter:v1 ./```. 

The following yaml file can be used to deploy the exporter as a pod in Kuberenetes and expose it as a service. Populate the ```args:``` section to include the IPs of the Ingress VPX/MPX to be monitored and deploy the exporter using```kubectl create -f exporter_ingress.yaml```. 
```
apiVersion: v1
kind: Pod
metadata:
  name: exp
  labels:
    app: exp
spec:
  containers:
    - name: exp
      image: ns-exporter:v1
      args:
        - "--target-nsip=x.x.x.x:xx"
        - "--target-nsip=y.y.y.y:yy"
        - "--port=8080"
      imagePullPolicy: IfNotPresent
---
apiVersion: v1
kind: Service
metadata:
  name: exp
  labels:
    app: exp
spec:
  type: ClusterIP
  ports:
  - port: 8080
    targetPort: 8080
    name: exp-port
  selector:
    app: exp
---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  labels:
    app: exp
  name: exp
  namespace: monitoring
spec:
  endpoints:
  - interval: 30s
    port: exp-port
#  jobLabel: app
  selector:
    matchLabels:
      k8s-app: exp
  namespaceSelector:
    matchNames:
    - monitoring
    - default
```
Additional parameters such as username, password, and TLS query can be enabled by providing additional flags in the ```args:``` section. The table below describes flags which can be provided:

flag&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | Description
-----------------|--------------------
--target-nsip    |Provide the &lt;IP:port&gt; of the Netscalers to be monitored
--port	        |Specify on which port the stats collected by the exporter should be exposed. Agents like Prometheus will need to scrape this port of the container to access stats being exported
--username       |Provide the username of the NetScaler to be monitored. Default: 'nsroot'
--password       |Provide the password of the NetScaler to be monitored. Default: 'nsroot'
--secure         |Option 'yes' can be provided to run stat collection from NetScalers over TLS. Default: 'no'.


**NOTE:** The labels of svcmon, svc, and prometheus, and namespace restrictions should match. 

Validate that the Exporter appears in the targets list in UP state.
</details>


<details>
<summary>2. Exporter Outside K8s Cluster</summary>
<br>

This [link](https://github.com/Rakshith1342/netscaler-stat-exporter) explains how the Exporter can be setup to monitor any given NetScaler device in a non-Kubernetes environment. By following that documentation and providing the IPs of the Ingress VPX/MPX machines, they can be monitored.
</details>


Validation:
---
<details>
<summary>1. Exporter Inside K8s Cluster</summary>
<br>
   
Validate that the Exporter appears in the targets list in UP state.
Now can qury for metrics in Grafana and get create graphs.
</details>


<details>
<summary>2. Exporter Outside K8s Cluster</summary>
<br>

The [verification section](https://github.com/Rakshith1342/netscaler-stat-exporter#verification-of-exporter-functionality) on the [Exporter](https://github.com/Rakshith1342/netscaler-stat-exporter) page explains how this setup which is running outside a Kubernetes cluster can be validated.
</details>


# test-exp-ns-vpx

Name for exporter - Citrix Exporter??

VALIDATION SECTION --- Include kubectl apply command and modified <>-service yaml files here.

Match yaml files with latest set
