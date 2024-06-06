### 1. Ceph 集群 Monitors 的 IP 地址（MONITORS）

Ceph monitors (MONs) 的IP地址可以通过以下命令获取：

`kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- ceph mon dump`

**[root@master rook-ceph]# kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- ceph mon dump**

epoch 3

fsid 98946052-26a9-49b5-911e-0e2ffb6f9bb7

last_changed 2024-06-04T07:51:51.672013+0000

created 2024-06-04T07:32:07.604477+0000

min_mon_release 17 (quincy)

election_strategy: 1

0: [v2:10.233.18.223:3300/0,v1:10.233.18.223:6789/0] mon.a

1: [v2:10.233.23.205:3300/0,v1:10.233.23.205:6789/0] mon.b

2: [v2:10.233.2.106:3300/0,v1:10.233.2.106:6789/0] mon.c

此时，存储类monIP为：10.1.1.1:6789,10.1.1.2:6789,10.1.1.3:6789

## kubectl apply -f storageclass-rbd.yaml
创建pvc.xml

    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: test-pvc
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 1Gi
      storageClassName: rook-ceph-block
**[root@master data]# kubectl apply -f pvc.yaml**
**[root@master data]# kubectl get pvc**

NAME       STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE  
test-pvc   Bound    pvc-35e5bfa8-a707-4827-888c-945bb7ebd55d   1Gi        RWO            rook-ceph-block   32s

创建pod.xml

    apiVersion: v1
    kind: Pod
    metadata:
      name: test-pod
    spec:
      containers:
      - name: test-container
        image: busybox
        command: [ "sh", "-c", "sleep 3600" ]
        volumeMounts:
        - mountPath: "/mnt/test"
          name: test-volume
      volumes:
      - name: test-volume
        persistentVolumeClaim:
          claimName: test-pvc
**[root@master data]# kubectl apply -f pod.yaml**
**[root@master data]# kubectl get pod**
NAME       READY   STATUS    RESTARTS   AGE
test-pod   1/1     Running   0          9s
表示ceph部署成功！此时查看osd状态：

**[root@master data]# kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- ceph osd status**

ID  HOST     USED  AVAIL  WR OPS  WR DATA  RD OPS  RD DATA  STATE      
 0  master  49.9M  39.9G      0        0       0        0   exists,up  
 1  node1   49.9M  39.9G      0     5734       0        0   exists,up  
 2  node2   49.9M  39.9G      0        0       0        0   exists,up
