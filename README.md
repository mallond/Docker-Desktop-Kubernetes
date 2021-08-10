# Docker-Desktop-Kubernetes
Docker Desktop Kubernetes - Persistent Volume and Claim

Steps
```

cmd-> kubectl create -f https://raw.githubusercontent.com/mallond/Docker-Desktop-Kubernetes/main/storageClass.yaml
out: storageclass.storage.k8s.io/my-local-storage created

cmd-> mkdir /mnt/disk/vol1
out: Will list the directory...

cmd-> kubectl create -f https://raw.githubusercontent.com/mallond/Docker-Desktop-Kubernetes/main/persistentVolume.yaml
out: persistentvolume/my-local-pv created

cmd-> kubectl create -f https://raw.githubusercontent.com/mallond/Docker-Desktop-Kubernetes/main/persistentVolumeClaim.yaml
out: persistentvolumeclaim/my-claim created

cmd:> kubectl get pv
out: 
NAME          CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS       REASON   AGE
my-local-pv   500Gi      RWO            Retain           Available           my-local-storage            5m46s



```
