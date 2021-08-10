# Play With K8s 
Persistent Volume and Claim: Local-Storage

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
```
kubectl taint nodes --all node-role.kubernetes.io/master-
```

