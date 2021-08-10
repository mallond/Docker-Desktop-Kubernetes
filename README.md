# Play With K8s 
Persistent Volume and Claim: Local-Storage

## Set up the LAB 
Step 1 
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

## Steps

### Step 1 Setup Local Shared Folder
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

### Step 2 Create Presistent Volume and Presistent Volume Claim
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

out: Will list the directory...

### Step 3 Create Persistent Volume 500Gi
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
kubectl create -f  https://raw.githubusercontent.com/mallond/Docker-Desktop-Kubernetes/main/http-pod.yaml
```  
out: pod/www created  
```
kubectl get pods
```
