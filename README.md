## Links
https://microk8s.io/

https://kubernetes.io/docs/home/

## Hilfe bei Problemen
Ihr koennt euch jeder Zeit an mich wenden. Am besten ihr beschreibt eure Probleme detailiert in einer Mail und schickt mir die. Dann habe ich Zeit um das zu beurteilen.

## Schritte fuer das Beispiel

 - Installation von Kubernetes

> Mit snap refresh kann man ein Update machen. snap info microk8s zeigt
> ebenfalls an welche Versionen zur Auswahl stehen (Beta/Alpha..)
> microk8s.start enabled ebenfalls den automatischen Start ueber systemd

    snap install microk8s --classic
    microk8s.start

 - Fuer das Beispiel brauchen wir folgende services:

> Ingress fuer den Zugriff von Aussen
> DNS fuer die interne Aufloesung von Namen
> Registry fuer die Verwaltung von Images

    microk8s.enable ingress registry dns

 - iptables Konfiguration fuer den Zugriff von ausserhalb der VM

> iptables persistent sichert die Rules und muss bei einer manuellen Aenderung aufgerufen werden, sonst sind die Rules nach dem naechsten Boot verloren

    iptables -P FORWARD ACCEPT
    apt install iptables-persistent

- Warten bis der Cluster und alle Services und Pods gestartet sind

> fuer alle Subcommandos steht immer ein --help bereit um Optionen anzeigen zu lassen
> mit get lassen sich verschiedene Cluster Komponenten anzeigen
> **Achtung** hier muss immer der Namespace Kontext beachtet werden
> 
> die IP fuer die container-registry im zweiten Kommando wird dann fuer die Konfiguration des Docker Daemon benoetigt

    microk8s.kubectl get all -A
    microk8s.kubectl get svc -n container-registry

- Installation von Docker

> Wer nicht  root nutzen will muss Mitglied der Gruppe docker sein (siehe usermod)

    apt install docker.io git
    usermod -a -G docker $USER

- git clone und Docker Image Build

> nach dem git clone findet der image build mit docker statt. Schaut euch das simple Beispiel in Dockerfile an. Lest die Doku um mehr Optionen in Dockerfiles zu sehen
>
> der Eintrag im daemon.json file benoetigt eure IP vom get svc registry Kommando. Also bitte ersetzen

    git clone https://github.com/christiansetzer/microk8s-miniapp.git  
    cd microk8s-miniapp
    docker build . -t mytest
    
    echo "{ \\"insecure-registries\\":[\\"10.152.183.38:5000\\"] }" > /etc/docker/daemon.json
    
    systemctl restart docker

 - Docker Image in die Kubernetes Registry pushen

> Um das Image in die Registry hochzuladen muss es mit der IP und Port getagt werden. Hier also wieder die IP der Registry verwenden
> 
> mytest ist das Image. Ihr koennt fuer jeden Image push Vorgang einen Versionstag noch angeben. In dem Fall latest. 

    docker tag mytest 10.152.183.38:5000/mytest:latest
    docker push 10.152.183.38:5000/mytest:latest

 - Deployment unseres mini-app Services
 
> Dazu sind 3 Schritte notwendig. BItte mal in die yaml Files schaun. Doku erklaert die Optionen
> 
> 1. Deployment der App. Hier werden die Pods erzeugt
> 2. die Ingress Regel anlegen. Hier definiert ihr wie euer Service erreichbar sein soll
> 3. am Ende muss noch die App exposed werden. Damit ist dann der Loadbalancer Zugriff moeglich
> mit get pods koennt ihr sehen ob eure App auf auf running steht

    microk8s.kubectl apply -f mydepl.yaml
    microk8s.kubectl apply -f ingress.yml
    microk8s.kubectl expose deployment mytest --type=LoadBalancer --port=80
    
    microk8s.kubectl get pods

 - List item

Zugriff auf den Service
 
> Ihr könnt nun auf den Service zugreifen. Browser oeffnen und die IP eurer VM verwenden
> 
> Wenn ihr Autorefresh einstellt dann solltet ihr die IP switchen sehen.

    Bsp: https://172.19.0.77/hello/


### microk8s Features
microk8s hat nativen Support fuer eine Menge an Services fuer Kubernetes. Diese koennen ganz einfach wie oben mit *enable* installiert werden.

| Available addons | 
|--|
| coredns |
| dashboard |
| dns |
| fluentd |
| gpu |
| ingress |
| istio |
| jaeger |
| knative |
| linkerd |
| metrics-server |
| prometheus |
| rbac |
| registry |
| storage |
 

### Tips 
Um  sich das Leben einfach machen, wird man oft sehen, dass Kubernetes Admins fuer kubectl einfach nur k gesetzt haben. Das hilft ungemein

    echo "alias k='microk8s.kubectl'" >> ~/.bashrc

### Was wird noch kommen im Beispiel

 - [ ] Loadbalancer
 - [ ] Istio
 - [ ] DNS
