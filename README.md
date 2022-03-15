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
$ kubectl get pods -n kube-system |grep csi
csi-nfs-controller-7c4498c745-94xl9        3/3     Running   0             19s
csi-nfs-controller-7c4498c745-c2sgb        3/3     Running   0             18s
csi-nfs-node-59sqr                         3/3     Running   0             19s
csi-nfs-node-5zxpl                         3/3     Running   0             19s
csi-nfs-node-kg5rj                         3/3     Running   0             19s
csi-nfs-node-t7sls                         3/3     Running   0             19s
csi-nfs-node-z9xk7                         3/3     Running   0             19s
```

If you wanna uninstall csi-driver-nfs, then do a following command.
```
$ helm uninstall csi-driver-nfs -n kube-system
```

# 3. Storage Class Usage (Dynamic Provisioning)
This step is for Dynamic Provisioning, so you don't need PV. It will be created autonomously.
> https://github.com/kubernetes-csi/csi-driver-nfs/tree/master/deploy/example
```
$ git clone https://github.com/developer-onizuka/persistentVolume-CSI
$ cd persistentVolume-CSI
$ kubectl create -f storageclass-vm-nfs.yaml
$ kubectl create -f pvc-nfs-vm-csi-dynamic.yaml
```
```
$ kubectl get sc
NAME         PROVISIONER      RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
nfs-vm-csi   nfs.csi.k8s.io   Delete          Immediate           false                  9s

$ kubectl get pvc
NAME                 STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc-nfs-vm-dynamic   Bound    pvc-43e19a03-6332-4fa6-92eb-146d1ad54802   1Gi        RWX            nfs-vm-csi     4s
```

# 4. Check Persistent Volume on Ubuntu
```
$ kubectl apply -f ubuntu3.yaml
```
