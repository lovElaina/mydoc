### K8S & kubesphere & ceph 部署文档

- 购买三个云服务器，在同一区域，可以实现内网互通，分别为master、node1、node2。

配置信息如下：

![202406240855946](https://aemon-1309182592.cos.ap-hongkong.myqcloud.com/202406240855946.png)

实例列表如下，各实例均为30G系统盘，120G数据盘，数据盘不要分区：

![image-20240624141543244](https://aemon-1309182592.cos.ap-hongkong.myqcloud.com/image-20240624141543244.png)



> [!NOTE]
>
> 能使用 KubeKey 安装的 Kubernetes 版本与 KubeSphere 3.4 支持的 Kubernetes 版本不同。如需在现有 Kubernetes 集群上安装 KubeSphere 3.4， Kubernetes 版本必须为 v1.21.x、v1.22.x、v1.23.x、* v1.24.x、* v1.25.x 和 * v1.26.x。
>
> 带星号的版本可能出现边缘节点部分功能不可用的情况。为了避免兼容性问题，建议安装 v1.23.x 版本的 Kubernetes。

- 在https://github.com/kubesphere/kubekey/releases页面下载kubekey-v3.1.1-linux-amd64.tar.gz，上传至master服务器。

- 解压文件**tar -xzvf kubekey-v3.1.1-linux-amd64.tar.gz**，得到二进制可执行文件kk。

- 执行./kk create config [--with-kubernetes version] [--with-kubesphere version] [(-f | --file) path]，如果不指定 Kubernetes 版本，KubeKey 将默认安装 Kubernetes v1.23.15。例如：**./kk create config --with-kubesphere 3.4.1**

- 执行上述命令后，将会在当前目录下，创建默认文件 config-sample.yaml。

- 调整host文件为云服务器配置，注意hosts下的内网ip地址、用户名、密码，并且etcd和control-plane均需要使用master节点。

  ![image-20240624141827829](https://aemon-1309182592.cos.ap-hongkong.myqcloud.com/image-20240624141827829.png)

另外由于需要启用devops，最好在安装之前就修改配置文件，如果等kubesphere安装完成后再修改配置文件，可能出现错误。

![image-20240624142021635](https://aemon-1309182592.cos.ap-hongkong.myqcloud.com/image-20240624142021635.png)

- 启用应用商店

![image-20240625091152744](https://aemon-1309182592.cos.ap-hongkong.myqcloud.com/image-20240625091152744.png)

- 编辑完成配置文件后，执行 **./kk create cluster -f config-sample.yaml**
- 执行上述命令后，首先会检测缺少的依赖项。一般来说会缺少conntrack（连接管理工具）和socat（双向数据传输工具），需要在三个节点上均安装他们。使用命令 **yum install -y conntrack  socat**

- 通过下面命令来查看哪些pod启动了：**kubectl get pod -A**

![image-20240624143921715](https://aemon-1309182592.cos.ap-hongkong.myqcloud.com/image-20240624143921715.png)

不应出现Running和Completed之外的其他状态。如图，现在安装顺利完成，可以可以正常启动kubesphere。

使用主节点的地址，加上默认端口30880，例如http://43.135.88.244:30880/ 访问kubesphere控制台。 注意，**初始用户名为admin，初始密码为P@88w0rd。**

出现以下页面，并可顺利登录进入控制台，说明kubesphere和k8s安装完成。

![image-20240624144334388](https://aemon-1309182592.cos.ap-hongkong.myqcloud.com/image-20240624144334388.png)

#### 部署ceph

- 在此之前，需要清除master节点上的污点。

**kubectl taint nodes master node-role.kubernetes.io/master-**

- 下载rook-ceph文件，并提取核心文件到自己的部署文件夹

**cd /data/**

**yum install -y git**

**git clone --single-branch --branch v1.11.6 https://github.com/rook/rook.git**

- 提取部署文件

**mkdir -p /data/rook-ceph/**

**cp /data/rook/deploy/examples/crds.yaml /data/rook-ceph/crds.yaml**

**cp /data/rook/deploy/examples/common.yaml /data/rook-ceph/common.yaml**

**cp /data/rook/deploy/examples/operator.yaml /data/rook-ceph/operator.yaml**

**cp /data/rook/deploy/examples/cluster.yaml /data/rook-ceph/cluster.yaml**

**cp /data/rook/deploy/examples/filesystem.yaml /data/rook-ceph/filesystem.yaml**

**cp /data/rook/deploy/examples/toolbox.yaml /data/rook-ceph/toolbox.yaml**

**cp /data/rook/deploy/examples/csi/rbd/storageclass.yaml /data/rook-ceph/storageclass-rbd.yaml**

**cp /data/rook/deploy/examples/csi/cephfs/storageclass.yaml /data/rook-ceph/storageclass-cephfs.yaml**

**cp /data/rook/deploy/examples/csi/nfs/storageclass.yaml /data/rook-ceph/storageclass-nfs.yaml**

**cd /data/rook-ceph**

- 部署operator

> [!NOTE]
>
> 修改镜像仓库信息，operator.yaml中镜像仓库修改为阿里云的镜像仓库配置（注意：如果服务器是海外，就不需要设置这些了）
>
> ROOK_CSI_CEPH_IMAGE: "quay.io/cephcsi/cephcsi:v3.8.0"
>
> ROOK_CSI_REGISTRAR_IMAGE: "registry.cn-hangzhou.aliyuncs.com/google_containers/csi-node-driver-registrar:v2.7.0"
>
> ROOK_CSI_RESIZER_IMAGE: "registry.cn-hangzhou.aliyuncs.com/google_containers/csi-resizer:v1.7.0"
>
> ROOK_CSI_PROVISIONER_IMAGE: "registry.cn-hangzhou.aliyuncs.com/google_containers/csi-provisioner:v3.4.0"
>
> ROOK_CSI_SNAPSHOTTER_IMAGE: "registry.cn-hangzhou.aliyuncs.com/google_containers/csi-snapshotter:v6.2.1"
>
> ROOK_CSI_ATTACHER_IMAGE: "registry.cn-hangzhou.aliyuncs.com/google_containers/csi-attacher:v4.1.0"

- 执行部署

**cd /data/rook-ceph**

**kubectl create -f crds.yaml**

**kubectl create -f common.yaml**

**kubectl create -f operator.yaml**

- 检查operator的创建运行状态

**kubectl -n rook-ceph get pod**

- 输出

    NAME                                 READY   STATUS    RESTARTS   AGE
    rook-ceph-operator-585f6875d-qjhdn   1/1     Running   0          4m36s

- 创建ceph集群

修改cluster.yaml：

![image-20240625145758710](https://aemon-1309182592.cos.ap-hongkong.myqcloud.com/image-20240625145758710.png)

**databaseSizeMB: "1024"** # uncomment if the disks are smaller than 100 GB，如果数据盘小于100GB，取消注释。

- 执行部署cluster-test.yaml：

**kubectl create -f cluster.yaml**

等待一段时间

查看部署结果，当全部为Running之后部署工具容器进行集群确认
**kubectl -n rook-ceph get pod**

    NAME                                               READY   STATUS      RESTARTS   AGE
    csi-cephfsplugin-5v55s                             2/2     Running     0          6h23m
    csi-cephfsplugin-bcnrf                             2/2     Running     0          7h11m
    csi-cephfsplugin-dxclh                             2/2     Running     0          7h11m
    csi-cephfsplugin-provisioner-6f5d88b7ff-ql7mr      5/5     Running     0          7h11m
    csi-cephfsplugin-provisioner-6f5d88b7ff-sptk8      5/5     Running     0          7h11m
    csi-rbdplugin-l986f                                2/2     Running     0          7h11m
    csi-rbdplugin-provisioner-57f5ddbd7-6sfp2          5/5     Running     0          7h11m
    csi-rbdplugin-provisioner-57f5ddbd7-j2bt2          5/5     Running     0          7h11m
    csi-rbdplugin-rcjzp                                2/2     Running     0          6h23m
    csi-rbdplugin-zpcg8                                2/2     Running     0          7h11m
    rook-ceph-crashcollector-master-86cd877564-2sfqp   1/1     Running     0          6h23m
    rook-ceph-crashcollector-node1-6d84b896d9-kbxgn    1/1     Running     0          113m
    rook-ceph-crashcollector-node2-64964c9cd4-qzs6s    1/1     Running     0          113m
    rook-ceph-mgr-a-74647647c5-4c8k6                   3/3     Running     0          6h37m
    rook-ceph-mgr-b-7c6744684d-4wmsv                   3/3     Running     0          6h37m
    rook-ceph-mon-a-64dc994ddc-2dzwk                   2/2     Running     0          6h43m
    rook-ceph-mon-b-c8bc85f4f-856t9                    2/2     Running     0          6h43m
    rook-ceph-mon-c-94b97cbff-qz6bm                    2/2     Running     0          6h24m
    rook-ceph-operator-585f6875d-5dhvq                 1/1     Running     0          7h16m
    rook-ceph-osd-0-7d9cfb9cc5-hlnht                   2/2     Running     0          113m
    rook-ceph-osd-1-7bb6b4ffb5-9fq2h                   2/2     Running     0          113m
    rook-ceph-osd-2-789d8f854b-bg9jn                   2/2     Running     0          113m
    rook-ceph-osd-prepare-master-6gw45                 0/1     Completed   0          113m
    rook-ceph-osd-prepare-node1-5f7hn                  0/1     Completed   0          113m
    rook-ceph-osd-prepare-node2-p6bsv                  0/1     Completed   0          113m
    rook-ceph-tools-598b59df89-cxqhb                   1/1     Running     0          7h3m

创建工具容器，检查集群状态

**kubectl apply -f toolbox.yaml**

执行命令查询

**kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- ceph -s**

输出：

      cluster:
        id:     98946052-26a9-49b5-911e-0e2ffb6f9bb7
        health: HEALTH_OK
     
      services:
        mon: 3 daemons, quorum a,b,c (age 6h)
        mgr: a(active, since 2h), standbys: b
        osd: 3 osds: 3 up (since 114m), 3 in (since 114m)
     
      data:
        pools:   2 pools, 33 pgs
        objects: 20 objects, 27 MiB
        usage:   193 MiB used, 120 GiB / 120 GiB avail
        pgs:     33 active+clean
     
      io:
        client:   4.0 KiB/s wr, 0 op/s rd, 0 op/s wr

![image-20240625113757026](https://aemon-1309182592.cos.ap-hongkong.myqcloud.com/image-20240625113757026.png)

如上图，是三个120GB数据盘节点，安装ceph成功后的结果。

或进入工具容器内执行命令查看集群状态

**kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- bash**

准备dashboard的nodeport端口映射服务

    cat > /data/rook-ceph/dashboard-external-https.yaml <<EOF
    apiVersion: v1
    kind: Service
    metadata:
      name: rook-ceph-mgr-dashboard-external-https
      namespace: rook-ceph
      labels:
        app: rook-ceph-mgr
        rook_cluster: rook-ceph
    spec:
      ports:
      - name: dashboard
        port: 8443
        protocol: TCP
        targetPort: 8443
        nodePort: 30808
      selector:
        app: rook-ceph-mgr
        rook_cluster: rook-ceph
      sessionAffinity: None
      type: NodePort
    
    EOF

这里的nodeport端口可更换

**kubectl apply -f dashboard-external-https.yaml**

**service/rook-ceph-mgr-dashboard-external-https created**

获取admin用户密码

**kubectl -n rook-ceph get secret rook-ceph-dashboard-password -o jsonpath="{['data']['password']}" | base64 --decode && echo**

使用浏览器访问端口x.x.x.x:30808，使用admin用户登陆，登陆后可修改密码


创建cephrbd存储类

**kubectl apply -f  storageclass-rbd.yaml**

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



### 注：关于apply cluster.yml时，出现osd缺失的问题

硬盘需要满足条件：未格式化，未挂载，缺一不可

如果已经格式化并挂载，在每个节点执行以下命令：

**umount /mnt/data**

**wipefs -a /dev/vdb1**

此时使用lsblk命令查看硬盘情况，应该如下所示：

**[root@master rook-ceph]# lsblk**

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