### Links
https://docs.google.com/presentation/d/19M1AnODM-a4bsQjC_7Pz_NGVZrcTIpLk-IEoNGzp8kw/edit?usp=sharing
### Excercises
#### 0. Examine your Cluster
```
kubectl get svc
kubectl get ns
kubectl get pods --all-namespaces
```
#### 1. Run simple pod
##### run pod
```
kubectl run -i --tty ubuntu --image=ubuntu --restart=Never -- bash
```
##### cleanup
```
# exit then
kubectl get pods
# explain what you see and why
kubectl get deployments
kubectl delete pod ubuntu
```

#### 2. Run nginx
##### run
```
# create deployment
kubectl create deployment nginx --image=nginx
# scale deployment
kubectl scale deployment nginx --replicas=3
# create service
kubectl expose deployment nginx --port 80
```
##### inspect
```
kubectl get deployments
kubectl get pods
kubectl get svc
# find out pods IP addresses
kubectl get pods -o wide
```
##### delete one of the pods
```
kubectl get pods
...
kubectl delete pod <pod_name>
kubectl get pods
```
###### start inspecting pod
```
# Create pod inside cluster from exc 1
kubectl run -i --tty ubuntu --image=ubuntu --restart=Never -- bash
# apt-get update, install ping :)
# or use another base image
# ping each pods
# curl each pods
curl <podip>:<port>
# ping service ClusterIP
# curl service ClusterIP
curl <clusterIP>:<port>
```
#### 3. Run local registry inside k8s
```
# create deployment
kubectl create deployment registry --image=registry
# expose
kubectl expose deploy/registry --port=5000 --type=NodePort
# verify
kubectl get deployment registry
kubectl get pods
kubectl get svc registry
```
##### verify endpoints available
```
# from ubuntu inspecting pod
ping <pod_ip>
curl <pod_ip>:5000/v2/_catalog
curl <svc_cluster_ip>:5000/v2/_catalog
# from local workstation
curl <minikube_ip>:<node_port>/v2/_catalog
# example curl 192.168.99.100:31135/v2/_catalog
```
##### Add insecure registry exclusion into local Docker Installation
```
# put here actual ip address/port
192.168.99.100:31155
```
##### relabel/push some image
```
# on your workstation
REGISTRY=192.168.99.100:31155
docker pull busybox
docker tag busybox $REGISTRY/busybox
docker push $REGISTRY/busybox
# verify image in registry
curl $REGISTRY/v2/_catalog
```
##### break things
```
# kill registry pod
kubectl get pods
kubectl delete pod <registry_pod_name>
# registry deployment will restart deployment
kubectl get pods
# verify image in registry.. wait a minute
curl $REGISTRY/v2/_catalog
# explain?
```
##### save deployment/svc templates into files
```
kubectl get deployment registry -o yaml
kubectl get deployment registry -o yaml --export >> deployment-registry-actual.yaml
kubectl get svc registry -o yaml --export >> less service-registry-actual.yaml
```

#### 4. Run registry in k8s, improved
##### recreate app use manifests from ./registry
```
# check your exported registry first
less deployment-registry-actual.yaml
less service-restry-actual.yaml
# look inside registry/registry-deployment.yaml
# compare with exported
#    - check volumes - pay attention
kubectl apply -f registry/registry-deployment.yaml
kubectl get pods
kubectl apply -f registry/service-registry.yaml
kubectl get svc
curl $REGISTRY/v2/_catalog
```
##### push busybox image one more time
```
docker push $REGISTRY/busybox
# verify registry persistant
kubectl get pods
kubectl delete pod <registry_pod_name>
curl $REGISTRY/v2/_catalog
```
##### check registry persistent volume content
```
# on your workstation
minikube ssh
sudo su -
cd /data
du -sh
```
##### check registry DNS name from inspection pod ( ubuntu/centos/ whatever)
```
ping registry
curl registry:5000/v2/_catalog
# check DNS settings
cat /etc/resolv.conf
```
##### Problem: your node/nodes can't use local registry, because it's Name is not exposed and not resolvable
##### Note: you still can use ClusterIP
##### Solution: configure minikube node to resolve local registry
##### Note: this will not survice minikube node restart
###### pre: find out your cluster DNS Service
```
kubectl get svc --all-namespaces
```
###### update DNS on master node
```
minikube ssh
sudo su -
# check what you can see in cluster from master node
ping <registry_pod>
ping <registry_cluster_ip>
curl <registry_cluster_ip>:5000/v2/_catalog
nslookup registry
# check DNS settings
cat /etc/resolv.conf
ls -lah /etc/resolv.conf
cat /etc/systemd/resolv.conf
# update DNS settings on master node
# update /etc/systemd/resolv.conf , place actual IPs
DNS=10.96.0.10 10.0.2.3   
Domains=default.svc.cluster.local svc.cluster.local cluster.local
# restart resolved service
systemctl restart systemd-resolved
# verify
cat /etc/resolv.conf
curl registry:5000/v2/_catalog
```
### Flask-Hello k8s app
##### build image, publish in local registry
```
cd ./flask-hello/image
docker build -t flask-hello .
# verify image working
docker run -b -p 5700:5000 flask-hello
# tag
docker tag flask-hello $REGISTRY/flask-hello
# push
docker push $REGISTRY/flask-hello
# verify
curl $REGISTRY/v2/_catalog
```
##### Create app deployment/service files
```
# review stack
less ./flask-hello/deployment-flask-hello.yaml
less ./flask-hello/service-flask-hello.yaml
# create deployments
kubectl apply -f flask-hello/deployment-flask-hello.yaml
kubectl apply flask-hello/service-flask-hello.yaml
# inspecting
kubectl get pods
kubectl get deployments
kubectl get svc
kubectl describe svc flask-hello
# verify, from localworkstation node
curl 192.168.99.100:<nodeport>
# create endless loop in dedicated windows
whire true; do curl 192.168.99.100:<nodeport>; sleep 1; done;
```
###### Update code
```
# add printing hostname and line ending to print
# rebuild image
# verify image locally
# tag image for local registry
# push
# kill all pods
# watch deployment
# watch verifying loop
```
