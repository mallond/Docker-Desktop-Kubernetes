# Docker-Desktop-Kubernetes
Docker Desktop Kubernetes - Persistent Volume and Claim

## Environment
```
Using PowerShell 7 and Windows Docker Desktop Kubernetes
Kubernetes v1.21.2
PowerShell 7.0.6
Windows 10 Enterprise 19041.1110
Docker Community 20.10.7
Ubuntu 20.x

```

## Steps

### Step 1 Create Storage Class
```
kubectl create -f https://raw.githubusercontent.com/mallond/Docker-Desktop-Kubernetes/main/storageClass.yaml
```  
out: storageclass.storage.k8s.io/my-local-storage created

### Step 2 Create Volume
```
mkdir /mnt/disk/vol1
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
