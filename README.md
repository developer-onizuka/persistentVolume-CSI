# persistentVolume-CSI

# Set up a NFS Server on a Kubernetes cluster
> https://github.com/kubernetes-csi/csi-driver-nfs/blob/master/deploy/example/nfs-provisioner/README.md
```
kubectl create -f https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/deploy/example/nfs-provisioner/nfs-server.yaml
```

# Install CSI driver with Helm 3
> https://github.com/kubernetes-csi/csi-driver-nfs/tree/master/charts
```
helm repo add csi-driver-nfs https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/charts
helm install csi-driver-nfs csi-driver-nfs/csi-driver-nfs --namespace kube-system --version v3.1.0
```
```
helm uninstall csi-driver-nfs -n kube-system
```

# Storage Class Usage (Dynamic Provisioning)
> https://github.com/kubernetes-csi/csi-driver-nfs/tree/master/deploy/example
```
# create StorageClass
kubectl create -f https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/deploy/example/storageclass-nfs.yaml

# create PVC
kubectl create -f https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/deploy/example/pvc-nfs-csi-dynamic.yaml
```

# Ubuntu
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ubuntu3
spec:
  selector:
    matchLabels:
      app: ubuntu3
  replicas: 1
  template:
    metadata:
      labels:
        app: ubuntu3
    spec:
      containers:
      - name: ubuntu
        image: ubuntu:20.04
        command:
          - sleep
          - infinity
        volumeMounts:
        - name: ubuntu3-data
          mountPath: /mnt
      volumes:
        - name: ubuntu3-data
          persistentVolumeClaim:
           claimName: pvc-nfs-dynamic
```
