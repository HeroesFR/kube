# KUBE

[>>Container Training<<](https://github.com/jpetazzo/container.training)

[Website](https://qconuk2019.container.training/)

[Github Awesome + techno](https://github.com/ramitsurana/awesome-kubernetes)


Containers = POD

PORTUS = registry like Nexus

Logs (Greylog, ELK,EFK) + metrics (prometheus) + tracing (open tracing) = observability O11y

12factor.net (la différence entre les env = la configuration, pas le binaire)

Site Reliability Engineer (Alice GoldFuss = github)

Sciladb = 10 * *la performance de cassandra* 

Persistance des données : dejà bien outillé avec oracle, mysql etc.

Utilisation de *MOSH* = ssh via UDP (utile pour les connexions pas stable)

---

## Première app

Facilitation de l'unboarding sur n'importe quel environnement. Utilisation de _git clone_ et _docker-compose up_ 

docker compose up -> idempotent, si un service est up, pas besoin de le relancer

* build
* ship
* run
  * network
  * volumes
  * container
  * link (service discovery = lier le service __redis__ sans passer par les adresses IP)


Best open source license

* MIT
* BSD

Worst open source license 

* GPLv3 = obligation de publier le code source !! (Lors de la distribution)

1 stack docker-compose = plusieurs  

* 1 container ==
  * 1 _processus_ du kernel Linux 
    * né à partir de _tarball_
    * vit dans son _namespace_ (crois qu'il est tout seul dans l'univers)
    * voit ses _cgroups_       (voit un p'tit peu de RAM, CPU, net etc.)

Tant qu'un container ne fait rien, il ne consomme rien.

__docker-compose up -d__ = detach the logs

__docker-compose down__ 

docker-compose régit dans le dossier ou il se trouve.

best practice image docker version

* [semser.org](semver.org) 
  * v1.3.x
  * git hash, build #
  * date (2019-03-25)

Flannel / Cilium / Calico => gestion du réseau dans Kube

---

## Concepts

pet vs cattle

* pet = monolithe, difficile à scale
* cattle = 5000 tête => easy to replace/scale

Il vaut mieux supprimer un slave et le remplacer que de faire de la maintenance



* FaaS = Function as a Service =/= Serverless (offre cloud)
* PaaS 
* KaaS
* CaaS = Cloud
* IaaS

Kube = Event driven
* Réaction suite à cet évènement.

---

## Déclaratif vs Impératif (vs Fonctionnel)

kubectl create
* create => creation yml => appel API server


### Déclaratif

* Gitlab CI 
* Ansible <-> ServerSpace
* JenkinsFile
* Docker-Compose
* Behaviour Driven Development (BDD)

Ingress = exposer vers l'extérieur des services

--- 

## First Contact with _kubectl_

_kubeadm_ a créé l'intégralité du cluster avec une API, un TLS certificates etc...

`kubectl get all --all-namespaces --export -o yaml > backup-k8s.yaml` save the config

`kubectl get nodes -o json | jq ".items[] | {name:.metadata.name} + .status.capacity"`

```json
{
  "name": "node1",
  "cpu": "4",
  "ephemeral-storage": "47830060Ki",
  "hugepages-2Mi": "0",
  "memory": "4044596Ki",
  "pods": "110"
}
{
  "name": "node2",
  "cpu": "4",
  "ephemeral-storage": "47830060Ki",
  "hugepages-2Mi": "0",
  "memory": "4044596Ki",
  "pods": "110"
}
{
  "name": "node3",
  "cpu": "4",
  "ephemeral-storage": "47830060Ki",
  "hugepages-2Mi": "0",
  "memory": "4044596Ki",
  "pods": "110"
}
```

F5 a racheté Nginx (hardware a racheté le software)

`kubectl get ns`

`kubectl -n kube-system get pods` avec `-n` = namespace

```
NAME                            READY   STATUS    RESTARTS   AGE
coredns-86c58d9df4-gs2gm        1/1     Running   0          13h
coredns-86c58d9df4-j65xn        1/1     Running   0          13h
etcd-node1                      1/1     Running   0          13h
kube-apiserver-node1            1/1     Running   0          13h
kube-controller-manager-node1   1/1     Running   0          13h
kube-proxy-2jr7w                1/1     Running   0          13h
kube-proxy-5trtl                1/1     Running   0          13h
kube-proxy-ljqk2                1/1     Running   0          13h
kube-scheduler-node1            1/1     Running   0          13h
weave-net-b7ng8                 2/2     Running   0          13h
weave-net-p8xzq                 2/2     Running   0          13h
weave-net-q2fpr                 2/2     Running   1          13h
```

`-node1` = lancé seulement sur cette node

`coredns` default port = 53, ne pas être trop restrictif sur les ports, sinon plus de gestion du réseau

---

## Setting up K8s

`kubectl apply` = `curl ... | sh` (_careful_)

`pssh` = parallel SSH (lancer une commande sur plusieurs hosts)


__@Youtube__ : KelseyHighTower, AliceGoldFuss, JessieFrazelle, JoeBeda

---

## Running first containers on Kube

_alpine_ distribution Linux super allégé

`kubectl run pingpong --image alpine ping 1.1.1.1`

* `nginx` inclus dans 
  * `Pod` inclus dans 
  * `ReplicaSet` inclus dans 
  * `deploy` inclus dans
  * `NameSpace`


FeaturesFlags : La feature a été livré, mais elle ne peut être déployé qu'à mon activation

Labeliser le code/environnement/frontend/network/scaling/pod/owner/astreinte(?)

`kubectl logs -l run=pingpong env=qual --tail 1`


prometheus in kube or outside ?
* both !
	* Grafana = outside
	* logs = inside

---

## Deploy self hosted registry 




```bash
kubectl run registry --image=registry
kubectl expose deploy/registry --port=5000 --type=NodePort # NodePort expose the port in every node

NODEPORT=$(kubectl get svc/registry -o json | jq .spec.ports[0].nodePort)
REGISTRY=127.0.0.1:$NODEPORT

cd ~/container.training/stacks
export REGISTRY
export TAG=v0.1
docker-compose -f dockercoins.yml build
docker-compose -f dockercoins.yml push


kubectl run redis --image=redis

for SERVICE in hasher rng webui worker; do
  kubectl run $SERVICE --image=$REGISTRY/$SERVICE:$TAG
done

# Expose ports of every services
# By default, the type is ClusterIP
kubectl expose deployment redis --port 6379
kubectl expose deployment rng --port 80
kubectl expose deployment hasher --port 80
kubectl expose deploy/webui --type=NodePort --port=80

```
:info: how to check if everything is okay ? 
* `kubectl get all` and check if everything is up and running
* go to the web browser `http://51.15.35.49:30913/index.html` and check if the page is showing

```
$ kubectl get all
NAME                            READY   STATUS    RESTARTS   AGE
pod/hasher-684c9fbdc7-567r4     1/1     Running   0          6m23s
pod/redis-6f9b48bb97-cdjb4      1/1     Running   0          7m
pod/registry-654cc89557-w75pr   1/1     Running   0          15m
pod/rng-d9548c789-bb74m         1/1     Running   0          6m23s
pod/webui-54879f744-ctbr7       1/1     Running   0          6m22s
pod/worker-686cc5944b-k25gm     1/1     Running   0          6m22s

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/hasher       ClusterIP   10.96.18.23     <none>        80/TCP           4m56s
service/kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP          49m
service/redis        ClusterIP   10.100.171.87   <none>        6379/TCP         5m4s
service/registry     NodePort    10.96.74.239    <none>        5000:30318/TCP   15m
service/rng          ClusterIP   10.97.198.52    <none>        80/TCP           5m1s
service/webui        NodePort    10.98.92.214    <none>        80:30913/TCP     4m45s

NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/hasher     1/1     1            1           6m23s
deployment.apps/redis      1/1     1            1           7m
deployment.apps/registry   1/1     1            1           15m
deployment.apps/rng        1/1     1            1           6m23s
deployment.apps/webui      1/1     1            1           6m22s
deployment.apps/worker     1/1     1            1           6m22s

NAME                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/hasher-684c9fbdc7     1         1         1       6m23s
replicaset.apps/redis-6f9b48bb97      1         1         1       7m
replicaset.apps/registry-654cc89557   1         1         1       15m
replicaset.apps/rng-d9548c789         1         1         1       6m23s
replicaset.apps/webui-54879f744       1         1         1       6m22s
replicaset.apps/worker-686cc5944b     1         1         1       6m22s
```

* node1:51.15.35.49
* node2:51.15.60.92
* node3:51.15.96.171

---

## Controlling the cluster remotely

Bastion = 
* serveur d'exploitation qui lance des commandes sur le cluster en remote
* avoir la liste des accès pour les clusters

--- 

## Accessing internal Services

-> acces a TCP service `kubectl port-forward service/name_of_service local_port:remote_port`

---

## Dashboard

We need to 
- create a _deployment_ and a _service_ for the dashboard
- also a _secret_ a _service account_, a _role_ and a _role binding_

[Securising kube](https://blog.heptio.com/on-securing-the-kubernetes-dashboard-16b09b1b7aca)

```
[51.15.35.49] (kubernetes-admin@kubernetes:default) docker@node1 ~
$ kubectl get namespaces 
NAME              STATUS   AGE
default           Active   106m
kube-node-lease   Active   106m
kube-public       Active   106m
kube-system       Active   106m
[51.15.35.49] (kubernetes-admin@kubernetes:default) docker@node1 ~
$ kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
hasher       ClusterIP   10.96.18.23     <none>        80/TCP           61m
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP          106m
redis        ClusterIP   10.100.171.87   <none>        6379/TCP         61m
registry     NodePort    10.96.74.239    <none>        5000:30318/TCP   71m
rng          ClusterIP   10.97.198.52    <none>        80/TCP           61m
webui        NodePort    10.98.92.214    <none>        80:30913/TCP     61m
[51.15.35.49] (kubernetes-admin@kubernetes:default) docker@node1 ~
$ kubectl -n kube-system get svc
NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                  AGE
kube-dns               ClusterIP   10.96.0.10      <none>        53/UDP,53/TCP,9153/TCP   106m
kubernetes-dashboard   NodePort    10.97.16.46     <none>        443:31032/TCP            13m
socat                  NodePort    10.106.127.67   <none>        80:30962/TCP             13m

```
---

## __Kubectl apply__ 

careful with this command because we apply a yml file to our cluster

---

## Scale a deployment

`kubectl scale deploy/worker --replicas=10`

if we scale the replicaset, the deploy will

Scale ? When 

HPA (Horizontal Pod AutoScaler) 

Scale if %CPU > X% ,

min = 2 
max = 2-3 fois le nombre de pods ?



--- 

## Daemon Sets

__Deployer un exemplaire d'un pod sur chaque node !__

Contrairement à deploy, qui peut en lancer 10 n'importe ou.

They can also be restricted to run only on some nodes : 

Likewise if you specify a .spec.template.spec.affinity, then DaemonSet controller will create Pods on nodes which match that node affinity.

affinity = label

taints =

(tolerations = ouvrir et assouplir)

__traiter les logs en tant que flux__, afin de décentraliser l'information [TOREAD](12factor.net)

If we delete pod, the replicaset will recreate it and will double it

--- 

## Updating a service through labels ans selectors

Difference between :

* the label of a resource in the __`metadata`__ block
  * decription
* the selector of a resource in the __`spec`__ block
  * 
* the label of the resource created by the first resource (in the __`template`__ block)
  * 


How to get logs from containers
* /var/lib/docker/containerHash/hash-json.log

How to avoid extra pods :

`kubectl patch`

---

## Rolling updates 

* update = rollout the pod and update it
* change spec = create new pod and leave the old pods

update the image `kubectl set image deploy worker worker=$REGISTRY/worker:$TAG` or use `kubectl edit deploy worker` and change the image name

`kubectl rollout undo deploy worker` will rollback to version N-1

`kubectl rollout status deploy worker` check the rollout status
 
### How to deploy

* Product Owner sign, promote and scan the image
* Pushed in the registry of prod as a promotion
* broadcast to Slack/Skype etc that a new version is ready do be deployed
* -> __`/deploy JIRA:v1.4`__
* ...
* ...
* __`OKAY: deployed on https://test.....`__ and we check that every metrics that matters the most are okay.

---

## Healthcheck

Needed in prod image `Dockerfile`, `docker-compose-prod.yml`, Resource Deployments K8s

* `Dockerfile` => `HEALTHCHECK`
* `docker-compose` => `healthcheck`
* resource Deploy K8s => liveness / readiness
  * liveness = pod dead or alive ?
  * readiness = is he available to serve traffic

if the charge overload, => __autoscaling__

---

## Accessing logs from the CLI

`stern <POD>` better than `kubectl logs` (until 1.14)

---

## Centralized logging

__ELK => EFK => Graylog 2 => Splunk (logz.io) => DataDog => Sematex__

### Create namespace

`kubectl create namespace efk`

`kns` and `kns efk`

A container writes a line on `stderr` or `stdout`

---

## Managing stacks with `Helm`

Templating 

`ClusterRoleBinding` = bind un _ClusterRole_ à un _serviceAccount_ 

```
kubectl create clusterrolebinding add-on-cluster-admin \
  --clusterrole=cluster-admin --serviceaccount=kube-system:default
```

`helm search` search on github charts

`helm search prometheus`

```
helm install stable/prometheus \
     --set server.service.type=NodePort \
     --set server.persistentVolume.enabled=false

```


