# LAB for taking snapshot

# Question: Create a snapshot of the existing etcd instance running at https://127.0.0.1:2379, saving the snapshot to /var/lib/etcd-snapshot.db.
## Next, restore an existing, previous snapshot located at /var/lib/etcd-snapshot-previous.db.

## Solution, this time using etcd static pod yaml file "/etc/kubernetes/manifests/etcd.yaml"

### https://kubernetes.io/

```
cat /etc/kubernetes/manifests/etcd.yaml 
```
```
ETCDCTL_API=3 etcdctl                      \
--endpoints=https://127.0.0.1:2379         \
--cacert=/etc/kubernetes/pki/etcd/ca.crt   \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key  \
snapshot save /var/lib/etcd-snapshot.db
```

## Now, how to restore it? 
### Let's assume that ETCD data directory is corrupted. In this case, we can restore the snapshot.
### Data directory is the directory, where all the ETCD's configuration and data files are present.
### Check the etcd data directory from ETCD yaml file.

```
cat /etc/kubernetes/manifests/etcd.yaml | egrep "data-dir"
```
``
--data-dir=/var/lib/etcd
``

```
ls -ld /var/lib/etcd
```
### As per the question, a previously taken snapshot is "etcd-snapshot-previous.db", thus, we have to restore it from this file only. And, please note that while restoring the snapshot, we have to use new data directory.

```
ETCDCTL_API=3 etcdctl                        \
--endpoints=https://127.0.0.1:2379           \
--cacert=/etc/kubernetes/pki/etcd/ca.crt     \
--cert=/etc/kubernetes/pki/etcd/server.crt   \
--key=/etc/kubernetes/pki/etcd/server.key    \
snapshot restore etcd-snapshot-previous.db   \
--data-dir="/var/lib/etcd-backup"
```
```
ls -ld /var/lib/etcd-backup
```
```
ls -ld /var/lib/etcd
```

## Now, we have changed the directory "--data-dir", thus we have to change the ownership of this directory also. 

sudo chown -R etcd:etcd /var/lib/etcd-backup
sudo systemctl start etcd

