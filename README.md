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
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 \
&& chmod 700 get_helm.sh \
&& ./get_helm.sh
$ helm repo add csi-driver-nfs https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/charts
$ helm install csi-driver-nfs csi-driver-nfs/csi-driver-nfs --namespace kube-system --version v3.2.0
```
```
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

You can find PV was automatically created as below after creating PVC.
```
$ kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                        STORAGECLASS   REASON   AGE
pvc-43e19a03-6332-4fa6-92eb-146d1ad54802   1Gi        RWX            Delete           Bound    default/pvc-nfs-vm-dynamic   nfs-vm-csi              11m
```


# 4. Check Persistent Volume on Ubuntu
```
$ kubectl apply -f ubuntu-csi.yaml
```
```
$ kubectl exec -it ubuntu-csi-58dcbc877f-qtbzn -- mount |grep pvc
192.168.33.11:/pvc-43e19a03-6332-4fa6-92eb-146d1ad54802 on /mnt type nfs4 (rw,relatime,vers=4.2,rsize=524288,wsize=524288,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=192.168.33.104,local_lock=none,addr=192.168.33.11)
```
```
$ kubectl exec -i ubuntu-csi-58dcbc877f-qtbzn -- bash <<EOC
echo \$HOSTNAME \$(date) >> /mnt/ubuntu_in_persistent_volume.txt
EOC

$ kubectl exec -i ubuntu-csi-58dcbc877f-qtbzn -- bash <<EOC
cat /mnt/ubuntu_in_persistent_volume.txt
EOC
ubuntu-csi-58dcbc877f-qtbzn Tue Mar 15 15:20:31 UTC 2022
```

# 5. Delete Pod
```
$ kubectl delete pods ubuntu-csi-58dcbc877f-qtbzn 
pod "ubuntu-csi-58dcbc877f-qtbzn" deleted
```
You can find each pod starting on different node soon.
```
$ kubectl get pods
NAME                               READY   STATUS    RESTARTS      AGE
ubuntu-csi-58dcbc877f-tckf2        2/2     Running   0             52s
```

# 5-1. Check Persistent Volume on Ubuntu again
```
$ kubectl exec -i ubuntu-csi-58dcbc877f-tckf2 -- bash <<EOC
echo \$HOSTNAME \$(date) >> /mnt/ubuntu_in_persistent_volume.txt
EOC

$ kubectl exec -i ubuntu-csi-58dcbc877f-tckf2 -- bash <<EOC
cat /mnt/ubuntu_in_persistent_volume.txt
EOC
ubuntu-csi-58dcbc877f-qtbzn Tue Mar 15 15:20:31 UTC 2022
ubuntu-csi-58dcbc877f-tckf2 Tue Mar 15 15:25:44 UTC 2022
```

# 6. Can Ubuntu-csi2 simultaneously mount the filesystem which is already mounted by Ubuntu-csi?
```
$ kubectl apply -f ubuntu-csi2.yaml
$ kubectl get pods -o wide
NAME                               READY   STATUS    RESTARTS      AGE     IP              NODE      NOMINATED NODE   READINESS GATES
ubuntu-csi-58dcbc877f-tckf2        2/2     Running   0             4m58s   10.10.235.142   worker1   <none>           <none>
ubuntu-csi2-5cdbdccd79-dl6ls       2/2     Running   0             9s      10.10.199.167   worker4   <none>           <none>
ubuntu-csi2-5cdbdccd79-gg446       2/2     Running   0             9s      10.10.189.74    worker2   <none>           <none>
```

Yes, it can.
```
$ kubectl exec -i ubuntu-csi2-5cdbdccd79-dl6ls -- bash <<EOC
cat /mnt/ubuntu_in_persistent_volume.txt
EOC
ubuntu-csi-58dcbc877f-qtbzn Tue Mar 15 15:20:31 UTC 2022
ubuntu-csi-58dcbc877f-tckf2 Tue Mar 15 15:25:44 UTC 2022
```
