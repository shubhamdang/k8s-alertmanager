# Kubernetes cluster Installation
  

 

We are using RKE(Rancher Kubernetes Engine) to install k8s cluster.


## Types of Nodes in K8s installations

- rke node(devops node): Node where rke utility is installed that is responsible for installation of k8s cluster on other nodes.
- k8s nodes(master/worker nodes): Nodes that are the part of k8s cluster where installation takes place.


## Prerequisites(master/worker)

- Docker must be installed on all the nodes of k8s cluster.


```
    curl https://releases.rancher.com/install-docker/20.10.sh | sh
    sudo usermod -aG docker $USER
    sudo systemctl restart docker 
    Password-less ssh to all k8s nodes from rke node.
```


## Setup RKE/Devops Node

### Install RKE


```
$ mkdir rke
$ cd rke
$ wget https://github.com/rancher/rke/releases/download/v1.3.17/rke_linux-amd64 -O rke
$ sudo chmod +x rke
```

### Install kubectl 
Use steps as per the Operating System.


 

## Install k8s cluster

In RKE node, go to rke directory that we created in above step and create file cluster.yml and add the following cluster data.

```
touch cluster.yml
```

Here adding a cluster.yml file for all-in-one cluster setup

```
cluster.yml
------------
nodes:
- address: 10.42.10.81
  user: cast
  role:
    - controlplane
    - etcd
    - worker
kubernetes_version: v1.20.8-rancher1-1
network:
    plugin: calico
```

Once cluster.yml configuration is updated , then run `rke up`  in the same directory to create kubernetes cluster.

Once cluster installation is successful, then a file is created kube_config_cluster.yml  in the same directory or we can call it kubeconfig file for that cluster.

 

### Cluster Installation Verification
Verify k8s installation with command (it will output the nodes in cluster.) 

```
kubectl --kubeconfig kube_config_cluster.yaml get nodes
```

All the nodes must have status as READY.

Copy and paste the kubeconfig created in above step to ~.kube/config and after that you don't need to pass kubeconfig parameter with kubectl command.


```
mkdir ~/.kube
cp ~/rke/kube_config_cluster.yaml ~/.kube/config
```

and now command will change to 

```
kubectl get nodes
```


## Install Kube-Prometheus-Stack

Install helm cli using the link https://helm.sh/docs/intro/install/
    
I am using kube-prometheus-stack to install prometheus, grafana and alertmanager.

Once helm is install follow the below steps.

First, create a custom-values.yaml file to specify our custom configurations:
```
# custom-values.yaml
prometheus:
  service:
    type: NodePort
grafana:
  service:
    type: NodePort
alertmanager:
  service:
    type: NodePort
```

Next, install the kube-prometheus-stack using Helm:

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm upgrade --install kube-prometheus-stack prometheus-community/kube-prometheus-stack -f custom-values.yaml
```

This command will deploy Prometheus, Alertmanager, and Grafana to your cluster with the services exposed as NodePort.


In order to access nodeport ports, run the below command
```
kubectl get svc
kube-prometheus-stack-alertmanager               NodePort    10.43.236.102   <none>        9093:30903/TCP,8080:32258/TCP   22h
kube-prometheus-stack-grafana                    NodePort    10.43.45.117    <none>        80:31455/TCP                    22h
kube-prometheus-stack-kube-state-metrics         ClusterIP   10.43.18.42     <none>        8080/TCP                        22h
kube-prometheus-stack-operator                   ClusterIP   10.43.140.138   <none>        443/TCP                         22h
kube-prometheus-stack-prometheus                 NodePort    10.43.16.75     <none>        9090:30090/TCP,8080:31013/TCP   22h
kube-prometheus-stack-prometheus-node-exporter   ClusterIP   10.43.224.174   <none>        9100/TCP                        22h
kubernetes                                       ClusterIP   10.43.0.1       <none>        443/TCP                         350d
```

Lets say your node ip is 10.42.10.81 then you can access the services on the following URL
```
Prometheus: 10.42.10.81:30090
Alermanager: 10.42.10.81:30903
Grafana: 10.42.10.81:31455
```


## Install Robusta

Using Robusta we can extends Prometheus/VictoriaMetrics/Coralogix (and more) with features like:
**Smart Grouping** - reduce notification spam with Slack threads 🧵
**AI Investigation** - Kickstart your alert investigations with AI (optional)
**Alert Enrichment** - see pods log and other data alongside your alerts
**Self-Healing** - define auto-remediation rules for faster fixes
**K8s Problem-Detection** - alert on OOMKills or failing Jobs without PromQL
**Change Tracking** - correlate alerts and Kubernetes rollouts
**Auto-Resolve** - send alerts, resolve them when updated (e.g. in Jira)
**Dozens of Integrations** - Slack, Teams, Jira, and more


### How to install robusta in K8s cluster
Here we have prometheus stack already installed so we will using `Integrate with Existing Prometheus` instllation to install robusta.

Installation consist of generation of config file and then install the robusta using helm chart and config file generated in last step.


#### Generate Config file

- In this step it will ask for some information regarding the slack channel etc.
```
chmod +x robusta
cats@cats-virtual-machine:~/Documents/monitoring$ ./robusta gen-config --no-enable-prometheus-stack

Please wait
 [|]
Robusta reports its findings to external destinations (we call them "sinks").
We'll define some of them now.

Configure Slack integration? This is HIGHLY recommended. [Y/n]: y
If your browser does not automatically launch, open the below url:
https://api.robusta.dev/integrations/slack?id=f20535d0-3c4xxxxxxxxxxxxx
You've just connected Robusta to the Slack of: ddxx
Which slack channel should I send notifications to? # event
Configure MsTeams integration? [y/N]: N
Configure Robusta UI sink? This is HIGHLY recommended. [Y/n]: n
Would you like to enable two-way interactivity (e.g. fix-it buttons in Slack) via Robusta's cloud? [y/N]: n
Last question! Would you like to help us improve Robusta by sending exception reports? [y/N]: n
 Saved configuration to ./generated_values.yaml - save this file for future use!
 ```
 
 - Install Robusta using below commands
 ```
 helm repo add robusta https://robusta-charts.storage.googleapis.com && helm repo update
 helm install robusta robusta/robusta -f ./generated_values.yaml --set clusterName=dev
 ```
 
 
#### Verifying Installation
Confirm that two Robusta pods are running:
```
kubectl get pods -A | grep robusta
 ```
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
