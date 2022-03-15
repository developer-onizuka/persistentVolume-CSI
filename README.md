# persistentVolume-CSI

# 1. Set up a NFS Server
> https://github.com/kubernetes-csi/csi-driver-nfs/blob/master/deploy/example/nfs-provisioner/README.md
```
$ git clone https://github.com/developer-onizuka/persistentVolume-CSI
$ cd persistentVolume-CSI
$ vagrant up --provider=libvirt
```

# 2. Install CSI driver with Helm 3
> https://github.com/kubernetes-csi/csi-driver-nfs/tree/master/charts
```
$ helm repo add csi-driver-nfs https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/charts
$ helm install csi-driver-nfs csi-driver-nfs/csi-driver-nfs --namespace kube-system --version v3.1.0
```

If you wanna uninstall csi-driver-nfs, then do a following command.
```
$ helm uninstall csi-driver-nfs -n kube-system
```

# 3. Storage Class Usage (Dynamic Provisioning)
This step is for Dynamic Provisioning, so you don't need PV. It will be created autonomously.
> https://github.com/kubernetes-csi/csi-driver-nfs/tree/master/deploy/example
```
$ kubectl create -f storageclass-vm-nfs.yaml
$ kubectl create -f pvc-nfs-vm-csi-dynamic.yaml
```

# 4. Check Persistent Volume on Ubuntu
```
$ kubectl apply -f ubuntu3.yaml
```
