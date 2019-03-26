# KUBE

Containers = POD

PORTUS = registry like Nexus

Logs (Greylog, ELK,EFK) + metrics (prometheus) + tracing (open tracing) = observability O11y

12factor.net (la différence entre les env = la configuration, pas le binaire)

Site Reliability Engineer (Alice GoldFuss = github)

Sciladb = 10 * *la performance de cassandra* 

Persistance des données : dejà bien outillé avec oracle, mysql etc.

Utilisation de *MOSH* = ssh via UDP (utile pour les connexions pas stable)

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

## Setting up K8s

`kubectl apply` = `curl ... | sh` (_careful_)

`pssh` = parallel SSH (lancer une commande sur plusieurs hosts)


__@Youtube__ : KelseyHighTower, AliceGoldFuss, JessieFrazelle

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

## Deploy self hosted registry 

```bash
kubectl run registry --image=registry
kubectl expose deploy/registry --port=5000 --type=NodePort
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

kubectl expose deployment redis --port 6379
kubectl expose deployment rng --port 80
kubectl expose deployment hasher --port 80
kubectl expose deploy/webui --type=NodePort --port=80
```

node1:51.15.35.49
node2:51.15.60.92
node3:51.15.96.171

## Controlling the cluster remotely

Bastion = 
* serveur d'exploitation qui lance des commandes sur le cluster en remote
* avoir la liste des accès pour les clusters

## Accessing internal Services





