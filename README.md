## Links
https://microk8s.io/

https://kubernetes.io/docs/home/

## Schritte fuer das Beispiel

snap install microk8s --classic

microk8s.start

microk8s.enable ingress registry dns

iptables -P FORWARD ACCEPT

apt install iptables-persistent

-> rules sichern

microk8s.kubectl get all -A

-> wenn alle services oben sind dann..

microk8s.kubectl get svc -n container-registry

-> die IP unten dann fuer die IP bei insecure-registry einsetzen und bei docker tag

apt install docker.io

usermod -a -G docker $USER

-> wenn ihr nicht mit root arbeitet 

cd pathtorepo/ 

docker build . -t mytest

echo "{ "insecure-registries":["10.152.183.38:5000"] }" > /etc/docker/daemon.json

systemctl restart docker

docker tag mytest 10.152.183.38:5000/mytest

docker push 10.152.183.38:5000/mytest:latest

microk8s.kubectl apply -f mydepl.yaml

microk8s.kubectl apply -f ingress.yml

microk8s.kubectl expose deployment mytest --type=LoadBalancer --port=80

microk8s.kubectl get pods

-> wenn die mytest pods running sind dann solltet ihr mit dem browser ueber die IP eurer VM und /hello eine nginx Seite sehen

Bsp: https://172.19.0.77/hello/

Wenn ihr Autorefresh einstellt dann solltet ihr die IP switchen sehen.

### microk8s Features
Available addons:

  coredns
  
  dashboard
  
  dns
  
  fluentd
  
  gpu
  
  ingress
  
  istio
  
  jaeger
  
  knative
  
  linkerd
  
  metrics-server
  
  prometheus
  
  rbac
  
  registry
  
  storage
  

### Tips 
echo "alias k='microk8s.kubectl'" >> ~/.bashrc

### Was wird noch kommen im Beispiel
Loadbalancer

Istio

DNS
