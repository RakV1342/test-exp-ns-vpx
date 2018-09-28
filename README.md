# test-exp-ns-vpx

Run a pod in cluster to monitor vpx/mpx -- use static promethues OR prometheus op

Run exporter container (and prom and graf) outside to monitor vpx/mpx 

Name for exporter - Citrix Exporter??

Change label such that it works with prometheus out of the box -- so label should be k8s-app

Exporter to Monitor Ingress VPX/MPX Devices
===

Description:
---

This guide describes how to setup an Exporter to monitor Ingress VPX/MPX devices which are linked to a Kubernetes cluster. More details about the Exporter can be obtained from this [link](https://github.com/Rakshith1342/netscaler-stat-exporter).

There are two ways to configure the monitoring system:
1. Exporter inside k8s cluster as a pod detected by Prometheus Operator
2. Exporter outside k8s cluster as a container detected by a Prometheus container

The following diagram shows the two approaches with a diagram. 

![exporter_diagram](https://user-images.githubusercontent.com/40210995/41391720-f89ee57e-6fb9-11e8-9550-02dc60dcfa43.png)

   In the above diagram, ................... blue boxes represent physical machines or VMs and grey boxes represent containers. 
There are two physical/virual NetScaler instances present with IPs 10.0.0.1 and 10.0.0.2 and a NetScaler CPX (containerized NetScaler) with an IP 172.17.0.2.
To monitor stats and counters of these NetScaler instances, an exporter (172.17.0.3) is being run as a container. It collects NetScaler stats such as total hits to a vserver, http request rate, ssl encryption-decryption rate, etc from the three NetScaler instances and holds them until the Prometheus containter 172.17.0.4 pulls the stats for storage as a time series. Grafana can then be pointed to the Prometheus container to fetch the stats, plot them, set alarms, create heat maps, generate tables, etc as needed to analyse the NetScaler stats. 

   Details about setting up the exporter to work in an environment as given in the figure is provided in the following sections. A note on which NetScaler entities/metrics the exporter scrapes by default and how to modify it is also explained.

Usage:
---
<details>
<summary>1. Exporter Inside K8s Cluster</summary>
<br>
   
This method assumes Prometheus Operator has been configured in K8s for monitoring. The deployment guide for Prometheus Operator given here
GIVE LINK
AGAIN?? **NOTE:** The labels of svcmon, svc, and prometheus, and namespace restrictions should match. 


An image for the exporter will need to be built and loaded to docker on all the nodes. The image can be built using ```docker build -f Dockerfile -t ns-exporter:v1 ./```. Once loaded to docker on all the worker nodes, the following yaml file can be used to deploy the exporter as a pod in Kuberenetes and expose it as a service. 
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
The parameters which can be provide in the ```args:``` section are:

flag&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | Description
-----------------|--------------------
--target-nsip    |Provide the &lt;IP:port&gt; of the Netscalers to be monitored
--port	        |Specify on which port the stats collected by the exporter should be exposed. Agents like Prometheus will need to scrape this port of the container to access stats being exported
--username       |Provide the username of the NetScaler to be monitored. Default: 'nsroot'
--password       |Provide the password of the NetScaler to be monitored. Default: 'nsroot'
--secure         |Option 'yes' can be provided to run stat collection from NetScalers over TLS. Default: 'no'.

The pod can now be deployed using ```kubectl create -f exporter.yaml```. The serviceMonitor provided in the yaml file should allow for the Exporter to be auto-detected by the Prometheus Operator. 

**NOTE:** The labels of svcmon, svc, and prometheus, and namespace restrictions should match. 

Validate that the Exporter appears in the targets list in UP state.
</details>


<details>
<summary>2. Exporter Outside K8s Cluster</summary>
<br>

To be filled in.
</details>
