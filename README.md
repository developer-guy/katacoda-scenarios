## How to connect with Walid?

Social networks:

  - issues/pull requests here GitHub
  - https://www.linkedin.com/in/walidshaari/
  - https://twitter.com/walidshaari

<img src="./images/CNJ.png" width="450"/>

# Interactive Katacoda Scenarios

[![](http://shields.katacoda.com/katacoda/walidshaari/count.svg)](https://www.katacoda.com/walidshaari "Get your profile on Katacoda.com")

Visit https://www.katacoda.com/walidshaari to view the profile and interactive scenarios

### Writing Scenarios
Visit https://www.katacoda.com/docs to learn more about creating Katacoda scenarios

For examples, visit https://github.com/katacoda/scenario-example


## Update on KataCoda (in-progress)

As I had issues with limits with katacoda.com and was not able to create these scenarios on katacod due to these limits and the lack of support, I have created the below file with hints regardin these area of the CKS exam, will work on Katacoda or Instruqt. [Instruqt](https://instruqt.com/katacoda-alternative-flexible-content-creation/) is quite promising as it has more flexibility in how your create your enviornment and its life cycle, however needs a sponsorship. if you are interested in creating such a content in Instruqt, and would like to help, let me know 


- video of the [Exam environment preview](https://www.youtube.com/watch?v=9UqkWcdy140)

   ![exam environment](images/ExamEnvPreview.png)

- practice on killer.sh when you are ready before the exam at least a week

   ![killer.sh](images/killer.shell-include.png)
   
 

- Ensure you address the right namespace and cluster always
- KISS:  Do not overthink it,start with the basics, do your backups, and remmber what you practiced.
- Get familiar with the options you need and files
- spelling mistakes
- Focused troubleshooting: for static manifests: `journalctl -u kubelet`  or for a service like falco `journalctl -u falco`
  - problems with api-server after new configs: check logs in `cat /var/log/pods`  or `/var/log/containers` or `crictl ps -a`  or `crictl logs`
- Make sure you backup the apiserver before working on it
  - and try to break it and see errors and you investigate them  https://itnext.io/cks-exam-series-4-crash-that-apiserver-5f4d3d503028  and #Klustered might help
- The exam environments comes ready with auto-completion and all command line tools you need, try to capatilize that, if you can do it from the command line, no     need to visit that web page, if you need that webpage, make sure you already know where in the web page its is what you need, you can bookmark if you wish.

## The mindset: What to expect in CKA

 -  the course curiculim, FAQ
 -  other webinars to watch
 -  Courses
 - Tips and tricks

## etcd
- Documentation:  
   - https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster
   - https://etcd.io/docs/v3.5/op-guide/recovery/#restoring-a-cluster
- Why etcd backup/restore?
  point of time copy of the cluster state
- is it enough?
-  Day 2 operations, required for resiliency, done periodically and before upgrades or maintenance updates
-  Componenets
   - command line tool: etcdctl
     `ubectl exec -n kube-system -it etcd-control-plane -- etcdctl version`
   - hostPath volume  /var/lib/etcd
   - ETCD Certfificates:  CA, Server cert and key
   - etcd static pod manifest: /etc/kubernetes/manifests/etcd.yaml
- etcdctl --help
```
  root@control-plane:~# kubectl exec -n kube-system -it etcd-control-plane -- etcdctl --help|grep snapshot
	snapshot restore	Restores an etcd member snapshot to an etcd directory
	snapshot save		Stores an etcd node backend snapshot to a given file
	snapshot status		Gets backend snapshot status of a given file
  
  
  kubectl exec -n kube-system -it etcd-control-plane -- etcdctl \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt  \
  --key=/etc/kubernetes/pki/etcd/server.key \
  --write-out=table \
  member list
  
  +-----------------+---------+---------------+------------------------+------------------------+------------+
  |       ID        | STATUS  |     NAME      |       PEER ADDRS       |      CLIENT ADDRS      | IS LEARNER |
  +-----------------+---------+---------------+------------------------+------------------------+------------+
  | 3b17aaa147134dd | started | control-plane | https://10.0.0.10:2380 | https://10.0.0.10:2379 |      false |
  +-----------------+---------+---------------+------------------------+------------------------+------------+
  
## if default --write-out it would be simple like below
###  https://stackoverflow.com/questions/63433622/is-the-following-output-of-etcdctl-member-list-correct-and-etcd-cluster-is-in
3b17aaa147134dd, started, control-plane, https://10.0.0.10:2380, https://10.0.0.10:2379, false

#####

```
- Backup a snapshot of etcd:  `etcdctl --endpoints snapshot save <filename>` 

```bash

################################  backup #################
# kubectl exec -n kube-system -it etcd-control-plane -- etcdctl  \
--cacert=/etc/kubernetes/pki/etcd/ca.crt  \
--cert=/etc/kubernetes/pki/etcd/server.crt  \
--key=/etc/kubernetes/pki/etcd/server.key  \
snapshot save /var/lib/etcd-backup

{"level":"info","ts":1627240895.1983745,"caller":"snapshot/v3_snapshot.go:119","msg":"created temporary db file","path":"/var/lib/etcd-backup.part"}
{"level":"info","ts":"2021-07-25T19:21:35.212Z","caller":"clientv3/maintenance.go:200","msg":"opened snapshot stream; downloading"}
{"level":"info","ts":1627240895.2146122,"caller":"snapshot/v3_snapshot.go:127","msg":"fetching snapshot","endpoint":"127.0.0.1:2379"}
{"level":"info","ts":"2021-07-25T19:21:35.325Z","caller":"clientv3/maintenance.go:208","msg":"completed snapshot read; closing"}
{"level":"info","ts":1627240895.3396266,"caller":"snapshot/v3_snapshot.go:142","msg":"fetched snapshot","endpoint":"127.0.0.1:2379","size":"5.2 MB","took":0.141213026}
{"level":"info","ts":1627240895.339843,"caller":"snapshot/v3_snapshot.go:152","msg":"saved","path":"/var/lib/etcd-backup"}
Snapshot saved at /var/lib/etcd-backup


# kubectl exec -n kube-system -it etcd-control-plane -- etcdctl   --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot status /var/lib/etcd-backup
  ef379630, 113704, 901, 5.2 MB

# kubectl exec -n kube-system -it etcd-control-plane -- etcdctl   --cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
snapshot status --write-out=table /var/lib/etcd-backup

+----------+----------+------------+------------+
|   HASH   | REVISION | TOTAL KEYS | TOTAL SIZE |
+----------+----------+------------+------------+
| ef379630 |   113704 |        901 |     5.2 MB |
+----------+----------+------------+------------+

```

- Check backup:  
```
etcdctl snapshot status  --write-out=table <backup-filename> 
```
- Restore a snapshot of etcd: 
  - restore backup to another location   
    - you can use the new location if you modify the etcd datadirectory configuration in the etcd pod manifest 
  - move the original etcd cluster data out of the way
  - stop etcd
  - move the restored data to the etcd datadirectory  /var/lib/etcd
  - startup etcd
- 
- `etcdctl --endpoints snapshot restore <filename>` 
```

$ etcdctl version

$ kubectl get pods -n kube-system|grep etcd
  etcd-control-plane                         1/1     Running   1          11d

$ kubectl describe -n kube-system pod etcd-control-plane 

Name:                 etcd-control-plane
Namespace:            kube-system
Priority:             2000001000
Priority Class Name:  system-node-critical
Node:                 control-plane/10.0.0.10
Start Time:           Sat, 24 Jul 2021 12:17:21 +0000
Labels:               component=etcd
                      tier=control-plane
Annotations:          kubeadm.kubernetes.io/etcd.advertise-client-urls: https://10.0.0.10:2379
                      kubernetes.io/config.hash: 9d7ebe35aa278df29e333e9778092061
                      kubernetes.io/config.mirror: 9d7ebe35aa278df29e333e9778092061
                      kubernetes.io/config.seen: 2021-07-13T11:57:12.197720051Z
                      kubernetes.io/config.source: file
Status:               Running
IP:                   10.0.0.10
IPs:
  IP:           10.0.0.10
Controlled By:  Node/control-plane
Containers:
  etcd:
    Container ID:  containerd://8ea4561c3d1cfe3b6f8bf119e16b23237d222b7d2bd2fd84e5f9260a70042b9d
    Image:         k8s.gcr.io/etcd:3.4.13-0
    Image ID:      k8s.gcr.io/etcd@sha256:4ad90a11b55313b182afc186b9876c8e891531b8db4c9bf1541953021618d0e2
    Port:          <none>
    Host Port:     <none>
    Command:
      etcd
      --advertise-client-urls=https://10.0.0.10:2379
      --cert-file=/etc/kubernetes/pki/etcd/server.crt
      --client-cert-auth=true
      --data-dir=/var/lib/etcd
      --initial-advertise-peer-urls=https://10.0.0.10:2380
      --initial-cluster=control-plane=https://10.0.0.10:2380
      --key-file=/etc/kubernetes/pki/etcd/server.key
      --listen-client-urls=https://127.0.0.1:2379,https://10.0.0.10:2379
      --listen-metrics-urls=http://127.0.0.1:2381
      --listen-peer-urls=https://10.0.0.10:2380
      --name=control-plane
      --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
      --peer-client-cert-auth=true
      --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
      --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
      --snapshot-count=10000
      --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
    State:          Running
      Started:      Sat, 24 Jul 2021 12:17:22 +0000
    Last State:     Terminated
      Reason:       Unknown
      Exit Code:    255
      Started:      Tue, 13 Jul 2021 11:56:58 +0000
      Finished:     Sat, 24 Jul 2021 12:17:09 +0000
    Ready:          True
    Restart Count:  1
    Requests:
      cpu:                100m
      ephemeral-storage:  100Mi
      memory:             100Mi
    Liveness:             http-get http://127.0.0.1:2381/health delay=10s timeout=15s period=10s #success=1 #failure=8
    Startup:              http-get http://127.0.0.1:2381/health delay=10s timeout=15s period=10s #success=1 #failure=24
    Environment:          <none>
    Mounts:
      /etc/kubernetes/pki/etcd from etcd-certs (rw)
      /var/lib/etcd from etcd-data (rw)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  etcd-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/kubernetes/pki/etcd
    HostPathType:  DirectoryOrCreate
  etcd-data:
    Type:          HostPath (bare host directory volume)
    Path:          /var/lib/etcd
    HostPathType:  DirectoryOrCreate
QoS Class:         Burstable
Node-Selectors:    <none>
Tolerations:       :NoExecute op=Exists
Events:            <none>

###################################################
# ETCD Network ports
# hostNetwork: true

root@control-plane:~# ss -tulpn|grep 2379
tcp   LISTEN   0        128              127.0.0.1:2379           0.0.0.0:*      users:(("etcd",pid=2047,fd=6))                                                 
tcp   LISTEN   0        128              10.0.0.10:2379           0.0.0.0:*      users:(("etcd",pid=2047,fd=5))                                                 
root@control-plane:~# ss -tulpn|grep 2380
tcp   LISTEN   0        128              10.0.0.10:2380           0.0.0.0:*      users:(("etcd",pid=2047,fd=3)) 


#########################
# get ETCD configuration paramters othern than "kubectl pod describe/view etcd manifest"
$ ps auxw|grep 'etcd '|grep -v grep
root      2047  1.8  3.5 10612728 72536 ?      Ssl  12:17   4:55 etcd --advertise-client-urls=https://10.0.0.10:2379 --cert-file=/etc/kubernetes/pki/etcd/server.crt --client-cert-auth=true --data-dir=/var/lib/etcd --initial-advertise-peer-urls=https://10.0.0.10:2380 --initial-cluster=control-plane=https://10.0.0.10:2380 --key-file=/etc/kubernetes/pki/etcd/server.key --listen-client-urls=https://127.0.0.1:2379,https://10.0.0.10:2379 --listen-metrics-urls=http://127.0.0.1:2381 --listen-peer-urls=https://10.0.0.10:2380 --name=control-plane --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt --peer-client-cert-auth=true --peer-key-file=/etc/kubernetes/pki/etcd/peer.key --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt --snapshot-count=10000 --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt



##################################


# No need to set the API env varaible ETCDCTL_API to 3, it defaults to 3 now  
etcdctl version: 3.4.10
API version: 3.4

a key to verifiy backup
etcdctl  get cluster.name --endpoints=https://10.0.1.101:2379 --cacert=./etcd-ca.pem --cert=etcd-server.crt --key=etcd-server.key
cluster.name
beebox




etcdctl snapshot save /home/cloud_user/etcd_backup.db  --endpoints=https://10.0.1.101:2379 --cacert=./etcd-ca.pem --cert=etcd-server.crt --key=etcd-server.key
sudo etcdctl snapshot restore  /home/cloud_user/etcd_backup.db  --initial-cluster etcd-restore=https://10.0.1.101:2380 --initial-advertise-peer-urls https://10.0.1.101:2380 --name etcd-restore --data-dir /var/lib/etcd
sudo chown etcd:etcd  /var/lib/etcd/



```
## Todo:
- test vagrant in different Oss and after a destory
- backup etcd CronJob
- secure your backups (encyrpt them and store in a secure location), for DR ensure you have a copy offsite

 
  

