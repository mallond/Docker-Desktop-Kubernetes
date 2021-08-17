# Play With K8s 

Persistent Volume and Claim: Local-Storage  
[Reference Article - Configure POD to use a Persitent Volume Storage - Local ](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)

## Step 1 - Set up the LAB 
```
https://labs.play-with-k8s.com/
```
Step 2 Initializes cluster master node:
```
kubeadm init --apiserver-advertise-address $(hostname -i) --pod-network-cidr 10.5.0.0/16
```
Step 3 Initialize cluster networking:
```
kubectl apply -f https://raw.githubusercontent.com/cloudnativelabs/kube-router/master/daemonset/kubeadm-kuberouter.yaml
```



## Step 2 - Setup Local Shared Folder
```
mkdir /mnt/data
```  
```
chmod 777 /mnt/data
```
```
sh -c "echo 'Hello from Kubernetes storage' > /mnt/data/index.html"
```
```
cat /mnt/data/index.html
```
out: storageclass.storage.k8s.io/my-local-storage created

## Step 3 - Create Presistent Volume and Presistent Volume Claim
```
kubectl apply -f https://raw.githubusercontent.com/mallond/Kubernetes-Labs-PV/main/pv-volume.yaml
```
```
kubectl get pv task-pv-volume
```
```
out:
NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
task-pv-volume   10Gi       RWO            Retain           Available           manual                  21s
```

```
kubectl apply -f https://raw.githubusercontent.com/mallond/Kubernetes-Labs-PV/main/pv-claim.yaml
```
```
kubectl get pv task-pv-volume
```
```
out:
NAME             CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                   STORAGECLASS   REASON   AGE
task-pv-volume   10Gi       RWO            Retain           Bound    default/task-pv-claim   manual                  3m56s
```


## Step 4 Create Pods
```
kubectl get nodes --show-labels
```
```
Note: 

The node affinity will need to match the kubernetes.io/hostname=
below this would be docker-desktop
out: 
NAME             STATUS   ROLES                  AGE     VERSION   LABELS
docker-desktop   Ready    control-plane,master   2d23h   v1.21.2   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=docker-desktop,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node-role.kubernetes.io/master=,node.kubernetes.io/exclude-from-external-load-balancers=

```
```
kubectl create -f https://raw.githubusercontent.com/mallond/Docker-Desktop-Kubernetes/main/persistentVolume.yaml
```  
out: persistentvolume/my-local-pv created

### Step 4 Create Persistent Volume Claim 500Gi
```
kubectl create -f https://raw.githubusercontent.com/mallond/Docker-Desktop-Kubernetes/main/persistentVolumeClaim.yaml
```  
out: persistentvolumeclaim/my-claim created

### Inspect PV
```
kubectl get pv
```  
out:   
```
NAME          CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS       REASON   AGE
my-local-pv   500Gi      RWO            Retain           Available           my-local-storage            5m46s
```

### Step 5 Create PODS
```
kubectl create -f https://raw.githubusercontent.com/mallond/Kubernetes-Labs-PV/main/pv-pod.yaml
```  
```
kubectl get pod task-pv-pod 
```
> GOTCHA - Sometimes the create pods works right from the start; howver, if not DEBUG  

> kubectl get pod task-pv-pod   

> [Taints and Tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)

```
kubectl describe  -f https://k8s.io/examples/pods/storage/pv-pod.yaml
```
```
out:
You will see by the describe that this pod is tainted and needs to have this removed
Events:
  Type     Reason            Age                 From               Message
  ----     ------            ----                ----               -------
  Warning  FailedScheduling  71s (x27 over 28m)  default-scheduler  0/1 nodes are available: 1 node(s) had taint {node-role.kubernetes.io/master: }, that the pod didn't tolerate.

```
```
kubectl taint nodes --all node-role.kubernetes.io/master-
```

Once the taint is removed (if needed) Test your work
```
> kubectl exec -it task-pv-pod -- /bin/bash
# Once in the POD
> apt update
> apt install curl
> curl http://localhost/
# Change the local content
> curl http://localhost/

There you go!

```

# Other
-  gcr.io/google-samples/hello-app:1.0 Great node that will print the pods unique address 
- 
```
apply/create - create resource(s) 
run - start a pod from an image 
explain - documentation of resources 
delete - delete resource(s) 
get - list resources 
describe - detailed resource information 
exec - execute a command on a container 
logs - view logs on a container
```
- https://kubernetes.io/docs/reference/kubectl/kubectl/  
- https://kubernetes.io/docs/reference/kubectl/cheatsheet/  

```
kubectl cluster-info
kubectl get nodes
kubectl get nodes -o wide
kubectl get pods 
kubectl get pods --namespace kube-system
kubectl get pods --namespace kube-system -o wide
kubectl get all --all-namespaces | more
kubectl api-resources | more
kubectl api-resources | grep pod
kubectl explain pod | more 
kubectl explain pod.spec | more 
kubectl explain pod.spec.containers | more 
kubectl explain pod --recursive | more 
kubectl describe nodes c1-cp1 | more 
kubectl describe nodes c1-node1 | more
kubectl -h | more
kubectl get -h | more
kubectl create -h | more

# Hello World Exercise - Muscle memory
kubectl run hello-world-pod --image=gcr.io/google-samples/hello-app:1.0
kubectl get pods
kubectl get pods -o wide
ssh aen@c1-node[XX]
sudo docker ps
kubectl logs hello-world-pod
kubectl exec -it hello-world-pod -- /bin/sh
kubectl get deployment hello-world
kubectl get replicaset
kubectl get pods
kubectl describe deployment hello-world | more
kubectl describe replicaset hello-world | more
kubectl describe pod hello-world-[tab][tab] | more
kubectl expose deployment hello-world \
     --port=80 \
     --target-port=8080
kubectl get service hello-world
kubectl describe service hello-world
curl http://$SERVCIEIP:$PORT
kubectl get endpoints hello-world
curl http://$ENDPOINT:$TARGETORT
kubectl get deployment hello-world -o yaml | more 
kubectl get deployment hello-world -o json | more 
kubectl get all
kubectl delete service hello-world
kubectl delete deployment hello-world
kubectl delete pod hello-world-pod
kubectl get all
kubectl create deployment hello-world \
     --image=gcr.io/google-samples/hello-app:1.0 \
     --dry-run=client -o yaml | more 
kubectl create deployment hello-world \
     --image=gcr.io/google-samples/hello-app:1.0 \
     --dry-run=client -o yaml > deployment.yaml
kubectl apply -f deployment.yaml
kubectl expose deployment hello-world \
     --port=80 --target-port=8080 \
     --dry-run=client -o yaml | more
kubectl expose deployment hello-world \
     --port=80 --target-port=8080 \
     --dry-run=client -o yaml > service.yaml 
kubectl apply -f service.yaml 
kubectl get all

#Scale up our deployment...in code
vi deployment.yaml
Change spec.replicas from 1 to 20
     replicas: 20

kubectl apply -f deployment.yaml
kubectl get deployment hello-world
kubectl get pods | more 
kubectl edit deployment hello-world
kubectl get deployment hello-world
kubectl scale deployment hello-world --replicas=40
kubectl get deployment hello-world

kubectl delete deployment hello-world
kubectl delete service hello-world
kubectl get all

```
