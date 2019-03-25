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











