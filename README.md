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

Now, the following yaml file can be used to deploy the exporter as a pod in Kuberenetes and expose it as a service. Populate the ```args:``` section to include the IPs of the Ingress VPX/MPX to be monitored and deploy the exporter using ```kubectl create -f exporter_ingress.yaml```. 
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
      env:
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
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
  selector:
    matchLabels:
      app: exp
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

To validate the proper working of the Exporter to monitor the Ingress devices, the Prometheus and Grafana pods need to be accessible from a browser. This can be done by exposing the Prometheus and Grafana services as NodePorts. The ```prometheus-service.yaml``` can be changed as follows to expose them using NodePort.
```
apiVersion: v1
kind: Service
metadata:
  labels:
    prometheus: k8s
  name: prometheus-k8s
  namespace: monitoring
spec:
  type: NodePort
  ports:
  - name: web
    port: 9090
    nodePort: 30100
    targetPort: web
  selector:
    app: prometheus
    prometheus: k8s
```

After making these changes to the yaml files, the following commands will expose the services:
```
kubectl apply -f prometheus-service.yaml
```

Now, going to the targets page ```http://<k8s-node-IP>:30100/targets``` of Prometheus can be used to verify if the state of the Exporter in the list is ```UP```. 

[IMAGE OF TARGETS PAGE]

**NOTE:** If large metrics... may not get response in reqd time. If the endpoint appear in the list of targets but are in DOWN with the error "context deadline exceeded", then increase the ```scrape_timing``` AND ```timeout``` of the prometheus-operator till UP state.

Finally, Grafana can be used to plot stats. 
EXPOSE GRAFANA AS NODE PORT
The ```grafana-service.yaml``` can be changed as follows to expose them using NodePort.
```
apiVersion: v1
kind: Service
metadata:
  name: grafana
  namespace: monitoring
spec:
  type: NodePort
  ports:
  - name: http
    port: 3000
    nodePort: 30300
    targetPort: http
  selector:
    app: grafana
```
```
kubectl apply -f grafana-service.yaml
```
OPEN IN BROWSER WITH ADMIN/ADMIN
PLOT SOME NETSCALER COUNTER --- TYPE NETSCALER IN QUERY BAR, OR JUST IMPORT THE JSON FILE.
[IMAGE OF SOME STAT PAGE]

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
