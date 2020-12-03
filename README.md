## Create pool and ceph info
On ceph server create pool
```bash
# exec to ceph shell
sudo /usr/local/bin/cephadm shell
# create pool
ceph osd pool create cl3-k8s 16 replicated

# create user for ceph k8s
ceph auth get-or-create client.cl3-k8s mon 'profile rbd' osd 'profile rbd pool=cl3-k8s' mgr 'profile rbd pool=cl3-k8s'

```
Store password for user
```
[client.<<UserName>>]
	key = <<UserPass>>
```

Get info about cluster
```
ceph mon dump
dumped monmap epoch 6
epoch 6
fsid <<ClusterID>>
last_changed 2020-09-30T23:44:31.786016+0000
created 2020-07-21T05:42:49.553747+0000
min_mon_release 15 (octopus)
0: [v2:10.94.0.13:3300/0,v1:<<mon1:port>>/0] mon.super3
1: [v2:10.94.0.11:3300/0,v1:<<mon2:port>>/0] mon.super1
```
## Prepare K8s conf
Files are in k8s-cfg replaced element
```
grep "<<" *
csi-config-map.yaml:        "clusterID": "<<ClusterID>>",
csi-config-map.yaml:          "<<mon1:port>>",
csi-config-map.yaml:          "<<mon2:port>>"
csi-rbd-sc.yaml:   clusterID: <<ClusterID>>
csi-rbd-sc.yaml:   pool: <<PoolName>>
csi-rbd-secret.yaml:  userID: <<UserName>>
csi-rbd-secret.yaml:  userKey: <<UserPass>>
```

## install to cluster
```
kubectl apply -f csi-config-map.yaml
kubectl apply -f csi-rbd-secret.yaml
kubectl apply -f https://raw.githubusercontent.com/ceph/ceph-csi/master/deploy/rbd/kubernetes/csi-provisioner-rbac.yaml
kubectl apply -f https://raw.githubusercontent.com/ceph/ceph-csi/master/deploy/rbd/kubernetes/csi-nodeplugin-rbac.yaml
kubectl apply -f csi-rbdplugin-provisioner.yaml
kubectl apply -f csi-rbdplugin.yaml
kubectl apply -f csi-rbd-sc.yaml
kubectl apply -f kms-config.yaml
```

## deploy test pod with pvc
```
kubectl apply -f pod.yaml
kubectl apply -f pvc.yaml
```

## Check disk in ceph pool
```
# list of all images (files)
rbd ls -p cl1-k8s
# get img info
rbd info -p cl1-k8s csi-vol-5f4dc0e2-34c2-11eb-8db9-a6881a2c9063
```