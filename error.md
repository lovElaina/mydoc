## 关于apply cluster.yml时，出现osd缺失的问题
硬盘需要满足条件：未格式化，未挂载，缺一不可
如果已经格式化并挂载，在每个节点执行以下命令：
**umount /mnt/data
wipefs -a /dev/vdb1**
此时使用lsblk命令查看硬盘情况，应该如下所示：
[root@master rook-ceph]# lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sr0     11:0    1 223.6M  0 rom  
vda    253:0    0    20G  0 disk 
└─vda1 253:1    0    20G  0 part /
vdb    253:16   0    40G  0 disk 
└─vdb1 253:17   0    40G  0 part 
此外，在cluster.yaml文件中，不要手动分区，并且将
**databaseSizeMB: "1024"** # uncomment if the disks are smaller than 100 GB，取消注释。
如果一切执行顺利，使用kubectl命令检查pods状态
**[root@master rook-ceph]# kubectl get pods -n rook-ceph**
NAME                                               READY   STATUS      RESTARTS   AGE
csi-cephfsplugin-5v55s                             2/2     Running     0          4h30m
csi-cephfsplugin-bcnrf                             2/2     Running     0          5h18m
csi-cephfsplugin-dxclh                             2/2     Running     0          5h18m
csi-cephfsplugin-provisioner-6f5d88b7ff-ql7mr      5/5     Running     0          5h18m
csi-cephfsplugin-provisioner-6f5d88b7ff-sptk8      5/5     Running     0          5h18m
csi-rbdplugin-l986f                                2/2     Running     0          5h18m
csi-rbdplugin-provisioner-57f5ddbd7-6sfp2          5/5     Running     0          5h18m
csi-rbdplugin-provisioner-57f5ddbd7-j2bt2          5/5     Running     0          5h18m
csi-rbdplugin-rcjzp                                2/2     Running     0          4h30m
csi-rbdplugin-zpcg8                                2/2     Running     0          5h18m
rook-ceph-crashcollector-master-86cd877564-2sfqp   1/1     Running     0          4h30m
rook-ceph-crashcollector-node1-6d84b896d9-kbxgn    1/1     Running     0          25s
rook-ceph-crashcollector-node2-64964c9cd4-qzs6s    1/1     Running     0          22s
rook-ceph-mgr-a-74647647c5-4c8k6                   3/3     Running     0          4h44m
rook-ceph-mgr-b-7c6744684d-4wmsv                   3/3     Running     0          4h44m
rook-ceph-mon-a-64dc994ddc-2dzwk                   2/2     Running     0          4h50m
rook-ceph-mon-b-c8bc85f4f-856t9                    2/2     Running     0          4h49m
rook-ceph-mon-c-94b97cbff-qz6bm                    2/2     Running     0          4h31m
rook-ceph-operator-585f6875d-5dhvq                 1/1     Running     0          5h23m
rook-ceph-osd-0-7d9cfb9cc5-hlnht                   2/2     Running     0          28s
rook-ceph-osd-1-7bb6b4ffb5-9fq2h                   2/2     Running     0          25s
rook-ceph-osd-2-789d8f854b-bg9jn                   2/2     Running     0          22s
rook-ceph-osd-prepare-master-6gw45                 0/1     Completed   0          35s
rook-ceph-osd-prepare-node1-5f7hn                  0/1     Completed   0          32s
rook-ceph-osd-prepare-node2-p6bsv                  0/1     Completed   0          29s
rook-ceph-tools-598b59df89-cxqhb                   1/1     Running     0          5h10m


**[root@master rook-ceph]# kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- ceph -s**
  cluster:
    id:     98946052-26a9-49b5-911e-0e2ffb6f9bb7
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum a,b,c (age 4h)
    mgr: a(active, since 17m), standbys: b
    osd: 3 osds: 3 up (since 23s), 3 in (since 41s)
 
  data:
    pools:   2 pools, 2 pgs
    objects: 3 objects, 449 KiB
    usage:   89 MiB used, 120 GiB / 120 GiB avail
    pgs:     2 active+clean
    
使用ceph osd status命令，检查健康状态
**[root@master rook-ceph]# kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- ceph osd status**
ID  HOST     USED  AVAIL  WR OPS  WR DATA  RD OPS  RD DATA  STATE      
 0  master  30.6M  39.9G      0        0       0        0   exists,up  
 1  node1   30.5M  39.9G      0        0       0        0   exists,up  
 2  node2   30.5M  39.9G      0        0       0        0   exists,up  
