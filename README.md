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

Skalieren per kubectl
`$ kubectl scale rc kubia --replicas=3`
replicationcontroller "kubia" scaled

# Kapitel 3

Nodes können zwischeneinander über IP Adressen kommunizieren. (Es existiert kein Network Address Translation)

Nicht jede App braucht einen eigenen Pod. Pods sind recht leichgewichtig und sollten mehrere (containerzed) Apps enthalten. Die Apps in einem Pod sollten logisch und technisch zusammenpassen (ähnlich wie bei Microservices und DDD).

Man könnte auch Frontend und Backend im selben Pod laufen lassen, aber das wiedrum wäre nicht so sinnvoll: Grund ist Skalierung (Kubernetes skaliert Pods, nichts die Apps darin) oder dass die immer auf einem Note laufen würden. (S.59 Action with Kubernetes). UI ist z.B. stateless, während die Datenbank es wohl nicht ist. 

Beispiel für zwei containerized Apps, wenn sie z.B. auf dieselbe Datenbank zugreifen. 

Leitfragen:
- Do they need to be run together or can they run on different hosts?
- Do they represent a single whole or are they independent components?
- Must they be scaled together or individually?

Man könnte auch Frontend und Backend im selben Pod laufen lassen, aber das wiedrum wäre nicht so sinnvoll: Grund ist Skalierung (Kubernetes skaliert Pods, nichts die Apps darin) oder dass die immer auf einem Note laufen würden. (S.59 Action with Kubernetes). UI ist z.B. stateless, während die Datenbank es wohl nicht ist. 

YAML: Für die Felder, die unterstützt werden steht die API-Dokumentation online zur Verfügung: https://kubernetes.io/docs/concepts/overview/kubernetes-api/
Oder mittels: `kubectl explain pods|notes|...`

Beispiel-YAML:
`
apiVersion: v1
kind: Pod
metadata:
  name: kubia-manual
spec:
  containers:
  - image: kuzdu/kubia
    name: kubia
    ports: 
    - containerPort: 8080
      protocol: TCP 
`

YAML erstellen: 
`kubectl create -f kubia-manual.yaml`

Logs sehen vom Pod:
`kubectl logs name_of_pod`
`kubectl logs kubia-manual -c name_of_container` //logs eines einzelnen Containers im Pod

Port Forwarding: 
`kubectl port-forward name_of_pod 8888:8080` // Anwendung erreichbar unter localhost:8888

##Labels

