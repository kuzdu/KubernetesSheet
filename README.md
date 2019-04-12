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
```
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
```

YAML erstellen: 
`kubectl create -f kubia-manual.yaml`

Logs sehen vom Pod:
`kubectl logs name_of_pod`
`kubectl logs kubia-manual -c name_of_container` //logs eines einzelnen Containers im Pod

Port Forwarding: 
`kubectl port-forward name_of_pod 8888:8080` // Anwendung erreichbar unter localhost:8888

##Labels
Organisation über Labels, um Pods logisch (horizontal) zu untertrennen.
```
apiVersion: v1
kind: Pod
metadata:
  name: kubia-manual-v2
  labels:
    creation_method: manual
    env: prod
spec:
  containers:
  - image: kuzdu/kubia
    name: kubia
    ports: 
    - containerPort: 8080
      protocol: TCP 
```

A label selector can select resources based on whether the resource
- Contains (or doesn’t contain) a label with a certain key
- Contains a label with a certain key and value
- Contains a label with a certain key, but with a value not equal to the one you
specify

Filterbar per `kubectl get po -l creation_method=manual`

Ebenfalls kann man auch nodes labeln. Z.B. so
`kubectl get nodes -l local_kub=true`

Hat man nun mehrere Nodes, kann in der Deployment-YAML-Datei festgelegt werden, dass ein Pod nur in Nodes mit dem entsprechenden Label erstellt wird. 
BeispieL: Sinnvoll kann dies z.B. sein, wenn Pods Grafikleistungen durchführen und extra ein Node auf GPU ausgelegt ist. 

```
...
kind: Pod
metadata:
  name: kubia-gpu
spec:
  nodeSelector:
    gpu: "true"
  containers:
...
```

### Annotation
Ebenfalls gibt es noch Annotations, die sich von den Labeln unterscheiden, da nach ihnen nicht sortiert/gefiltert (...) werden kann. Dafür können Annotations viel mehr Informationen erhalten. Teilweise setzt Kubernetes diese Infos.

Kann man kriegen mittels: `kubectl get pods my_pod -o yaml` 
Ebenfalls kann man sie manipulieren.

### Namespaces

Mit Namespaces wird ein System unterteilt, was sinnvoll ist, wenn viele User ihre eigenen Anwendungen haben. Ein Namespace umfasst also z.B. viele Pods. Die Resources müssen nur im Namespace unqiue sein. 


### Pods löschen

Löschen von Pods geht via Labels, Namespaces ... 
Man kann auch alles Erstellte (außer Secrets) löschen via `kubectl delete all --all`

Die Reihenfolge des Löschens ist wichtig. Besteht z.B. noch ein Replica-Set, wird ein Pod logischerweise neu erstellt. Löscht man ein Deployment wird Pod(+ReplicationController) mitgelöscht.

### Logs einsehen 
kubectl logs mypod --previous //previous ist optional

# Kapitel 4 

### Liveness Probe
- erreichbar sein
- innere Werte prüfen, bei denen ein Neustart helfen könnte
- leichtgewichtige Antwort


## Replication Controller
Replication Controller händeln das Erstellen von mehreren Kopien (Replicas) von Pods. Ist ein Pod gecrashed, wird ein neuer erstellt und der alte verworfen. 

Wer im Replica Controller ist ergibt sich über das *Label*, *pod template* gibt an, wie ein Replica auszusehen hat (Maximum, minimum an replicas)

Ändert man das Label oder das Template, betrifft dies nur die neuen Pods. 

Man kann Pods aus dem RC nehmen, in dem man das Label ändert:
`kubectl label pod name_des_pods app=foo --overwrite` 
Sinnvolle Anwendungen können sein, wenn ein einziger Pod falsch läuft und man ihn untersuchen möchte. 

Ändert man dagegen das Label vom RC, dann fallen alle Pods raus. 

Später sind ReplicaSets rausgekommen: Es ist zu erwarten, dass der Replication Controller deprecated wird.
Man sollte daher immer ReplicaSets verwenden. 
- Also, for example, a single ReplicationController can’t match pods with the label env=production and those with the label env=devel at the same time. It can only match either pods with the env=production label or pods with the env=devel label. But a sin- gle ReplicaSet can match both sets of pods and treat them as a single group.

Create ReplicaSet `kubectl apply -f kubia-replicaset.yaml ` 
