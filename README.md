# KubernetesSheet

Masternode hostet Kubernetes Control Plane, darüber wird das gesamte Kubernetes System gemangaged, enthält folgende Komponenten  
- hat Kubernetes Api Server (gegen den man Befehle ausführt)
- Scheduler (händelt Workernodes)
- Controller Manager (clustern, replicating components, keeping tracks of nodes ?)
- etcd (Daten speichern)

Workernodes: Dort laufen meine Applications drauf
- Docker 
- kubelet (redet zum API Server vom Masternode)
- kube-proxy (load balancing)
- 1 node kann 1..n pods laufen lassen

Gutes Bild auf Seite 20: Figure 1.10 

Summary aus Kapitel 1:
- Monolithic apps are easier to deploy, but harder to maintain over time and sometimes impossible to scale.
- Microservices-based application architectures allow easier development of each component, but are harder to deploy and configure to work as a single system. Linux containers provide much the same benefits as virtual machines, but are far more lightweight and allow for much better hardware utilization.
- Docker improved on existing Linux container technologies by allowing easier and faster provisioning of containerized apps together with their OS environments. Kubernetes exposes the whole datacenter as a single computational resource for running applications.
- Developers can deploy apps through Kubernetes without assistance from sysadmins.
- Sysadmins can sleep better by having Kubernetes deal with failed nodes auto- matically.



# Kubernetesbefehle

*Zeigt Info über das derzeitige Cluster* 
kubectl cluster-info

*Zeigt alle Nodes* 
kubectl get nodes
kubectl describe node your_node_name







# Ablauf (?)
1. Man deplyoed seine App auf einem Pod 
*Start pod manuell (normalerweise über yaml)*
`kubectl run kubia --image=luksa/kubia --port=8080 --generator=run/v1`
replicationcontroller "kubia" created

2. Man erstellt einen Service drauf
*To create the service, you’ll tell Kubernetes to expose the ReplicationController you created earlier*
`kubectl expose rc kubia --type=LoadBalancer --name kubia-http`

Normalerweise steht einem dann eine external IP bereit, worüber der Service ansprechbar ist, bei Minikube muss man 
`minikube service kubia-http` aufrufen

Wie hängt das alles zusammen:
Request geht an den Service -> Service leitet weiter an Pod, der durch einen Replication Controller erstellt worden ist. Wenn im Replication Controller mehrere Pods erstellt werden, kümmert der Service sich ums Load Balancing.

Pods sind flüchtig (Anwendung kann crashen). 
Services haben eine statische IP, weshalb mit diesen kommunziert werden sollte. 
